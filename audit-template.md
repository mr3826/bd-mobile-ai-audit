# BD Mobile AI Audit Template

Copy this file for each AI feature you need to evaluate.

---

## Project Information

| Field | Value |
|---|---|
| Project name | |
| Feature name | |
| Date | |
| Auditor | |
| Version | |

---

## 1. Executive Decision

| Question | Answer |
|---|---|
| Recommended architecture | Cloud / Hybrid / On-device |
| Confidence level | High / Medium / Low |
| Key reason | |

---

## 2. Workload Profile

### 2.1 Use case description
*Describe the AI feature in one paragraph.*

### 2.2 Input / Output
| Input | Output |
|---|---|
| | |

### 2.3 Typical request size
- Input tokens / bytes:
- Output tokens / bytes:

### 2.4 Expected frequency
| Metric | Value |
|---|---|
| Requests per user per day | |
| Peak concurrent users | |
| Monthly active users | |

---

## 3. Constraints & Weights

*Score each dimension 1–5 (1 = not important, 5 = critical). Multiply by weight. Sum = weighted score.*

| Dimension | Weight (1–5) | Score (1–5) | Weighted |
|---|---:|---:|---:|
| Weak network tolerance | | | |
| Privacy / data residency | | | |
| Cloud API cost sensitivity | | | |
| Large reasoning / context needed | | | |
| Low-end device support | | | |
| Time to market | | | |
| Team ML expertise | | | |
| Regulatory / compliance | | | |
| **Total** | | | |

---

## 4. Option Assessment

For each option, rate feasibility 1–5 and note evidence.

### Cloud-only AI
| Criterion | Score (1–5) | Evidence / Notes |
|---|---:|---|
| Network availability | | |
| Latency budget | | |
| Monthly API cost estimate | | |
| Data sensitivity | | |
| Implementation effort | | |

### Hybrid AI
| Criterion | Score (1–5) | Evidence / Notes |
|---|---:|---|
| Local model complexity | | |
| Escalation trigger definition | | |
| Cache / offline strategy | | |
| Sync / conflict handling | | |
| Implementation effort | | |

### On-device AI
| Criterion | Score (1–5) | Evidence / Notes |
|---|---:|---|
| Model size fit (MB) | | |
| RAM usage at runtime | | |
| Cold-start latency | | |
| Battery impact | | |
| Quantization quality loss | | |
| Device fragmentation coverage | | |
| Update / rollout mechanism | | |
| Implementation effort | | |

---

## 5. Risk Matrix

| Risk | Likelihood (1–5) | Impact (1–5) | Mitigation |
|---|---:|---:|---|
| | | | |
| | | | |

---

## 6. Economics

| Cost Item | Cloud-only | Hybrid | On-device |
|---|---:|---:|---:|
| API cost / month (USD) | | | N/A |
| Model hosting / month | N/A | | |
| Engineering effort (person-weeks) | | | |
| Maintenance burden | | | |
| **Total monthly estimate** | | | |

---

## 7. Rollout Plan

| Phase | Milestone | Owner | Target date |
|---|---|---|---|
| Prototype | | | |
| Offline evaluation | | | |
| Beta (internal) | | | |
| Beta (external) | | | |
| GA | | | |

---

## 8. Evidence Log

*Attach links to benchmarks, screenshots, profiling traces, user testing notes.*

| Date | Artifact | Link / Location | Notes |
|---|---|---|---|
| | | | |

---

## 9. Sign-off

| Role | Name | Date | Decision |
|---|---|---|---|
| Product | | | Approve / Reject / Defer |
| Engineering | | | Approve / Reject / Defer |
| Security / Privacy | | | Approve / Reject / Defer |
| Finance | | | Approve / Reject / Defer |

---

## Appendix: Quick Scoring Reference

### Network tolerance
- 1: Always online, fiber/5G
- 3: Mixed, occasional drops
- 5: Rural, 2G/3G, frequent offline

### Privacy sensitivity
- 1: Public content
- 3: PII but not regulated
- 5: Financial / health / identity / legal hold

### API cost sensitivity
- 1: Unlimited budget
- 3: Fixed monthly cap
- 5: Cost must trend to zero

### Reasoning need
- 1: Classification / embedding only
- 3: Short generation, few shots
- 5: Long-form, multi-step, tool use

### Low-end device support
- 1: Flagship only
- 3: Mid-range (4–6 GB RAM)
- 5: Android Go (2 GB RAM), 5+ year old devices