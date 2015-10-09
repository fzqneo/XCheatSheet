#OProfile

**OProfile** is a sampling-based tool.

##*operf* --- the recommended tool

`operf` is the newer and recommended command to do profiling --- compared with `opcontrol` --- the legacy and deprecated tool.

```bash
operf --events=LLC_MISSES:6000:0x41,LLC_REFS:6000:0x4f ./myapp
```

The sample data is automatically saved in the default session directory. Callgraph can be collected with the `--callgraph` option. When callgraph is enabled, the count must be at least 15X the minimal count.

Available event types, minimal counts and possible unit masks can be displayed with:
```bash
ophelp
```

##*opreport* --- displaying profile result

Running `opreport` automatically displays the profile from last run of `operf`. By default it is per-image. It displays per-symbol with:
```bash
opreport -l
```

If callgraph is collected, it can be reported with `opreport ---callgraph`.

##*opannotate* --- annotating source file
```bash
opannotate --source --assembly
opannotate --source --output-dir=annotated
```
