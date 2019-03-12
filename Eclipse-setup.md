# Basic eclipse installation
* Download and extract Eclipse IDE for C/C++ Developers from https://www.eclipse.org/downloads/eclipse-packages/
* Start eclipse executable

# Setup SG++ Project
* File->Import... Git->Projects from Git

 ![Import__014](/uploads/6369200f71aac2852f5ac19295fecdce/Import__014.png)
* Select "Clone URI"
* Enter URI (SSH or HTPPS) and authetication details from [this projects main page](https://simsgs.informatik.uni-stuttgart.de:8444/SGpp/SGpp)
* Select branches to clone locally
* Choose destination folder (preferably inside current workspace)
* Wait for clone operation to finish, select "Import as general project" and finish
* Rightclick newly created Project->New->Convert to a C/C++ Project (Adds C/C++ nature)
* Select "Makefile project" and "Linux GCC" as toolchain

![Convert_to_a_C-C___Project__018](/uploads/5808d4043c74e794ddfe6787320aa56d/Convert_to_a_C-C___Project__018.png)

# Fix C++11 header indexing
* Rightclick Project->Properties ->C/C++ General->Preprocessor Include Paths, Macros etc.->Providers (Tab)
* Select _only_ CDT User Settings and CDT GCC Built-in Compiler Settings
* Move the latter to the top of the list
* Add "-std=c++11" to the "Command to get compiler specs" field

![Properties_for_SGpp__019](/uploads/2c3ff36e319b9e2f0243bd8ae564bd28/Properties_for_SGpp__019.png)
* Rightclick Project->Index->Rebuild

# Configure SCons build 
Either overwrite Eclipse build or install SConsolidator plugin

## Overwrite Eclipse Build
* Rightclick Project->C/C++ Build
* Deselect "Use default build command" and enter "scons"
* Switch to "Behaviour" tab
* Enter scons parameters (e.g: "-j4 SG_ALL=0 SG_BASE=1") in "Build (Incremental build)"
* Enter "-c" in Clean

![Properties_for_SGpp__020](/uploads/d8e4cf52e9b4704f61e8915681a736a3/Properties_for_SGpp__020.png)
* Optionally add different build configurations

## Install SConsolidator
* SConsolidator is an Eclipse plugin that Introduces native support for SCons scripts in Eclipse and provides nice features like dependency visualization 
* Install SConsolidator from update page as proposed in [Sconsolidator Wiki](http://www.sconsolidator.com/projects/sconsolidator/wiki/Installation)
* Restart Eclipse
* Right click on your Project > SCons > Use self-provided SCons build
* Add your desired SCons options e.g. ("SG_ALL=0 SG_BASE=1"). DO NOT specify parallel building. (e.g. "-j 4"). This is done automatically by SConsolidator.
* SConsolidator rebuilds the project 
* Build will from now on automatically use Sconsolidator. 

NOTE: Sometimes "Converting projects to SCons projects" hangs. I don't know why. Then use the above method overriding Eclipse build.

# Configure Code Styling
* CppStyle Eclipse Plugin: fixes the style of your C++ code and prevents style warnings/errors on jenkins
  * Install CppStyle from Eclipse Marketplace
  * Restart eclipse
  * Rightclick Project->Preferences -> C/C++ -> CppStyle and set the corresponding
     * clang format path to '<wherever your clang-format is located>' (/usr/bin/clang-format by default)
     * cpplint path to "<SGpp home>/tools/cpplint.py"
  * Check "enable cpplint" and "run clang-format on file save"
