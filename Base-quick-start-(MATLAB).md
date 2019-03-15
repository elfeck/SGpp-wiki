

To be able to quickly start with a toolkit, it is often advantageous
(not only for the impatient users), to look at some code examples first.
In this tutorial, we give a short example program which interpolates a
bivariate function on a regular sparse grid.
Identical versions of the example are given in all languages
currently supported by SG++: C++, Python, Java, and MATLAB.

In the example, we create a two-dimensional regular sparse grid of level 3
(with grid points ![f0])
using piecewise bilinear basis functions
![f1].
We then interpolate the function

![f2]

with

![f3]

by calculating the coefficients ![f4] such that
![f5] for all ![f6].
This process is called <i>hierarchization</i> in sparse grid contexts;
the ![f4] are called <i>(hierarchical) surpluses</i>.
Note that ![f7] vanishes at the boundary of the domain ![f8];
therefore, we don't have to spend sparse grid points on the boundary.
Finally, we evaluate the sparse grid function ![f9] at a point
![f10].

Before starting, the function ![f7], which we want to interpolate, is defined.

```matlab
f = @(x0, x1) (16.0 * (x0 - 1.0) * x0 * (x1 - 1.0) * x1);
```

First, we create a two-dimensional grid (type `sgpp::base::Grid)`
with piecewise bilinear basis functions with the help of the factory method
`sgpp::base::Grid.createLinearGrid()`.

```matlab
dim = 2;
grid = sgpp.Grid.createLinearGrid(dim);
```

Then we obtain a reference to the grid's
`sgpp::base::GridStorage` object which allows us, e.g., to access grid
points, to obtain the dimensionality (which we print) and the
number of grid points.

```matlab
gridStorage = grid.getStorage();
fprintf('dimensionality:         %u\n', gridStorage.getDimension());
```

Now, we use a `sgpp::base::GridGenerator` to
create a regular sparse grid of level 3.
Thus, `gridStorage.getSize()` returns 17, the number of grid points
of a two-dimensional regular sparse grid of level 3.

```matlab
level = 3;
grid.getGenerator().regular(level);
fprintf('number of grid points:  %u\n', gridStorage.getSize());
```

We create an object of type `sgpp::base::DataVector`
which is essentially a wrapper around a `double` array.
The `DataVector` is initialized with as many
entries as there are grid points. It serves as a coefficient vector for the
sparse grid interpolant we want to construct. As the entries of a
freshly created `DataVector` are not initialized, we set them to
0.0. (This is superfluous here as we initialize them in the
next few lines anyway.)

```matlab
alpha = sgpp.DataVector(gridStorage.getSize());
alpha.setAll(0);
fprintf('length of alpha vector: %u\n', alpha.getSize());
```

The `for` loop iterates over all grid points: For each grid
point `gp`, the corresponding coefficient ![f4] is set to the
function value at the grid point's coordinates which are obtained by
`getStandardCoordinate(dim)`.
The current coefficient vector is then printed.

```matlab
for i = 0:gridStorage.getSize()-1
    gp = gridStorage.getPoint(i);
    alpha.set(i, f(gp.getStandardCoordinate(0), gp.getStandardCoordinate(1)));
end

fprintf('alpha before hierarchization: %s\n', char(alpha.toString()));
```

An object of `sgpp::base::OperationHierarchisation` is created and used to
hierarchize the coefficient vector, which we print.

```matlab
sgpp.createOperationHierarchisation(grid).doHierarchisation(alpha);
fprintf('alpha after hierarchization:  %s\n', char(alpha.toString()));
```

Finally, a second `DataVector` is created which is used as a point to
evaluate the sparse grid function at. An object is obtained which
provides an evaluation operation (of type `sgpp::base::OperationEvaluation)`,
and the sparse grid interpolant is evaluated at ![f11],
which is close to (but not exactly at) a grid point.

```matlab
p = sgpp.DataVector(dim);
p.set(0, 0.52);
p.set(1, 0.73);
opEval = sgpp.createOperationEval(grid);
fprintf('u(0.52, 0.73) = %.4f\n', opEval.eval(alpha,p));
```

The example results in the following output:
```
dimensionality:         2
number of grid points:  17
length of alpha vector: 17
alpha before hierarchization: [1, 0.75, 0.75, 0.4375, 0.9375, 0.9375, 0.4375, 0.75, 0.75, 0.4375, 0.9375, 0.9375, 0.4375, 0.5625, 0.5625, 0.5625, 0.5625]
alpha after hierarchization:  [1, 0.25, 0.25, 0.0625, 0.0625, 0.0625, 0.0625, 0.25, 0.25, 0.0625, 0.0625, 0.0625, 0.0625, 0.0625, 0.0625, 0.0625, 0.0625]
u(0.52, 0.73) = 0.7696
```

It can be clearly seen that the surpluses decay with a factor of 1/4:
On the first level, we obtain 1, on the second 1/4, and on the third
1/16 as surpluses.

[f0]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cvec%7Bx%7D_j%20%5Cin%20%5B0%2C%201%5D%5E2
[f1]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cvarphi_j%3A%5C%3B%20%5B0%2C%201%5D%5E2%20%5Cto%20%5Cmathbb%7BR%7D
[f2]: http://chart.apis.google.com/chart?cht=tx&chl=%0A%20%20f%3A%5C%3B%20%5B0%2C%201%5D%5E2%20%5Cto%20%5Cmathbb%7BR%7D%2C%5Cquad%0A%20%20f%28x_0%2C%20x_1%29%20%3A%3D%2016%20%28x_0%20-%201%29%20x_0%20%28x_1%20-%201%29%20x_1%0A
[f3]: http://chart.apis.google.com/chart?cht=tx&chl=%0A%20%20u%3A%5C%3B%20%5B0%2C%201%5D%5E2%20%5Cto%20%5Cmathbb%7BR%7D%2C%5Cquad%0A%20%20u%28x_0%2C%20x_1%29%20%3A%3D%20%5Csum_%7Bj%3D0%7D%5E%7BN-1%7D%20%5Calpha_j%20%5Cvarphi_j%28x_0%2C%20x_1%29%0A
[f4]: http://chart.apis.google.com/chart?cht=tx&chl=%5Calpha_j
[f5]: http://chart.apis.google.com/chart?cht=tx&chl=u%28%5Cvec%7Bx%7D_j%29%20%3D%20f%28%5Cvec%7Bx%7D_j%29
[f6]: http://chart.apis.google.com/chart?cht=tx&chl=j
[f7]: http://chart.apis.google.com/chart?cht=tx&chl=f
[f8]: http://chart.apis.google.com/chart?cht=tx&chl=%5B0%2C%201%5D%5E2
[f9]: http://chart.apis.google.com/chart?cht=tx&chl=u
[f10]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cvec%7Bp%7D%20%3D%20%280.52%2C%200.73%29
[f11]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cvec%7Bp%7D