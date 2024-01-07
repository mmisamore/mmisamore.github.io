---
title: Perfectly Correlated Bernoulii Random Variables 
date: 2024-01-06 
tags: 
  - probability 
---

Here is a fun little exercise: show that any two perfectly correlated Bernoulii random variables \\(X\\) and \\(Y\\) must be
equal. 

Proof: The assumption can be stated as
$$\frac{Cov(X, Y)}{\sigma_X \cdot \sigma_Y} = 1$$ 

Let \\(Z_X = (X - \mu_X) / \sigma_X\\), \\(Z_Y = (Y - \mu_Y) / \sigma_Y\\) be the standardizations so that
\\(\mathbb{E}(Z_X) = 0\\), \\(Var(Z_X) = 1\\), and similarly for \\(Z_Y\\). Then we have 
$$Cov(Z_X, Z_Y) = 1.$$

Now observe that
$$Var(Z_X - Z_Y) = Var(Z_X) + Var(Z_Y) - 2 \cdot Cov(Z_X, Z_Y)
= 2 - 2 \cdot 1 = 0$$

so that \\(Z_X - Z_Y = 0\\) a.e. and therefore \\(Z_X = Z_Y\\) a.e. But then \\(Y = aX + b\\) for some constants \\(a\\)
and \\(b\\), so either \\(Y = X\\) or \\(Y = 1 - X\\) since \\(X\\) and \\(Y\\) are Bernoulli. In the second case we have 

$$Cov(X, 1 - X) = Cov(X, 1) + Cov(X, -X) = -Var(X)$$

and \\(\sigma_X \cdot \sigma_{1 - X} = \mu_X \cdot (1 - \mu_X) = Var(X)\\) since \\(X\\) is Bernoulli so the correlation
in that case is \\(-1\\). Therefore \\(X = Y\\) as was to be shown. \\(\square\\)
