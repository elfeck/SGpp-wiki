@page linux Linux (GCC/Clang/ICC)
This page contains instructions for compiling and using SG++ with
GCC, Clang or ICC under Linux.
For brevity, we assume you want to compile with GCC;
the compiler can be changed easily, however.
@section linux_overview Overview
- \ref linux_dependencies
- \ref linux_compilation
- \ref linux_using
@section linux_dependencies Dependencies
@subsection linux_dependencies_required Required
The following software is required in order to build SG++:
- <a href="https://gcc.gnu.org/" target="_blank">GCC (&ge; 4.8)</a> or
<a href="http://clang.llvm.org/" target="_blank">Clang (&ge; 3.8)</a>
- <a href="http://www.scons.org/" target="_blank">SCons (&ge; 2.3)</a>
is required for building (build system).
SCons is written in Python 2.7, which is therefore needed by SG++ as well.
@subsection linux_dependencies_recommended Recommended
The following software is recommended for core functionality:
- <a href="http://www.boost.org/" target="_blank">Boost.Test</a>
for compiling and running the unit tests.
You can also skip the unit tests, but this is not recommended.
- <a href="http://www.swig.org/" target="_blank">SWIG (&ge; 3.0)</a>,
if you want to use SG++ within Python, Java, or MATLAB.
SWIG creates bindings from C/C++ functions in order to make
them available as a Python module or
prepare them to be called from Java code.
- <a href="http://www.python.org/" target="_blank">
Python development headers</a> are needed,
if you want to compile the Python bindings.
- <a href="http://www.numpy.org/" target="_blank">NumPy for Python 2.7</a>
is needed, if you want to run the Python tests/examples.
- <a href="http://www.java.com/" target="_blank">Java Development Kit (JDK)</a>,
if you want to compile the Java bindings.
- <a href="http://www.doxygen.org/" target="_blank">Doxygen</a> is required to
create and compile the documentation, which you are reading right now.
It is also required to automatically annotate the Python bindings
generated by SWIG with docstrings.
@subsection linux_dependencies_optional Optional
The following software can be installed for full functionality:
- <a href="http://www.graphviz.org/" target="_blank">
dot in the Graphviz package</a>
is optional and generates inheritance diagrams in the Doxygen documentation.
- <a href="http://www.mathworks.com/" target="_blank">MATLAB</a>,
if you want to use SG++ within MATLAB.
- <a href="https://www.gnu.org/software/gsl/" target="_blank">
GNU Scientific Library (GSL)</a>,
is required for some functionality of the <tt>SG_DATADRIVEN</tt> module.
- <a href="http://eigen.tuxfamily.org/" target="_blank">Eigen</a>
is required for some functionality of the <tt>SG_COMBIGRID</tt> module
and recommended for the <tt>SG_OPTIMIZATION</tt> module.
- <a href="http://faculty.cse.tamu.edu/davis/suitesparse.html" target="_blank">
UMFPACK (in SuiteSparse)</a>
is recommended for the <tt>SG_OPTIMIZATION</tt> module.
- <a href="http://arma.sourceforge.net/" target="_blank">Armadillo</a>
is recommended for the <tt>SG_OPTIMIZATION</tt> module.
- <a href="http://getfem.org/gmm.html" target="_blank">Gmm++</a>
can be used in conjunction with the <tt>SG_OPTIMIZATION</tt> module.
- <a href="https://dakota.sandia.gov/">DAKOTA</a> is required for the Polynomial Chaos Expansion functionalities of the <tt>SG_COMBIGRID</tt> module. [Installation instructions](@ref install_dakota)
- <a href="https://www.cgal.org/index.html">CGAL</a> is required for a new Sparse Grid Density Estimation of the <tt>SG_DATADRIVEN</tt> module. It is required to solve a quadratic optimization problem to guarantee positive function values at the grid points.
@subsection linux_dependencies_installation Installation
On a recent Ubuntu system (&ge; 14.04 LTS),
you can install most dependencies by installing the following packages:
- GCC: <tt>g++</tt>
- Clang: <tt>clang</tt> on Ubuntu &ge; 16.04.
On the <a href="http://llvm.org/releases/download.html" target="_blank">
LLVM download page</a>, there are binaries for many other systems,
including older Ubuntu versions.
- SCons: <tt>scons</tt>
- Boost.Test: <tt>libboost-test-dev</tt>
- SWIG: <tt>swig3.0</tt> on Ubuntu &ge; 14.10.
For Ubuntu 14.04 LTS, download the
<a href="https://launchpad.net/ubuntu/utopic/amd64/swig3.0/3.0.2-1ubuntu1"
target="_blank">SWIG 3.0 .deb file</a> here.
- Python development headers: <tt>libpython-dev</tt>
- NumPy: <tt>python-numpy</tt>
- Java: <tt>openjdk-7-jdk</tt> or <tt>openjdk-8-jdk</tt> on Ubuntu &ge; 15.10
- Doxygen: <tt>doxygen</tt>
- Dot: <tt>graphviz</tt>
- GSL: <tt>libgsl-dev</tt>
@section linux_compilation Compilation with SCons
Compilation of the C++ libraries is done with SCons. Execute
@verbatim
scons -j <number of cores>
@endverbatim
in the main folder to compile SG++ with GCC.
For configuration (including other compilers) and
optimization, see below.
If SCons does not seem to find external dependencies
even if they are installed,
you might want to clear the SCons cache before trying again:
@verbatim
rm -r .sconf_temp .sconsign.dblite
@endverbatim
To obtain help on parameters for compilation, type
@verbatim
scons --help
@endverbatim
After compilation, all unit-tests
(located in the <tt>tests</tt>-folder of each module) are executed,
if Boost.Test is installed.
There are also some tests written in Python,
but the majority is written with Boost.Test.
When the build is finished,
the shared libraries are installed in <tt>lib/sgpp</tt>.
If you use it, add this directory to your <tt>LD_LIBRARY_PATH</tt>.
Instructions are also displayed at the end of the build.
@subsection linux_compilation_configuration Configuration
SCons uses the file <tt>SConstruct</tt>. This file contains all information for
compiling SGpp.
If you just execute <tt>scons</tt>,
the default compilation with GCC for SSE3 instruction set is selected.
The compiler can be changed using <tt>COMPILER=clang</tt> or
<tt>COMPILER=intel</tt>, e.g.
To use other instruction sets, use <tt>ARCH=avx</tt>, etc.
You are able to compile different SG++ modules independently.
However, you should take into account the dependencies between
the modules to avoid "undefined symbol" errors:
When using them, depending on the dependencies,
other modules might have to be included, too.
The currently available modules are (see the @ref modules page):
- <tt>SG_BASE</tt>: basic functionality
- <tt>SG_DATADRIVEN</tt>: operations on data
- <tt>SG_SOLVER</tt>: classes for solving the systems of equations
- <tt>SG_PDE</tt>: partial differential equations
- <tt>SG_FINANCE</tt>: financial module
- <tt>SG_PARALLEL</tt>: classes for parallel computing
- <tt>SG_COMBIGRID</tt>: combigrid classes
- <tt>SG_OPTIMIZATION</tt>: optimization of objective functions
Also, there are two switches for supported high-level back-ends:
- <tt>SG_PYTHON</tt>: Python bindings
- <tt>SG_JAVA</tt>: Java bindings
For example, the command
@verbatim
scons SG_OPTIMIZATION=0
@endverbatim
will compile all modules except optimization.
Additionally, you can pass some specific flags to the compiler
using the <tt>CPPFLAGS</tt> environment variable:
@verbatim
scons CPPFLAGS='-g -O0'
@endverbatim
@subsection linux_compilation_pysgpp Python Bindings
The Python bindings are important,
because some unit tests are written in Python.
By default, the Python bindings are built, too.
If not, then some prerequisites are missing
(see \ref linux_dependencies).
By default, the Python bindings will be annotated with Python docstrings,
if Doxygen is installed.
Disabling this feature, which is recommended if you have to recompile the whole codebase frequently, is done by setting <tt>PYDOC=0</tt> in the
SCons command line.
When the build is finished,
the Python bindings are installed in <tt>lib/pysgpp</tt>.
If you use them, add the <tt>lib</tt> directory to your <tt>PYTHONPATH</tt>.
Alternatively, you can install the bindings into your local
<tt>site-packages</tt> directory:
@verbatim
python setup.py install --user
@endverbatim
Instructions are also displayed at the end of the build.
In Python, you can import the library and print its contents via
@code{.py}
import pysgpp
dir(pysgpp)
@endcode
@subsection linux_compilation_jsgpp Java Bindings
By default, the Java bindings are built, too.
If not, then the JDK is missing
(see \ref linux_dependencies).
@subsection linux_compilation_eclipse Eclipse and SCons
Create a Makefile project and change the project properties as follows:
- <i>Properties</i> &rarr; <i>C/C++ Build</i> &rarr; <i>Builder Settings</i>:
Disable <i>Use default build command</i> and set <i>Build command</i> to
<tt>scons</tt>.
- <i>Properties</i> &rarr; <i>C/C++ Build</i> &rarr; <i>Behaviour</i>:
Set <i>Build (Incremental build)</i> to, e.g., <tt>-j 2</tt>
and <i>Clean</i> to <tt>-c</tt>.
@section linux_using Using SG++
In this section, we show how SG++ can be used as a library in other programs.
For C++, this includes compilation, linking, and execution of the program
using SG++.
We also show how to use SG++ from the other supported languages
(Python, Java, and MATLAB).
As an example application, we consider the @ref example_tutorial_cpp
from the directory <tt>base/examples</tt>;
however, the instructions can be analogously applied to other programs.
Note that all examples, including @ref example_tutorial_cpp, are automatically built
after each SCons run.
Therefore, the following steps are not necessary to compile the examples;
rather, the intent is to show the steps to build an application using SG++.
In the following, the current directory is always <tt>base/examples</tt> and
@c /PATH_TO_SGPP refers to the absolute path of the SG++ directory.
We assume that SG++ or its bindings have been successfully built before.
@subsection linux_using_cpp C++
First, compile the program
while supplying the include paths of the relevant modules:
@verbatim
g++ tutorial.cpp \
-c -std=c++11 -fopenmp \
-I/PATH_TO_SGPP/base/src \
-o tutorial.o
@endverbatim
Then, link the program by indicating the SG++ library path and the modules
you want to link against:
@verbatim
g++ tutorial.o \
-fopenmp \
-L/PATH_TO_SGPP/lib/sgpp \
-lsgppbase \
-o tutorial
@endverbatim
To run the program, note that you have to set the @c LD_LIBRARY_PATH
environment variable to include the SG++ library path:
@verbatim
export LD_LIBRARY_PATH="/PATH_TO_SGPP/lib/sgpp:$LD_LIBRARY_PATH"
./tutorial
@endverbatim
@subsection linux_using_python Python
The Python bindings pysgpp can be used either by setting the
@c PYTHONPATH environment variable to include the @c lib directory, i.e.
@verbatim
export PYTHONPATH="/PATH_TO_SGPP/lib:$PYTHONPATH"
@endverbatim
or by installing pysgpp in the local @c site-packages folder:
@verbatim
python setup.py install --user
@endverbatim
To run your Python program, don't forget to update the @c LD_LIBRARY_PATH
environment variable in any case:
@verbatim
export LD_LIBRARY_PATH="/PATH_TO_SGPP/lib/sgpp:$LD_LIBRARY_PATH"
python tutorial.py
@endverbatim
@subsection linux_using_java Java
Java programs using the Java bindings jsgpp have to be compiled in this way:
@verbatim
javac -cp .:/PATH_TO_SGPP/lib/jsgpp/jsgpp.jar tutorial.java
@endverbatim
When running Java programs, you have to augment @c LD_LIBRARY_PATH
not only by the SG++ library path, but also by a path specific for jsgpp:
@verbatim
export LD_LIBRARY_PATH="/PATH_TO_SGPP/lib/sgpp:/PATH_TO_SGPP/lib/jsgpp:$LD_LIBRARY_PATH"
java -cp .:/PATH_TO_SGPP/lib/jsgpp/jsgpp.jar tutorial
@endverbatim
@subsection linux_using_matlab MATLAB
MATLAB can use SG++ in three ways.
@subsubsection linux_using_matlab_binaries Via Binaries
The recommended way is to download the binaries for use with MATLAB
that are listed at \ref downloads.
For instructions, please see the \ref matlab page.
@subsubsection linux_using_matlab_mex Via MEX Interface
If the binaries don't work for you, then it is possible to write and
compile a MEX interface yourself (similar to the interface that the
binaries would provide).
This means that you have to write a C++ program interacting
with SG++ directly in C++ and converting input and output arguments
from and to MATLAB's data structures.
The parameters to be passed to
<a href="https://www.mathworks.com/help/matlab/ref/mex.html" target="_blank">
MATLAB's @c mex function</a> which compiles
the program are largely the same as for plain C++.
Alternatively, you can link and compile by yourself:
@verbatim
g++ your_mex_program.cpp \
-c -std=c++11 -fopenmp -fPIC \
-Wall -Wextra \
-I/PATH_TO_MATLAB/extern/include \
-I/PATH_TO_SGPP/base/src \
-o your_mex_program.o
g++ your_mex_program.o \
-shared -fopenmp \
-L/PATH_TO_MATLAB/bin/glnxa64 \
-L/PATH_TO_SGPP/lib/sgpp \
-lmex \
-lsgppbase \
-o your_mex_program.mexa64
@endverbatim
Of course, you have to add include paths and library switches for each
module that @c your_mex_program uses.
@subsubsection linux_using_matlab_jsgpp Via jsgpp
The third way of using SG++ from within MATLAB is jsgpp,
i.e., using the Java library of SG++ and import it to MATLAB.
Before we can use these methods in MATLAB, we have to add
@c /PATH_TO_SGPP/lib/jsgpp to the @c librarypath.txt file of MATLAB.
(Hint: Typing @c matlabroot in MATLAB returns the path of your MATLAB
installation.)
Open the file @c /PATH_TO_MATLAB/toolbox/local/librarypath.txt in a text editor
and add the line
@verbatim
/PATH_TO_SGPP/lib/jsgpp
@endverbatim
at the end of the file.
Now we can start MATLAB.
However, we have to set the environment variables @c LD_LIBRARY_PATH and
@c LD_PRELOAD before:
@verbatim
export LD_LIBRARY_PATH="/PATH_TO_SGPP/lib/sgpp:/PATH_TO_SGPP/lib/jsgpp:$LD_LIBRARY_PATH"
export LD_PRELOAD="/usr/lib/x86_64-linux-gnu/libstdc++.so.6:$LD_PRELOAD"
matlab
@endverbatim
The variable @c LD_LIBRARY_PATH has to be set to allow MATLAB to find
the shared libraries (see above).
However, this does not suffice:
MATLAB ships its own version of @c libstdc++
(usually in @c /PATH_TO_MATLAB/sys/os/glnxa64) and prepends its location
to MATLAB's internal @c LD_LIBRARY_PATH.
Since this version of @c libstdc++ is most likely incompatible with the
version used to compile SG++, you will get errors like this
when executing SG++ programs in MATLAB:
@verbatim
Error using tutorial (line 5)
Java exception occurred:
java.lang.UnsatisfiedLinkError: /PATH_TO_SGPP/lib/jsgpp/libjsgpp.so:
PATH_TO_MATLAB/bin/glnxa64/libstdc++.so.6: version
`GLIBCXX_3.4.20' not found (required by /PATH_TO_SGPP/lib/jsgpp/libjsgpp.so)
at java.lang.ClassLoader$NativeLibrary.load(Native Method)
at java.lang.ClassLoader.loadLibrary1(Unknown Source)
at java.lang.ClassLoader.loadLibrary0(Unknown Source)
at java.lang.ClassLoader.loadLibrary(Unknown Source)
at java.lang.Runtime.loadLibrary0(Unknown Source)
at java.lang.System.loadLibrary(Unknown Source)
at sgpp.LoadJSGPPLib.loadJSGPPLib(LoadJSGPPLib.java:10)
@endverbatim
To work around this issue, set the @c LD_PRELOAD variable
to load the correct version @c libstdc++ before calling MATLAB.
You can find the correct version by examining the output of
@verbatim
ldd /PATH_TO_SGPP/lib/jsgpp/libjsgpp.so
@endverbatim
and searching for a line containing @c libstdc++.
On 64-bit Ubuntu systems, it can be located in
<tt>/usr/lib/x86_64-linux-gnu/libstdc++.so.6</tt>,
but it can differ on other systems.
When starting MATLAB, make sure that the Java version of
MATLAB's internal Java Runtime Environment
(check with <tt>version -java</tt> in MATLAB)
is newer than that of the Java you built jsgpp with
(check with <tt>java -version</tt> in a terminal).
Sometimes, you have to force MATLAB to use your installed JDK
by using the <tt>MATLAB_JAVA</tt> environment variable:
@verbatim
export MATLAB_JAVA="/PATH_TO_JDK/jre"
@endverbatim
(e.g., <tt>MATLAB_JAVA="/usr/lib/jvm/java-7-openjdk-amd64/jre"</tt>).
You can get errors like this otherwise:
@verbatim
Undefined variable "sgpp" or class "sgpp.LoadJSGPPLib.loadJSGPPLib".
@endverbatim
Keep in mind that, however, MATLAB seems not to be compatible with Java 8 yet.
After starting MATLAB,
we have to add the @c jsgpp.jar file to MATLAB's class path with the command
@verbatim
javaaddpath('/PATH_TO_SGPP/lib/jsgpp/jsgpp.jar');
@endverbatim
The final step consists in loading the jsgpp library via
@verbatim
sgpp.LoadJSGPPLib.loadJSGPPLib();
@endverbatim
You should now be able to use SG++ in MATLAB.
For an example, look at <tt>base/examples/tutorial.m</tt>.
However, note that the example was written for the MATLAB SG++ binaries
explained above.
Therefore, you likely have to change some calls especially to static
methods like <tt>sgpp.createOperationEval</tt>, which are now located
at <tt>sgpp.jsgpp.*</tt>.
@subsubsection linux_using_matlab_hints Hints
- Use
@verbatim
javaclasspath();
@endverbatim
to see the loaded jar files (upper part static, lower
part dynamic; our jsgpp.jar should be in the latter part).
- Write
@verbatim
import sgpp.*
@endverbatim
in MATLAB to not have to write @c sgpp. in front of every method.
- Call
@verbatim
methods('sgpp.Classname')
@endverbatim
to see all methods of the class "Classname" (e.g. @c Grid).
- The methods itself can also be called like Java methods in the MATLAB
command window, e.g.:
@verbatim
dataVector = sgpp.DataVector(10)
dataVector.setAll(0)
dataVector
@endverbatim