| Left panel                            | IMDE                                                                                               | Meaning                                       | 
| ------------------------------------  | -------------------------------------------------------------------------------------------------- | --------------------------------------------- | 
| $X_0$                                 | $x$                                                                                                | start point                                   | 
| $X_1$                                 | $\varphi_{Nh,f(x)}$                                                                                | ideal end point                               | 
| $v_{Base}(t, x)$                      | $f$                                                                                                | Base velocity field                           |
| $v_{perturbed}(t, x)$                 | $f_h$                                                                                              | Ideal perturbed velocity field                |
| $v_{\theta}(t, x)$                    | $f_{\theta}$                                                                                       | learned velocity field                        |
| $`\hat{X}_1^{Euler, Base}`$           | $`\left(\Phi_{h,f}\right)^N \left(x \right)`$                                                      | Base Euler endpoint                           |
| $`\hat{X}_1^{Euler, Perturbed}`$      | $`\left(\Phi_{h,f_{\theta}}\right)^N \left(x \right)`$                                             | Perturbed Euler endpoint                      |
| $`e_B^{disc}`$                        | $`\left(\Phi_{h,f}\right)^N \left(x \right) - \varphi_{Nh,f(x)}`$                                  | discretisation error of base model            |
| $`d_N`$                               | $`\left(\Phi_{h,f_{\theta}}\right)^N \left(x \right) - \left(\Phi_{h,f}\right)^N \left(x \right)`$ | (Perturbed-Base) Euler endpoint displacement  |

