---
layout: post
title: Analyzing Makefiles
subtitle: How to dump makefile variable definitions
tags: [make, debug]
---

OP-TEE uses a very extensive Makefile infrastructure with recurive invocations of make. A Make debugger with breakpoint would have been useful. To at least allow inspecting the state of the variables within Make, this script can be used:

```make
some-target:
	$(eval $@_MAKEFILE_ENV_FILE = $(abspath $(firstword $(MAKEFILE_LIST))).$(subst /,_,$@).env)
	$(eval $@_MAKEFILE_VARS_FILE = $(abspath $(firstword $(MAKEFILE_LIST))).$(subst /,_,$@).vars)
	env > $($@_MAKEFILE_ENV_FILE)
	$(file > $($@_MAKEFILE_VARS_FILE),)
	$(foreach v, $(.VARIABLES), \
		$(file >> $($@_MAKEFILE_VARS_FILE),$(v)) \
		$(file >> $($@_MAKEFILE_VARS_FILE),    := $(value $(v))) \
		$(file >> $($@_MAKEFILE_VARS_FILE),    @ $(origin $(v))) \
		$(file >> $($@_MAKEFILE_VARS_FILE),    == $($(v))) \
		$(file >> $($@_MAKEFILE_VARS_FILE),) \
	)
	some-recipe
```

As a result, the environment and variables of the currently running make invocation for are printed out in a separate file each, beside the original Makefile itself.

Please note that such an expansion of all macros may, depending on the Makefile, cause unexpected side-effects.
