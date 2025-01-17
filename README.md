# RangeEnclosures.jl

[![Build Status](https://travis-ci.org/JuliaReach/RangeEnclosures.jl.svg?branch=master)](https://travis-ci.org/JuliaReach/RangeEnclosures.jl)
[![Docs latest](https://img.shields.io/badge/docs-latest-blue.svg)](http://juliareach.github.io/RangeEnclosures.jl/latest/)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/JuliaReach/RangeEnclosures.jl/blob/master/LICENSE.md)
[![Code coverage](http://codecov.io/github/JuliaReach/RangeEnclosures.jl/coverage.svg?branch=master)](https://codecov.io/github/JuliaReach/RangeEnclosures.jl?branch=master)
[![Join the chat at https://gitter.im/JuliaReach/Lobby](https://badges.gitter.im/JuliaReach/Lobby.svg)](https://gitter.im/JuliaReach/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

A [Julia](http://julialang.org) package to compute range enclosures of
real-valued functions.

## Resources

- [Manual](http://juliareach.github.io/RangeEnclosures.jl/latest/)
- [Contributing](https://juliareach.github.io/RangeEnclosures.jl/latest/about/#Contributing-1)
- [Release notes of tagged versions](https://github.com/JuliaReach/RangeEnclosures.jl/releases)
- [Release notes of the development version](https://github.com/JuliaReach/RangeEnclosures.jl/wiki/Release-log-tracker)

## Installing

This package requires Julia v1.0 or later.
Refer to the [official documentation](https://julialang.org/downloads) on how to
install and run Julia on your system.

Depending on your needs, choose an appropriate command from the following list
and enter it in Julia's REPL.
To activate the `pkg` mode, type `]` (and to leave it, type `<backspace>`).

#### [Install the latest release version](https://julialang.github.io/Pkg.jl/v1/managing-packages/#Adding-registered-packages-1)

```julia
pkg> add RangeEnclosures
```

#### Install the latest development version

```julia
pkg> add RangeEnclosures#master
```

#### [Clone the package for development](https://julialang.github.io/Pkg.jl/v1/managing-packages/#Developing-packages-1)

```julia
pkg> dev RangeEnclosures
```

## Quickstart

`RangeEnclosures` can be used to find bounds on the global minimum and maximum of a
real-valued function `f`, either univariate or multivariate, over a hyperrectangular
(i.e. box-shaped) domain.

This is a "meta" package in the sense it defines a unique `enclose` function that
calls one of the available solvers to do the optimization;
see the [documentation](http://juliareach.github.io/RangeEnclosures.jl/latest/)
or the examples below as an illustration of the available solvers.

The inputs to `enclose` are standard Julia functions and the domains are intervals
(resp. products of intervals): these are given as `Interval` (resp. `IntervalBox`)
types, both defined in `IntervalArithmetic`.

For instance, let `f(x) = -x^3/6` over the interval `[-4.5, -0.3]` whose the exact
lower and upper bounds are `[0.0045, 15.1875]`. We compute a range enclosure using
Taylor models substitution of order 4 like so:

```julia
julia> using RangeEnclosures

julia> f = x -> -x^3/6;

julia> dom = -4.5 .. -0.3
[-4.5, -0.299999]

julia> enclose(f, dom, :TaylorModels, order=4)
[-5.28751, 15.1876]
```

The same computation can be done using interval arithmetic substitution,
```julia
julia> enclose(f, dom, :IntervalArithmetic)
[0.00449999, 15.1875]

julia> enclose(f, dom) # use default solver
[0.00449999, 15.1875]
```
This result is actually tight; it can be confirmed using interval (global)
optimization:

```julia
julia> enclose(f, dom, :IntervalOptimisation)
[0.00449999, 15.1875]
```
or polynomial optimization:
```julia
julia> enclose(f, dom, :SumOfSquares)
Strange behavior : primal < dual :: line 144 in sdpa_solve.cpp
[0.00450017, 15.1876]
```
In the last calculation, the default semidefinite programming (SDP) solver `SDPA` is used,
but you can pass in any other SDP solver accepted by `JuMP`. For instance, if you
have `MosekTools` (and a Mosek license) installed, and want to use the maximum
degree of the relaxation to be `4`, do:

```julia
julia> using MosekTools

julia> enclose(f, dom, :SumOfSquares, order=4, backend=MosekTools.Mosek.Optimizer, QUIET=true)
[0.00450004, 15.1875]
```
(the optional `QUIET` argument is used to turn off the verbose mode).

Multivariate functions are handled similarly:

```julia
julia> g(x, y) = (x + 2y - 7)^2 + (2x + y - 5)^2;

julia> dom = IntervalBox(-10..10, 2)
[-10, 10] × [-10, 10]

julia> enclose(g, dom, :IntervalArithmetic)
[0, 2594]

julia> enclose(g, dom, :SumOfSquares, order=5, backend=MosekTools.Mosek.Optimizer, QUIET=true)
[9.30098e-07, 2594]
```

## Acknowledgements

If you use `RangeEnclosures.jl`, consider acknowledging or citing the Julia package
that implements the specific solver that you are using.

This package was completed during Aadesh Deshmuhk's [Julia Seasons of Contributions
(JSOC)](https://julialang.org/soc/ideas-page) 2019. 
In addition, we are grateful to the following persons for enlightening discussions
during the preparation of this package:

- [Luis Benet](https://github.com/lbenet)
- [Benoît Legat](https://github.com/blegat/)
- [David P. Sanders](https://github.com/dpsanders/)
