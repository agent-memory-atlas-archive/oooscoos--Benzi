# mini-swe-agent on SWE-bench Verified

**410 of 500 resolved (82.0%), pass@1, on DeepSeek V4.1-Flash, for $6.39 total: 1.6¢ per resolved instance.**

A baseline run: the same benchmark and model as [Benzi's October result](../swebench-october/), through mini-swe-agent instead of Benzi. One configuration, one attempt per instance, no ensembling, no test-time selection, graded by the official SWE-bench harness. This report describes the system, the setup, the results, and how to reproduce them.

---

## 1. Headline results

| | |
|---|---|
| **Resolved** | **410 / 500 (82.0%)** |
| Attempts per instance | 1 (pass@1) |
| Model | DeepSeek V4.1-Flash, thinking mode at its default effort |
| Total generation cost, all 500 | $6.39 |
| Cost per resolved instance | $0.016 |
| Model turns: total / median per instance | 12,065 / 18 |
| Input tokens served from prompt cache | 96.8% |
| Output tokens, total (of which reasoning) | 6.3M (4.2M) |

Costs are DeepSeek list prices at the standard (off-peak) rates, over measured tokens.

Grading: `swebench.harness.run_evaluation` (swebench 5.0.2), official per-instance Docker images, official Hugging Face dataset (`SWE-bench/SWE-bench_Verified`). An empty prediction counts as unresolved.

### By repository

| Repository | Instances | Resolved | Rate |
|---|---:|---:|---:|
| flask | 1 | 1 | 100.0% |
| scikit-learn | 32 | 31 | 96.9% |
| pytest | 19 | 17 | 89.5% |
| xarray | 22 | 19 | 86.4% |
| matplotlib | 34 | 29 | 85.3% |
| sphinx | 44 | 37 | 84.1% |
| astropy | 22 | 18 | 81.8% |
| django | 231 | 187 | 81.0% |
| sympy | 75 | 59 | 78.7% |
| requests | 8 | 6 | 75.0% |
| pylint | 10 | 5 | 50.0% |
| seaborn | 2 | 1 | 50.0% |

## 2. The system

mini-swe-agent (version 2.4.6) is the SWE-bench team's minimal reference agent. The model gets one tool, a bash shell: no file viewer, no editor, no index. Every action is a shell command run in its own subprocess inside the instance's container, and the model reads code, edits files and runs tests through that shell alone.

The run uses mini-swe-agent's stock `swebench.yaml` configuration, its prompts unchanged. The model is called through LiteLLM as `deepseek/deepseek-flash`, in DeepSeek's thinking mode at its default effort.

No FAIL_TO_PASS or PASS_TO_PASS test names, files or contents were shown to the agent at solve time. The agent sees the issue text and the repository, nothing else.

## 3. Setup

- **Containers.** Each instance ran inside its official SWE-bench per-instance Docker image. GitHub and PyPI hosts resolved to loopback inside every container, so nothing could fetch the fixed version of itself. The only other change from the stock configuration: the image-pull timeout was raised from 120 to 900 seconds.
- **Budgets.** mini-swe-agent's stock limits: 250 steps and $3 per instance, no wall-clock limit. All 500 instances finished by submitting, none at a limit.
- **Retries.** Instances whose container failed to start (image pulls that hit the 120-second timeout before it was raised) were re-attempted, and two grading runs marked as infrastructure failures after a container failed to stop were re-graded. No instance was retried for being wrong; every instance is reported from its one attempt.

## 4. Limitations

- Single configuration, single run, pass@1: no ensembling, no multiple samples, no test-time selection, no cherry-picking.
- All numbers are on DeepSeek V4.1-Flash. This result is not a claim about other models.

## 5. Reproducibility

```
mini-extra swebench --subset SWE-bench/SWE-bench_Verified --split test -w 8 \
  -m deepseek/deepseek-flash -c swebench.yaml -c mini_override.yaml -o mini_swe_out
```

`mini_override.yaml` sets the loopback hosts and the image-pull timeout described above. Generation ran on EC2 hosts (Docker 25, Python 3.12); grading used the official harness against the official dataset.

---

*Run by Variant Technologies as a baseline for Benzi. [benzi.fly.dev](https://benzi.fly.dev/about) · [github.com/oooscoos/Benzi](https://github.com/oooscoos/Benzi)*
