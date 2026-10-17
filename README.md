# Flow_plus_Distribution-Matching-loss
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

Because there was the claim of "reward hacking" from FD paper, i.e., the model was post-trained on same metric that it was evaluated. We decide to use W2. W2 has its own issue, too. Despite all of our fixes to W2 such as fixing its bias to contracted distribution via implementing [this paper](https://neurips.cc/virtual/2025/loc/san-diego/124284), nothing can really fix W2 issue with high dim. In other words, despite all its flaws, FD is still better to use than W2 in high dim. The best we could do is lowering ambient dimensionality so that W2 does not suck.

Anyway, from all these results, we find that only Flow + KL loss could consistently work with more accurate solvers such as Heun and Dormand-Prince 5(4) at 4 sampling steps, whereas all other configurations only work well for Euler at 4 sampling steps. This means than only Flow + KL loss could outperform pretrained flow with accurate solver at > 75 sampling steps. This carries the following implications:
1. Mgflow paper was correct to pursue for Flow + KL loss. And unlike Mgflow paper, we did not have to reweight our loss.
2. TDM paper was published long before FD paper, advFD paper and Mgflow paper. Yet TDM paper did not have good result because its pretrained flow model was not the best. I.e., [JiT paper](https://arxiv.org/abs/2511.13720v1) found out among 3 choice of learned parameters {data, score, velocity}, predicting data is the best choice by a large gap. TDM paper chose to predict score. If your pretrained model sucks, there's only so much you can polish it with KL loss during post-training.
3. We think this may pave the way to understand why KL penalty plays a in MDP + RLHF line-of-research for image generation. Looking at eq(9) in [Qwen-Image 2.0 technical report](https://arxiv.org/abs/2606.27608v1), you can see that KL is again used during post-training. 
