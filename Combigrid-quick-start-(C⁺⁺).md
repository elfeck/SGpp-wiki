This tutorial contains examples with increasing complexity to introduce you to the combigrid
module. The combigrid module is quite separated from the other modules. It only refers to the
base module for things like DataVector and DataMatrix.
First, we need some includes.

```c++
#include <sgpp/combigrid/operation/CombigridMultiOperation.hpp>
#include <sgpp/combigrid/operation/CombigridOperation.hpp>
#include <sgpp/combigrid/operation/CombigridTensorOperation.hpp>
#include <sgpp/combigrid/operation/Configurations.hpp>
#include <sgpp/combigrid/operation/multidim/AveragingLevelManager.hpp>
#include <sgpp/combigrid/operation/multidim/WeightedRatioLevelManager.hpp>
#include <sgpp/combigrid/functions/OrthogonalPolynomialBasis1D.hpp>
#include <sgpp/combigrid/storage/FunctionLookupTable.hpp>
#include <sgpp/combigrid/utils/Stopwatch.hpp>
#include <sgpp/combigrid/utils/Utils.hpp>

#include <cmath>

#include <iostream>
#include <vector>
```

The first thing we need is a function to evaluate. This function will be evaluated on the domain
![f0]. This particular function can be used with any number of dimensions.

```c++
double f(sgpp::base::DataVector const &x) {
  double prod = 1.0;
  for (size_t i = 0; i < x.getSize(); ++i) {
    prod *= exp(-x[i]);
  }
  return prod;
}

// We have to wrap f in a sgpp::combigrid::MultiFunction object.
static const sgpp::combigrid::MultiFunction func(f);

// Let's use a 3D-function.
size_t d = 3;
```

### Example 1: Leja quadrature with linear growth of grid points

Here comes the first and very simple example.

```c++
void example1() {
```

Let's increase the number of points by two for each level.

```c++
  size_t growthFactor = 2;
```

Now create the operation object that handles the evaluation. The evaluation mode is quadrature,
so it will approximate the integral of f over [0, 1]^d. It uses Leja points with 1 + 2*l
points in level l. The level starts from zero, higher level means finer grid.
Slower growth of the number of points per level means that the total number of points used can
be controlled better.

```c++
  std::shared_ptr<sgpp::combigrid::CombigridOperation> operation =
      sgpp::combigrid::CombigridOperation::createLinearLejaQuadrature(d, func, growthFactor);
```

Now, we compute the result. The parameter 2 means that grid at level-multi-indices with a
1-norm (i.e. sum of entries) less than or equal to 2 are used. In our 3D case, these are
exactly the levels (0, 0, 0), (1, 0, 0), (2, 0, 0), (1, 1, 0), (1, 0, 1), (0, 1, 0), (0, 2, 0),
(0, 1, 1), (0, 0, 1) and (0, 0, 2).

```c++
  double result = operation->evaluate(2);
```

Now compare the result to the analytical solution:

```c++
  std::cout << "Quadrature result: " << result
            << ", analytical solution: " << pow(1 - 1.0 / M_E, static_cast<double>(d)) << "\n";
```

We can also find out how many function evaluations have been used:

```c++
  std::cout << "Number of function evaluations: " << operation->numGridPoints() << "\n";
}
```

### Example 2: Polynomial interpolation on nested Clenshaw Curtis grids

The next example uses interpolation.

```c++
void example2() {
```

This time, we use Clenshaw-Curtis points with exponentially growing number of points per level.
This is helpful for CC points to make them nested. Nested means that the set of grid points at
one level is a subset of the set of grid points at the next level. Nesting can drastically
reduce the number of needed function evaluations. Using these grid points, we will do
polynomial interpolation at a single point.

```c++
  std::shared_ptr<sgpp::combigrid::CombigridOperation> operation =
      sgpp::combigrid::CombigridOperation::createExpClenshawCurtisPolynomialInterpolation(d, func);
```

Now create a point where to evaluate the interpolated function:

```c++
  sgpp::base::DataVector evaluationPoint(d);

  evaluationPoint[0] = 0.1572;
  evaluationPoint[1] = 0.6627;
  evaluationPoint[2] = 0.2378;
```

We can now evaluate the interpolation at this point (using 3 as a bound for the 1-norm of the
level multi-index):

```c++
  double result = operation->evaluate(3, evaluationPoint);
```

Now compare the result to the actual function value:

```c++
  std::cout << "Interpolation result: " << result << ", function value: " << func(evaluationPoint)
            << "\n";
```

Again, print the number of function evaluations:

```c++
  std::cout << "Function evaluations: " << operation->numGridPoints() << "\n";
```

Now, let's do another (more sophisticated) evaluation at a different point, so change the point
and re-set the parameter. This method will automatically clear all intermediate values that
have been computed internally up to now.

```c++
  evaluationPoint[0] = 0.4444;
  std::cout << "Target function value: " << func(evaluationPoint) << "\n";
  operation->setParameters(evaluationPoint);
```

The level manager provides more options for combigrid evaluation, so let's get it:

```c++
  std::shared_ptr<sgpp::combigrid::LevelManager> levelManager = operation->getLevelManager();
```

We can add regular levels like before:

```c++
  levelManager->addRegularLevels(3);
```

The result can be fetched from the CombigridOperation:

```c++
  std::cout << "Regular result 1: " << operation->getResult() << "\n";
  std::cout << "Total function evaluations: " << operation->numGridPoints() << "\n";
```

We can also add more points in a regular structure, using at most 50 new function evaluations.
Most level-adding variants of levelManager also have a parallelized version. This version
executes the calls to func in parallel with a specified number of threads, which is okay here
since func supports parallel evaluations. Since func takes very little time to evaluate and the
parallelization only concerns function evaluations and not the computations on the resulting
function values, parallel evaluation is not actually useful in this case.
We will use 4 threads for the function evaluations.

```c++
  levelManager->addRegularLevelsByNumPointsParallel(50, 4);
  std::cout << "Regular result 2: " << operation->getResult() << "\n";
  std::cout << "Total function evaluations: " << operation->numGridPoints() << "\n";
```

We can also use adaptive level generation. The adaption strategy depends on the subclass of
LevelManager that is used. If you do not want to use the default LevelManager, you can specify
your own LevelManager:

```c++
  operation->setLevelManager(std::make_shared<sgpp::combigrid::AveragingLevelManager>());
  levelManager = operation->getLevelManager();
```

It was necessary to use setLevelManager(), because this links the LevelManager to the
computation. Now, let's add at most 80 more function evaluations adaptively.
Note that the adaption here is only based on the result at our single evaluation point, which
might give inaccurate results. The same holds for quadrature.
In practice, you should probably do an interpolation at a lot of Monte-Carlo points via
CombigridMultiOperation (cf. Example 3) and then transfer the generated level structure to
another CombigridOperation or CombigridMultiOperation for your actual evaluation (cf. Example
4).

```c++
  levelManager->addLevelsAdaptive(60);
  std::cout << "Adaptive result: " << operation->getResult() << "\n";
  std::cout << "Total function evaluations: " << operation->numGridPoints() << "\n";
}
```

### Example 3: Evaluation at multiple points

Now, we want to do interpolation at multiple evaluation points efficiently.

```c++
void example3() {
```

Use Leja points unlike example 2 and use CombigridMultiOperation for evaluation at multiple
points.

```c++
  std::shared_ptr<sgpp::combigrid::CombigridMultiOperation> operation =
      sgpp::combigrid::CombigridMultiOperation::createLinearLejaPolynomialInterpolation(d, func);
```

One method to pass the data is via a std::vector<sgpp::base::DataVector>. We will use 2
interpolation points.

```c++
  std::vector<sgpp::base::DataVector> parameters(2, sgpp::base::DataVector(d));
  parameters[0][0] = 0.2;
  parameters[0][1] = 0.6;
  parameters[0][2] = 0.7;
  parameters[1][0] = 0.3;
  parameters[1][1] = 0.9;
  parameters[1][2] = 1.0;
```

Let's use the simple interface for this example and stop the time:

```c++
  sgpp::combigrid::Stopwatch stopwatch;
  sgpp::base::DataVector result = operation->evaluate(3, parameters);
  stopwatch.log();
  std::cout << "First result: " << result[0] << ", function value: " << func(parameters[0])
            << std::endl
            << "Second result: " << result[1] << ", function value: " << func(parameters[1])
            << "\n";
```

We can also set the parameters via a DataMatrix containing the vectors as columns:

```c++
  sgpp::base::DataMatrix matrix(3, 2);
  for (size_t i = 0; i < parameters.size(); ++i) {
    for (size_t j = 0; j < parameters[0].getSize(); ++j) {
      matrix(j, i) = parameters[i][j];
    }
  }

  result = operation->evaluate(3, matrix);
  std::cout << "First result: " << result[0] << ", function value: " << func(parameters[0]) << "\n";
  std::cout << "Second result: " << result[1] << ", function value: " << func(parameters[1])
            << "\n";
}  // end example3
```

### Example 4: Serialization and lookup tables

This example shows how to store and retrieve computed function values.

```c++
void example4() {
```

First, we create a function that prints a string if it is called:

```c++
  sgpp::combigrid::MultiFunction loggingFunc([](sgpp::base::DataVector const &x) {
    std::cout << "call function \n";
    return x[0];
  });
```

Next, we create a FunctionLookupTable. This will cache the function values by their DataVector
parameter. Note, however, that even slightly differing DataVectors will lead to separate
function evaluations.

```c++
  sgpp::combigrid::FunctionLookupTable lookupTable(loggingFunc);
  auto operation = sgpp::combigrid::CombigridOperation::createLinearLejaQuadrature(
      d, sgpp::combigrid::MultiFunction(lookupTable));
```

Do a normal computation...

```c++
  double result = operation->evaluate(2);
  std::cout << "Result computed: " << result << "\n";
```

The first (and most convenient) possibility to store the data is serializing the lookup table.
The serialization is not compressed and will roughly use 60 Bytes per entry. If you have lots
of data, you might consider compressing it.

```c++
  sgpp::combigrid::writeToFile("lookupTable.log", lookupTable.serialize());
```

It is also possible to store which levels have been evaluated:

```c++
  sgpp::combigrid::writeToFile("levels.log",
                               operation->getLevelManager()->getSerializedLevelStructure());
```

Restore the data into another lookup table. The function is still needed for new evaluations.

```c++
  sgpp::combigrid::FunctionLookupTable restoredLookupTable(loggingFunc);
  restoredLookupTable.deserialize(sgpp::combigrid::readFromFile("lookupTable.log"));
  auto operation2 = sgpp::combigrid::CombigridOperation::createLinearLejaQuadrature(
      d, sgpp::combigrid::MultiFunction(restoredLookupTable));
```

A new evaluation with the same levels does not require new function evaluations:

```c++
  operation2->getLevelManager()->addLevelsFromSerializedStructure(
      sgpp::combigrid::readFromFile("levels.log"));
  result = operation2->getResult();
  std::cout << "Result computed (2nd time): " << result << "\n";
```

Another less general way of storing the data is directly serializing the storage underlying the
operation. This means that retrieval is faster, but it only works if the same grid is used
again.
For demonstration purposes, we use loggingFunc directly this time without a lookup table:

```c++
  sgpp::combigrid::writeToFile("storage.log", operation->getStorage()->serialize());
  auto operation3 = sgpp::combigrid::CombigridOperation::createLinearLejaQuadrature(
      d, sgpp::combigrid::MultiFunction(loggingFunc));
  operation3->getStorage()->deserialize(sgpp::combigrid::readFromFile("storage.log"));
  result = operation3->evaluate(2);
  std::cout << "Result computed (3rd time): " << result << "\n";
}
```

### Example 5: Using different operations in each dimension

This example shows how to apply different operators in different dimensions.

```c++
void example5() {
```

First, we want to configure which grid points to use in which dimension.
We use Chebyshev points in the 0th dimension. To make them nested, we have to use at least ![f1] points at level ![f2]. This is why this method contains the prefix exp.
CombiHierarchies provides some matching configurations for grid points. If you nevertheless
need your own configuration or you want to know which growth strategy and ordering fit to which
point distribution, look up the implementation details in CombiHierarchies, it is not
difficult to implement your own configuration.

```c++
  sgpp::combigrid::CombiHierarchies::Collection grids;
  grids.push_back(sgpp::combigrid::CombiHierarchies::expChebyshev());
```

Our next set of grid points are Leja points with linear growth (![f3]).
For the last dimension, we use equidistant points with boundary. These are suited for linear
interpolation. To make them nested, again the slowest possible exponential growth is selected
by the CombiHierarchies class.

```c++
  grids.push_back(sgpp::combigrid::CombiHierarchies::linearLeja(3));
  grids.push_back(sgpp::combigrid::CombiHierarchies::expUniformBoundary());
```

The next thing we have to configure is the linear operation that is performed in those
directions. We will use polynomial interpolation in the 0th dimension, quadrature in the 1st
dimension and linear interpolation in the 2nd dimension.
Roughly spoken, this means that a quadrature is performed on the 1D function that is the
interpolated function with two fixed parameters. But since those operators "commute", the
result is invariant under the order that the operations are applied in.
The CombiEvaluators class also provides analogous methods and typedefs for the multi-evaluation
case.

```c++
  sgpp::combigrid::CombiEvaluators::Collection evaluators;
  evaluators.push_back(sgpp::combigrid::CombiEvaluators::polynomialInterpolation());
  evaluators.push_back(sgpp::combigrid::CombiEvaluators::quadrature());
  evaluators.push_back(sgpp::combigrid::CombiEvaluators::linearInterpolation());
```

To create a CombigridOperation object with our own configuration, we have to provide a
LevelManager as well:

```c++
  std::shared_ptr<sgpp::combigrid::LevelManager> levelManager(
      new sgpp::combigrid::WeightedRatioLevelManager());

  auto operation =
      std::make_shared<sgpp::combigrid::CombigridOperation>(grids, evaluators, levelManager, func);
```

The two interpolations need a parameter ![f4]. If ![f5] is the interpolated
function, the operation approximates the result of ![f6].

```c++
  sgpp::base::DataVector parameters(2);
  parameters[0] = 0.777;
  parameters[1] = 0.14159;
  double result = operation->evaluate(2, parameters);
  std::cout << "Result: " << result << "\n";
}
```

### Example 6: Using a function operating on grids

This example shows how to apply different operators in different dimensions.

```c++
void example6() {
```

In some applications, you might not want to have a callback function that is called at single
points, but on a full grid. One of these applications is solving PDEs. This example provides a
simple framework where a PDE solver can be included. It is also suited for other tasks.
The core part is a function that computes grid values on a full grid.

```c++
  sgpp::combigrid::GridFunction gf([](std::shared_ptr<sgpp::combigrid::TensorGrid> grid) {
    // We store the results for each grid point, encoded by a MultiIndex, in a TreeStorage
    auto result = std::make_shared<sgpp::combigrid::TreeStorage<double> >(d);

    // Creates an iterator that yields all multi-indices of grid points in the grid.
    sgpp::combigrid::MultiIndexIterator it(grid->numPoints());

    while (it.isValid()) {
      // Customize this computation for your algorithm
      double value = func(grid->getGridPoint(it.getMultiIndex()));

      // Store the result at the multi index encoding the grid point
      result->set(it.getMultiIndex(), value);
      it.moveToNext();
    }

    return result;
  });
```

To create a CombigridOperation, we currently have to use the longer way as in example 5.

```c++
  sgpp::combigrid::CombiHierarchies::Collection grids(
      d, sgpp::combigrid::CombiHierarchies::expUniformBoundary());
  sgpp::combigrid::CombiEvaluators::Collection evaluators(
      d, sgpp::combigrid::CombiEvaluators::cubicSplineInterpolation());
  std::shared_ptr<sgpp::combigrid::LevelManager> levelManager(
      new sgpp::combigrid::WeightedRatioLevelManager());
```

We have to specify if the function always produces the same value for the same grid points.
This can make the storage smaller if the grid points are nested. In this implementation, this
is true. However, it would be false in the PDE case, so we set it to false here.

```c++
  bool exploitNesting = false;
```

Now create an operation as usual and evaluate the interpolation with a test parameter.

```c++
  auto operation = std::make_shared<sgpp::combigrid::CombigridOperation>(
      grids, evaluators, levelManager, gf, exploitNesting);

  sgpp::base::DataVector parameter(std::vector<double>{0.1, 0.2, 0.3});

  double result = operation->evaluate(4, parameter);

  std::cout << "Target function value: " << func(parameter) << "\n";
  std::cout << "Numerical result: " << result << "\n";
}
```

### Example 7:
```
void example7() {
  //  std::shared_ptr<sgpp::combigrid::CombigridOperation> operation =
  //      sgpp::combigrid::CombigridOperation::createExpUniformBoundaryLinearInterpolation(1, func);

  std::shared_ptr<sgpp::combigrid::CombigridOperation> operation =
      sgpp::combigrid::CombigridOperation::createExpClenshawCurtisBsplineInterpolation(1, func, 3);
```

Now create a point where to evaluate the interpolated function:

```c++
  sgpp::base::DataVector evaluationPoint(d);
  evaluationPoint[0] = 0.1572;
```

We can now evaluate the interpolation at this point (using 3 as a bound for the 1-norm of the
level multi-index):

```c++
  double result = operation->evaluate(1, evaluationPoint);
```

Now compare the result to the actual function value:

```c++
  std::cout << "Interpolation result: " << result << ", function value: " << func(evaluationPoint)
            << std::endl;
}
```
### Example 8: UQ setting with variance refinement

This example shows how to use the variance refinement method that uses the PCE transformation for
variance computation on each subspace.

```c++
void example8() {
  sgpp::combigrid::OrthogonalPolynomialBasis1DConfiguration config;
  config.polyParameters.type_ = sgpp::combigrid::OrthogonalPolynomialBasisType::LEGENDRE;

  auto basisFunction = std::make_shared<sgpp::combigrid::OrthogonalPolynomialBasis1D>(config);

  auto tensor_operation =
      sgpp::combigrid::CombigridTensorOperation::createExpClenshawCurtisPolynomialInterpolation(
          basisFunction, d, func);

  auto levelManager = std::make_shared<sgpp::combigrid::AveragingLevelManager>();
  tensor_operation->setLevelManager(levelManager);

  // add at least the regular levels up to 1 otherwise the refinement criteria
  // does not converge due to the fact that the variance for the constant basis
  // is zero.
  levelManager->addRegularLevels(1);
  std::cout << "Total function evaluations: " << tensor_operation->numGridPoints() << "\n";
  levelManager->addLevelsAdaptive(200);
  std::cout << "Total function evaluations: " << tensor_operation->numGridPoints() << "\n";

  // print the expectation value
  auto tensorResult = tensor_operation->getResult();
  std::cout << "result of transformation: E(u) = "
            << tensorResult.get(sgpp::combigrid::MultiIndex(d, 0)) << std::endl;

  // we use the same level structure and compute the expectation value via quadrature
  auto quadrature_operation =
      sgpp::combigrid::CombigridOperation::createExpClenshawCurtisQuadrature(d, func);

  // copy the level information and initialize the combigrid
  quadrature_operation->getLevelManager()->addLevelsFromStructure(
      levelManager->getLevelStructure());

  double quadratureResult = quadrature_operation->getResult();
  std::cout << "result of quadrature    : E(u) = " << quadratureResult << std::endl;
}

int main() {
  std::cout << "Example 1: \n";
  example1();

  std::cout << "\nExample 2: \n";
  example2();

  std::cout << "\nExample 3: \n";
  example3();

  std::cout << "\nExample 4: \n";
  example4();

  std::cout << "\nExample 5: \n";
  example5();

  std::cout << "\nExample 6: \n";
  example6();

  std::cout << "\nExample 7: \n";
  example7();

  std::cout << "\nExample 8: \n";
  example8();
}  // end of main
```


[f0]: http://chart.apis.google.com/chart?cht=tx&chl=%5B0%2C%201%5D%5Ed
[f1]: http://chart.apis.google.com/chart?cht=tx&chl=n%0A%3D%203%5El
[f2]: http://chart.apis.google.com/chart?cht=tx&chl=l
[f3]: http://chart.apis.google.com/chart?cht=tx&chl=n%20%3D%201%20%2B%203l
[f4]: http://chart.apis.google.com/chart?cht=tx&chl=%28x%2C%20z%29
[f5]: http://chart.apis.google.com/chart?cht=tx&chl=%5Ctilde%7Bf%7D
[f6]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cint_0%5E1%20%5Ctilde%7Bf%7D%28x%2C%20y%2C%20z%29%20%5C%2Cdy