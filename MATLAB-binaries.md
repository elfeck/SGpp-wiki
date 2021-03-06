This page explains how to install and use the binaries we provide for
use with MATLAB as part of every SGpp [release](https://github.com/SGpp/SGpp/releases).
# Linux
1. Extract the archive to an arbitrary location, say
<tt>/PATH_TO_BINARIES</tt>.
2. Start MATLAB with the following environment variables:
```console
export LD_LIBRARY_PATH="/PATH_TO_BINARIES:$LD_LIBRARY_PATH"
export LD_PRELOAD="/usr/lib/x86_64-linux-gnu/libstdc++.so.6:$LD_PRELOAD"
matlab
```
3. Add <tt>/PATH_TO_BINARIES</tt> to the MATLAB search path, either by
changing to that directory or using <tt>pathtool</tt>.
4. Check if it works via <tt>sgpp.Grid.createLinearGrid(2)</tt>.
# Windows
1. Extract the archive to an arbitrary location, say
<tt>C:\\PATH_TO_BINARIES</tt>.
2. Start MATLAB.
3. Add <tt>C:\\PATH_TO_BINARIES</tt> to the MATLAB search path, either by
changing to that directory or using <tt>pathtool</tt>.
4. Check if it works via <tt>sgpp.Grid.createLinearGrid(2)</tt>.

