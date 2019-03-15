

On this page, we look at an example application of the `sgpp::optimization` module.
Versions of the example are given in all languages
currently supported by SG++: C++, Python, Java, and MATLAB.

The example interpolates a bivariate test function with B-splines instead
of piecewise linear basis functions to obtain a smoother interpolant.
The resulting sparse grid function is then minimized with the method of steepest descent.
For comparison, we also minimize the objective function with Nelder-Mead's method.

First, we import pysgpp and the required modules.

```python
import pysgpp
import math
import sys
```

The function ![f0] to be minimized
is called <i>objective function</i> and has to derive from pysgpp.OptScalarFunction.
In the constructor, we give the dimensionality of the domain
(in this case ![f1]).
The eval method evaluates the objective function and returns the function
value ![f2] for a given point ![f3].

```python
class ExampleFunction(pysgpp.OptScalarFunction):
    """Example objective function from the title of my Master's thesis."""
    def __init__(self):
        super(ExampleFunction, self).__init__(2)

    def eval(self, x):
        """Evaluates the function."""
        return math.sin(8.0 * x[0]) + math.sin(7.0 * x[1])

def printLine():
    print("----------------------------------------" + \
          "----------------------------------------")
```

We have to disable OpenMP within pysgpp since it interferes with SWIG's director feature.

```python
pysgpp.omp_set_num_threads(1)

print("sgpp::optimization example program started.\n")
# increase verbosity of the output
pysgpp.OptPrinter.getInstance().setVerbosity(2)
```

Here, we define some parameters: objective function, dimensionality,
B-spline degree, maximal number of grid points, and adaptivity.

```python
# objective function
f = ExampleFunction()
# dimension of domain
d = f.getNumberOfParameters()
# B-spline degree
p = 3
# maximal number of grid points
N = 30
# adaptivity of grid generation
gamma = 0.95
```

First, we define a grid with modified B-spline basis functions and
an iterative grid generator, which can generate the grid adaptively.

```python
grid = pysgpp.Grid.createModBsplineGrid(d, p)
gridGen = pysgpp.OptIterativeGridGeneratorRitterNovak(f, grid, N, gamma)
```

With the iterative grid generator, we generate adaptively a sparse grid.

```python
printLine()
print("Generating grid...\n")

if not gridGen.generate():
    print("Grid generation failed, exiting.")
    sys.exit(1)
```

Then, we hierarchize the function values to get hierarchical B-spline
coefficients of the B-spline sparse grid interpolant
![f4].

```python
printLine()
print("Hierarchizing...\n")
functionValues = gridGen.getFunctionValues()
coeffs = pysgpp.DataVector(len(functionValues))
hierSLE = pysgpp.OptHierarchisationSLE(grid)
sleSolver = pysgpp.OptAutoSLESolver()

# solve linear system
if not sleSolver.solve(hierSLE, gridGen.getFunctionValues(), coeffs):
    print("Solving failed, exiting.")
    sys.exit(1)
```

We define the interpolant ![f5] and its gradient
![f6] for use with the gradient method (steepest descent).
Of course, one can also use other optimization algorithms from
`sgpp::optimization::optimizer`.

```python
printLine()
print("Optimizing smooth interpolant...\n")
ft = pysgpp.OptInterpolantScalarFunction(grid, coeffs)
ftGradient = pysgpp.OptInterpolantScalarFunctionGradient(grid, coeffs)
gradientDescent = pysgpp.OptGradientDescent(ft, ftGradient)
x0 = pysgpp.DataVector(d)
```

The gradient method needs a starting point.
We use a point of our adaptively generated sparse grid as starting point.
More specifically, we use the point with the smallest
(most promising) function value and save it in x0.

```python
gridStorage = gridGen.getGrid().getStorage()

# index of grid point with minimal function value
x0Index = 0
fX0 = functionValues[0]
for i in range(1, len(functionValues)):
    if functionValues[i] < fX0:
        fX0 = functionValues[i]
        x0Index = i

x0 = gridStorage.getCoordinates(gridStorage.getPoint(x0Index));
ftX0 = ft.eval(x0)

print("x0 = {}".format(x0))
print("f(x0) = {:.6g}, ft(x0) = {:.6g}\n".format(fX0, ftX0))
```

We apply the gradient method and print the results.

```python
gradientDescent.setStartingPoint(x0)
gradientDescent.optimize()
xOpt = gradientDescent.getOptimalPoint()
ftXOpt = gradientDescent.getOptimalValue()
fXOpt = f.eval(xOpt)

print("\nxOpt = {}".format(xOpt))
print("f(xOpt) = {:.6g}, ft(xOpt) = {:.6g}\n".format(fXOpt, ftXOpt))
```

For comparison, we apply the classical gradient-free Nelder-Mead method
directly to the objective function ![f7].

```python
printLine()
print("Optimizing objective function (for comparison)...\n")
nelderMead = pysgpp.OptNelderMead(f, 1000)
nelderMead.optimize()
xOptNM = nelderMead.getOptimalPoint()
fXOptNM = nelderMead.getOptimalValue()
ftXOptNM = ft.eval(xOptNM)

print("\nxOptNM = {}".format(xOptNM))
print("f(xOptNM) = {:.6g}, ft(xOptNM) = {:.6g}\n".format(fXOptNM, ftXOptNM))

printLine()
print("\nsgpp::optimization example program terminated.")
```

The example program outputs the following results:
```
sgpp::optimization example program started.

--------------------------------------------------------------------------------
Generating grid...

Adaptive grid generation (Ritter-Novak)...
    100.0% (N = 29, k = 3)
Done in 3ms.
--------------------------------------------------------------------------------
Hierarchizing...

Solving linear system (automatic method)...
    estimated nnz ratio: 59.8% 
    Solving linear system (Armadillo)...
        constructing matrix (100.0%)
        nnz ratio: 58.0%
        solving with Armadillo
    Done in 0ms.
Done in 1ms.
--------------------------------------------------------------------------------
Optimizing smooth interpolant...

x0 = [0.625, 0.75]
f(x0) = -1.81786, ft(x0) = -1.81786

Optimizing (gradient method)...
    9 steps, f(x) = -2.000780
Done in 1ms.

xOpt = [0.589526, 0.673268]
f(xOpt) = -1.99999, ft(xOpt) = -2.00078

--------------------------------------------------------------------------------
Optimizing objective function (for comparison)...

Optimizing (Nelder-Mead)...
    280 steps, f(x) = -2.000000
Done in 2ms.

xOptNM = [0.589049, 0.673198]
f(xOptNM) = -2, ft(xOptNM) = -2.00077

--------------------------------------------------------------------------------

sgpp::optimization example program terminated.
```



We see that both the gradient-based optimization of the smooth sparse grid
interpolant and the gradient-free optimization of the objective function
find reasonable approximations of the minimum, which lies at
![f8].

[f0]: http://chart.apis.google.com/chart?cht=tx&chl=f%3A%5C%3B%20%5B0%2C%201%5D%5Ed%20%5Cto%20%5Cmathbb%7BR%7D
[f1]: http://chart.apis.google.com/chart?cht=tx&chl=d%20%3D%202
[f2]: http://chart.apis.google.com/chart?cht=tx&chl=f%28%5Cvec%7Bx%7D%29
[f3]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cvec%7Bx%7D%20%5Cin%20%5B0%2C%201%5D%5Ed
[f4]: http://chart.apis.google.com/chart?cht=tx&chl=%5Ctilde%7Bf%7D%3A%5C%3B%20%5B0%2C%201%5D%5Ed%20%5Cto%20%5Cmathbb%7BR%7D
[f5]: http://chart.apis.google.com/chart?cht=tx&chl=%5Ctilde%7Bf%7D
[f6]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cnabla%5Ctilde%7Bf%7D
[f7]: http://chart.apis.google.com/chart?cht=tx&chl=f
[f8]: http://chart.apis.google.com/chart?cht=tx&chl=%283%5Cpi/16%2C%203%5Cpi/14%29%20%5Capprox%20%280.58904862%2C%200.67319843%29