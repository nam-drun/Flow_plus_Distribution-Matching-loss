| Left panel                            | [IMDE paper](https://arxiv.org/abs/2206.07335) | Meaning                                           | How to access them during training/sampling                        |
| ------------------------------------  | -----------------------------------------------| --------------------------------------------------|------------------------------------------------------------------- | 
| N                                     | S                                              | number of sampling steps. $h$ is time interval    | $\frac1N = \Delta t = h$                                           | 
| $X_0$                                 | $x$                                            | start point                                       | $X_0 \sim \mathcal{N}(0, I_d)$                                     |
| $X_1$                                 | $\phi_{Sh,f}(x)$                               | Base continuous end point. $Sh$ is time interval  | $X_0 + \int_0^1 v(s, X_s) ds$. $Sh$ is time interval               |
| $v_B(t, x)$                           | $f$                                            | Base velocity field                               | $`v_B(t, x) \approx \dot{X}_t`$                                    |
| $v_P(t, x)$                           | $f_h$                                          | Perturbed velocity field                          | $`v_P(t, x) = v_B(t, x) + \Delta \theta`$. $\theta$ is NN weight   |
| $`\hat{X}_1^{Euler, B}`$              | $(\Phi_{h,f})^S(x)$                            | Base Euler end point.                             | $`\hat{X}_1^{Euler, B} = \Delta t \sum_{k=0}^{N-1} v_B(t_k, x)`$   |
| $`\hat{X}_1^{Euler, P}`$              | $(\Phi_{h,f_h})^S(x)$                          | Perturbed Euler end point                         | $`\hat{X}_1^{Euler, P} = \Delta t \sum_{k=0}^{N-1} v_P(t_k, x)`$   |
| $`e_B^{disc}`$                        | $(\Phi_{h,f})^S(x) - \phi_{Sh,f}(x)$           | discretisation error at $t=1$ of Base model       | $`e_B^{disc} = \hat{X}_1^{Euler, B} - X_1`$                        |
| $`d_N`$                               | $(\Phi_{h,f_h})^S(x) - (\Phi_{h,f})^S(x)$      | discretisation error counteraction at $t=1$       | $`d_N = \hat{X}_1^{Euler, P} - \hat{X}_1^{Euler, B}`$              |

# What to get out from IMDE paper
Besides illustrating that we want $`\hat{X}_1^{Euler, P}`$ to be $X_1$, the left panel also displays our challenge. We don't really know Perturbed Continuous trajectory $`X_t^P`$ really is. This matters because we'll need to quantify the relationship between $`v_P(t, X_t^P)`$ and $`v_B(t, X_t)`$. So that given pair of $`\left\{X_0,X_1\right\}`$, if we can derive $`v_P(t, X_t^P)`$, then we can measure on how well does distribution matching loss makes learned velocity field getting closer to ideal perturbed velocity field. 

# Theorem 3.2 in IMDE
Now, the first expression in theorem 3.2 is:
```math
\begin{align*}
& \Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
& \Phi_{h,f}(x) - \phi_{h,f}(x) = h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}), \\
\end{align*}
```

Which is backed by Figure 2 in the paper. This expression implicates that its equation (3) in IMDE paper (i.e., essentially power series expansion of $`v_P(t, X_t^P)`$) now has at least 1 term that leverage discretisation error to recover clean image. That term is $-\delta f$.
```math
\begin{align*}
& f_h - f = - h^p -\delta f + \mathcal{O}(h^{p+1}) \\
& f_h - f = -\delta f h^p + \mathcal{O}(h^{p+1}) \\
& v_P(t,x) - v_B(t,x) = -\delta_{v_B}(t,x) h^p + \mathcal{O}(h^{p+1}) \\
\end{align*}
```
Here, we can see that $v_P(t,x) - v_B(t,x)$  has some specific relationship with $p^{th}$-order integration solver and $\Delta t$. But this is not yet $v_P(t, X_t^P) - v_B(t, X_t)$, which is our goal if we want to quantify $v_P(t, X_t^P)$ and $v_B(t, X_t)$. ***Now, this is where the left panel and the IMDE theory diverges***: Essentially, left panel assumes that $`e_B^{disc} = e_P^{disc}`$ because there is no reason that Perturbed Continuous $X_t^p$ cannot be treated as it is another continuous trajectory in $`\mathbb{R}^d`$ like Base Continuous $X_t$; but IMDE predicts that $`e_P^{disc} = e_B^{disc} + \mathcal{O}(h^2)`$. So, there are 2 different hypotheses of how $v_P(t, X_t^P) - v_B(t, X_t)$ should be.


