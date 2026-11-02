# DKB ROM / Switch 2 NSP bootstrap fix (test build)

This branch is based on the existing PS5 port at commit
40eb286c96ee568b8ac8efab49fd25aeb392bfe8. The pinned Eglantine core
is updated to e96917cd0e68851ec389dd519f03b52adc11d493 to match the
newer core that was tested on the console.

## Evidence from the DKB ROM PS5 test

- Loader identifies Switch 2 NPDM ("NPDM uses the Switch 2 ACID layout").
- System::Load nevertheless logs "Guest counter frequency: 19200000 Hz".
- stderr.log reports exactly 4,294,967,296 bytes (4 GiB) of guest memory.
- After loading, six nx2:gpu 214 allocations of 0xC800000 bytes (200 MiB each)
  fail per game session.
- MainThread later reads NULL and executes at PC 0x4. This is not evidence
  that System::Load itself blocks: boot-trace says "game loaded (status 0)".

## Code changes

- The Switch 2 bootstrap detection now reads NPDM from Program NCA's ExeFS
  inside NSP/NSZ/XCI/XCZ, as well as standalone NCA and loose ExeFS, before
  InitializeKernel().
- The original auto-stub and huge BCn mip-trimming patches are preserved and
  rebased where necessary for the newer Eglantine core.
- The old System::Load diagnostic-only patch is removed from this test branch.
- No game, user settings, saves, firmware, or console installation is changed.

## Verification and remaining uncertainties

GitHub Actions build success is required before deploying. It is not proof
that DKB ROM runs. After building, check that the DKB session reports:
  - "Switch 2 program: using the 12 GiB memory layout"
  - "Guest counter frequency: 31250000 Hz"
  - "EDEN_GUEST_MEMORY ... bytes=21474836480 ..." (20 GiB host guest layout,
    12 GiB application memory + 8 GiB graphics-server reservation in this core)
Then check whether nx2:gpu 214 allocation errors and the null dereference
persist. If the guest still faults, investigate the guest call stack rather
than hiding or bypassing the prefetch abort.

Do not overwrite the installed build without first backing it up. The
current console boot-trace indicates /system_ex/app/PPSA98515 is used, so a
change to /data/homebrew/PPSA98515 alone may not affect the active version.
