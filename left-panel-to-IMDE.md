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
We want to quantify the relationship between $`v_P(t, X_t^P)`$ and $`v_B(t, X_t)`$. So that we can verify on whether distribution matching loss makes learned velocity field getting closer to $`v_P(t, X_t^P)`$. Here, the left panel also displays our challenge: We don't really know Perturbed Continuous trajectory $`X_t^P`$ really is, so we can't derive $`v_P(t, X_t^P)`$ like how Flow Matching typically does it. Fortunately, theorem 3.2 in IMDE paper can help us quantify $`v_P(t, X_t^P) - v_B(t, X_t)`$.


# Bounding theorem 3.2 from IMDE paper
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
\end{align*}
```

Although this is not yet $`v_P(t, X_t^P) - v_B(t, X_t)`$, it is nonetheless enables us to delegate the unknown to a simpler expression
```math
\begin{align*}
& v_P(t, X_t^P) - v_B(t, X_t) = v_P(t, X_t^P) - v_P(t, X_t) + v_P(t, X_t) - v_B(t, X_t) \\
& v_P(t, X_t^P) - v_B(t, X_t) = v_P(t, X_t^P) - v_P(t, X_t) -\delta_{v_B}(t,X_t) h^p + \mathcal{O}(h^{p+1}) 
\end{align*}
```

Here, we invoke the Lipschitz regularity assumption on $v_B(t, X_t)$: $`\left\| v_B(t, X_t^P) - v_B(t, X_t) \right\| \le L(t) \left\| X_t^P - X_t \right\|`$. Note, if you feel confused about this statement, just read it as $`\left\| v_B(t, x + c) - v_B(t, x) \right\| \le L(t) \left\| (x + c) - x \right\|`$ with $`x \in X_t`$ and $`(x + c) \in X_t^P`$; then when I add $`c \to 0`$, this statement now reads like 1st principle of differentiation on $`v_B(t, x)`$. In other words, as $`c \to 0`$, $`L(t)`$ quite literally becomes the slope of $`v_B(t, x)`$. Anyway, this assumption generally holds true in Flow Matching, as shown by section 3.2 in [Error Bounds for Flow Matching Methods paper](https://openreview.net/forum?id=uqQPyWFDhY). 

From this assumption on $v_B(t, X_t)$, we evoke another Lipschitz regularity assumption on $v_P(t,x)$: $`\left\| v_P(t, X_t^P) - v_P(t, X_t) \right\| \le L(t)_P \left\| X_t^P - X_t \right\|`$. ***The central theme of our work is that Perturbed Continuous is to be treated like another trajectory (continuous ODE)*** $\in \mathbb{R}_d$!! Sure, it cannot be a Flow Matching since the core tenet of Flow Matching is that there is only 1 trajectory $\in \mathbb{R}_d$ that transports $X_0 \to X_1$. But that doesn't mean Perturbed Continuous cannot be treated as some other continuous ODE $\in \mathbb{R}_d$. So now we can bound $`v_P(t, X_t^P) - v_B(t, X_t)`$ as follows:
```math
\left\| v_P(t, X_t^P) - v_B(t, X_t) \right\| \le L(t)_P \left\| X_t^P - X_t \right\| + \left\| \delta_{v_B}(t,X_t) h^p \right\| + \mathcal{O}(h^{p+1}) 
```

Now we just need to figure out $`\left\| X_t^P - X_t \right\|`$ with Gronwall's inequality. Gronwall's inequality has this basic structure:
```math
\begin{align*}
& \text{Given } \dot{f}(t) \le B(t)f(t) \Leftrightarrow f(t) \le f(0) + \int_0^t g(s)f(s) ds, \\
& \text{thus } f(t) \le f(0)\exp \left(\int_0^t g(s)ds \right) \\
\end{align*}
```
where $`g(s)`$ and $`f(t)`$ are some real-valued functions. Denote $`f(t) := \left\| X_t^P - X_t \right\|`$, hence $`\dot{f}(t) := \left\| v_P(t, X_t^P) - v_B(t, X_t) \right\|`$ and $`f(0) = 0`$ (Perturbed Continuous and Base Continuous starts from the same $`X_0`$). Now, we have just figured out the upper bound for $`\left\| v_P(t, X_t^P) - v_B(t, X_t) \right\|`$, so plug that into the RHS of Gronwall's inequality, we get:
```math
\left\| X_t^P - X_t \right\| \le \exp \left(\int_0^t g(s)ds \right)
```
