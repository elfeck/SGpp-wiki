The easiest way to use SG⁺⁺ are the debian packages we provide with every [release](https://github.com/SGpp/SGpp/releases).

SG⁺⁺ is developed on Linux, which is the officially recommended OS. It can be installed and used on OSX and Windows, although currently we are not able to offer proper user support for those platforms.

To compile and customize SG⁺⁺ select your operating system:

* [Linux (GCC/Clang/ICC)](https://github.com/SGpp/SGpp/wiki/Linux-(GCC-Clang-ICC)) [recommended]
* [OSX (GCC/ICC)](https://github.com/SGpp/SGpp/wiki/OSX-(GCC-ICC)) 
* [Windows (MinGW)](https://github.com/SGpp/SGpp/wiki/Windows-(MinGW))

SG⁺⁺ could also build with other OS/compiler combinations that are not listed here. 


If you want to use SG⁺⁺ with MATLAB, we recommend to use the binaries provided in the [releases](https://github.com/SGpp/SGpp/releases) (see installation instructions on the [MATLAB binaries](https://github.com/SGpp/SGpp/wiki/MATLAB-binaries) page). There are also other options, if you do not want or are not able to use the binaries (in this case, proceed as explained on the page for your platform).

Instructions for setting up Eclipse IDE for SG⁺⁺ can be found [here](https://github.com/SGpp/SGpp/wiki/Eclipse-setup).

There are also example repositories available, showing how to integrate SG⁺⁺ into your own project using git submodules and CMake (or SCons). These work without a prior installation of SG⁺⁺ and instead automatically build and include it in the current project only. The example with CMake can be found [here](https://github.com/SGpp/SGpp_Example_Application_CMake), the one using SCons [here](https://github.com/SGpp/SGpp_Example_Application_CMake).