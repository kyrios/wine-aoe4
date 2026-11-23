# Age of Empires II: Definitive Edition with the soft-fault path

Since its 2 June 2026 patch, AoE2 DE (`AoE2DE_s.exe`, Steam app 813780) shows the same slowdown under Rosetta as AoE IV: the game raises processor exceptions many thousands of times per second and handles each one itself, and every one takes the slow route through macOS and Rosetta. In busy scenes that is about 15,000 per second, and FPS collapses with the number of units that have health bars or are selected.

The engine needs **no code change** for AoE2: the generic `X87_COOP_EXE` path in `__wine_main` plus environment variables is enough.

## Environment

```sh
export WINELOADER=<engine>/bin/wine
export AOELAB_SIDECAR_PATH=<path>/x87sidecar     # NerRobDog/x87sidecar >= 4b6bad8
export X87_COOP_EXE=AoE2DE_s.exe
export AOELAB_SOFTFAULT_BRIDGE=1                 # maps the engine's bridge page
export AOE_SOFTFAULT_TARGET1=0x180000000
export AOE_SOFTFAULT_TARGET=0x180000010
export X87_ALWAYS_NONE=1
# .text of AoE2DE_s.exe; read it from the PE header, it changes with game updates.
# 101.103.57257.0 (sha256 1e415af8b7fbbe6aeb4e1eebf888f43e9d9b8b9129df2bd010d0a7a60dc3ca16):
export AOE_SOFTFAULT_PC=0x140001000
export AOE_SOFTFAULT_END=0x143195ef2
```

Do **not** set `AOELAB_CODE_CACHE_GAME`: the code cache is specific to AoE IV and is gated to that executable anyway.

## Results

Measured with an equivalent port of the same soft-fault code into the stock CrossOver 26.3.0 ntdll. It was **not** run on this engine build. Patch and notes: https://github.com/kyrios/TrebuchetApp/tree/main/fixes/wine-fast-exception-delivery-aoe2

Setup: M2 Max, macOS 27.2.

| | Stock | Soft-fault |
|---|---|---|
| Test program, cost per exception | 53.5 µs | 0.62 µs |
| Game: exceptions taking the slow route (DTrace) | about 15,000/s | 0 |
| Game: paused busy scene, 60 units selected | about 10–12 FPS | about 22 FPS |

**Remaining cost:** the game threads still spend much of their time in Rosetta's runtime, re-translating code that changes while the game runs.
