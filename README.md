
# Uncertainty Routing in Instruction-Tuned LMs

This repo contains experiments probing how an instruction-tuned language model represents “answerability” and routes that internal signal into either a direct answer or an explicit abstention (e.g., “UNCERTAIN”). The focus is mechanistic: localizing where the decision is formed, testing causal leverage across depth, and identifying high-impact attention components that control the final routing behavior.

## Core idea
Uncertainty and abstention are often treated as the same thing (high uncertainty ⇒ abstain). In practice, they can decouple: a model can be internally uncertain yet still answer, or abstain even when token-level uncertainty looks low. This project studies the internal computation that turns “answerability evidence” into a discrete answer vs abstain decision.

## What’s in here
- **FORCE vs ALLOW setup**
  - *FORCE-GUESS*: force an answer and estimate internal uncertainty from sampling disagreement (entropy / majority agreement over normalized answers).
  - *ALLOW-UNCERTAINTY*: allow abstention and measure how often the model emits an abstain token.
- **Decision readout**
  - For mechanistic analysis, the routing decision is tracked with a simple binary readout (e.g., `ANSWERABLE` vs `UNANSWERABLE`) using the **binary margin**:
    \[
    s = \log p(\text{ANSWERABLE}) - \log p(\text{UNANSWERABLE})
    \]
  - Reported metrics: **Δmargin** and **flip rate** (fraction of examples whose predicted label changes under an intervention).
- **Layerwise localization**
  - **Decodability probes** to see where the decision becomes readable from residual-stream activations.
  - **Activation patching** (bidirectional) to identify a late “routing band” with high causal leverage.
- **Steering / control**
  - Compute an “answerability direction” and inject \(+\epsilon \hat v\) at selected layers to causally modulate the margin/flip rate.
  - Component localization: compare injection at block output vs attention vs MLP.
- **Head-level mechanism**
  - **Head ablations** (necessity): ablate candidate heads and compare against low-impact control heads.
  - **Head patching** (conditional sufficiency): transplant head outputs between answering/abstaining runs; includes wrong-token and random-donor controls.

## Results at a glance
- Internal disagreement (FORCE) and explicit abstention (ALLOW) often diverge.
- Causal influence is concentrated in a late routing band rather than uniformly distributed.
- Within that band, the effect is attention-dominant (stronger through attention outputs than MLP).
- A small set of heads has outsized leverage; single-head patches can flip some borderline cases, suggesting head/token-specific influence within a distributed circuit.

## Running
This repo is currently organized as research notebooks/scripts. Typical workflow:
1. Generate/collect FORCE and ALLOW runs on a QA-style dataset (SQuAD 2.0–style answerable/unanswerable prompts).
2. Compute uncertainty metrics (FORCE) and abstention frequency (ALLOW).
3. Compute the binary margin readout and identify borderline cases.
4. Run localization (probes, layerwise patching), then steering/component splits.
5. Run head ablation + head patching with controls.

> If you’re trying to reproduce a specific figure/result, start from the activation patching sweep and steering notebooks, then follow the head-level sections.

## Notes / limitations
- Many flip results are most visible on borderline examples (small base margin).
- Small-N runs can inflate flip-rate variance; reruns with different seeds/samples are recommended.
- “UNANSWERABLE” is used as a controlled proxy for abstention in mechanistic experiments; free-form “UNCERTAIN” behavior is evaluated separately.

## Citation
If you use ideas or code from this repo, please cite the associated writeup (link TBD).

