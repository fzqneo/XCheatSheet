#VTune

[1]:http://cseweb.ucsd.edu/classes/wi14/cse141/project/VTUNE_PC_P.pdf

[2]:https://software.intel.com/sites/default/files/managed/c0/a8/hotspots_amplxe_lin.pdf

**Reference:** 

+ [from UCSD][1] 
+ [from Intel][2]

##Install

Download the source from Intel's website, extract and install. Quick steps see [1]

##Launch

Every time before using VTune, execute the following:

```
source /opt/intel/vtune_amplifier_xe_2013/amplxe-vars.sh
echo 0 | sudo tee /proc/sys/kernel/nmi_watchdog
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

Note: the 1st line sets some environment variables. The 2nd line is necessary for general exploration. The 3nd is necessary for basic hotspot analysis.
