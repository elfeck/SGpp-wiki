To insure extendability and maintainability, SG++ is divided into a number of modules that implement different functionality.

Different modules can be compiled independently from each other.

The modules can depend on each other. Especially all depend on the base module. Libraries are generated for all modules. The following modules are available:

* Module sgpp::base Fundamental functionality required by all other modules.
* Module sgpp::combigrid Combination technique functionality.
* Module sgpp::datadriven Data mining and machine learning.
* Module sgpp::optimization SG++ module for optimization of smooth sparse grid interpolants.
* Module sgpp::pde Operations and functionality related to PDEs.
* Module sgpp::quadrature Stochastic and deterministic quadrature algorithms.
* Module sgpp::solver Solvers in the broadest sense: PDE, linear equations, gradient descent, etc.

The modules correspond to the C++ namespaces. Roughly speaking, all files from one namespace belong to the respective module. But a module can contain files belonging to some another namespace, i.e. static factory methods in the namespace sgpp::op_factory.

#Folder structure

Please note that the modularization results in a somewhat unconventional folder structure. Keeping

/path/to/SGpp/module/

as "modular" as possible requires to have everything belonging to a module below that directory. This especially applies to the src folder. Its subdirectories reflect the namespaces. Therefore, the module name reappears once again:

/path/to/SGpp/module/src/sgpp/module/...

