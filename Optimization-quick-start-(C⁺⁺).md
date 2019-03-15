

On this page, we look at an example application of the `sgpp::optimization` module.
Versions of the example are given in all languages
currently supported by SG++: C++, Python, Java, and MATLAB.

The example interpolates a bivariate test function with B-splines instead
of piecewise linear basis functions to obtain a smoother interpolant.
The resulting sparse grid function is then minimized with the method of steepest descent.
For comparison, we also minimize the objective function with Nelder-Mead's method.

First, we include all the necessary headers, including those of the `sgpp::base` and
`sgpp::optimization` module.

```c++
#include <sgpp_base.hpp>
#include <sgpp_optimization.hpp>

#include <algorithm>
#include <iostream>
#include <iterator>
```

The function ![f0] to be minimized
is called <i>objective function</i> and has to derive from
`sgpp::optimization::ScalarFunction`.
In the constructor, we give the dimensionality of the domain
(in this case ![f1]).
The eval method evaluates the objective function and returns the function
value ![f2] for a given point ![f3].
The clone method returns a `std::unique_ptr` to a clone of the object
and is used for parallelization (in case eval is not thread-safe).

```c++
class ExampleFunction : public sgpp::optimization::ScalarFunction {
 public:
  ExampleFunction() : sgpp::optimization::ScalarFunction(2) {}

  double eval(const sgpp::base::DataVector& x) {
    // minimum is f(x) = -2 for x[0] = 3*pi/16, x[1] = 3*pi/14
    return std::sin(8.0 * x[0]) + std::sin(7.0 * x[1]);
  }

  virtual void clone(std::unique_ptr<sgpp::optimization::ScalarFunction>& clone) const {
    clone = std::unique_ptr<sgpp::optimization::ScalarFunction>(new ExampleFunction(*this));
  }
};
```

Now, we can start with the `main` function.

```c++
void printLine() {
  std::cout << "----------------------------------------"
               "----------------------------------------\n";
}

int main(int argc, const char* argv[]) {
  (void)argc;
  (void)argv;

  std::cout << "sgpp::optimization example program started.\n\n";
  // increase verbosity of the output
  sgpp::optimization::Printer::getInstance().setVerbosity(2);
```

Here, we define some parameters: objective function, dimensionality,
B-spline degree, maximal number of grid points, and adaptivity.

```c++
  // objective function
  ExampleFunction f;
  // dimension of domain
  const size_t d = f.getNumberOfParameters();
  // B-spline degree
  const size_t p = 3;
  // maximal number of grid points
  const size_t N = 30;
  // adaptivity of grid generation
  const double gamma = 0.95;
```

First, we define a grid with modified B-spline basis functions and
an iterative grid generator, which can generate the grid adaptively.

```c++
  sgpp::base::ModBsplineGrid grid(d, p);
  sgpp::optimization::IterativeGridGeneratorRitterNovak gridGen(f, grid, N, gamma);
```

With the iterative grid generator, we generate adaptively a sparse grid.

```c++
  printLine();
  std::cout << "Generating grid...\n\n";

  if (!gridGen.generate()) {
    std::cout << "Grid generation failed, exiting.\n";
    return 1;
  }
```

Then, we hierarchize the function values to get hierarchical B-spline
coefficients of the B-spline sparse grid interpolant
![f4].

```c++
  printLine();
  std::cout << "Hierarchizing...\n\n";
  sgpp::base::DataVector functionValues(gridGen.getFunctionValues());
  sgpp::base::DataVector coeffs(functionValues.getSize());
  sgpp::optimization::HierarchisationSLE hierSLE(grid);
  sgpp::optimization::sle_solver::Auto sleSolver;

  // solve linear system
  if (!sleSolver.solve(hierSLE, functionValues, coeffs)) {
    std::cout << "Solving failed, exiting.\n";
    return 1;
  }
```

We define the interpolant ![f5] and its gradient
![f6] for use with the gradient method (steepest descent).
Of course, one can also use other optimization algorithms from
`sgpp::optimization::optimizer`.

```c++
  printLine();
  std::cout << "Optimizing smooth interpolant...\n\n";
  sgpp::optimization::InterpolantScalarFunction ft(grid, coeffs);
  sgpp::optimization::InterpolantScalarFunctionGradient ftGradient(grid, coeffs);
  sgpp::optimization::optimizer::GradientDescent gradientDescent(ft, ftGradient);
  sgpp::base::DataVector x0(d);
  double fX0;
  double ftX0;
```

The gradient method needs a starting point.
We use a point of our adaptively generated sparse grid as starting point.
More specifically, we use the point with the smallest
(most promising) function value and save it in x0.

```c++
  {
    sgpp::base::GridStorage& gridStorage = grid.getStorage();

    // index of grid point with minimal function value
    size_t x0Index =
        std::distance(functionValues.getPointer(),
                      std::min_element(functionValues.getPointer(),
                                       functionValues.getPointer() + functionValues.getSize()));

    x0 = gridStorage.getCoordinates(gridStorage[x0Index]);

    fX0 = functionValues[x0Index];
    ftX0 = ft.eval(x0);
  }

  std::cout << "x0 = " << x0.toString() << "\n";
  std::cout << "f(x0) = " << fX0 << ", ft(x0) = " << ftX0 << "\n\n";
```

We apply the gradient method and print the results.

```c++
  gradientDescent.setStartingPoint(x0);
  gradientDescent.optimize();
  const sgpp::base::DataVector& xOpt = gradientDescent.getOptimalPoint();
  const double ftXOpt = gradientDescent.getOptimalValue();
  const double fXOpt = f.eval(xOpt);

  std::cout << "\nxOpt = " << xOpt.toString() << "\n";
  std::cout << "f(xOpt) = " << fXOpt << ", ft(xOpt) = " << ftXOpt << "\n\n";
```

For comparison, we apply the classical gradient-free Nelder-Mead method
directly to the objective function ![f7].

```c++
  printLine();
  std::cout << "Optimizing objective function (for comparison)...\n\n";

  sgpp::optimization::optimizer::NelderMead nelderMead(f, 1000);
  nelderMead.optimize();
  sgpp::base::DataVector xOptNM = nelderMead.getOptimalPoint();
  const double fXOptNM = nelderMead.getOptimalValue();
  const double ftXOptNM = ft.eval(xOptNM);

  std::cout << "\nnxOptNM = " << xOptNM.toString() << "\n";
  std::cout << "f(xOptNM) = " << fXOptNM << ", ft(xOptNM) = " << ftXOptNM << "\n\n";

  printLine();
  std::cout << "\nsgpp::optimization example program terminated.\n";

  return 0;
}
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