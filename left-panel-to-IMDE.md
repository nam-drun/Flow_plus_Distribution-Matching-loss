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
The left panel in the main page was not just merely illustrative. We want to visually invoke an idea that if you want $`\hat{X}_1^{Euler, P}`$ to be $X_1$, then you need $e_B^{disc} + d_N$ to be 0. More importantly, if we want to understand the implication of IMDE paper with respects to post training Flow Matching with distribution matching loss, then we'll need to quantify the relationship between $v_P(t, X_t^P)$ and $v_B(t, X_t)$. Because for a given pair of $`\left\{X_0,X_1\right\}`$, if we know can derive $v_P(t, X_t^P)$, then we can measure on how well does distribution matching loss makes learned velocity field getting closer to ideal perturbed velocity field. 

# IMDE + Alekseev-Grobner formula
Now, $e_B^{disc} + d_N$ in IMDE's notation is $(\Phi_{h,f_h})^S(x) - \phi_{Sh,f}(x)$. Now, the first expression in theorem 3.2 in IMDE paper is:
```math
\begin{align*}
& \Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
& \Phi_{h,f}(x) - \phi_{h,f}(x) = h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
\end{align*}
```
As shown above, once we arrange this expression, now it means that the discretisation error at each time step has some specific relationship with arbitrary $p^{th}$-order integration solver and $\Delta t$. In IMDE paper, we further see that this relationship is backed by their Figure 2, but whether it holds for Flow Matching's discretisation error, that's an open question. For the moment, what matters is that how do we connect $\Phi_{h,f}(x) - \phi_{h,f}(x)$ to $(\Phi_{h,f_h})^S(x) - \phi_{Sh,f}(x)$. Here, IMDE paper provides this equality to enable this connection:
```math
\begin{align*}
(\Phi_{h,f_h})^S(x) = (\phi_{h,f})^S(x) = \phi_{Sh,f}(x)
\end{align*}
```
$\phi_{Sh,f}(x) = (\phi_{h,f})^S(x)$ just simply means you segment Base Continuous trajectory into $S$ segments, each segment now has $h$ time interval. Now, $(\Phi_{h,f_h})^S(x) = \phi_{Sh,f}(x)$ is basically saying we wish $`\hat{X}_1^{Euler, P}`$ to be $X_1$. So now we have $`(\Phi_{h,f_h})^S(x) = (\phi_{h,f})^S(x)`$. But this is not yet related to $`\Phi_{h,f}(x) - \phi_{h,f}(x)`$, and we are still not remotely closer to finding out the relationship between $v_P(t, X_t^P)$ and $v_B(t, X_t)$. Enter Alekseev-Grobner formula from [Benton paper](https://openreview.net/forum?id=uqQPyWFDhY).
```math
\begin{align*}
& Y_{0,t}^x - Z_{0,t}^x = \int_0^t \left(\nabla_x Y_{s,t}^{Z_s} \right) \left(v_\theta(Z_s,s) - v^X(Z_s,s) \right) ds \\
& \forall t \in [s,t], Z_s \subset Z_{0,t}^x \text{ trajectory } \\
& \left(\nabla_x Y_{s,t}^{Z_s} \right) = \lim_{\epsilon \to 0} \frac{Y_{s,t}^{Z_s + \epsilon} - Y_{s,t}^{Z_s}}{\epsilon} \\
& \text{Recontextualise this equation for the left panel}: \\
& X_t^P - X_t = \int_0^t \left(\nabla_x X_{s,t}^{P, X_s} \right) \left(v_P(X_s,s) - v_B(X_s,s) \right) ds, \\
& \forall t \in [s,t], X_s \subset \text{ Base Continuous } X_t \text{ trajectory } \\
& \nabla_x X_{s,t}^{P, X_s} = \lim_{\epsilon \to 0} \frac{X_{s,t}^{P, X_s + \epsilon} - X_{s,t}^{P, X_s}}{\epsilon}
\end{align*}
```
Rather than just believing our words. It's better to see it for yourself of what Alekseev-Grobner formula from Benton paper was trying to describe with the pictures below. As we recontextualise the Alekseev-Grobner formula, we simply say: Let dashed grey line be Base Continuous $X_t$, dashed purple line be Perturbed Continuous $X_t^P$, and the purple line branching from dashed grey line be $X_{s,t}^{P, X_s}$. 

![alekseev-grobner-p1.png](https://github.com/nam-drun/Flow_plus_Distribution-Matching-loss/blob/main/plots/alekseev-grobner-p1.png)

![alekseev-grobner-p2.png](https://github.com/nam-drun/Flow_plus_Distribution-Matching-loss/blob/main/plots/alekseev-grobner-p2.png)



