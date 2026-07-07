# ptxas 13.3 (C7907) internal compiler error on `red.async.release.sys` with an mbarrier operand

**Toolchain:** `ptxas` **V13.3.73** (CUDA 13.3; `pip install nvidia-cuda-nvcc==13.3.73`), PTX ISA 9.3.
**Affects:** `sm_100a` (datacenter) **and** `sm_121a` (consumer) — reproduced on both.

## Summary
```
red.async.release.sys.global.add.u32 [addr], val, [mbar];
```
A sys-scope `red.async` with an mbarrier completion operand crashes `ptxas` with
`(C7907) Internal compiler error` (a hard crash / segfault) instead of assembling
or emitting a clean diagnostic. **The identical instruction without the mbarrier
operand assembles fine**, so the crash is specific to the mbarrier-operand form.

## Reproduce
```
ptxas -arch=sm_100a red_async_ice.ptx -o /dev/null    # -> (C7907) Internal compiler error
ptxas -arch=sm_100a red_async_ok.ptx  -o /dev/null    # -> assembles cleanly (control)
ptxas -arch=sm_121a red_async_ice.ptx -o /dev/null    # -> same ICE on consumer
```

## Observed — see `observed_error.txt`
```
ptxas fatal   : (C7907) Internal compiler error.
ptxas fatal   : Ptx assembly aborted due to errors
```

## Files
- `red_async_ice.ptx` — the crashing input (mbarrier-operand form)
- `red_async_ok.ptx` — the no-mbarrier control that assembles
- `observed_error.txt` — the crash output
