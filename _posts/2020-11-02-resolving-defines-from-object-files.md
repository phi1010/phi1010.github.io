---
layout: post
title: Resolving Defines from Object Files
subtitle: How to get to the actual value of recursive defines
tags: [binary, dwarf, c]
---

During the analysis of OP-TEE, many C-defines were defined using other defined constants, scattered across many header files, makefiles and other data specified by the build system.
This made the analysis difficult, requiring validation of the results using a disassembler, to avoid 
To quickly resolve the value of such defines, one may use the debug output attached to object files to re-evaluate those defines with gcc.

This script is not engineered with security in mind, please do not use it on untrusted object files or with untrusted macro input parameter.

This script probably does the job for you, if your object file was not build using varying definitions of the same define:

```py
#!/usr/bin/env python3
# Usage: ./undefine.py "path/to/object/file.o" "MACRO"
# Or: ./undefine.py "path/to/object/file.o" "MACRO(param1, param2)"
import os
import sys
from subprocess import run, PIPE
from re import fullmatch
import tempfile

OBJFILE = sys.argv[1]
QUESTION = sys.argv[2]
GCC = "aarch64-linux-gnu-gcc"
OBJDUMP = "objdump"

dwarf = run([OBJDUMP,OBJFILE,"--dwarf"],capture_output=True,check=True,encoding="utf8").stdout
with tempfile.NamedTemporaryFile(mode="w",encoding="utf8",suffix=".h") as header:
    for line in dwarf.splitlines():
        match = fullmatch(r"^ DW_MACRO_define_strp - lineno : \d+ macro : (.*)$",line)
        if match: header.write("#define "+match.groups(1)[0]+"\n")
    header.write("#ifdef "+QUESTION+"\n")
    header.write('"Search term was defined."\n')
    header.write("#else\n")
    header.write('"Search term was not defined."\n')
    header.write("#endif\n")
    header.write(QUESTION+"\n")
    header.flush()
    print(header.name)
    result = run([GCC, "-E",header.name],stdout=PIPE,check=True,encoding="utf8").stdout
    print(result)
```


