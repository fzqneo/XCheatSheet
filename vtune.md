#VTune

[ucsd]:http://cseweb.ucsd.edu/classes/wi14/cse141/project/VTUNE_PC_P.pdf

[intel]:https://software.intel.com/sites/default/files/managed/c0/a8/hotspots_amplxe_lin.pdf

**Reference:** 

+ [from UCSD][ucsd] 
+ [from Intel][intel]

##Install

Download the source from Intel's website, extract and install. Quick steps see [ucsd]

##Launch preparation

Every time before using VTune, execute the following:

```
source /opt/intel/vtune_amplifier_xe_2013/amplxe-vars.sh
echo 0 | sudo tee /proc/sys/kernel/nmi_watchdog
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

Note: the 1st line sets some environment variables. The 2nd line is necessary for general exploration. The 3nd is necessary for basic hotspot analysis.

Launch VTune with (remember to run the preparation commands beforehand):
```
amplxe-gui
```

##Basic hotspot analysis

More details in [intel]. Quick steps:

Profiling a **selected code section** --- insert in your source:

```
#include	<ittnotify.h>
…
__itt_resume();
//your code to be profiled
…
__itt_pause();
...
```

When compiling your program:
```
gcc -O3 -g -I/opt/intel/vtune_amplifier_xe/include/ -L/opt/intel/vtune_amplifier_xe/lib64/ app.c -littnotify -ldl -o app
```

When using VTune, select **Start pause** in VTune. Profiling is disabled when you start the program, until it encounters `__itt_resume()` in the source, and pauses with `__itt_pause()`.
