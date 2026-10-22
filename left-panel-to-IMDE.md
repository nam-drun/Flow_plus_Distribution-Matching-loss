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
Besides illustrating that we want $`\hat{X}_1^{Euler, P}`$ to be $X_1$, the left panel also displays our challenge. We don't really know Perturbed Continuous trajectory $X_t^P$ really is. This matters because we'll need to quantify the relationship between $v_P(t, X_t^P)$ and $v_B(t, X_t)$. So that given pair of $`\left\{X_0,X_1\right\}`$, if we know can derive $v_P(t, X_t^P)$, then we can measure on how well does distribution matching loss makes learned velocity field getting closer to ideal perturbed velocity field. 

# Theorem 3.2 in IMDE + Alekseev-Grobner formula
Now, theorem 3.2 in IMDE paper is:
```math
\begin{align*}
& \text{Given that:} 
& \Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
& \Phi_{h,f}(x) - \phi_{h,f}(x) = h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}), \\
& \text{which is backed by Figure 2 in the paper}. \\
& \text{This means that equation (3) in IMDE paper now becomes: } \\
& f_h = f + h^p f_p + h^{p+1} f_{p+1} + \cdots \\
& f_h = f + h^p f_p + \mathcal{O}(h^{p+1}) \\
& \text{We want to leverage discretisation error, therefore } f_p = -\delta f \\
& f_h - f = -\delta f h^p + \mathcal{O}(h^{p+1}) \\
& \text{Rewrite this in our notation} \\
& v_P(t,x) - v_B(t,x) = -\delta_{v_B}(t,x) h^p + \mathcal{O}(h^{p+1}) \\
\end{align*}
```
Here, we can see that $v_P(t,x) - v_B(t,x)$  has some specific relationship with $p^{th}$-order integration solver and $\Delta t$. But this is not yet $v_P(t, X_t^P) - v_B(t, X_t)$, which is our goal if we want to quantify $v_P(t, X_t^P)$ and $v_B(t, X_t)$. Now, enter Alekseev-Grobner formula from [Benton paper](https://openreview.net/forum?id=uqQPyWFDhY):
```math
\begin{align*}
& Y_{0,t}^x - Z_{0,t}^x = \int_0^t \left(\nabla_x Y_{s,t}^{Z_s} \right) \left(v_\theta(Z_s,s) - v^X(Z_s,s) \right) ds \\
& \forall t \in [s,t], Z_s \subset Z_{0,t}^x \text{ trajectory } \\
& \left(\nabla_x Y_{s,t}^{Z_s} \right) = \lim_{\epsilon \to 0} \frac{Y_{s,t}^{Z_s + \epsilon} - Y_{s,t}^{Z_s}}{\epsilon} \\
\\
& \text{Recontextualise this equation for the left panel}: \\
& X_1^P - X_1 = \int_0^1 \left(\nabla_x X_{s,1}^{P, X_s} \right) \left(v_P(X_s,s) - v_B(X_s,s) \right) ds, \\
& \forall s \in [s,1], X_s \subset \text{ Base Continuous } X_t \text{ trajectory } \\
& \nabla_x X_{s,1}^{P, X_s} = \lim_{\epsilon \to 0} \frac{X_{s,1}^{P, X_s + \epsilon} - X_{s,1}^{P, X_s}}{\epsilon}
\end{align*}
```
Rather than just believing our words. It's better to see it for yourself of what Alekseev-Grobner formula was trying to describe with the pictures below. As we recontextualise this formula, we simply say: Let dashed grey trajectory be Base Continuous that arrives $X_1$ at final time $t$, dashed purple line be Perturbed Continuous that arrives $X_1^P$ at final time $t$, and the purple line branching from dashed grey line be $X_{s,1}^{P, X_s}$. As shown by the 2nd picture below, $\left(\nabla_x X_{s,1}^{P, X_s} \right)$ simply tells us how changing the start point $X_s$ of the $X_{s,1}^{P, X_s}$ trajectory could alter its endpoint.

![alekseev-grobner-p1.png](https://github.com/nam-drun/Flow_plus_Distribution-Matching-loss/blob/main/plots/alekseev-grobner-p1.png)

![alekseev-grobner-p2.png](https://github.com/nam-drun/Flow_plus_Distribution-Matching-loss/blob/main/plots/alekseev-grobner-p2.png)



