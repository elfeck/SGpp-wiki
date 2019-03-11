Install and enable Dakota for the sgpp::combigrid module.

1. Obtain source code either from [git](https://dakota.sandia.gov/content/getting-dakota-source-code) or downloading the [appropriate source package](https://dakota.sandia.gov/downloads).

2. Follow the [compilation and installation guide](https://dakota.sandia.gov/content/build-compile-source-code).

On Ubuntu most [dependencies](https://dakota.sandia.gov/content/linux-ubuntu-1404) can be installed from standard package repository.

3. Point to dakota include and library directory when building SG++
Example custom.py with the appropriate options for building the combigrid module
`
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
`
