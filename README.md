# Why care?
We have figured out that most distribution matching losses could hunt for discretisation error (in theory). And in practice, by having pretrained flow model serving as a good initialisation during training, distribution matching loss could actually hunt for discretisation error in practice. Then, we also have figured out that among the choice of distribution matching losses, such as Frechet Distance, GAN Adversarial, and KL, KL works best. This carries 2 practical implications:
1. It might help us understand how RLHF helps in MDP in image generation (see Qwen-Image 2.0). Until someone proves that RLHF cannot hunt for discretisation error, this remains an interesting possibility to figure out.
2. Flow + KL loss could be more computational efficient than MDP + RLHF in image generation. 

# How does distribution matching loss hunt for discretisation error (in theory)
First, we want to establish the idea that modern diffusion/flow matching is about reconstructing trajectory using initial position $X_0$, velocity field $v(t, X_t)$ and integrating solver such euler, heun, or dormand-prince5(4). In high school physics, people have learn the idea that *distance = velocity x time*. The same idea applies for flow matching during sampling. If you use Euler, it is quite literally *next position = current position + velocity x time*, and you keep doing this until *next position* = $\hat{X}_1$, which is a generated image. 

In Flow Matching, you train neural network to learn ground truth velocity field via mean-squared error. So where is ground truth velocity field coming from? Simple, during training, you sample a random pair of endpoints $`\left\{ X_0, X_1 \right\}`$. Then you compute current position $X_t$ using noise schedule. E.g., if you use VP schedule from DDPM diffusion, then $X_t = \sqrt{1 - t^2}X_0 + tX_1$; if you use linear schedule from [JiT paper](https://arxiv.org/abs/2511.13720v1) then $X_t = (1-t)X_0 + tX_1$. Then given current position $X_t$, you take its derivative to get velocity field at current position $v(t, X_t)$. Again, this is a classical idea from high school physics where velocity = derivative of current position. Now, on paper velocity field in flow matching is the average of derivative of current position, i.e., $v(t, X_t) = \mathbb{E}[\dot{X}_t \mid X_t = x]$. People design velocity field in this way so that if there are multiple trajectories passing through the point $x$, we can still derive some ground truth velocity field to train the neural network. However, in practice, the ambient space has so many dimensions such that the chance of multiple trajectories intersecting each other is basically none. So $\mathbb{E}[\dot{X}_t \mid X_t = x] \approx \dot{X}_t$. In other words, so long that ambient space has high number of dimensions (which is practically always true in image generation), then you can think of flow matching like high-school physics *distance = velocity x time*.

Second, we want to establish a simple idea that discretisation error is just the difference between $X_1$ and generated $\hat{X}_1$. See picture below 

This repo contains toy experiment results for post-training flow model in the following configurations:
1. Flow + Frechet Distance (FD) loss
2. Flow + FD loss + Adversarial loss using FD as distance metric
3. Flow + Adversarial using FD as instance metric
4. Flow + FD loss + KL loss
5. Flow + KL loss

(1) follows the same setup as [FD paper by Jiawei Yang](https://arxiv.org/abs/2604.28190). This is the baseline configuration since it inspires our research.
(2)(4) follows the same setup as [advFD paper](https://arxiv.org/abs/2608.11205), which combines FD with Adversarial loss - see eq(2) in the paper.
(3) is an ablation of eq(2) in [advFD paper](https://arxiv.org/abs/2608.11205), i.e., what if we just simply use Adversarial loss.
(5) follows the same setup as [TDM paper](https://arxiv.org/abs/2503.06674) and [MGflow paper](https://arxiv.org/abs/2609.35763)

Because there was the claim of "reward hacking" from FD paper, i.e., the model was post-trained on same metric that it was evaluated. We decide to use W2. W2 has its own issue, too. Despite all of our fixes to W2 such as fixing its bias to contracted distribution via implementing [this paper](https://neurips.cc/virtual/2025/loc/san-diego/124284), nothing can really fix W2 issue with high dim. In other words, despite all its flaws, FD is still better to use than W2 in high dim. The best we could do is lowering ambient dimensionality so that W2 does not suck. All in all, even if W2 isn't a perfect metric. We recommend evaluating on both FD and W2 and look for common trends. In other words, think of W2 as "2nd opinion".

Anyway, from all these results, we find that only Flow + KL loss could consistently work with more accurate solvers such as Heun and Dormand-Prince 5(4) at 4 sampling steps, whereas all other configurations only work well for Euler at 4 sampling steps. This means than only Flow + KL loss could outperform pretrained flow with accurate solver at > 75 sampling steps. This carries the following implications:
1. Mgflow paper was correct to pursue for Flow + KL loss. And unlike Mgflow paper, we did not have to reweight our loss.
2. TDM paper was published long before FD paper, advFD paper and Mgflow paper. Yet TDM paper did not have good result because its pretrained flow model was not the best. I.e., [JiT paper](https://arxiv.org/abs/2511.13720v1) found out among 3 choice of learned parameters {data, score, velocity}, predicting data is the best choice by a large gap. TDM paper chose to predict score. If your pretrained model sucks, there's only so much you can polish it with KL loss during post-training.
3. We think this may pave the way to understand why KL penalty plays a in MDP + RLHF line-of-research for image generation. Looking at eq(9) in [Qwen-Image 2.0 technical report](https://arxiv.org/abs/2606.27608v1), you can see that KL is again used during post-training. 
