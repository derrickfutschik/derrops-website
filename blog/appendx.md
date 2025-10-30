---
slug: appendix
title: appendix
authors: [derrops]
tags: []
draft: true
---

| **Contact Type**                   | **Formula (LaTeX)**                                                                                                                                                                                                   |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sphere–Sphere**                  | $$\text{contact} \iff \|c_2 - c_1\| \le r_1 + r_2$$                                                                                                                                                                   |
| **Sphere–AABB**                    | $$p = \operatorname{clamp}(c, m, M), \quad \text{contact} \iff \|p - c\| \le r$$                                                                                                                                      |
| **Capsule–Capsule**                | $$d = \operatorname{dist}([a_1, b_1], [a_2, b_2]), \quad \text{contact} \iff d \le r_1 + r_2$$                                                                                                                        |
| **Triangle–Triangle / Primitive**  | $$\text{contact} \iff d_{\min} \le 0 \quad (\text{or } d_{\min} \le \varepsilon)$$                                                                                                                                    |
| **Separating Axis Theorem (SAT)**  | $$\max_{a \in A} \langle a, \hat{n} \rangle < \min_{b \in B} \langle b, \hat{n} \rangle$$; $$\text{contact} \iff \text{no separating axis exists}$$                                                                   |
| **GJK (Minkowski Difference)**     | $$A \ominus B = \{\, a - b \mid a \in A,\, b \in B \,\}$$; $$\operatorname{support}_{A \ominus B}(d) = \operatorname{support}_A(d) - \operatorname{support}_B(-d)$$; $$0 \in A \ominus B \Rightarrow \text{contact}$$ |
| **Signed Distance Field (SDF)**    | $$\min_{x \in \mathbb{R}^3}\big( \phi_A(x), \phi_B(x) \big) \le 0$$                                                                                                                                                   |
| **Continuous Collision (Spheres)** | $$\|v + u t\|^2 = (r_1 + r_2)^2, \quad v = c_2^0 - c_1^0, \quad u = v_2 - v_1, \quad t \in [0, 1]$$                                                                                                                   |
| **Continuous Collision (Convex)**  | $$\Delta t = \alpha \frac{d(t)}{v_{\text{rel, along normal}}}, \quad \text{advance until } d(t) \le \varepsilon \text{ or } t > 1$$                                                                                   |
| **Epsilon Tolerance**              | $$\text{contact if } d \le \varepsilon, \quad \varepsilon \in [10^{-6}, 10^{-4}] \times \text{bbox diagonal}$$                                                                                                        |
|                                    |
