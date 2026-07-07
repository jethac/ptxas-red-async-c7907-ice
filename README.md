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

## Prior art — C7907 is a known Blackwell ptxas ICE class
The **(C7907) internal compiler error is already reported** on Blackwell/SM100 by
others, all with high-register-pressure Triton/kernel triggers:
- NVIDIA/numba-cuda#725 — https://github.com/NVIDIA/numba-cuda/issues/725 (nvJitLink 13.1 + sm120)
- state-spaces/mamba#904 — https://github.com/state-spaces/mamba/issues/904 (Mamba3 bwd, GB200)
- triton-lang/triton#9933 — https://github.com/triton-lang/triton/issues/9933 (SM100, autotuner configs eliminated)

**What this repo adds:** a *minimal single-instruction* trigger for C7907 — one
`red.async` line, no register pressure, no Triton — which should make the ptxas
codegen path far easier to isolate than the existing large-kernel repros.

## Files
- `red_async_ice.ptx` — the crashing input (mbarrier-operand form)
- `red_async_ok.ptx` — the no-mbarrier control that assembles
- `observed_error.txt` — the crash output
