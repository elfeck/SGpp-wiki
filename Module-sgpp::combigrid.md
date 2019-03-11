Conbination technique functionality.

The combination technique is an older variant of sparse grids which only allows dimensional, but not spatial adaptivity. However, a central advantage is that global interpolation and quadrature schemes can be used, e.g. polynomial interpolation. This allows for high orders of convergence when using sufficiently smooth functions.
The functionality of the module includes
- an anistropic implementation of the Combination Technique. This means that different 1D-grids and 1D  - operators can be used for each dimension. This allows e.g. interpolating in one direction and integrating the interpolated function in another direction.
- 1D Grids : Uniform (with or without boundary), Clenshaw-Curtis (sgpp::combigrid::ClenshawCurtisDistribution), Chebyshev (sgpp::combigrid::ClenshawCurtisDistribution), Leja (sgpp::combigrid::LejaPointDistribution), weighted ![f0]-Leja sgpp::combigrid::L2LejaPointDistribution
- Operators: Interpolation (sgpp::combigrid::PolynomialInterpolationEvaluator, sgpp::combigrid::BSplineInterpolationEvaluator, sgpp::combigrid::LinearInterpolationEvaluator), quadrature (sgpp::combigrid::PolynomialQuadratureEvaluator, sgpp::combigrid::BSplineQuadratureEvaluator), tensors (sgpp::combigrid::InterpolationCoefficientEvaluator)
- regular and adaptive (also parallel) level-generation: various level managers (sgpp::combigrid::RegularLevelManager, sgpp::combigrid::AveragingLevelManager) are available that predict the surplus of each subspace
- support for directly working on full-grid subspaces: enables to solve PDEs, regression problems, density estimation on each full-grid separately
- serialization and deserialization of computed function values
- optimizations: function evaluations (but not the following computations) can be easily parallelized, interpolation at multiple points at once, operator coefficients and grid points are stored for recomputations at other evaluation points
- frequent use of abstract base classes such that extensions are easy to implement (e.g. new grids, operators and adaption schemes)
- various surrogate models for Uncertainty Quantification that are based on the Combination Technique are available:
- B-Spline Collocation
-Stochastic Collocation (global polynomial approximation with Legendre polynomials)
- Polynomial Chaos Expansion (global polynomial approximation with orthogonal polynomials with respect to univariate weight functions)
We will introduce some basic terminology and conventions here. For usage and more details on features, please refer to the [tutorials](https://github.com/SGpp/SGpp/wiki/Quick-Start).


The combination technique is an abstract scheme to combine one-dimensional numeric (linear) operations into a multi-dimensional operation. It provides a far better tradeoff between grid points and accuracy than working on a single regular/full grid ![f1]. For each dimension ![f2], the combination technique uses a sequence ![f3] of 1D-grids of increasing size. The index ![f4] is also called the level of the grid. Unlike many papers, the level ![f4] starts from zero in this implementation.
The combination technique evaluates a given function on multiple (rather small) regular/full grids ![f5]. Here, ![f4] is a multi-index which is also called level of the (multi-dimensional) grid. ![f6] is the set of multi-indices of levels that corresponds to the regular/full grids that are evaluated. It must satisfy ![f7]. The results of all evaluated grids are combined to obtain a more precise numeric result. When we speak of adding levels, we mean inserting a level multi-index into ![f6] and incorporating the result of the evaluation on the added regular/full grid into the numerical approximation.
Standard approximations indicate that level multi-indices with a higher sum of their components are potentially less important. For ![f8], this yields the set ![f9] of regular levels. Often, it is better to choose the set of levels adaptively. This module provides such adaption algorithms; they also work in parallel.
The combigrid module separates operators and grid points. For example, the quadrature operation computes its quadrature weights by integrating Lagrange polynomials on grid that you choose for it. If you choose a sequence of points for the first dimension, you also have to choose ![f10] for all ![f11], i.e. the number of grid points per level, which is also referred to as the growth of the (number of) grid points. It is generally advisable that the grid points are nested, i.e. ![f12], which may require a specific growth of the grid points. To simplify the decisions to be made, the combigrid module already provides suitable combinations of grid points, growths and operators.
The main functionality of the module is containted in
- interface classes that try to hide the the computation chain
- grid point and growth classes
- 1D-operator classes
- storage classes that store computed function values
- FullGrid*Evaluator, which takes the grid points, operators and a storage to compute a value on a regular/full grid ![f13]
- CombigridEvaluator, which uses FullGridTensorEvaluator to perform the numeric computation on different regular/full grids and combines the obtained values
- level manager classes that provide functionality to add a set of levels to the CombigridEvaluator, e.g. regular levels or adaptively generated levels.
To use the UQ methods of the combigrid module, Dakota is required as a dependency. Click [here]() for installation instructions.


[f0]: http://chart.apis.google.com/chart?cht=tx&chl=L%5E%7B2%7D
[f1]: http://chart.apis.google.com/chart?cht=tx&chl=X%5E%7B%281%29%7D%20%5Ctimes%20%5Cldots%20%5Ctimes%20X%5E%7B%28d%29%7D
[f2]: http://chart.apis.google.com/chart?cht=tx&chl=k%20%5Cin%20%5C%7B1%2C%20%5Cldots%2C%20d%5C%7D
[f3]: http://chart.apis.google.com/chart?cht=tx&chl=%28X%5E%7B%28i%29%7D_l%29_%7Bl%20%5Cin%20%5Cmathbb%7BN%7D_0%7D
[f4]: http://chart.apis.google.com/chart?cht=tx&chl=l
[f4]: http://chart.apis.google.com/chart?cht=tx&chl=l
[f5]: http://chart.apis.google.com/chart?cht=tx&chl=X_l%20%3A%3D%20X%5E%7B%281%29%7D_%7Bl_1%7D%20%5Ctimes%20%5Cldots%20%5Ctimes%20X%5E%7B%28d%29%7D_%7Bl_d%7D%2C%20l%20%5Cin%20I%20%5Csubseteq%20%5Cmathbb%7BN%7D_0%5Ed
[f6]: http://chart.apis.google.com/chart?cht=tx&chl=I
[f7]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cforall%20l%20%5Cin%20I%3A%20%5Cforall%20j%20%5Cin%20%5Cmathbb%7BN%7D_0%3A%20%28%28%5Cforall%20i%20%5Cin%20%5C%7B1%2C%20%5Cldots%2C%20d%5C%7D%3A%20j_i%20%5Cleq%20l_i%29%20%5CRightarrow%20j%20%5Cin%20I%29
[f8]: http://chart.apis.google.com/chart?cht=tx&chl=q%20%5Cin%20%5Cmathbb%7BN%7D_0
[f9]: http://chart.apis.google.com/chart?cht=tx&chl=I%20%3D%20%5C%7Bl%20%5Cin%20%5Cmathbb%7BN%7D_0%5Ed%20%5Cmid%20l_1%20%2B%20%5Cldots%20%2B%20l_d%20%5Cleq%20q%5C%7D
[f10]: http://chart.apis.google.com/chart?cht=tx&chl=%7CX%5E%7B%281%29%7D_l%7C
[f11]: http://chart.apis.google.com/chart?cht=tx&chl=l%20%5Cin%20%5Cmathbb%7BN%7D_0
[f12]: http://chart.apis.google.com/chart?cht=tx&chl=X%5E%7B%28i%29%7D_0%20%5Csubseteq%20X%5E%7B%28i%29%7D_1%20%5Csubseteq%20X%5E%7B%28i%29%7D_2%20%5Csubseteq%20%5Cldots
[f13]: http://chart.apis.google.com/chart?cht=tx&chl=X_l