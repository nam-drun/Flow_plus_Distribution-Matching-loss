| Left panel                            | [IMDE paper](https://arxiv.org/abs/2206.07335) | Meaning                                                  | 
| ------------------------------------  | -----------------------------------------------| ---------------------------------------------------------| 
| N                                     | S                                              | number of sampling steps                                 |
| $X_0$                                 | $x$                                            | start point                                              | 
| $X_1$                                 | $\phi_{Sh,f}(x)$                               | Base continuous end point                                | 
| $v_B(t, x)$                           | $f$                                            | Base velocity field                                      |
| $v_P(t, x)$                           | $f_h$                                          | Perturbed velocity field                                 |
| $`\hat{X}_1^{Euler, B}`$              | $(\Phi_{h,f})^S(x)$                            | Base Euler end point. $h$ is step size of $              | 
| $`\hat{X}_1^{Euler, P}`$              | $(\Phi_{h,f_h})^S(x)$                          | Perturbed Euler end point                                |
| $`e_B^{disc}`$                        | $(\Phi_{h,f})^S(x) - \phi_{Sh,f}(x)$           | discretisation error of Base model                       |
| $`d_N`$                               | $(\Phi_{h,f_h})^S(x) - (\Phi_{h,f})^S(x)$      | Perturbed Euler end point - Base Euler end point         |

The left panel in the main page was not just merely illustrative. We want to visually invoke an idea that if you want $`\hat{X}_1^{Euler, P}`$ to be $X_1$, then $e_B^{disc} + d_N$ needs to 0. In IMDE's notation, we get $e_B^{disc} + d_N = (\Phi_{h,f_h})^S(x) - \phi_{Sh,f}(x)$. Now, theorem 3.2 in IMDE paper tells us that:
```math
\Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2})
```
