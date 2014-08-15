#Valgrind

##Callgrind

[Official Manual](http://valgrind.org/docs/manual/cl-manual.html)

Callgrind generates a **call-graph**, showing caller-callee relationships, counting functions calls and counting Ir (instruction references) (by default). It can also simulate cache misses and branch predictions, by setting command line flags.

```bash
valgrind --tool=callgrind ./myapp
```

It will generate a profile file like *callgrind.out.4990*. The session number may change.

###Profiling only selected piece of code

```cpp
#include  <valgrind/callgrind.h>

//code before

CALLGRIND_START_INSTRUMENTATION;
//the meat
//...
CALLGRIND_STOP_INSTRUMENTATION;
CALLGRIND_DUMP_STATS;

//code after
```

On the command line, start profiling with

```bash
 valgrind --tool=callgrind --instr-atstart=no ./myapp
 ```
