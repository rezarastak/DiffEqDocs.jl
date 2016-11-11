# Convergence Simulations

The convergence simulation type is useful for deriving order of convergence estimates
from a group of simulations. This object will automatically assemble error vectors
into a more useful manner and provide plotting functionality. Convergence estimates
are also given by pair-wise estimates.

One can automatically have DifferentialEquations.jl perform the error analysis by
passing a `ConvergenceSimulation` a vector of solutions, or using one of the provided
`test_convergence` functions. These will give order of convergence estimates and
provide plotting functionality. This requires that the true solution was provided
in the problem definition.

`ConvergenceSimulation`s can either be created by passing the constructor the
appropriate solution array or by using one of the provided `test_convergence` functions.

## The ConvergenceSimulation Type

A type which holds the data from a convergence simulation.

### Fields

* `solutions::Array{<:DESolution}`: Holds all the PdeSolutions.
* `errors`: Dictionary of the error calculations. Can contain:

    - `h1Errors`: Vector of the H1 errors.
    - `l2Errors`: Vector of the L2 errors.
    - `maxErrors`: Vector of the nodal maximum errors.
    - `node2Errors`: Vector of the nodal l2 errors.

* `N`: The number of simulations.
* `auxdata`: Auxillary data of the convergence simluation. Entries can include:

    - `dts`: The dt's in the simulations.
    - `dxs`: The dx's in the simulations.
    - `μs`: The CFL μ's in the simulations.
    - `νs`: The CFL ν's in the simulations.

* `𝒪est`: Dictionary of order estimates. Can contain:

    - `ConvEst_h1`: The H1 error order of convergence estimate for the convergence simulation.
       Generated via `log2(error[i+1]/error[i])`. Thus only valid if generated by halving/doubling
       the dt/dx. If alternate scaling, modify by dividing of log(base,ConvEst_h1)
    - `ConvEst_l2`: The L2 error order of convergence estimate for the convergence simulation.
       Generated via `log2(error[i+1]/error[i])`. Thus only valid if generated by halving/doubling
       the dt/dx. If alternate scaling, modify by dividing of log(base,ConvEst_l2)
    - `ConvEst_max`: The nodal maximum error order of convergence estimate for the convergence simulation.
       Generated via `log2(error[i+1]/error[i])`. Thus only valid if generated by halving/doubling
       the dt/dx. If alternate scaling, modify by dividing of log(base,ConvEst_max)
    - `ConvEst_node2`: The nodal l2 error order of convergence estimate for the convergence simulation.
       Generated via `log2(error[i+1]/error[i])`. Thus only valid if generated by halving/doubling
       the dt/dx. If alternate scaling, modify by dividing of log(base,ConvEst_node2)

* `convergence_axis`: The axis along which convergence is calculated. For example, if
   we calculate the dt convergence, convergence_axis is the dts used in the calculation.

## Plot Functions

The plot functionality is provided by a Plots.jl recipe. What is plotted is a
line series for each calculated error along the convergence axis. To plot a
convergence simulation, simply use:

```julia
plot(sim::ConvergenceSimulation)
```

All of the functionality (keyword arguments) provided by Plots.jl are able to
be used in this command. Please see the Plots.jl documentation for more information.

## ODE

`test_convergence(dts::AbstractArray,prob::AbstractODEProblem)`

Tests the order of the time convergence of the given algorithm on the given problem
solved over the given dts.

### Keyword Arguments

* `T`: The final time. Default is 1
* `save_timeseries`: Denotes whether to save at every timeseries_steps steps. Default is true.
* `timeseries_steps`: Denotes the steps to save at if `save_timeseries=true`. Default is 1
* `alg`: The algorithm to test.
* `tableau`: The tableau used for generic methods. Defaults to ODE_DEFAULT_TABLEAU.

## SDE

`test_convergence(dts::AbstractArray,prob::AbstractSDEProblem)`

Tests the strong order time convergence of the given algorithm on the given problem
solved over the given dts.

### Keyword Arguments

* `T`: The final time. Default is 1
* `numMonte`: The number of simulations for each dt. Default is 10000.
* `save_timeseries`: Denotes whether to save at every timeseries_steps steps. Default is true.
* `timeseries_steps`: Denotes the steps to save at if `save_timeseries=true`. Default is 1
* `alg`: The algorithm to test. Defaults to "EM".

## Poisson

`test_convergence(dxs::AbstractArray,prob::PoissonProblem)`

Tests the convergence of the solver algorithm on the given Poisson problem with
dxs as given. Uses the square mesh [0,1]x[0,1].

### Keyword Arguments

* `solver`: Which solver to use. Default is "Direct".

## Heat

test_convergence(dts::AbstractArray,dxs::AbstractArray,prob::AbstractHeatProblem,convergence_axis)

Tests the convergence of the solver algorithm on the given Heat problem with
the dts and dxs as given. Uses the square mesh [0,1]x[0,1]. The convergence
axis is the axis along which convergence is calculated. For example, when testing
dt convergence, `convergence_axis = dts`.

### Keyword Arguments

* `T`: The final time. Defaults to 1
* `alg`: The algorithm to test. Default is "Euler".

## Utilities

### Order Estimation

`calc𝒪estimates(error::Vector{Number})``

Computes the pairwise convergence estimate for a convergence test done by
halving/doubling stepsizes via

log2(error[i+1]/error[i])

Returns the mean of the convergence estimates.