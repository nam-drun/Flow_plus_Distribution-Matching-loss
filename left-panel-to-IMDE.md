| Left panel                            | IMDE                                    | Meaning                                          | 
| ------------------------------------  | ----------------------------------------| -----------------------------------------------  | 
| N                                     | S                                       | number of sampling steps                         |
| $X_0$                                 | $x$                                     | start point                                      | 
| $X_1$                                 | $\phi_{Sh,f}(x)$                        | Base Continuous end point                        | 
| $v_{Base}(t, x)$                      | $f$                                     | Base velocity field                              |
| $v_{perturbed}(t, x)$                 | $f_h$                                   | Ideal perturbed velocity field                   |
| $`\hat{X}_1^{Euler, Base}`$           | $\Phi_{Sh,f}(x)$                        | Base Euler endpoint                              |
| $`\hat{X}_1^{Euler, Perturbed}`$      | $\Phi_{Sh,f_h}(x)$                      | Perturbed Euler endpoint                         |
| $`e_B^{disc}`$                        | $\Phi_{Sh,f}(x) - \phi_{Sh,f}(x)$       | discretisation error of base model               |
| $`d_N`$                               | $\Phi_{Sh,f_h}(x) - \Phi_{Sh,f}(x)$     | Perturbed Euler end point - Base Euler end point |

Now, say you want to reproduce the left panel, then naturally $e_B^{disc} + d_N$ needs to 0. In IMDE paper, this is $\Phi_{Sh,f_h}(x) - \phi_{Sh,f}(x)`$. Theorem 3.2 tells us that:
```math
\Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2})
```
