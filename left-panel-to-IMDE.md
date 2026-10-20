| Left panel                            | IMDE                                                                                               | Meaning                                        | 
| ------------------------------------  | -------------------------------------------------------------------------------------------------- | ---------------------------------------------- | 
| N                                     | S                                                                                                  | number of sampling steps                       |
| $X_0$                                 | $x$                                                                                                | start point                                    | 
| $X_1$                                 | $\phi_{Sh,f(x)}$                                                                                   | ideal end point                                | 
| $v_{Base}(t, x)$                      | $f$                                                                                                | Base velocity field                            |
| $v_{perturbed}(t, x)$                 | $f_h$                                                                                              | Ideal perturbed velocity field                 |
| $`\hat{X}_1^{Euler, Base}`$           | $`\left(\Phi_{h,f}\right)^S \left(x \right)`$                                                      | Base Euler endpoint                            |
| $`\hat{X}_1^{Euler, Perturbed}`$      | $`\left(\Phi_{h,f_h}\right)^S \left(x \right)`$                                             | Perturbed Euler endpoint                       |
| $`e_B^{disc}`$                        | $`\left(\Phi_{h,f}\right)^S \left(x \right) - \phi_{Sh,f(x)}`$                                     | discretisation error of base model             |
| $`d_N`$                               | $`\left(\Phi_{h,f_h}\right)^S \left(x \right) - \left(\Phi_{h,f}\right)^S \left(x \right)`$ | Perturbed Euler endpoint - Base Euler endpoint |

Suppose you want to reproduce the left panel, then naturally $e_B^{disc} + d_N = 0$. In IMDE paper, this is $`\left(\Phi_{h,f_h}\right)^S \left(x \right) - \phi_{Sh,f(x)}`$. Theorem 3.2 tells us that:
```math
\Phi_{h,f}(x) = \phi_{h,f(x)} + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2})
```
