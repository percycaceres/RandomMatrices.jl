RandomMatrices.jl
=================

Random matrix package for [Julia](http://julialang.org).

This extends the [Distributions](https://github.com/JuliaStats/Distributions.jl)
package to provide methods for working with matrix-valued random variables,
a.k.a. random matrices. State of the art methods for computing random matrix
samples and their associated distributions are provided.

The names of the various ensembles can vary widely across disciplines. Where possible,
synonyms are listed.

[![Build Status](https://travis-ci.org/jiahao/RandomMatrices.jl.png?branch=master)](https://travis-ci.org/jiahao/RandomMatrices.jl)
[![RandomMatrices](http://pkg.julialang.org/badges/RandomMatrices_release.svg)](http://pkg.julialang.org/?pkg=RandomMatrices&ver=release)
[![RandomMatrices](http://pkg.julialang.org/badges/RandomMatrices_nightly.svg)](http://pkg.julialang.org/?pkg=RandomMatrices&ver=nightly)
[![Coverage Status](https://img.shields.io/coveralls/jiahao/RandomMatrices.jl.svg)](https://img.shields.io/coveralls/jiahao/RandomMatrices.jl.svg)

Additional functionality is provided when these optional packages are installed:
- Symbolic manipulation of Haar matrices with [GSL.jl](https://github.com/jiahao/GSL.jl)
- Invariant ensembles with [ApproxFun.jl](https://github.com/dlfivefifty/ApproxFun.jl)

# Gaussian matrix ensembles

Much of classical random matrix theory has focused on matrices with matrix elements comprised of
independently and identically distributed (iid) real, complex or quaternionic Gaussians.
(Traditionally, these are associated with a parameter `beta` tracking the number of independent
real random variables per matrix element, i.e. `beta=1,2,4` respectively. This is also referred
to as the Dyson 3-fold way.)
Methods are provided for calculating random variates (samples) and various properties of these
random matrices.

The hierarchy of dense matrices provided are

- Ginibre ensemble - all matrix elements are iid with no global symmetry
- Hermite ensemble - one global symmetry
  - Gaussian orthogonal ensemble (GOE, `beta=1`) - real and symmetric
  - Gaussian unitary ensemble (GUE, `beta=2`) - complex and Hermitian
  - Gaussian symplectic ensemble (GSE, `beta=4`) - quaternionic and self-dual
- Circular ensemble - uniformly distributed with `|det|=1`
  - Circular orthogonal ensemble (GOE, `beta=1`)
  - Circular unitary ensemble (GUE, `beta=2`)
  - Circular symplectic ensemble (GSE, `beta=4`)
- Laguerre matrices = white Wishart matrices
- Jacobi matrices = MANOVA matrices
- Unitary invariant ensembles

Unless otherwise specified, `beta=1,2,4` are supported. For the symplectic matrices `beta=4`,
the 2x2 outer block-diagonal complex representation `USp(2N)` is used.

## Joint probability density functions (jpdfs)

Given eigenvalues `lambda` and the `beta` parameter of the random matrix distribution:

- `VandermondeDeterminant(lambda, beta)` computes the Vandermonde determinant
- `HermiteJPDF(lambda, beta)` computes the jpdf for the Hermite ensemble
- `LaguerreJPDF(lambda, n, beta)` computes the jpdf for the Laguerre(n) ensemble
- `JacobiJPDF(lambda, n1, n2, beta)` computes the jpdf for the Jacobi(n1, n2) ensemble

## Matrix samples

Constructs samples of random matrices corresponding to the classical Gaussian
Hermite, Laguerre(m) and Jacobi(m1, m2) ensembles.

- `GaussianHermiteMatrix(n, beta)`, `GaussianLaguerreMatrix(n, m, beta)`,
   `GaussianJacobiMatrix(n, m1, m2, beta)`
   each construct a sample dense `n`x`n` matrix for the corresponding matrix
   ensemble with `beta=1,2,4`

- `GaussianHermiteTridiagonalMatrix(n, beta)`,
  `GaussianLaguerreTridiagonalMatrix(n, m, beta)`,
  `GaussianJacobiSparseMatrix(n, m1, m2, beta)`
  each construct a sparse `n`x`n` matrix for the corresponding matrix ensemble
  for arbitrary positive finite `beta`. 
  `GaussianHermiteTridiagonalMatrix(n, Inf)` is also allowed.
  These sampled matrices have the same eigenvalues as above but are much faster
  to diagonalize oweing to their sparsity. They also extend Dyson's threefold
  way to arbitrary `beta`.
- `GaussianHermiteSamples(n, beta)`,
  `GaussianLaguerreSamples(n, m, beta)`,
  `GaussianJacobiSamples(n, m1, m2, beta)`
  return a set of `n` eigenvalues from the sparse random matrix samples

- `HaarMatrix(n, beta)`
  Generates a random matrix from the `beta`-circular ensemble.
 - `HaarMatrix(n, beta, correction)` provides fine-grained control of what kind of correction
   is applied to the raw QR decomposition. By default, `correction=1` (Edelman's correction) is
   used. Other valid values are `0` (no correction) and `2` (Mezzadri's correction).
 - `NeedsPiecewiseCorrection()` implements a simple test to see if a correction is necessary.
 
- `InvariantEnsemble(str,n)`
   Generates a unitary invariant ensemble, where str determines the
   potential of the ensemble, see below.
   Only available if ApproxFun package is installed.

The parameters `m`, `m1`, `m2` refer to the number to independent "data" degrees of freedom.
For the dense samples these must be `Integer`s but can be `Real`s for the rest.

# Formal power series

Allows for manipulations of formal power series (fps) and formal Laurent series
(fLs), which come in handy for the computation of free cumulants.

## Types
- `FormalPowerSeries`: power series with coefficients allowed only for
  non-negative integer powers
- `FormalLaurentSeries`: power series with coefficients allowed for all
  integer powers

## FormalPowerSeries methods

### Elementary operations
- basic arithmetic operations `==`, `+`, `-`, `^`
- `*` computes the Cauchy product (discrete convolution)
- `.*` computes the Hadamard product (elementwise multiplication)
- `compose(P,Q)` computes the series composition P.Q
- `derivative` computes the series derivative
- `reciprocal` computes the series reciprocal

### Utility methods
- `trim(P)` removes extraneous zeroes in the internal representation of `P`
- `isalmostunit(P)` determines if `P` is an almost unit series
- `isconstant(P)` determines if `P` is a constant series
- `isnonunit(P)` determines if `P` is a non-unit series
- `isunit(P)` determines if `P` is a unit series
- `MatrixForm(P)` returns a matrix representation of `P` as an upper triangular
  Toeplitz matrix
- `tovector` returns the series coefficients

# Densities

Famous distributions in random matrix theory

- `Semicircle` provides the semicircle distribution
- `TracyWidom` computes the Tracy-Widom density distribution
  by brute-force integration of the Painlevé II equation

# Utility functions

- `hist_eig` computes the histogram of eigenvalues of a matrix using the
  method of Sturm sequences.
  This is recommended for `SymTridiagonal` matrices as it is significantly
  faster than `hist(eigvals())`
  This is also implemented for dense matrices, but it is pretty slow and
  not really practical.

# Stochastic processes

Provides finite-dimensional matrix representations of stochastic operators.

In the following, `dt` is the time interval being discretized over and `t_end` is the final time.

- `BrownianProcess(dt, t_end)` generates a vector corresponding to a Brownian random walk starting
   from time `t=0` and position `x=0` 
- `WhiteNoiseProcess(dt, t_end)` generates a vector corresponding to white noise.
- `StochasticAiryProcess(dt, t_end, beta)` generates the largest eigenvalue corresponding to the
   stochastic Airy process with real positive `beta`. This is known to be distributed in the `t_end -> Inf`
   limit to the `beta`-Tracy-Widom law.
   
# Invariant ensembles

`InvariantEnsemble(str,n)` supports n x n unitary invariant ensemble 
 with distribution

`exp(- Tr Q(M)) dM`

 `str` specifies an ensemble with precomputed recurrence coefficients.
  The currently include ensembles are

 - Quartic:            		Q(M) = n M^4
 - Eight:              		Q(M) = n M^8
 - HODecay:            		Q(M) = n (M^4/20 - 4/15M^3 +M^2/5 + 8/5M)
 - CoshUnscaled:       		Q(M) = cosh(M)
 -  QuarticUnscaled:    		Q(M) = M^4
 -  EightUnscaled:      		Q(M) = M^8

# References
- James Albrecht, Cy Chan, and Alan Edelman,
    "Sturm Sequences and Random Eigenvalue Distributions",
    *Foundations of Computational Mathematics*,
    vol. 9 iss. 4 (2009), pp 461-483.
  [[pdf]](www-math.mit.edu/~edelman/homepage/papers/sturm.pdf)
  [[doi]](http://dx.doi.org/10.1007/s10208-008-9037-x)

- Ioana Dumitriu and Alan Edelman,
    "Matrix Models for Beta Ensembles",
    *Journal of Mathematical Physics*,
    vol. 43 no. 11 (2002), pp. 5830-5547
  [[doi]](http://dx.doi.org/doi: 10.1063/1.1507823)
  [arXiv:math-ph/0206043](http://arxiv.org/abs/math-ph/0206043)

- Alan Edelman, Per-Olof Persson and Brian D Sutton,
    "The fourfold way",
    *Journal of Mathematical Physics*,
    submitted (2013).
  [[pdf]](http://www-math.mit.edu/~edelman/homepage/papers/ffw.pdf)

- Alan Edelman and Brian D. Sutton,
    "The beta-Jacobi matrix model, the CS decomposition,
     and generalized singular value problems",
    *Foundations of Computational Mathematics*,
    vol. 8 iss. 2 (2008), pp 259-285.
  [[pdf]](http://www-math.mit.edu/~edelman/homepage/papers/betajacobi.pdf)
  [[doi]](http://dx.doi.org/10.1007/s10208-006-0215-9)

- Peter Henrici,
    *Applied and Computational Complex Analysis,
    Volume I: Power Series---Integration---Conformal Mapping---Location of Zeros*,
    Wiley-Interscience: New York, 1974
  [[worldcat]](http://www.worldcat.org/oclc/746035)

- Frank Mezzadri,
    "How to generate random matrices from the classical compact groups",
    Notices of the AMS, vol. 54 (2007), pp592-604
  [[arXiv]](http://arxiv.org/abs/math-ph/0609050)
