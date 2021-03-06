# SDE Solvers

## Recommended Methods

For most Ito diagonal and scalar noise problems where a good amount of accuracy is
required and mild stiffness may be an issue, the `SRIW1` algorithm should
do well. If the problem has additive noise, then `SRA1` will be the
optimal algorithm. For commutative noise, `RKMilCommute` is a strong order 1.0
method which utilizes the commutivity property to greatly speed up the Wiktorsson
approximation and can choose between Ito and Stratonovich. For non-commutative noise,
difficult problems usually require adaptive time stepping in order to be efficient.
In this case, `LambaEM` and `LambaEulerHeun` are adaptive and handle general
non-diagonal problems (for Ito and Stratonovich interpretations respectively).
If adaptivity isn't necessary, the `EM` and `EulerHeun` are good choices
(for Ito and Stratonovich interpretations respectively).

For stiff problems with diagonal noise, `ImplicitRKMil` is the most efficient
method and can choose between Ito and Stratonovich. If the noise is non-diagonal,
`ImplicitEM` and `ImplicitEulerHeun` are for Ito and Stratonovich respectively.
For each of these methods, the parameter `theta` can be chosen. The default is
`theta=1/2` which will not dampen numerical oscillations and thus is symmetric
(and almost symplectic) and will lead to less error when noise is sufficiently
small. However, `theta=1/2` is not L-stable in the drift term, and thus one
can receive more stability (L-stability in the drift term) with `theta=1`, but
with a tradeoff of error efficiency in the low noise case. In addition, the
option `symplectic=true` will turns these methods into an implicit Midpoint
extension which is symplectic in distribution but has an accuracy tradeoff.

## Mass Matrices and Stochastic DAEs

The stiff methods can solve stochastic equations with mass matrices
(including stochastic DAEs written in mass matrix form) when either
`symplectic=true` or `theta=1`. These methods interpret the mass matrix
equation as:

```math
Mu' = f(t,u)dt + Mg(t,u)dW_t
```

i.e. with no mass matrix inversion applied to the `g` term. Thus these methods
apply noise per dependent variable instead of on the combinations of the
dependent variables and this is designed for phenomenological noise on the
dependent variables (like multiplicative or additive noise)

## Special Noise Forms

Some solvers are for specialized forms of noise. Diagonal noise is the default
setup. Non-diagonal noise is specified via setting `noise_rate_prototype` to
a matrix in the `SDEProblem` type. A special form of non-diagonal noise,
commutative noise, occurs when the noise satisfies the following condition:

```math
\sum_{i=1}^d g_{i,j_1}(t,x) \frac{\partial g_{k,j_2}(t,x)}{\partial x_i} = \sum_{i=1}^d g_{i,j_2}(t,x) \frac{\partial g_{k,j_1}(t,x)}{\partial x_i}
```

for every ``j_1,j_2`` and ``k``. Additive noise is when ``g(t,u)=g(t)``,
i.e. is independent of `u`. Multiplicative noise is ``g_i(t,u)=a_i u``.

## Special Keyword Arguments

* `save_noise`: Determines whether the values of `W` are saved whenever the timeseries
  is saved. Defaults to true.
* `delta`: The `delta` adaptivity parameter for the natural error estimator.
  Determines the balance between drift and diffusion error. For more details, see
  [the publication](http://chrisrackauckas.com/assets/Papers/ChrisRackauckas-AdaptiveSRK.pdf).

# Full List of Methods

## StochasticDiffEq.jl

Each of the StochasticDiffEq.jl solvers come with a linear interpolation.
Orders are given in terms of strong order.

### Nonstiff Methods

- `EM`- The Euler-Maruyama method. Strong Order 0.5 in the Ito sense. Can handle
  all forms of noise, including non-diagonal, scalar, and colored noise. Fixed
  time step only.†
- `LambaEM`- A modified Euler-Maruyama method with adaptive time stepping with
  an error estimator based on Lamba and Rackauckas. Strong Order 0.5 in the Ito
  sense. Can handle all forms of noise, including non-diagonal, scalar, and
  colored noise.†
- `EulerHeun` - The Euler-Heun method. Strong Order 0.5 in the Stratonovich sense.
  Can handle all forms of noise, including non-diagonal, scalar, and colored noise.
  Fixed time step only.†
- `LambaEulerHeun` - A modified Euler-Heun method with adaptive time stepping
  with an error estimator based on Lamba due to Rackauckas. Strong Order 0.5 in
  the Stratonovich sense. Can handle all forms of noise, including non-diagonal,
  scalar, and colored noise.†
- `RKMil` - An explicit Runge-Kutta discretization of the strong Order 1.0
  Milstein method. Defaults to solving the Ito problem, but
  `RKMil(interpretation=:Stratonovich)` makes it solve the Stratonovich problem.
  Only handles scalar and diagonal noise. Uses a 1.5/2.0 error estimate for
  adaptive time stepping.†
- `RKMilCommute` - An explicit Runge-Kutta discretization of the strong Order 1.0
  Milstein method for commutative noise problems. Defaults to solving the Ito
  problem, but `RKMilCommute(interpretation=:Stratonovich)` makes it solve the
  Stratonovich problem. Uses a 1.5/2.0 error estimate for adaptive time stepping.†
- `SRA` - Adaptive strong Order 1.5 methods for additive Ito and Stratonovich SDEs.
  Default tableau is for SRA1. Can handle non-diagonal and scalar additive noise.
- `SRI` - Adaptive strong Order 1.5 methods for diagonal/scalar Ito SDEs.
  Default tableau is for SRIW1.
- `SRIW1` - Adaptive optimized version of SRIW1. Strong Order 1.5 for diagonal/scalar
  Ito SDEs.†
- `SRA1` - Adaptive optimized version of SRA1. Strong Order 1.5 for additive Ito and
  Stratonovich SDEs. Can handle non-diagonal and scalar additive noise.†

Example usage:

```julia
sol = solve(prob,SRIW1())
```

### Tableau Controls

For `SRA` and `SRI`, the following option is allowed:

* `tableau`: The tableau for an `:SRA` or `:SRI` algorithm. Defaults to SRIW1 or SRA1.

### Stiff Methods

- `ImplicitEM` - An order 0.5 Ito implicit method. This is a theta method which
  defaults to `theta=1/2` or the Trapezoid method on the drift term. This method
  defaults to `symplectic=false`, but when true and `theta=1/2` this is the
  implicit Midpoint method on the drift term and is symplectic in distribution.
  Can handle all forms of noise, including non-diagonal, scalar, and colored noise.
  Uses a 1.0/1.5 heuristic for adaptive time stepping.
- `ImplicitEulerHeun` - An order 0.5 Stratonovich implicit method. This is a
  theta method which defaults to `theta=1/2` or the Trapezoid method on the
  drift term. This method defaults to `symplectic=false`, but when true and
  `theta=1/2` this is the implicit Midpoint method on the drift term and is
  symplectic in distribution. Can handle all forms of noise, including
  non-diagonal, scalar, and colored noise. Uses a 1.0/1.5 heuristic for
  adaptive time stepping.
- `ImplicitRKMil` - An order 1.0 implicit method. This is a theta method which
  defaults to `theta=1/2` or the Trapezoid method on the drift term. Defaults
  to solving the Ito problem, but `ImplicitRKMil(interpretation=:Stratonovich)`
  makes it solve the Stratonovich problem. This method defaults to
  `symplectic=false`, but when true and `theta=1/2` this is the
  implicit Midpoint method on the drift term and is symplectic in distribution.
  Handles diagonal and scalar noise. Uses a 1.5/2.0 heuristic for adaptive
  time stepping.
- `ISSEM` - An order 0.5 split-step Ito implicit method. It is fully implicit,
  meaning it can handle stiffness in the noise term. This is a theta method which
  defaults to `theta=1` or the Trapezoid method on the drift term. This method
  defaults to `symplectic=false`, but when true and `theta=1/2` this is the
  implicit Midpoint method on the drift term and is symplectic in distribution.
  Can handle all forms of noise, including non-diagonal, scalar, and colored noise.
  Uses a 1.0/1.5 heuristic for adaptive time stepping.
- `ISSEulerHeun` - An order 0.5 split-step Stratonovich implicit method. It is
  fully implicit, meaning it can handle stiffness in the noise term. This is a
  theta method which defaults to `theta=1` or the Trapezoid method on the drift
  term. This method defaults to `symplectic=false`, but when true and `theta=1/2`
  this is the implicit Midpoint method on the drift term and is symplectic in
  distribution. Can handle all forms of noise, including non-diagonal,Q scalar,
  and colored noise. Uses a 1.0/1.5 heuristic for adaptive time stepping.

### Derivative-Based Methods

The following methods require analytic derivatives of the diffusion term.

- `PCEuler` - The predictor corrector euler method. Strong Order 0.5 in the Ito
  sense. Requires the ggprime function, which is defined as
  ```math
    \text{ggprime}^k(t,x) = \sum_{j=1}^m \sum_{i=1}^d g_{i,j}(t,x) \frac{\partial g_{k,j}(t,x)}{\partial x_i}.
  ```
  This can also be understood more intuitively in vector/matrix form as,
  ```math
  \text{ggprime}(t,x) = \sum_{j=1}^m \bar{\mathcal{J}}[\vec g^{(j)}(t,x)] \vec g^{(j)}(t,x).
  ```
  where ``\vec g^{(j)}`` is the noise vector for the j'th noise channel and ``\bar{\mathcal{J}}`` is the Jacobian of the j'th   noise vector.

  The default settings for the drift implicitness is `theta=0.5` and
  the diffusion implicitness is `eta=0.5`.  

## StochasticCompositeAlgorithm

One unique feature of StochasticDiffEq.jl is the `StochasticCompositeAlgorithm`, which allows
you to, with very minimal overhead, design a multimethod which switches between
chosen algorithms as needed. The syntax is `StochasticCompositeAlgorithm(algtup,choice_function)`
where `algtup` is a tuple of StochasticDiffEq.jl algorithms, and `choice_function`
is a function which declares which method to use in the following step. For example,
we can design a multimethod which uses `EM()` but switches to `RKMil()` whenever
`dt` is too small:

```julia
choice_function(integrator) = (Int(integrator.dt<0.001) + 1)
alg_switch = StochasticCompositeAlgorithm((EM(),RKMil()),choice_function)
```

The `choice_function` takes in an `integrator` and thus all of the features
available in the [Integrator Interface](@ref)
can be used in the choice function.

## BridgeDiffEq.jl

Bridge.jl is a set of fixed timestep algorithms written in Julia. These methods
are made and optimized for out-of-place functions on immutable (static vector)
types. Note that this setup is not automatically included with
DifferentialEquaitons.jl. To use the following algorithms, you must install and
use BridgeDiffEq.jl:

```julia
Pkg.clone("https://github.com/JuliaDiffEq/BridgeDiffEq.jl")
using BridgeDiffEq
```

- `BridgeEuler` - Strong order 0.5 Euler-Maruyama method for Ito equations.†
- `BridgeHeun` - Strong order 0.5 Euler-Heun method for Stratonovich equations.†
- `BridgeSRK` - Strong order 1.0 derivative-free stochastic Runge-Kutta method
  for scalar (`<:Number`) Ito equations.†

#### Notes

†: Does not step to the interval endpoint. This can cause issues with discontinuity
detection, and [discrete variables need to be updated appropriately](../features/diffeq_arrays.html).
