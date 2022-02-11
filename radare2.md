radare2
=======

RAnalPlugin
-----------

```cpp
// from libr/include/r_anal.h

typedef struct r_anal_plugin_t {
	char *name;
	char *desc;
	char *license;
	char *arch;
	char *author;
	char *version;
	int bits;
	int esil; // can do esil or not
	int fileformat_type;
	int (*init)(void *user);
	int (*fini)(void *user);
	//int (*reset_counter) (RAnal *anal, ut64 start_addr);
	int (*archinfo)(RAnal *anal, int query);
	ut8* (*anal_mask)(RAnal *anal, int size, const ut8 *data, ut64 at);
	RList* (*preludes)(RAnal *anal);

	// legacy r_anal_functions
	RAnalOpCallback op;

	// command extension to directly call any analysis functions
	RAnalCmdExt cmd_ext;

	RAnalRegProfCallback set_reg_profile;
	RAnalRegProfGetCallback get_reg_profile;
	RAnalFPBBCallback fingerprint_bb;
	RAnalFPFcnCallback fingerprint_fcn;
	RAnalDiffBBCallback diff_bb;
	RAnalDiffFcnCallback diff_fcn;
	RAnalDiffEvalCallback diff_eval;

	RAnalEsilCB esil_init; // initialize esil-related stuff
	RAnalEsilLoopCB esil_post_loop;	//cycle-counting, firing interrupts, ...
	RAnalEsilTrapCB esil_trap; // traps / exceptions
	RAnalEsilCB esil_fini; // deinitialize
} RAnalPlugin;

```

### By Example

https://radare.gitbooks.io/radare2book/content/plugins/dev-anal.html

```cpp
RAnalPlugin r_anal_plugin_avr = {
	.name = "avr",
	.desc = "AVR code analysis plugin",
	.license = "LGPL3",
	.arch = "avr",
	.esil = true,
	.archinfo = archinfo,
	.bits = 8 | 16, // 24 big regs conflicts
	.op = &avr_op,
	.set_reg_profile = &set_reg_profile,
	.esil_init = esil_avr_init,
	.esil_fini = esil_avr_fini,
	.anal_mask = anal_mask_avr,
};
```

`.op` scans opcode and builds `RAnalOp`. Its type: `RAnalOpCallback`

```cpp
typedef int (*RAnalOpCallback)(RAnal *a, RAnalOp *op, ut64 addr, const ut8 *data, int len, RAnalOpMask mask);
```

`.esil = true` means that it produces
[ESIL](https://radare.gitbooks.io/radare2book/content/disassembling/esil.html),
a stack-based language used for emulation. `.set_reg_profile` is required for
ESIL to work.

