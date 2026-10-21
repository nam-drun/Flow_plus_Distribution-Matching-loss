| Left panel                            | [IMDE paper](https://arxiv.org/abs/2206.07335) | Meaning                                           | How to access these expression during training/sampling            |
| ------------------------------------  | -----------------------------------------------| --------------------------------------------------|------------------------------------------------------------------- | 
| N                                     | S                                              | number of sampling steps                          | $\frac1N = \Delta t = h$                                           | 
| $X_0$                                 | $x$                                            | start point                                       | $X_0 \sim \mathcal{N}(0, I_d)$                                     |
| $X_1$                                 | $\phi_{Sh,f}(x)$                               | Base continuous end point                         | $X_0 + \int_0^1 v(s, X_s) ds$. $Sh$ is time interval               |
| $v_B(t, x)$                           | $f$                                            | Base velocity field                               | $`v_B(t, x) \approx \dot{X}_t`$                                    |
| $v_P(t, x)$                           | $f_h$                                          | Perturbed velocity field                          | $`v_P(t, x) = v_B(t, x) + \Delta \theta`$. $\theta$ is NN weight   |
| $`\hat{X}_1^{Euler, B}`$              | $(\Phi_{h,f})^S(x)$                            | Base Euler end point.                             | $`\hat{X}_1^{Euler, B} = \Delta t \sum_{k=0}^{N-1} v_B(t_k, x)`$   |
| $`\hat{X}_1^{Euler, P}`$              | $(\Phi_{h,f_h})^S(x)$                          | Perturbed Euler end point                         | $`\hat{X}_1^{Euler, P} = \Delta t \sum_{k=0}^{N-1} v_P(t_k, x)`$   |
| $`e_B^{disc}`$                        | $(\Phi_{h,f})^S(x) - \phi_{Sh,f}(x)$           | discretisation error at $t=1$ of Base model       | $`e_B^{disc} = X_1 - \hat{X}_1^{Euler, B}`$                        |
| $`d_N`$                               | $(\Phi_{h,f_h})^S(x) - (\Phi_{h,f})^S(x)$      | discretisation error counteraction at $t=1$       | $`d_N = \hat{X}_1^{Euler, P} - \hat{X}_1^{Euler, B}`$              |

The left panel in the main page was not just merely illustrative. We want to visually invoke an idea that if you want $`\hat{X}_1^{Euler, P}`$ to be $X_1$, then $e_B^{disc} + d_N$ needs to 0. In IMDE's notation, $e_B^{disc} + d_N$ is $(\Phi_{h,f_h})^S(x) - \phi_{Sh,f}(x)$. Now, the first expression in theorem 3.2 in IMDE paper is:
```math
\begin{align*}
& \Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
& \Phi_{h,f}(x) - \phi_{h,f}(x) = h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
\end{align*}
```
As shown above, once we arrange this expression, now it means that the discretisation error at each time step has some specific relationship with arbitrary $p^{th}$-order integration solver and $\Delta t$. In IMDE paper, we further see that this relationship is backed by their Figure 2, but whether it actually holds for Flow Matching's discretisation error, that's an open question. For the moment, what matters is that how do we connect $\Phi_{h,f}(x) - \phi_{h,f}(x)$ to $(\Phi_{h,f_h})^S(x) - \phi_{Sh,f}(x)$. Here, IMDE paper provides this equality to enable this connection:
```math
\begin{align*}
(\Phi_{h,f_h})^S(x) = (\phi_{h,f})^S(x) = \phi_{Sh,f}(x)
\end{align*}
```

$\phi_{Sh,f}(x) = (\phi_{h,f})^S(x)$ just simply means you segment Base Continuous trajectory into $S$ segments, each segment now has $h$ time interval. Now, $(\Phi_{h,f_h})^S(x) = \phi_{Sh,f}(x)$ is basically saying we wish $`\hat{X}_1^{Euler, P}`$ to be $X_1$.

