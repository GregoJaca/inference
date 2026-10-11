# Serving Config Tuning

**Engine defaults ≠ your traffic. Replay your requests, search settings on your box, ship only what passes latency + quality gates.**

Honest take: thin. Free tuners exist. Sell as a service or as an audit module.

## Numbers

| Lever | Result | Source |
| --- | --- | --- |
| Auto-tuned vs manual config | ≤1.7x tok/s/GPU, vendor example, Dynamo | [AIConfigurator](https://docs.nvidia.com/dynamo/latest/performance/aiconfigurator.html) |
| Tail latency | p99 1.36 → 0.70 s; in-target req/s 8 → 15 (TinyLlama, vLLM) | [SLO-Tuner](https://arxiv.org/abs/2603.11340) |
| CPU cores per GPU | TTFT 1.47-7.11x | [Georgia Tech](https://arxiv.org/abs/2603.22774) |
| Host noise | -8.8% / -20.5% tok/s; pinning +7.6% (1 GPU, 0.6B model) | [Eunomia](https://eunomia.dev/blog/2026/05/24/measuring-cpu-noise-in-gpu-inference-with-ebpf/) |
| Draft head vs load | Kimi K2.5: 1.90x @4 concurrent → 1.69x @64. MiniMax M2.5: 1.79x → 1.38x. Shrinks at 8K prompts | [AMD](https://rocm.blogs.amd.com/artificial-intelligence/eagle3-speculative-decoding/README.html) |
| $ | +10% tok/s on 8 GPUs @ $2.50/GPU-h ≈ $17.5k/yr | our math |

## Use vs build

| Job | Use (open source) | We build |
| --- | --- | --- |
| Replay load | [GuideLLM](https://github.com/vllm-project/guidellm) | logs → replay file |
| Search | vLLM `auto_tune.sh` ([docs](https://gitcode.com/SparkLLM/vllm/tree/main/benchmarks/auto_tune)), [llm-optimizer](https://bentoml.com/blog/announcing-llm-optimizer), [vllm-tuner](https://pypi.org/project/vllm-tuner/) | search space + rules |
| Agent search | [LLM-Tuna 2.0](https://pretalx.devconf.info/devconf-us-2026/talk/9SMF9X/), [autooptimizer](https://github.com/Trendyol/autooptimizer) | guardrails: quality gate, crash limits |
| Dynamo | [AIConfigurator](https://docs.nvidia.com/dynamo/latest/performance/aiconfigurator.html) | proof on customer traffic |
| Ideas | [SCOOT](https://liner.com/review/scoot-slooriented-performance-tuning-for-llm-inference-engines), [SLO-Tuner](https://arxiv.org/abs/2603.11340), [SLO-Guard](https://www.emergentmind.com/papers/2604.17627) (papers) | borrow: p99-first goal, crash-aware trials |
| Host | [Parca CUDA profiler](https://www.polarsignals.com/blog/posts/2025/10/22/gpu-profiling), cuda\_sched\_trace ([Eunomia](https://eunomia.dev/blog/2026/05/24/measuring-cpu-noise-in-gpu-inference-with-ebpf/)) | collector + fix catalog: core limits, pinning, IRQs, NUMA |
| Brain | Claude Code (closed) | skills pack, report, rollback |

## Client touchpoints

- **Logs:** 1-2 days: arrival time, in/out tokens, streaming, tool calls. No prompt text.
- **Config:** launch command, engine version, deploy files (k8s/Slurm/Docker).
- **Hardware:** GPU type + count, CPU cores, RAM, shared or dedicated.
- **Targets:** p99, TTFT, quality bar, monthly GPU $.
- **Metrics:** engine `/metrics`.
- **Box:** spare server or window; root for host settings.
- **PR access:** optional, if we deliver the change.
- **DPA** if EU and prompt text involved.

## Competitors

| Who | What | Open? | Gap |
| --- | --- | --- | --- |
| `auto_tune.sh`, llm-optimizer, vllm-tuner | Search settings under a latency target | Open | Free; no host layer, no quality gate |
| [LLM-Tuna](https://pretalx.devconf.info/devconf-us-2026/talk/9SMF9X/) (Red Hat) | Agent + knowledge base reads vLLM logs | Open | vLLM/llm-d focus; a tool |
| [autooptimizer](https://github.com/Trendyol/autooptimizer) (Trendyol) | LLM agent edits serve config | Open | No quality gate |
| [AIConfigurator](https://docs.nvidia.com/dynamo/latest/performance/aiconfigurator.html) | Dynamo configs | Open | Dynamo only |
| [Zymtrace](https://zymtrace.com/) | Profiling + agent skills | Closed, free tier | Code/kernels, not settings |
| Baseten, Fireworks | Tune for you | Closed | Must move platform |

Real rival: free tools + a team with Claude Code. Our edge: traffic replay, host layer, quality gate, doing the work. Consultancies: not verified.

## Sources

As of 2026-10-11. Mostly small models or vendor claims; "our math" = ours.

- [NVIDIA AIConfigurator](https://docs.nvidia.com/dynamo/latest/performance/aiconfigurator.html)
- [SLO-Tuner](https://arxiv.org/abs/2603.11340)
- [Georgia Tech: CPU slowdowns](https://arxiv.org/abs/2603.22774)
- [Eunomia: CPU noise](https://eunomia.dev/blog/2026/05/24/measuring-cpu-noise-in-gpu-inference-with-ebpf/)
- [AMD: EAGLE-3 on Kimi K2.5](https://rocm.blogs.amd.com/artificial-intelligence/eagle3-speculative-decoding/README.html)
- [GuideLLM v0.7.0](https://github.com/vllm-project/guidellm/releases/tag/v0.7.0)
- [Red Hat LLM-Tuna 2.0](https://pretalx.devconf.info/devconf-us-2026/talk/9SMF9X/)
