# Architecture Options for Mobile AI in Bangladesh & Similar Markets

This document expands on the three architecture patterns with Bangladesh-specific technical details, decision dimensions, and evaluation criteria.

---

## 1. Architecture Definitions

### Cloud-only AI
All inference runs on a remote server. The mobile app sends requests over HTTPS and receives responses.

Typical stack:
- App → HTTPS → API Gateway → LLM Provider (OpenAI, Anthropic, self-hosted vLLM/TGI, etc.)
- Stateless or session-based
- Optional: edge caching via CDN

### Hybrid AI
Split inference between device and cloud based on rules, confidence, or user context.

Common patterns:
- **Local-first with cloud escalation**: Small model handles simple cases; complex/low-confidence requests escalate to cloud.
- **Local embeddings + cloud generation**: App computes embeddings on-device for retrieval; cloud LLM generates final answer.
- **Cache + sync**: Frequent queries cached locally; cache refreshed when online.
- **Privacy gate**: Sensitive inputs processed locally; non-sensitive routed to cloud.

### On-device AI
All inference runs locally. No network required for core AI feature.

Typical stack:
- Quantized LLM (GGUF, MLC, ExecuTorch, MediaPipe LLM Inference, ONNX Runtime Mobile)
- Local vector store (SQLite-vec, FAISS mobile, custom)
- Optional: LoRA adapters swapped per feature

---

## 2. Decision Dimensions (Bangladesh Context)

| Dimension | Cloud-only | Hybrid | On-device |
|---|---|---|---|
| **Network reliability** | Requires stable 3G/4G; fails on 2G or intermittent | Graceful degradation; local fallback works offline | Works fully offline |
| **Typical latency (Dhaka → Singapore/US)** | 150–400 ms + model latency | Local: 50–300 ms; Cloud leg: same as cloud-only | 50–500 ms (device-dependent) |
| **Data cost for user** | ~0.5–2 MB per request (JSON + tokens) | Lower if local handles majority | Near zero after model download |
| **Cloud API cost (USD/month)** | $0.001–0.05 per 1K tokens × volume | 30–70% reduction vs cloud-only | $0 after model download |
| **Data residency / privacy** | Leaves Bangladesh; may hit cross-border rules | Sensitive data stays local | Fully local |
| **Regulatory (Bangladesh)** | ICT Act, Digital Security Act — data leaving country may need consent | Easier compliance for sensitive fields | Simplest compliance |
| **Device requirements** | Any smartphone with browser/app | Mid-range+ for local model; low-end falls back to cloud | 3 GB+ RAM preferred; 2 GB possible with tiny models |
| **Model quality ceiling** | Highest (GPT-4o, Claude 3.5, large open models) | Medium–High (cloud leg) + Low–Medium (local) | Low–Medium (≤7B quantized) |
| **Update / rollout** | Instant server-side | Model updates via Play Store / OTA; cloud instant | App update or dynamic model download |
| **Engineering complexity** | Low | Medium–High | High |
| **Battery impact** | Low (network only) | Medium (local + network) | High (sustained GPU/NPU/CPU) |

---

## 3. Technical Detail by Option

### 3.1 Cloud-only AI

**When to choose**
- Prototyping / MVP (days not weeks)
- Feature needs large context (100K+ tokens)
- Team has no ML engineering capacity
- Users are primarily urban, 4G/5G, unlimited data plans

**Bangladesh-specific gotchas**
- International bandwidth cost: ISPs may throttle or cache poorly.
- Power outages → mobile data fallback → high latency.
- Many users on prepaid caps; 2 MB/request adds up.
- Digital Security Act: personal data leaving BD may require user consent + DPO.

**Mitigations**
- Compress prompts (few-shot → zero-shot, structured output).
- Use smaller cloud models (GPT-4o-mini, Haiku, Llama 3.1 8B hosted).
- Implement request batching & caching.
- Add explicit offline queue with local draft storage.

---

### 3.2 Hybrid AI

**When to choose**
- Production app with mixed user base (urban + rural)
- Some data is sensitive (NID, bank SMS, health)
- Want to control cloud spend
- Can invest 4–8 weeks in local model integration

**Local model candidates (2024–2025, Android)**
| Model | Size (quantized) | RAM (loaded) | Use case |
|---|---:|---:|---|
| Gemma 2 2B Q4 | ~1.2 GB | ~1.6 GB | Classification, extraction, short gen |
| Phi-3.5-mini Q4 | ~2.2 GB | ~2.8 GB | Reasoning, summarization, chat |
| Llama 3.2 1B/3B Q4 | 0.7 / 1.9 GB | 1.1 / 2.4 GB | General purpose |
| Qwen 2.5 1.5B/3B Q4 | 0.9 / 1.8 GB | 1.3 / 2.3 GB | Multilingual (Bangla strong) |
| SmolLM2 1.7B Q4 | ~1.0 GB | ~1.4 GB | Ultra-light, good Bangla |

**Inference runtimes**
| Runtime | Pros | Cons |
|---|---|---|
| **MediaPipe LLM Inference** | Google-maintained, GPU/NPU delegate, APK size small | Limited model zoo, Android 10+ |
| **ExecuTorch** | PyTorch-native, XNNPACK/NPU, flexible | Larger binary, steeper learning curve |
| **MLC LLM / TVM** | Best NPU utilization, supports many SoCs | Complex build, larger size |
| **ONNX Runtime Mobile** | Broad HW support, CPU fallback solid | Slower than NPU delegates |
| **llama.cpp (GGUF) via JNI** | Mature, many quant formats, CPU fast | No NPU, JNI overhead, larger APK |

**Hybrid routing logic examples**
```kotlin
// Pseudocode
fun route(request: AIRequest): InferenceTarget {
    if (request.containsSensitiveData) return LOCAL
    if (request.complexity < THRESHOLD && localModel.canHandle(request)) return LOCAL
    if (!network.isAvailable) {
        if (localModel.canHandle(request)) return LOCAL
        else return QUEUE_FOR_SYNC
    }
    return CLOUD
}
```

**Offline cache strategy**
- SQLite + FTS5 for text; sqlite-vec for embeddings.
- Write-ahead log for pending cloud requests.
- Background sync when `ConnectivityManager` reports unmetered + charging.

---

### 3.3 On-device AI

**When to choose**
- Offline-first product (field agents, riders, rural health workers)
- Hard privacy requirement (no cloud egress ever)
- Recurring cloud cost must be near zero
- Team has ML engineering capacity (6+ weeks)

**Hard constraints on low-end Android (2–3 GB RAM)**
| Constraint | Practical limit |
|---|---|
| Model size (quantized) | ≤ 2 GB (Q4_K_M 3B–4B) |
| Runtime RAM (model + KV cache + app) | ≤ 1.5 GB headroom |
| Context window | 2K–4K tokens typical |
| Generation speed | 3–8 tok/s on CPU; 10–25 tok/s on NPU |
| Cold start (load + warmup) | 1–3 s |
| APK size increase | +20–50 MB (model bundled) or dynamic download |

**Quantization guidance**
- Q4_K_M / Q4_0: best quality/size trade-off.
- Q3_K_M: acceptable for classification, risky for generation.
- Q2_K: usually too lossy for Bangla.
- Use `llama.cpp` calibration or AWQ for custom quantization.

**Bangla language considerations**
- Qwen 2.5, Gemma 2, SmolLM2 have decent Bangla token coverage.
- Llama 3.2 tokenizer splits Bangla into many tokens → slower, more RAM.
- Fine-tune / LoRA on Bangla instruction data if quality critical.

**Battery / thermal**
- Sustained NPU/GPU inference can throttle after 2–5 min on mid-range.
- Implement:
  - Token streaming to UI (don't block main thread).
  - User-visible progress + cancel.
  - Thermal API listener → downshift to CPU or pause.

**Distribution**
- Bundle model in APK (simple, increases install size).
- Dynamic download via Play Asset Delivery (on-demand, saves install size).
- LoRA adapters per feature (small, swappable).

---

## 4. Evaluation Matrix (Weighted Scoring)

Use this matrix to score your feature. Weights should reflect your product priorities.

| Dimension | Weight (your product) | Cloud-only | Hybrid | On-device |
|---|---:|---:|---:|---:|
| Network resilience | | | | |
| Data privacy / residency | | | | |
| Cloud cost control | | | | |
| Model quality ceiling | | | | |
| Low-end device support | | | | |
| Time to market | | | | |
| Engineering capacity | | | | |
| Regulatory simplicity | | | | |
| Battery / thermal | | | | |
| **Weighted total** | **1.0** | | | |

Score each cell 1–5. Multiply by weight. Highest total wins.

---

## 5. Hybrid Pattern Decision Tree

```
START
│
├─ Does any input contain NID, bank, health, biometric?
│   └─ YES → LOCAL (privacy gate)
│
├─ Is network guaranteed (WiFi/4G, corporate device)?
│   └─ YES → CLOUD (simpler)
│
├─ Can a ≤3B model handle the task acceptably?
│   ├─ YES → LOCAL (cost/latency win)
│   └─ NO  → Can we define a clear escalation trigger?
│              ├─ YES → HYBRID (local + cloud escalation)
│              └─ NO  → CLOUD (or redesign task)
│
└─ Is cloud budget effectively zero?
    └─ YES → ON-DEVICE (or hybrid with aggressive local)
```

---

## 6. Cost Model Example (Monthly, 10K MAU, 5 req/user/day)

| Item | Cloud-only (GPT-4o-mini) | Hybrid (70% local) | On-device |
|---|---:|---:|---:|
| API calls | 1.5M | 450K | 0 |
| Input tokens (avg 500) | 750M | 225M | — |
| Output tokens (avg 200) | 300M | 90M | — |
| API cost (est. $0.15/1M in, $0.60/1M out) | ~$292 | ~$88 | $0 |
| Model hosting (if self-hosted cloud leg) | — | $50–200 | — |
| Engineering (amortized 12 mo) | $500 | $2,000 | $5,000 |
| **Total / month (yr 1)** | **~$792** | **~$2,138** | **~$5,000** |
| **Total / month (yr 2+)** | **~$292** | **~$138** | **~$0** |

*Adjust volumes, token counts, and model pricing for your case.*

---

## 7. Recommended Starting Path by Team Profile

| Team Profile | Start With | Migrate To |
|---|---|---|
| 2–3 engineers, no ML exp, 4-week deadline | Cloud-only | Hybrid (quarter 2) |
| 4+ engineers, 1 ML-capable, 8-week deadline | Hybrid | On-device (if needed) |
| Offline-first product, privacy mandate | On-device | — |
| Early prototype, uncertain product-market fit | Cloud-only | Re-evaluate at PMF |

---

## 8. Quick Benchmarks to Run Before Deciding

1. **Network profile**: Measure RTT, throughput, dropout rate on target devices in target areas (use `ping`, `iperf3`, custom probe).
2. **Device profile**: List minimum 5 devices covering your user base. Record RAM, SoC, Android version, NPU support.
3. **Local model smoke test**: Load candidate model on min-spec device. Measure:
   - Load time
   - First token latency
   - Steady-state tokens/sec
   - RAM delta
   - Battery drain (10 min inference)
4. **Cloud cost projection**: Run 100 representative prompts through target API. Record tokens, latency, errors.
5. **Privacy audit**: List every data field that touches AI. Classify: Public / PII / Sensitive / Regulated.

---

## 9. References & Tools

- MediaPipe LLM Inference: `https://github.com/google-ai-edge/mediapipe`
- ExecuTorch: `https://github.com/pytorch/executorch`
- MLC LLM: `https://github.com/mlc-ai/mlc-llm`
- llama.cpp Android: `https://github.com/ggml-org/llama.cpp/tree/master/examples/android`
- sqlite-vec: `https://github.com/asg017/sqlite-vec`
- Android Thermal API: `https://developer.android.com/reference/android/os/ThermalManager`
- Play Asset Delivery: `https://developer.android.com/guide/playcore/asset-delivery`

---

## 10. License

MIT