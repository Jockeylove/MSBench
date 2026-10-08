<h1 align="center">Know Where You Stand</h1>
<h3 align="center">Memory-Source Choice in Long-Context Dialogue Agents</h3>

<p align="center"><b>NeurIPS 2026</b></p>

<p align="center"><b>English</b> · <a href="README.zh-CN.md">简体中文</a></p>

<p align="center">
  Caishen Zhou · Yihong Tang · Xuefeng Bai · Kehai Chen · Min Zhang<br>
  Harbin Institute of Technology, Shenzhen
</p>

<p align="center">
  <a href="#msbench"><b>Benchmark</b></a> &nbsp;·&nbsp;
  <a href="#msr"><b>Method</b></a> &nbsp;·&nbsp;
  <a href="#results"><b>Results</b></a> &nbsp;·&nbsp;
  <a href="#release"><b>Code &amp; Data</b></a> &nbsp;·&nbsp;
  <a href="#citation"><b>Citation</b></a>
</p>

<p align="center"><b>When should a dialogue agent stay with recent context, and when should it retrieve prior memory?</b></p>

---

## Overview

Long-context agents face two competing errors: missing evidence that has left the visible window, and retrieving stale memory that overrides the current conversation. The key decision is **which source should support the answer**.

We introduce **MSBench**, a benchmark that isolates this source-choice problem, and **MSR (Memory-Source Reasoner)**, a training-free method that monitors support before selecting and organizing answer evidence.

<p align="center">
  <img src="assets/source-choice.png" width="100%" alt="Two source-choice failures: missing earlier evidence and allowing stale memory to override the current state.">
</p>
<p align="center"><sub>Both missed retrieval and unnecessary retrieval can lead to a wrong answer.</sub></p>

<a id="msbench"></a>

## MSBench: Evaluating Source Choice

MSBench replays multi-session dialogue in chronological order. At each checkpoint, two questions share the same dialogue state but require different evidence sources: one uses **visible recent context**, while the other needs **prior memory**.

| Scenes | Replay checkpoints | Questions | Source balance |
| :---: | :---: | :---: | :---: |
| **45** | **450** | **900** | **450 recent-context / 450 prior-memory** |

- **Explicit source boundaries.** The latest 16 dialogue turns form the visible context; earlier turns form prior memory. Future turns are hidden.
- **Paired source contrasts.** Questions at the same checkpoint test whether the agent can choose the source required by each question.
- **Evolving dialogue state.** The benchmark includes state updates and misleading recent context, so topical relevance alone does not determine the right source.

We report **Answer Accuracy**, which measures answer correctness, and **Selection Accuracy**, which measures whether the agent chooses the required source. MSBench is constructed from PersonaMem-v2; transfer experiments extend the evaluation to other corpora.

<a id="msr"></a>

## MSR: Monitor Support, Then Choose the Source

<p align="center">
  <img src="assets/msr-overview.png" width="100%" alt="MSR source monitoring and its visible-context and prior-memory answer branches.">
</p>

1. **Monitor support.** Decompose the question into the facts and constraints an answer must satisfy. Identify supporting dialogue turns in the visible context and a small hidden memory preview.
2. **Choose the source.** Select prior memory when the preview supports a requirement missing from the visible context. Otherwise, stay with the visible context.
3. **Ground the answer.** Use the same support state to organize evidence within the selected branch. For prior memory, recover the relevant exchanges and place them before question-ranked retrieval hits.

The hidden preview informs source monitoring rather than being passed directly to the answer model. MSR uses two calls to the same backbone, with **no fine-tuning or architecture changes**.

<a id="results"></a>

## Results

### Main comparison on MSBench

On GPT-5-mini, MSR reaches **60.0% Answer Accuracy** and **81.7% Selection Accuracy**. These exceed the strongest non-MSR result for each metric by **12.6** and **6.3 percentage points**, respectively.

| Method | Answer Accuracy | Selection Accuracy |
| :--- | ---: | ---: |
| No Memory | 28.0 | 50.0 |
| RAG | 47.4 | 50.0 |
| Prompted LLM Retrieval Gate | 46.8 | 75.4 |
| **MSR** | **60.0** | **81.7** |

All values are percentages. MSR also achieves the highest point estimates on both metrics with Gemini 3.1 Flash-Lite, Qwen3 235B-A22B, and DeepSeek V4 Pro under the shared MSBench protocol.

### Transfer and efficiency

<table>
  <tr>
    <th align="center">LongMemEval-M transfer</th>
    <th align="center">Memory-system references</th>
  </tr>
  <tr>
    <td width="50%"><img src="assets/longmemeval-transfer.png" width="100%" alt="Answer and selection accuracy under the source-choice transfer setting on LongMemEval-M."></td>
    <td width="50%"><img src="assets/memory-system-references.png" width="100%" alt="MSBench answer accuracy, per-question token cost, and latency for memory-system references."></td>
  </tr>
</table>

The left panel evaluates LongMemEval-M under the visible/prior source boundary used in MSBench. The right panel reports native memory-system results on MSBench under their respective configurations.

MSR also leads on Answer and Selection Accuracy in the **LoCoMo and QMSum source-choice evaluations**. In the controlled cost comparison, its per-question latency is comparable to the prompted gate. An **abstention extension** improves action decisions on unanswerable questions while maintaining responses to answerable controls.

<a id="release"></a>

## Code & Data

**Code and data are coming soon.** The planned release includes MSBench data, MSR and baseline implementations, prompts, and evaluation scripts. Installation and quick-start instructions will accompany the release.

<a id="citation"></a>

## Citation

If you use MSBench or MSR in your research, you may use the provisional citation below. We will update it after verifying the official bibliographic record.

```bibtex
@inproceedings{zhou2026know,
  title     = {Know Where You Stand: Memory-Source Choice in Long-Context Dialogue Agents},
  author    = {Zhou, Caishen and Tang, Yihong and Bai, Xuefeng and Chen, Kehai and Zhang, Min},
  booktitle = {Advances in Neural Information Processing Systems},
  year      = {2026},
  note      = {Accepted to NeurIPS 2026}
}
```

## Licensing & Data Attribution

Our original code will be released under the [Apache License 2.0](LICENSE), which permits commercial use subject to its terms. This code license does not cover the paper or datasets. Third-party code and data remain subject to their respective licenses.

**MSBench is derived from [PersonaMem-v2](https://huggingface.co/datasets/bowen-upenn/PersonaMem-v2), released by Bowen Jiang and collaborators under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).** We select dialogue scenes, construct chronological replay checkpoints, and add paired source-choice questions, source labels, and reference answers. These are our modifications to the source resource. The data release will include source-version information and the applicable attribution and license notices.

The external benchmarks used in our transfer experiments have separate upstream terms:

| Resource | Upstream license |
| :--- | :--- |
| [LongMemEval](https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned) | [MIT](https://github.com/xiaowu0162/LongMemEval/blob/main/LICENSE) |
| [LoCoMo](https://github.com/snap-research/locomo) | [CC BY-NC 4.0](https://github.com/snap-research/locomo/blob/main/LICENSE.txt), including its noncommercial-use restriction |
| [QMSum](https://github.com/Yale-LILY/QMSum) | [MIT](https://github.com/Yale-LILY/QMSum/blob/main/LICENSE), as stated in the official repository |

Redistributed third-party materials must retain the required attribution and license notices. Modified materials must also follow the applicable upstream terms; our code license does not relicense external datasets.

## Acknowledgements

We thank the [PersonaMem-v2](https://huggingface.co/datasets/bowen-upenn/PersonaMem-v2) team for the resource underlying MSBench, and the [LongMemEval](https://github.com/xiaowu0162/LongMemEval), [LoCoMo](https://github.com/snap-research/locomo), and [QMSum](https://github.com/Yale-LILY/QMSum) teams for the benchmarks used in our transfer evaluations.
