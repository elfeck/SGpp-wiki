Optimization of smooth sparse grid interpolants.

This SG++ module was developed as part of a Master's thesis with the translated title
"Hierarchical Optimization with Gradient-Based Methods on Sparse Grid Functions".
In the thesis, a new approach for minimizing objective functions
![f0] is developed.
It can be summarized in three steps:
1. Adaptive iterative sparse grid generation. Two methods are implemented for generating
adaptively a sparse grid according to the function values at the grid points.
2. B-spline or Mexican-Hat hierarchization. The objective function is interpolated by a
linear combination of sufficiently smooth sparse grid basis functions.
3. Gradient-based optimization of the smooth interpolant. Various optimization methods,
gradient-based as well gradient-free ones, are implemented for optimizating the sparse grid
surrogate.


[f0]: http://chart.apis.google.com/chart?cht=tx&chl=f%5Ccolon%20%5B0%2C%201%5D%5Ed%20%5Cto%20%5Cmathbb%7BR%7D