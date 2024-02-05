---
title: "World Models"
date: 2023-11-29
# weight: 1
# aliases: ["/first"]
tags: ["World Models", "Model-Based RL"]
author: "Yuheng"
showtoc: true
tocopen: true
draft: true
math: true
comments: false
description: "Planning with Prediction"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
ShowShareButtons: false
---

## World Models

> Learning the transition of the environment

Reference:
> [`world models paper`](https://arxiv.org/abs/1803.10122)
> [`world models author's blog`](https://worldmodels.github.io/)

The goal of World Model is to learn a transition model which is able to characterize the probability distribution $p(s_{t}|s_{t-1}, a_{t-1})$. Such pre-knowledge of the environment transition convert the tasks into model based, thereby enabling the application of advanced methodologies, including planning and model-based reinforcement learning (MBRL).

In this paper, the World Model is built with two distinct yet interconnected components: Vision Model (V) and Memory Model (M) where V learns to encode the high-dimensional observation into low-dimensional latent space and M learns to predict low-dimensional representation of the future observation. Specifically, the V model is a Variational AutoEncoder (VAE) and the M model is a mixture-density-network RNN (MDN-RNN). The Controller (C) is a policy (not a part of World Model in this context) which aims to maximize the long-term expected cumulative reward. It's flexible and can use different strategies, including reinforcement learning (RL) and evolutionary algorithms, to achieve this goal.

The World Model is illustrated in the following diagram:

{{< figure src="/post/world-models/world-model.png" caption="Flow diagram of World Model" >}}

In the described diagram, the raw pixel observation at each time step $t$, denoted as $s_t$, is encoded into the latent representation $z_t$ via a pre-trained VAE encoder. The Controller (C) then takes the hidden state $h_t$ of MDN-RNN and the latent representation $z_t$ of current observation to produce the action $a_t$ which interacts with the environment. M will encode the combination of $h_t$, $z_t$ and $a_t$ to produce the $h_{t+1}$ which will be used in the next time step $t+1$.

{{< figure src="/post/world-models/world-model-pseudocode.png" caption="Pseudo code of non-iterative procedure training of World Model" >}}

In the above description, the components of World Model, specifically V and M, are trained separately. We use a random policy to collect amount of episodes which records $s_t$, $a_t$, $s_{t+1}$ in sequence. First, the observation at each step $s_t$ is utilized to train the VAE. And then, this pre-trained VAE is applied to encoder all collected observations into latent representation $z_t$ which is then used to train the MDN-RNN to model $p(s_{t+1} | s_t, a_t, h_t)$. Such a comprehensive predictive capacity could significantly augment the World Model's effectiveness in complex scenarios.

To address the complexities of more dynamic environments, it may be beneficial to train the Memory model and Controller iteratively so that the learned transition could be refined over time. More advance, an enhancement could involve enabling the World Model to not only characterize the future $s_{t+1}$, but also the corresponding $r_{t+1}$, $a_{t+1}$ and termination signal $d_{t+1}$. Such a comprehensive predictive capacity could significantly augment the World Model's effectiveness in complex scenarios.

The procedure of iterative training is as follows:
1. initialize M and C with random parameters, and pre-train the V.
2. rollout in the environment N times. Save $s_t$, $a_t$.
3. train M to characterize $p(s_{t+1}, r_{t+1}, a_{t+1}, d_{t+1} | s_t, a_t, h_t$ and train C to maximize the expected reward in M.
4. if not completed, go back to step (2)

## PlaNet

> Learning latent dynamics for planning

Reference:
> [`Learning Latent Dynamics for Planning from Pixels`](https://arxiv.org/abs/1811.04551)

As mentioned above, characterizing $s_{t+1}$ along with associated reward signal $r_{t+1}$, action $a_{t+1}$ and termination signal $d_{t+1}$ may enhance performance in the complicated environments. Deep Planning Network (PlaNet) introduces a model that could capture these dynamics more effectively and is able to choose action through online planning in the learned compact latent space. In summary, PlaNet introduces a Recurrent State Space Model to learn latent dynamics and could utilize Multi-step Prediction to improve its capacity for accurate future prediction(*PlaNet doesn't requrie mutli-step prediction but it may be benificial for other dynammics model*).

### Recurrent State Space Model (RSSM)
{{< figure src="/post/world-models/RSSM.png" caption="Latent dynamics model design of PlaNet" >}}

Typically, there are two ways to learn latent dynamics. One is to represent the hidden state as deterministic variables while another one representing it as stochastic variables (as shown in the above figure). However, learning pure in deterministic model prevents the model from capturing multiple features and makes it easy for the planner to exploit inaccuracies. Learning in the purely from stochastic makes it difficult to remember information over multiple time steps.

To alleviate this problem, PlaNet introduces **Recurrent State Space Model (RSSM)** which decompose the state into stochastic and deterministic parts. RSSM has four models,  *Deterministic state model, Stochastic state model, Observation model, Reward model* where *deterministic model* is implemented as a RNN. Here, $s_t$ represents the encoded state in the latent space. $o_t$ represents the raw observation.
1. Deterministic state model: $h_t = f(h_{t-1}, s_{t-1}, a_{t-1})$
2. Stochastic model: $s_t \sim p(s_t | h_t)$
3. Observation model: $o_t \sim p(o_t | h_t, s_t)$
4. Reward model: $r_t \sim p(r_t | h_t, s_t)$

### Multi-step Prediction

> PlaNet doesn't leverage this technique, but it may be benificial for other dynamics models.

{{< figure src="/post/world-models/RSSM-latent-overshooting.png" caption="Unrolling schemes" >}}

In the unrolling schemes, The labels $s_{i|j}$ are short for the state at time step $i$ conditioned on observations up to time $j$. Arrows pointing at shaded circles indicate log-likelihood loss terms. Wavy arrows indicate KL-divergence loss terms.
(a) The standard variational objective decodes the posterior, denoted as $p(s|o)$, at every step to compute the reconstruction loss. (b) Observation overshooting decodes all multi-step predictions to apply reconstruction loss and it's too expensive in images domain (c) Latent overshooting predicts all multi-step prior, denoted as $p(s)$. These state beliefs are trained towards their corresponding posteriors $p(s|o)$ in latent space to encourage accurate multi-step predictions.

{{< figure src="/post/world-models/latent-overshooting-objective.png" caption="Loss function of latent overshooting" >}}

In the author's opinion, latent overshooting can be viewed as a regularization mechanism in the latent space. It promotes consistency between one-step and multi-step predictions, which, theoretically, should be equivalent when averaged over the data set.

### Pseudo Code

{{< figure src="/post/world-models/PlaNet-pseudocode.png" caption="Pseudo code of PlaNet" >}}

The loss function in equation 3 is as follows:

{{< figure src="/post/world-models/PlaNet-loss-function.png" caption="Loss function of PlaNet" >}}

In PlaNet, it learns a dynamic model and use a MPC to plan while interacting with the environment. In terms of planning time, it belongs to **Decision-Time Planning**. In decision-time planning, the agent learns the dynamics and use the simulated trajectories to decide on an action.

Another way to construct World Model is **Learning-Time Planning**. Different with Decision-Time Planning, it involves incorporating planning as part of the learning process of the model. Specifically, this approach involves using the learned model to simulate experiences and update the agent's policy and value function during the learning phase, as opposed to during the decision-making phase.

## Dreamer-V1

[`paper link`](https://arxiv.org/abs/1912.01603)
[`author's blog`](https://blog.research.google/2020/03/introducing-dreamer-scalable.html)
[`openreview link`](https://openreview.net/forum?id=S1lOTC4tDS)

{{< figure src="/post/world-models/dreamerv1.gif" caption="Dreamer-V1" >}}
  
Dreamer-V1 improves PlaNet in two key aspects. Firstly, it establishes a sophisticated World Model capable of predicting the future latent state $s_{t+1}$ and its corresponding reward signal $r_{t+1}$. Secondly, Dreamer-V1 integrates policy learning within the World Model development phase, classifying it as a Learning-Time Planning algorithm.

However, one issue is that deriving behaviors from predicted rewards within a finite prediction horizon might lead to a short-sighted problem, where the agent is only able to see the finite future rewards. Another challenge lies in mitigating cumulative errors produced by the learned World Model. Existing methods often resort to derivative-free algorithms, which may not be as robust as the derivative-based methods. To improve these issues, Dreamer-V1 concurrently predicts both actions $q_\phi(a|s)$ and latent state values $v_\phi(s_t)$.

The World Model of Dreamer-V1 resembles to the PlaNet. It learns a *Representation Model* $p_\theta(s_t | s_{t-1}, a_{t-1}, o_t)$, a *Transition Model* $q_\theta(s_t | s_{t-1}, a_{t-1})$, a *Reward Model* $q_\theta(r_t | s_t)$. For behavior learning, it employs an Actor-Critic framework, wherein the agent models an *Action Model* $q_\phi(a_t | s_t)$ and a *Value Model* $v_\psi(s_t)$. 

The training procedure is described as below:

{{< figure src="/post/world-models/Dreamerv1-pseudocode.png" caption="Pseudo code of Dreamer-V1" >}}

### Dynamics Learning

The objective of dynamics learning is to develop a World Model that not only acquires a more effective representation but also achieves precise transition prediction accuracy. For the purpose of updating parameters, the following loss function is utilized:

{{< figure src="/post/world-models/Dreamerv1-reconstruction.png" caption="Components of Dreamer-V1 World Model" >}}


There are various methods for learning the *Representation Model*. In Dreamer-V1, it utilizes reconstruction algorithm, specifically a VAE, to encode the raw observation into latent space. The *Observation Model* $q_\theta(o_t | s_t)$ is naturally learned through the decoder of VAE. The *Observation Model* here is only serves to provide a learning signal for the World Model.

The *Transition Model* is implemented as a **RSSM**. The *Representation Model* and *Observation Model* are implemented as an encoder and decoder, respectively, of a single VAE.

{{< figure src="/post/world-models/Dreamerv1-reconstruction-loss.png" caption="Loss function of Dreamer-V1 World Model" >}}

### Behavior Learning

Once we have a learned World Model, we can start imagination from any beginning state $s_t$. We constrain our imagine horizon to a fixed length $H$. In order to update the action and value models, we first compute the estimate $V(s)$ for all states along the imagined trajectories. The goal of *Action Model* is to predict actions that result is state trajectories with high value estimations. The objective for *Value Model* is to regress the value estimates.

{{< figure src="/post/world-models/Dreamerv1-vamodel-loss.png" caption="Loss function of Dreamer-V1 Action Model and Value Model" >}}

## Dreamer-V2

[`paper link`](https://arxiv.org/abs/2010.02193)
[`author's blog`](https://blog.research.google/2021/02/mastering-atari-with-discrete-world.html)
[`openreview link`](https://openreview.net/forum?id=0oabwyZbOu)

Dreamer-V2 shares the similar pipeline with the Dreamer-V1 and PlaNet. It learns a World Model and uses it to train actor-critic behaviors from imagination trajectories. In summary, it has two main modifications: (1) categorical latent representation, and (2) KL balancing.

### Categorical Latents

In the training phase, the encoder converts each image into a stochastic representation. This stochastic representation is then incorporated into the *recurrent state* of the World Model. Because the representations are stochastic, it implies that they do not have access to perfect information about the images and instead extract only what is necessary to make predictions, making the agent robust to unseen images. From each state, a decoder reconstructs the corresponding image to learn general representations. Moreover, a small reward network is trained to rank outcomes during planning. To enable planning without generating images, a *predictor* learns to guess the stochastic representations without access to the images from which they were computed.

Different from PlaNet and Dreamer-V1, the encoded features are represented as 32 distributions over 32 classes. The one-hot vectors sampled from these distributions are concatenated to a *sparse representation* that is passed onto the recurrent state.To backpropagate through the samples, we use straight-through gradients that are easy to implement using automatic differentiation. Representing images with categorical variables allows the predictor to accurately learn the distribution over the one-hot vectors of the possible next images. In contrast, earlier world models that use Gaussian predictors cannot accurately match the distribution over multiple Gaussian representations for the possible next images.

{{< figure src="/post/world-models/Dreamerv2-categorical-latent-benefit.png" caption="Multiple categoricals that represent possible next images can be accurately predicted by a categorical predictor, whereas a Gaussian predictor is not flexible enough to accurately predict multiple possible Gaussian representations." >}}


### KL Balancing

PlaNet and Dreamer series are based on the ELBO or variational free energy of a hidden markov model that is conditioned on the action sequence. In the ELBO objective, the KL loss serves two purposes: (1) it trains the prior $p_\phi(z_t | h_t)$ toward the representations (posterior) $q_\phi(z_t | h_t, x_t)$, and (2) it regularizes the representations toward the prior. However, in the training procedure, we want to avoid regularizing the representations toward a poorly trained prior.

To solve this, KL balancing minimize the KL loss faster with respect to the prior than the representations by using different learning rates. Specifically, $\alpha = 0.8$ for the prior and $1 - \alpha$ for the posterior.


### Interesting Facts

In the ablation experiments, the result shows that the reward prediction offers no improvement in the settings in the paper.