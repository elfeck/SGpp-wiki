

On this page, we look at an example application of the `sgpp::optimization` module.
Versions of the example are given in all languages
currently supported by SG++: C++, Python, Java, and MATLAB.

The example interpolates a bivariate test function with B-splines instead
of piecewise linear basis functions to obtain a smoother interpolant.
The resulting sparse grid function is then minimized with the method of steepest descent.
For comparison, we also minimize the objective function with Nelder-Mead's method.

The example uses the following external class (stored in <tt>ExampleFunction.java</tt>).
The function ![f0] to be minimized
is called <i>objective function</i> and has to derive from sgpp.OptScalarFunction.
In the constructor, we give the dimensionality of the domain
(in this case ![f1]).
The eval method evaluates the objective function and returns the function
value ![f2] for a given point ![f3].
```java
// Copyright (C) 2008-today The SG++ project
// This file is part of the SG++ project. For conditions of distribution and
// use, please see the copyright notice provided with SG++ or at
// sgpp.sparsegrids.org

/**
 * Example test function.
 */
public class ExampleFunction extends sgpp.OptScalarFunction {
  /**
   * Constructor.
   */
  public ExampleFunction() {
    super(2);
  }

  /**
   * Evaluates the test function.
   *
   * @param x     point \f$\vec{x} \in [0, 1]^2\f$
   * @return      \f$f(\vec{x})\f$
   */
  public double eval(sgpp.DataVector x) {
    if ((x.get(0) >= 0.0) && (x.get(0) <= 1.0) &&
        (x.get(1) >= 0.0) && (x.get(1) <= 1.0)) {
      // minimum is f(x) = -2 for x[0] = 3*pi/16, x[1] = 3*pi/14
      return Math.sin(8.0 * x.get(0)) + Math.sin(7.0 * x.get(1));
    } else {
      return Double.POSITIVE_INFINITY;
    }
  }

  /**
   * Dummy method needed for SWIG, which provides the actual method body.
   */
  public void clone(sgpp.SWIGTYPE_p_std__unique_ptrT_sgpp__optimization__ScalarFunction_t clone) {
  }
}
```



The actual example looks as follows.


```java
public class optimization {
  static void printLine() {
    System.out.println("----------------------------------------" +
                       "----------------------------------------");
  }

  static String numToStr(double number) {
    return new java.text.DecimalFormat("#.#####").format(number);
  }

  public static void main(String[] args) {
```

At the beginning of the program, we have to load the shared library object file.
We can do so by using <tt>java.lang.System.load</tt> or
<tt>sgpp.LoadJSGPPLib.loadJSGPPLib</tt>.
Also, we disable OpenMP within jsgpp since it interferes with SWIG's director feature.

```java
    sgpp.LoadJSGPPLib.loadJSGPPLib();
    sgpp.jsgpp.omp_set_num_threads(1);

    System.out.println("sgpp::optimization example program started.\n");
    // increase verbosity of the output
    sgpp.OptPrinter.getInstance().setVerbosity(2);
```

Here, we define some parameters: objective function, dimensionality,
B-spline degree, maximal number of grid points, and adaptivity.

```java
    // objective function
    ExampleFunction f = new ExampleFunction();
    // dimension of domain
    final long d = f.getNumberOfParameters();
    // B-spline degree
    final long p = 3;
    // maximal number of grid points
    final long N = 30;
    // adaptivity of grid generation
    final double gamma = 0.95;
```

First, we define a grid with modified B-spline basis functions and
an iterative grid generator, which can generate the grid adaptively.

```java
    sgpp.Grid grid = sgpp.Grid.createModBsplineGrid(d, p);
    sgpp.OptIterativeGridGeneratorRitterNovak gridGen =
        new sgpp.OptIterativeGridGeneratorRitterNovak(f, grid, N, gamma);
```

With the iterative grid generator, we generate adaptively a sparse grid.

```java
    printLine();
    System.out.println("Generating grid...\n");

    if (!gridGen.generate()) {
      System.out.println("Grid generation failed, exiting.");
      return;
    }
```

Then, we hierarchize the function values to get hierarchical B-spline
coefficients of the B-spline sparse grid interpolant
![f4].

```java
    printLine();
    System.out.println("Hierarchizing...\n");
    final sgpp.DataVector functionValues = gridGen.getFunctionValues();
    sgpp.DataVector coeffs = new sgpp.DataVector(functionValues.getSize());
    sgpp.OptHierarchisationSLE hierSLE = new sgpp.OptHierarchisationSLE(grid);
    sgpp.OptAutoSLESolver sleSolver = new sgpp.OptAutoSLESolver();

    // solve linear system
    if (!sleSolver.solve(hierSLE, gridGen.getFunctionValues(), coeffs)) {
      System.out.println("Solving failed, exiting.");
      return;
    }
```

We define the interpolant ![f5] and its gradient
![f6] for use with the gradient method (steepest descent).
Of course, one can also use other optimization algorithms from
`sgpp::optimization::optimizer`.

```java
    printLine();
    System.out.println("Optimizing smooth interpolant...\n");
    sgpp.OptInterpolantScalarFunction ft =
        new sgpp.OptInterpolantScalarFunction(grid, coeffs);
    sgpp.OptInterpolantScalarFunctionGradient ftGradient =
        new sgpp.OptInterpolantScalarFunctionGradient(grid, coeffs);
    sgpp.OptGradientDescent gradientDescent =
        new sgpp.OptGradientDescent(ft, ftGradient);
    sgpp.DataVector x0 = new sgpp.DataVector(d);
    double fX0;
    double ftX0;
```

The gradient method needs a starting point.
We use a point of our adaptively generated sparse grid as starting point.
More specifically, we use the point with the smallest
(most promising) function value and save it in x0.

```java
    {
      sgpp.GridStorage gridStorage = grid.getStorage();

      // index of grid point with minimal function value
      int x0Index = 0;
      fX0 = functionValues.get(0);

      for (int i = 1; i < functionValues.getSize(); i++) {
        if (functionValues.get(i) < fX0) {
          fX0 = functionValues.get(i);
          x0Index = i;
        }
      }

      x0 = gridStorage.getCoordinates(gridStorage.getPoint(x0Index));
      ftX0 = ft.eval(x0);
    }

    System.out.println("x0 = " + x0);
    System.out.println("f(x0) = " + numToStr(fX0) +
                       ", ft(x0) = " + numToStr(ftX0) + "\n");
```

We apply the gradient method and print the results.

```java
    gradientDescent.setStartingPoint(x0);
    gradientDescent.optimize();
    sgpp.DataVector xOpt = gradientDescent.getOptimalPoint();
    final double ftXOpt = gradientDescent.getOptimalValue();
    final double fXOpt = f.eval(xOpt);

    System.out.println("\nxOpt = " + xOpt);
    System.out.println("f(xOpt) = " + numToStr(fXOpt) +
                       ", ft(xOpt) = " + numToStr(ftXOpt) + "\n");
```

For comparison, we apply the classical gradient-free Nelder-Mead method
directly to the objective function ![f7].

```java
    printLine();
    System.out.println("Optimizing objective function (for comparison)...\n");

    sgpp.OptNelderMead nelderMead = new sgpp.OptNelderMead(f, 1000);
    nelderMead.optimize();
    sgpp.DataVector xOptNM = nelderMead.getOptimalPoint();
    final double fXOptNM = nelderMead.getOptimalValue();
    final double ftXOptNM = ft.eval(xOptNM);

    System.out.println("\nxOptNM = " + xOptNM);
    System.out.println("f(xOptNM) = " + numToStr(fXOptNM) +
                       ", ft(xOptNM) = " + numToStr(ftXOptNM) + "\n");

    printLine();
    System.out.println("\nsgpp::optimization example program terminated.");
  }
}  // end of class
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