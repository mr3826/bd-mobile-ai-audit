# BD Mobile AI Feasibility Audit

A practical framework for deciding whether a mobile app should use cloud AI, hybrid AI, or on-device AI.

This is especially useful for products targeting Bangladesh and similar markets where users may have:

- Low-end Android devices
- Limited RAM
- Weak or unstable network
- Sensitivity to cloud API costs
- Privacy or compliance constraints

## The core question

> Can this app deliver useful AI without failing when the network drops or when cloud API costs scale?

## Audit questions

### 1. Where does the AI feature run?

- City-only users
- Mixed urban and rural users
- Field workers, drivers, agents, or merchants
- Offline-first environments

If many users operate in low-connectivity areas, cloud-only AI becomes fragile.

### 2. What is the minimum target device?

Examples:

- 2GB RAM Android Go devices
- 3–4GB RAM mid-range Android
- 6GB+ RAM premium devices

If your minimum target is low-end Android, model size, RAM pressure, and UI-thread blocking become major constraints.

### 3. What type of AI feature is required?

- OCR / document scanning
- Image classification
- Chat assistant
- Search or retrieval
- Summarization
- Voice transcription
- Fraud or risk detection
- Product recommendation

Some tasks are cheap on-device. Some tasks are better in the cloud.

### 4. What data cannot leave the device?

Examples:

- Identity documents
- Financial records
- HR data
- Health information
- Merchant pricing or inventory data
- Private chat content

If sensitive data must remain local, on-device or hybrid architecture becomes more attractive.

### 5. What is the monthly cloud AI budget?

Cloud AI is easy to prototype but can become expensive at scale.

Ask:

- How many requests per user per day?
- What is the cost per request?
- What happens if usage grows 10x?
- Can the business absorb USD-based API costs?

### 6. What is the failure behavior?

When the network fails, should the app:

- Block the user?
- Show cached results?
- Use a smaller local model?
- Queue the request?
- Fall back to human review?

## Architecture options

### Option A: Cloud-only AI

Best for:

- Fast prototyping
- High-quality reasoning
- Large context
- Low device hardware requirements

Tradeoffs:

- Network dependent
- Recurring API cost
- Data leaves the device
- Latency can be high

### Option B: Hybrid AI

Best for:

- Most production apps
- Sensitive local processing with optional cloud escalation
- Cost control

Example:

- Local embedding or classification
- Local cache
- Small on-device model for simple tasks
- Cloud model for complex tasks

### Option C: On-device AI

Best for:

- Offline-first products
- Privacy-sensitive workflows
- Low-latency local inference
- Reducing recurring API cost

Tradeoffs:

- Model size limits
- RAM pressure
- Battery cost
- Device fragmentation
- Slower generation on low-end hardware

## Decision matrix

| Signal | Cloud-only | Hybrid | On-device |
|---|---:|---:|---:|
| Weak network tolerance | Low | Medium | High |
| Privacy sensitivity | Low | Medium | High |
| API cost sensitivity | Low | Medium | High |
| Need for large reasoning | High | Medium | Low |
| Low-end device support | High | Medium | Low/Medium |
| Implementation complexity | Low | Medium | High |

## Proof of work

I am building public benchmarks for on-device LLM inference on low-end Android.

Focus areas:

- Model quantization
- RAM pressure
- KV-cache overhead
- Cold-load time
- UI-thread isolation
- Offline-first mobile RAG

Benchmark repository: [ADD YOUR on-device-llm-lab LINK]

## How to use this audit

1. Copy `audit-template.md`.
2. Answer the questions for your product.
3. Identify whether your feature should be cloud-only, hybrid, or on-device.
4. If you are unsure, send me a DM with the word `AUDIT`.

## License

MIT