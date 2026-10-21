| Left panel                            | [IMDE paper](https://arxiv.org/abs/2206.07335) | Meaning                                           | How to access these expression during training/sampling            |
| ------------------------------------  | -----------------------------------------------| --------------------------------------------------|------------------------------------------------------------------- | 
| N                                     | S                                              | number of sampling steps                          | $\frac1N = \Delta t = h$                                           | 
| $X_0$                                 | $x$                                            | start point                                       | $X_0 \sim \mathcal{N}(0, I_d)$                                     |
| $X_1$                                 | $\phi_{Sh,f}(x)$                               | Base continuous end point                         | $X_0 + \int_0^1 v(s, X_s) ds$. $Sh$ is time interval               |
| $v_B(t, x)$                           | $f$                                            | Base velocity field                               | $`v_B(t, x) \approx \dot{X}_t`$                                    |
| $v_P(t, x)$                           | $f_h$                                          | Perturbed velocity field                          | $`v_P(t, x) = v_B(t, x) + \Delta \theta`$. $\theta$ is NN weight   |
| $`\hat{X}_1^{Euler, B}`$              | $(\Phi_{h,f})^S(x)$                            | Base Euler end point.                             | $`\hat{X}_1^{Euler, B} = \Delta t \sum_{k=0}^{N-1} v_B(t_k, x)`$   |
| $`\hat{X}_1^{Euler, P}`$              | $(\Phi_{h,f_h})^S(x)$                          | Perturbed Euler end point                         | $`\hat{X}_1^{Euler, P} = \Delta t \sum_{k=0}^{N-1} v_P(t_k, x)`$   |
| $`e_B^{disc}`$                        | $(\Phi_{h,f})^S(x) - \phi_{Sh,f}(x)$           | discretisation error of Base model                | $`e_B^{disc} = X_1 - \hat{X}_1^{Euler, B}`$                        |
| $`d_N`$                               | $(\Phi_{h,f_h})^S(x) - (\Phi_{h,f})^S(x)$      | discretisation error counteraction                | $`d_N = \hat{X}_1^{Euler, P} - \hat{X}_1^{Euler, B}`$              |

The left panel in the main page was not just merely illustrative. We want to visually invoke an idea that if you want $`\hat{X}_1^{Euler, P}`$ to be $X_1$, then $e_B^{disc} + d_N$ needs to 0. In IMDE's notation, we get $e_B^{disc} + d_N = (\Phi_{h,f_h})^S(x) - \phi_{Sh,f}(x)$. Now, theorem 3.2 in IMDE paper tells us that:
```math
\Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2})
```
