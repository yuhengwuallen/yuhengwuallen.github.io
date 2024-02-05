---
title: "Survey on Policy-Based Deep Reinforcement Learning Algorithms"
date: 2023-11-10
tags: ["RL", "policy-based"]
author: "Yuheng"
showtoc: true
tocopen: true
draft: true
math: true
comments: false
description: "review classic policy-based algorithms"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
ShowShareButtons: false
---

> Acknowledgement: this note is based on Lilian Weng's blog [Policy Gradient Algorithms](https://lilianweng.github.io/posts/2018-04-08-policy-gradient/). I have meticulously reviewed the referenced papers, reorganized the content to align with my thought process, and infused my own understanding into the material. For the original content authored by Lilian, please visit the provided link. Special gratitude to Lilian for delivering an exceptional tutorial that served as the foundation for this note.

## Terminology

Here, we list terminologies we will often see later.

| symbol                   | meaning                                                                                                                      |
| :----------------------- | :--------------------------------------------------------------------------------------------------------------------------- |
| $s \in \mathcal{S}$      | states                                                                                                                          |
| $a \in \mathcal{A}$      | actions                                                                                                                           |
| $r \in \mathcal{R}$      | reward                                                                                                                            |
| $\gamma$                 | discount factor of future rewards                                                                                                 |
| $P(s',r \vert s, a)$     | transition probability of environment                                                                     |                   
| $\pi_{\theta}(a\vert s)$ | stochastic policy parameterized by $\theta$, a conditional probability distribution given current state $s$      |
| $\mu(s)$                 | deterministic policy, we just use $\mu$ instead of $\pi$ to simply differ stochastic policy from deterministic policy           |
| $V^{\pi}(s)$             | state-value function measures expected cumulative reward from current state $s$, following policy $\pi$                          |
| $Q^{\pi}(s,a)$           | action-value function measures expected cumulative reward from current state $s$ taking action $a$, following policy $\pi$     |
| $A(s,a)$                 | advantage function $A(s,a) = Q(s,a) - V(s)$ measures "how the chosen action $a$ is better than average at current state $s$"    |

## Policy Gradient

### Policy Gradient Theorem

Different from value-based methods, which implicitly expresses policy through Q function, policy-based methods directly parameterize the policy $\pi(a|s)$. Here, we use deep neural network parameterization in which the policy is parameterized by $\theta$, denoted as $\pi_{\theta}(a | s)$. Our goal is to improve the policy so as to maximize the expected cumulative reward $J(\theta)$. With the explicitly expressed policy $\pi(a \vert s)$, the goal can be formalized as expected cumulative from start state. Thus, we can define this formally through state-value function $V(s)$.

## Proof
We formulate the goal as the following equation. Here, we assume we start from a particular state $s_0$.

$$ J(\theta) = V^\pi(s_0) $$ 

Then, we drive an expression from the gradient of $V^\pi(s)$,  

$$
\begin{aligned}
\nabla_\theta V^\pi(s) &= \nabla_\theta \left( \sum_a \pi(a|s)Q^\pi(s,a) \right) \\\\  
&= \sum_a \left[ \nabla_\theta \pi(a|s) Q^\pi(s,a) + \pi(a|s) \nabla_\theta Q^\pi(s,a) \right] \\\\ 
&= \sum_a \left[ \nabla_\theta \pi(a|s) Q^\pi(s,a) + \pi(a|s) \nabla_\theta \sum_{s',r}\left ( r +  p(s',r |s,a)V^\pi(s') \right) \right] \\\\
&= \sum_a\left[ \nabla_\theta \pi(a|s) Q^\pi(s,a) + \pi(a|s)\nabla_\theta \sum_{s'} p(s' |s,a)V^\pi(s') \right] \\\\
\end{aligned}
$$


This leads to a nice recursive form.  

$$
\textcolor{red}{\nabla_\theta V^\pi(s)} = \sum_a\left[ \nabla_\theta \pi(a|s) Q^\pi(s,a) +  \pi(a|s)\sum_{s'} p(s' |s,a) \textcolor{red}{\nabla_\theta V^\pi(s')} \right]
$$

However, computing $\nabla J(\theta)$ remains challenging because the target depends on both the action selection $\nabla_\theta \pi(a|s)$ and state-value function following the policy $\nabla_\theta V^\pi(s')$.   

Let's consider expanding the equation above. For simplicity, we use $\phi(s)$ to represent $\sum_a \nabla_\theta\pi(a|s)Q^\pi(s,a)$  

$$
\begin{aligned}
\textcolor{red}{\nabla_\theta V^\pi(s)} &= \phi(s) +  \sum_{a}\pi(a|s)\sum_{s'} p(s' |s,a) \textcolor{red}{\nabla_\theta V^\pi(s')} \\\\
&= \phi(s) + \sum_a\pi(a|s)\sum_{s'}p(s'|s,a) \textcolor{red}{ \left[ \phi(s')+\sum_{a'}\pi(a'|s')\sum_{s''}p(s''|s',a')\nabla_\theta V^\pi(s'') \right]} \\\\
\end{aligned}
$$

We draw the recursive form of the probability in the above equation again. Each probability can be regarded as a sum of probability that state s transit to state x following the policy $\pi$. We define state transition probability distribution under policy $\pi$ as $\rho(s\rightarrow x,\pi)$.  

$$
\begin{aligned}
s \rightarrow s'&:\sum_a \pi(a|s) \sum_{s'}p(s'|s,a) \\\\
s \rightarrow s' \rightarrow s'' &:\sum_a\pi(a|s)\sum_{s'}p(s'|s,a) \sum_{a'}\pi(a'|s')\sum_{s''}p(s''|s',a') \\\\
s \rightarrow s' \rightarrow s'' \rightarrow s'''&: \cdots
\end{aligned}
$$

The presented equations delineate the summation of probabilities that given the current state $s$, across all feasible actions $a$ leading to the subsequent state $s'$. When examining these transitions collectively, they encapsulate the scenario wherein, starting from a state $s$ and following the policy $\pi$, the attainment of a target state $x$ is considered with variable step counts $k$. This concept is formally expressed as $\rho(s \rightarrow x, k, \pi)$, signifying the probability of reaching the target state $x$ from the initial state $s$ within $k$ sequential steps, following the prescribed policy $\pi$. For simplicity, we call this $\rho$ *visitation probability* (It's not official. I don't know if it has a formal name).  

We integrate the visitation probability $\rho$ into the gradient form, then we get,  

$$
\begin{aligned}
\nabla_\theta V^\pi(s) &= \phi(s) + \sum_a \pi_\theta(a|s) \sum_{s'}p(s'|s,a) \nabla_\theta V^\pi(s') \\\\
&= \phi(s) + \sum_{s'}\sum_a \pi_\theta(a|s)p(s'|s,a)\nabla_\theta V^\pi(s') \\\\
&= \phi(s) + \sum_{s'} \rho(s \rightarrow s', 1, \pi) \nabla_\theta V^\pi(s') \\\\
&= \phi(s) + \sum_{s'} \rho(s \rightarrow s', 1, \pi) \left[ \phi(s') + \sum_{s''}\rho(s' \rightarrow s'', 1, \pi)\nabla_\theta V^\pi(s'') \right] \\\\
&= \phi(s) + \sum_{s'}\rho(s \rightarrow s', 1, \pi)\phi(s') + \sum_{s''}\rho(s \rightarrow s'', 2, \pi) \nabla_\theta V^\pi(s'') \\\\
&= \cdots \\\\
&= \textcolor{red}{\sum_{x \in S}\sum_{k=0}^{\infty}\rho(s \rightarrow x, k, \pi)\phi(x)}
\end{aligned}
$$

Recall that the $\phi(s)$ denotes the cumulative gradient when considering all actions $a$ from the current state $s$, representing the gradient of the specific state-value "$V^\pi(s)$". By computing the probability of reaching the terminal state $x$ and incorporating the know cumulative gradient of the specific state $\phi(x)$, the formula computes the expectation over all possible terminal states. An alternative perspective is to consider the sum of visitation probability $\rho$ as a stationary state distribution in Markov Decision Process. We compute the current $V^\pi(s)$ by coupling the long-term probabilities of being in each terminal state while following specific policy $\pi$ with the expected state value of the terminal states.  

In the context of the visitation probability $\rho$, we rewrite the goal $\nabla_\theta J(\theta)$ as follows:  

$$
\begin{aligned}
\nabla_\theta J(\theta) &= \nabla_\theta V^\pi(s_0) \\\\
&= \sum_s \textcolor{red}{\sum_{k=0}^{\infty} \rho (s_0 \rightarrow s, k, \pi)}\phi(s) \\\\
&= \sum_s \textcolor{red}{\eta(s)}\phi(s) \\\\
&= \left( \sum_s \eta(s) \right) \sum_s \frac{\eta(s)}{\sum_s \eta(s)} \phi(s) \\\\
&\propto \sum_s \textcolor{red}{\frac{\eta(s)}{\sum_s \eta(s)}} \phi(s) \\\\
&= \sum_s \textcolor{red}{d^\pi(s)} \sum_a \nabla_\theta \pi_\theta(a|s)Q^\pi(s,a) \\\\
&= \sum_{s}d^\pi(s)\sum_a \pi_\theta (a|s)Q^\pi(s,a) \frac{\nabla_\theta \pi_\theta(a|s)}{\pi_\theta(a|s)} \\\\
&= E_{s \sim d^\pi, a \sim \pi_\theta} \left[ Q^\pi(s,a) \nabla_\theta \ln \pi_\theta(a|s) \right]
\end{aligned}
$$

Finally, we arrive at the vanilla policy gradient theorem which lays the foundation of most policy-based algorithms. 

$$
\nabla_\theta J(\theta) = E_{s \sim d^\pi, a \sim \pi_\theta} \left[ Q^\pi(s,a) \nabla_\theta \ln \pi_\theta(a|s) \right]
$$

## Policy Gradient Algorithms

Most policy-gradient algorithms can be regarded as variants of what we have proved above. Vanilla policy gradient theorem is unbiased but has high variance. Many variants based on it aims to reduce the variance while keeping unbiased.  

### REINFORCE & Its Variants

Reference:
> - [`Policy Gradient`](https://proceedings.neurips.cc/paper/1999/file/464d828b85b0bed98e80ade0a5c43b0f-Paper.pdf): Policy Gradient Methods for Reinforcement Learning with Function Approximation
> - [`REINFORCE`](https://link.springer.com/article/10.1007/BF00992696): Simple Statistical Gradient-Following Algorithms for Connectionist Reinforcement Learning
> - [`Generalized Advantage Estimation`](https://arxiv.org/abs/1506.02438): High-Dimensional Continuous Control Using Generalized Advantage Estimation
> - [`Reinforcement Learning: An Introduction`](http://incompleteideas.net/book/the-book-2nd.html): by Sutton & Barto

**REINFORCE** estimates the expected reward return $Q^\pi(s,a)$ by monte carlo using sampling episodes to update the gradient of policy $\pi_\theta(a|s)$. The idea behind is pretty simple but beautiful. We want to measure $Q$, so we just sample a full trajectory and the sampled expected reward is the actual expected return of $Q$ we want. Then we use this unbiased return to update our policy $\pi$.

$$
\nabla_\theta J(\theta) = E_\pi \left[ G_t \nabla_\theta \ln \pi_\theta(a_t|s_t) \right]
$$

The pseudo code is the as follows:
![reinforce](post/policy-gradient/reinforce.png)

### Deterministic Policy Gradient & Its Variants


### TRPO & PPO

