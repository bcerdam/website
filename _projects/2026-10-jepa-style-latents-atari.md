---
title: "Do JEPA-Style Latents Improve Atari Scores in Dreamer-Like Agents?"
type: "Technical write-up"
date: "2026-10-01"
permalink: /projects/jepa-style-latents-atari/
---

## Introduction

In Joint-Embedding Predictive Architectures (JEPAs) like [LeWorldModel (Maes et al., 2026)](https://arxiv.org/abs/2603.19312), the latent is typically 1D, deterministic, and continuous, with unbounded real-valued components and no sampling step. By contrast, Sample Efficient World Models (SEWMs), a closely related branch of world models within model-based reinforcement learning that includes the Dreamer lineage ([Hafner et al., 2020](https://arxiv.org/abs/1912.01603), [Hafner et al., 2022](https://arxiv.org/abs/2010.02193), [Hafner et al., 2024](https://arxiv.org/abs/2301.04104), [Hafner et al., 2025](https://arxiv.org/abs/2509.24527)), use a 2D, stochastic, discrete latent. Each row is obtained by sampling from a categorical distribution over codes, producing a one-hot vector.

This brings us to the core question of this write-up: **Should SEWMs of the Dreamer lineage keep using a 2D stochastic discrete latent, or should they adapt their latent into something more JEPA-like: a 1D deterministic continuous latent?**

## Architecture

The world model used in this write-up is called UWM (Unnamed World Model). I use this name because it is my model and is currently under double-blind review for ICLR 2027.

### Structure of the latent

Both models use an encoder `e` to map observations `o` to latents. LeWM uses a 1D deterministic continuous latent <span class="math"><var>z</var><sub>t</sub><sup>1D</sup></span> of shape `(D,)`, with `D = 384`. UWM produces categorical logits <span class="math"><var>ℓ</var><sub>t</sub></span> of shape `K × C`, where `K` is the number of distributions and each distribution has `C` codes. Sampling from these distributions gives the 2D stochastic discrete latent <span class="math"><var>z</var><sub>t</sub><sup>2D</sup></span>, with one active code per row (Figure 1).

<figure class="article-figure">
  <img src="{{ '/assets/latent_comparison-1.png' | relative_url }}" alt="Diagram comparing LeWM's one-dimensional continuous latent with UWM's two-dimensional discrete categorical latent" loading="lazy">
  <figcaption>Figure 1. Latent representations and sampling in LeWM and UWM.</figcaption>
</figure>

Since neither model uses a decoder, both need a mechanism to prevent representation collapse of the latent. SIGReg ([Balestriero and LeCun, 2025](https://arxiv.org/abs/2511.08544)) is used in both models, but its application differs. My paper under review for ICLR 2027 explores this in depth. For this write-up, the only relevant thing to understand is that SIGReg prevents representational collapse.

### Self-supervised training

To learn environment dynamics, both models use self-supervised learning: given an observation <span class="math"><var>o</var><sub>t</sub></span> and action <span class="math"><var>a</var><sub>t</sub></span>, they predict the latent representation of the next observation <span class="math"><var>o</var><sub>t+1</sub></span>. LeWM emphasizes simplicity and heuristic-free training, avoiding techniques such as the stop-gradient operator that lacks clear theoretical justification ([Ponce et al., 2026](https://openreview.net/forum?id=f5MC1G6XhB)). UWM's training pipeline is also simple, but it requires stop-gradient on the posterior logits. Without this, training becomes unstable and the agent does not learn a competitive policy. Figure 2 marks the detachment on the path from the target logits <span class="math"><var>ℓ</var><sub>t+1</sub></span> to MSE.

<figure class="article-figure">
  <img src="{{ '/assets/wm_flow_comparison-1.png' | relative_url }}" alt="Diagram of LeWM and UWM self-supervised training flows, including the stop-gradient path" loading="lazy">
  <figcaption>Figure 2. Self-supervised training flows in LeWM and UWM.</figcaption>
</figure>

## Performance comparison

To compare agent performance across the two latent structures, we evaluate a representative subset of games from the Atari-100k benchmark ([Kaiser et al., 2024](https://arxiv.org/abs/1903.00374)). The subset includes Kung Fu Master, Bank Heist, Breakout, James Bond, and Kangaroo. Both conditions use UWM and differ in latent structure: one uses its 2D stochastic discrete latent, while the other uses a 1D deterministic continuous latent. The latter is a UWM variant, not the original LeWM implementation.

Figure 3 shows the mean rewards obtained by the model using a 1D deterministic continuous latent (LeWM-like) versus a 2D stochastic discrete latent (UWM, Dreamer-like). The x-axis shows the number of environment steps taken by the agent, limited to 100k by the Atari-100k benchmark. The y-axis shows the mean reward. Each agent was evaluated every 20k environment steps, with 100 rollouts per evaluation.

The results are clear: for downstream agent performance, the 2D stochastic discrete latent is crucial. It allows the agent to learn a policy in every game, while the world model using a 1D deterministic continuous latent learns a policy only in Breakout.

<figure class="article-figure">
  <img src="{{ '/assets/rewards_per_game_latent_comparison-1.png' | relative_url }}" alt="Line charts comparing mean reward for two-dimensional stochastic discrete latents and one-dimensional deterministic continuous latents across five Atari games" loading="lazy">
  <figcaption>Figure 3. Mean reward on five Atari-100k games, comparing a 2D stochastic discrete latent (UWM, Dreamer-like) with a 1D deterministic continuous latent (LeWM-like).</figcaption>
</figure>

## Discussion and conclusion

LeWM demonstrates that a simple world model can be trained without heuristics such as the stop-gradient operator and still perform well on Model Predictive Control benchmarks. However, the comparison above shows that UWM performs poorly on the Atari-100k subset when using a 1D deterministic continuous latent without stop-gradient. UWM scores much better with the 2D stochastic discrete latent (as proposed in DreamerV2).

So, should Dreamer-lineage models such as UWM replace the 2D stochastic discrete latent with a more theoretically grounded 1D deterministic continuous latent? The answer is: **Not yet.** The drop in agent performance is clear. However, I believe training approaches in JEPAs and SEWMs are converging, so the answer may change as both fields develop.

## References

1. [LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels](https://arxiv.org/abs/2603.19312), Maes et al., 2026.
2. [Dream to Control: Learning Behaviors by Latent Imagination](https://arxiv.org/abs/1912.01603), Hafner et al., 2020.
3. [Mastering Atari with Discrete World Models](https://arxiv.org/abs/2010.02193), Hafner et al., 2022.
4. [Mastering Diverse Domains through World Models](https://arxiv.org/abs/2301.04104), Hafner et al., 2024.
5. [Training Agents Inside of Scalable World Models](https://arxiv.org/abs/2509.24527), Hafner et al., 2025.
6. [LeJEPA: Provable and Scalable Self-Supervised Learning Without the Heuristics](https://arxiv.org/abs/2511.08544), Balestriero and LeCun, 2025.
7. [Dual Perspectives on Non-Contrastive Self-Supervised Learning](https://openreview.net/forum?id=f5MC1G6XhB), Ponce et al., 2026.
8. [Model-Based Reinforcement Learning for Atari](https://arxiv.org/abs/1903.00374), Kaiser et al., 2024.
