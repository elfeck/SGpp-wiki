In order to use orthogonal polynomials with combigrid PCE and the combigird probability density functions, you have to install the Dakota library. You can do this as follows (tested with Dakota 6.8):
Install dependencies as described here:
[https://dakota.sandia.gov/content/install-required-tools-and-libraries](https://dakota.sandia.gov/content/install-required-tools-and-libraries)

On Ubuntu most dependencies can be installed from standard package repository. See
[https://dakota.sandia.gov/content/linux-ubuntu-1404](https://dakota.sandia.gov/content/linux-ubuntu-1404)

for reference. Obtain source code either from downloading appropriate source package:
[https://dakota.sandia.gov/downloads](https://dakota.sandia.gov/downloads)

or the dakota git repository:
[https://dakota.sandia.gov/content/getting-dakota-source-code](https://dakota.sandia.gov/content/getting-dakota-source-code)

by performing 
```console
git clone https://software.sandia.gov/git/dakota
```

edit `dakota/.gitmodules`  to redirect each of the submodule URLs  to `https://software.sandia.gov/git/`

```console
git submodule sync
git submodule update --init
```
Then follow compilation and installation example at [https://dakota.sandia.gov/content/configure-compile-and-install-dakota](https://dakota.sandia.gov/content/configure-compile-and-install-dakota).

Point to dakota include and library directory when building SG++.

Example custom.py with the appropriate options for building the combigrid module:

```python
OPT=1
USE_UMFPACK=1
USE_EIGEN=1
USE_ARMADILLO=1
USE_DAKOTA=1
SG_BASE=1
SG_COMBIGRID=1
SG_OPTIMIZATION=1
SG_QUADRATURE=1
SG_PDE=1
SG_SOLVER=1
SG_FINANCE=0
SG_MISC=0
SG_PARALLEL=0
SG_DATADRIVEN=0
COMPILE_BOOST_TESTS=0
RUN_CPPLINT=0
RUN_PYTHON_TESTS=0
CPPPATH="<dakota_install>/include"
LIBPATH="<dakota_install>/lib"
```
