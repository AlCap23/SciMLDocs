![SurrogatesLogo](images/Surrogates.png)
# Overview
A surrogate model is an approximation method that mimics the behavior of a computationally
expensive simulation. In more mathematical terms: suppose we are attempting to optimize a function
``\; f(p)``, but each calculation of ``\; f`` is very expensive. It may be the case that we need to solve a PDE for each point or use advanced numerical linear algebra machinery, which is usually costly. The idea is then to develop a surrogate model ``\; g`` which approximates ``\; f`` by training on previous data collected from evaluations of ``\; f``.
The construction of a surrogate model can be seen as a three-step process:

1. Sample selection
2. Construction of the surrogate model
3. Surrogate optimization

The sampling methods are super important for the behavior of the Surrogate.
At the moment they are:

- Grid sample
- Uniform sample
- Sobol sample
- Latin Hypercube sample
- Low discrepancy sample

The available surrogates are:

- Linear
- Radial Basis
- Kriging
- Custom Kriging provided with Stheno
- Neural Network
- Support Vector Machine
- Random Forest
- Second Order Polynomial
- Inverse Distance

After the surrogate is built, we need to optimize it with respect to some objective function.
That is, simultaneously looking for a minimum **and** sampling the most unknown region.
The available optimization methods are:

- Stochastic RBF (SRBF)
- Lower confidence bound strategy (LCBS)
- Expected improvement (EI)
- Dynamic coordinate search (DYCORS)

## Multi-output Surrogates

In certain situations, the function being modeled may have a multi-dimensional output space.
In such a case, the surrogate models can take advantage of correlations between the
observed output variables to obtain more accurate predictions.

When constructing the original surrogate, each element of the passed `y` vector should
itself be a vector. For example, the following `y` are all valid.

```
using Surrogates
using StaticArrays

x = sample(5, [0.0; 0.0], [1.0; 1.0], SobolSample())
f_static = (x) -> StaticVector(x[1], log(x[2]*x[1]))
f = (x) -> [x, log(x)/2]

y = f_static.(x)
y = f.(x)
```

Currently, the following are implemented as multi-output surrogates:

- Radial Basis
- Neural Network (via Flux)
- Second Order Polynomial
- Inverse Distance
- Custom Kriging (via Stheno)

## Gradients

The surrogates implemented here are all automatically differentiable via Zygote. Because
of this property, surrogates are useful models for processes which aren't explicitly
differentiable, and can be used as layers in, for instance, Flux models.

# Installation
Surrogates is registered in the Julia General Registry. In the REPL:
```
]add Surrogates
```

You can obtain the current master with:
```
]add https://github.com/JuliaDiffEq/Surrogates.jl#master
```

# Quick example
```@example
using Surrogates
num_samples = 10
lb = 0.0
ub = 10.0

#Sampling
x = sample(num_samples,lb,ub,SobolSample())
f = x-> log(x)*x^2+x^3
y = f.(x)

#Creating surrogate
alpha = 2.0
n = 6
my_lobachevsky = LobachevskySurrogate(x,y,lb,ub,alpha=alpha,n=n)

#Approximating value at 5.0
value = my_lobachevsky(5.0)

#Adding more data points
surrogate_optimize(f,SRBF(),lb,ub,my_lobachevsky,UniformSample())

#New approximation
value = my_lobachevsky(5.0)
```
