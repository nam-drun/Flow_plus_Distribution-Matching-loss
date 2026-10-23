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
We want to quantify the relationship between $`v_P(t, X_t^P)`$ and $`v_B(t, X_t)`$. So that we can verify on whether distribution matching loss makes learned velocity field getting closer to ideal perturbed velocity field. Here, the left panel also displays our challenge: We don't really know Perturbed Continuous trajectory $`X_t^P`$ really is, so we can't derive $`v_P(t, X_t^P)`$ like how Flow Matching typically does it. Fortunately, theorem 3.2 in IMDE paper can help us quantify $`v_P(t, X_t^P) - v_B(t, X_t)`$.


# Theorem 3.2 in IMDE
Now, the first expression in theorem 3.2 is:
```math
\begin{align*}
& \Phi_{h,f}(x) = \phi_{h,f}(x) + h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}) \\
& \Phi_{h,f}(x) - \phi_{h,f}(x) = h^{p+1} \delta_f(x) + \mathcal{O}(h^{p+2}), \\
\end{align*}
```

Which is backed by Figure 2 in the paper. This expression implicates that $`v_P(t, X_t^P)`$ is affected by our intent to leverage discretisation error via $-\delta f$.
```math
\begin{align*}
& f_h - f = - h^p -\delta f + \mathcal{O}(h^{p+1}) \\
& f_h - f = -\delta f h^p + \mathcal{O}(h^{p+1}) \\
& v_P(t,x) - v_B(t,x) = -\delta_{v_B}(t,x) h^p + \mathcal{O}(h^{p+1}) \\
& v_P(t,x) - v_B(t,x) = \mathcal{O}(h^p)
\end{align*}
```

Although this is not yet $`v_P(t, X_t^P) - v_B(t, X_t)`$, it is nonetheless enables us to delegate the unknown to a simpler expression
```math
\begin{align*}
& v_P(t, X_t^P) - v_B(t, X_t) = v_P(t, X_t^P) - v_B(t, X_t^P) + v_B(t, X_t^P) - v_B(t, X_t) \\
& v_P(t, X_t^P) - v_B(t, X_t) = -\delta_{v_B}(t,x) h^p + \mathcal{O}(h^{p+1}) + v_B(t, X_t^P) - v_B(t, X_t)
\end{align*}
```

Now we just need to quantify $`v_B(t, X_t^P) - v_B(t, X_t)`$ with Taylor expansion.
```math
v_B(t, X_t^P) = v_B(t, X_t) + \nabla_x v_B(t, X_t)(X_t^P - X_t) + \mathcal{O}(\left\| X_t^P - X_t  \right\|^2)
```

Here, we invoke the Lipschitz regularity assumption on $v_B(t,x)$, where $\left\| v_B(t,x) - v_B(t, x+c) \right\| \le L_t \left\| (x+c) - x \right\|$. This assumption generally holds true in Flow Matching, as shown by section 3.2 in [Error Bounds for Flow Matching Methods paper](https://openreview.net/forum?id=uqQPyWFDhY).

