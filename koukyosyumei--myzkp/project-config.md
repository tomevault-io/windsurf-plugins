---
trigger: always_on
description: [Bootle, Jonathan, et al. "Gemini: Elastic SNARKs for diverse environments." Annual International Conference on the Theory and Applications of Cryptographic Techniques. Cham: Springer International Publishing, 2022.](https://eprint.iacr.org/2022/420.pdf)
---

# Gemini

[Bootle, Jonathan, et al. "Gemini: Elastic SNARKs for diverse environments." Annual International Conference on the Theory and Applications of Cryptographic Techniques. Cham: Springer International Publishing, 2022.](https://eprint.iacr.org/2022/420.pdf)

While many polynomial commitment schemes work for standard single-variable (univariate) polynomials, things get trickier when we deal with multilinear polynomials, which are polynomials with multiple variables where each variable has a maximum degree of one. The Gemini protocol offers an elegant way to handle these.

## Multilinear Polynomial

First, let's get a handle on the main object of interest. A multilinear polynomial with \\(m\\) variables (\\(f(X_0, X_1, \dots, X_{m- 1})\\)) can be written as a sum over all possible combinations of its variables. A general \\(m\\)-variate multilinear polynomial \\(f\\) is defined as:

\begin{align*}
    f(X_0, X_1, \dots, X_{m-1}) = \sum_{i = 0}^{2^m-1} c_i \prod_{j=0}^{m-1} X_j^{b_j(i)}
\end{align*}

Here, \\(c_i\\) is the \\(i\\)-th coefficient, and \\((b_{m-1}(i), \dots, b_0(i))\\) is the binary representation of the number \\(i\\). THis formula just means we sum up terms where each variable \\(X_{j}\\) is either present or absent.

For example, a three-variable (\\(m = 3\\)) multilinear polynomial looks like this:

\begin{align*}
    f(X_0, X_1, X_2) = c_{000} + c_{100} X_0 + c_{010} X_1 + c_{110} X_0 X_1 + c_{001} X_2 + c_{101} X_0 X_2 + c_{011} X_1 X_2 + c_{111} X_0 X_1 X_2
\end{align*}

The subscript of each coefficient \\(c\\) tells you which variables are in that term. For instance, \\(c_{110}\\) is the coefficient of the \\(X_0 X_1\\) term (reading the bits from right to left as \\(i_0\\),\\(i_1\\),\\(i_2\\)).

## Problem

Our goal is to create a commitment scheme for these polynomials. Specifically, a Prover wants to commit to a multilinear polynomial \\(f\\) and then convince a Verifier that for a specific, publicly known point \\(\rho = (\rho_0, \rho_1, \dots, \rho_{m-1})\\) and a public value \\(u\\), the following statement is true:

\begin{align*}
    f(\rho) = u
\end{align*}

The Prover must prove this without revealing the entire polynomial \\(f\\).

## Key idea 

Here's the core idea of Gemini: **transform the multivariable problem into a series of single-variable problems**, which are much easier to handle.

We first take the \\(2^{m}\\) coefficients of our multilinear polynomial \\(f\\) and use them to define a new **univariate** polynomial \\(g\\) of degree \\(2^{m} - 1\\). 

\begin{align*}
    g(X) = \sum_{i=0}^{2^{m} - 1} c_{i} X^{i}
\end{align*}

Using our three-variable example, the corresponding univariate polynomial \\(g(X)\\) would be:

\begin{align*}
g(X) = c_{000} + c_{100} X + c_{010} X^2 + c_{110} X^3 + c_{001} X^4 + c_{101} X^5 + c_{011} X^6 + c_{111} X^7
\end{align*}

### Split and Fold

Then, we are going to recursively fold this univariate polynomialan univariate polynomial with the **folding** trick, which repeatedly halves the degree by separating even and odd exponents:

For any univariate polynomial \\(g(X)\\), we can write:

\begin{align*}
    g(X) = g_E(X) + X \cdot g_O(X)
\end{align*}, where

\begin{align*}
    g_E(X) = \frac{g(X) + g(-X)}{2}, \quad \quad g_O(X) = \frac{g(X) - g(-X)}{2X}
\end{align*}

Here, \\(g_E\\) contains the even exponents of \\(g\\) and \\(g_O\\) contains the shifted odd exponents.

\begin{align*}
    g_E(X) = \sum_{i=0}^{\frac{m+1}{2} - 1} c_{2i} X^{2i}, \quad \quad g_O(X) = \sum_{i=0}^{\frac{m+1}{2} - 1} c_{2i + 1} X^{2i}
\end{align*}

Now treat \\(Y := X^2\\). Because the even and odd parts only use powers \\(X^{2k}\\), both \\(g_E\\) and \\(g_O\\) can be seen as polynomials in \\(Y\\) with half the degree.

We then define a sequence of folded polynomials as follows:

\begin{align*}
    g^{(0)}(X) &:= g(X) \\\\
    g^{(1)}(X) &:= g_E^{(0)}(X) + \rho_0 g_O^{(0)}(X) \\\\
    g^{(2)}(X^2) &:= g_E^{(1)}(X^2) + \rho_1 g_O^{(1)}(X^2) \\\\
        &\quad\vdots \\\\
    g^{(m)}(X^{2^{m-1}}) &:= g_E^{(m-1)}(X^{2^{m-1}}) + \rho_{m-1} g_O^{(m-1)}(X^{2^{m-1}}) = (\text{a constant}) \\\\
\end{align*}

Intuitively, each folding step substitutes the known scalar \\(\rho_{i-1}\\) for one of the original variables, and reduces the number of remaining variables by one. After \\(m\\) folds, we obtain a constant equal to \\(f(\rho)\\), i.e., \\(g^{(m)}(\cdot) = u\\).

```rust
pub fn split_and_fold<F: Field>(
    coef: &Vec<F>,
    rhos: &Vec<F>,
) -> Result<Vec<Polynomial<F>>, SplitFoldError> {
    let n = coef.len();
    if n.count_ones() != 1 {
        return Err(SplitFoldError::CoefsNotPowerOfTwo { found: n });
    }

    let log2_n = int_log2(n);
    if rhos.len() != log2_n as usize {
        return Err(SplitFoldError::PointsLenMismatch {
            expected: log2_n as usize,
            found: rhos.len(),
        });
    }

    let mut f = Polynomial::<F> { coef: coef.clone() };
    let mut fs = vec![f.clone()];
    for i in 1..(log2_n + 1) {
        let f_e = Polynomial::<F> {
            coef: f
                .coef
                .iter()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Koukyosyumei/MyZKP](https://github.com/Koukyosyumei/MyZKP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
