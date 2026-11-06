# FileBox patch notes

Upstream: https://github.com/OlehKulykov/PLzmaSDK

Base commit: `85e47662833ef0e368ba5323becd32d3427aa0c9` (1.6.2).

## ARM64 assembly debug-map warnings

`src/Asm/arm64/LzmaDecOpt.S` defines absolute assembly constants with `.equ`.
Xcode combines the Swift package's object files with `ld -r`, which retains
debug-map entries for these constants. The final app link reports 59 warnings
of the form `could not associate debug note to atom kStartOffset`.

The patch prefixes the file-local constants and their references with `Lplzma_`.
The Mach-O assembler treats `L`-prefixed names as temporary local symbols, so
these constants no longer enter the intermediate debug map. The exported
decoder symbol, instructions, constants' values, package products and original
licenses are unchanged.

Validation with Apple clang 21.0.0 / ld-27037.1:

- Reproduced all 59 warnings with a two-stage link (`ld -r`, then executable).
- The same link produces zero warnings with the patch, for both iOS ARM64 and
  ARM64 iOS Simulator targets.
- The original and patched `__text` sections are byte-for-byte identical:
  4,964 bytes, SHA-256
  `8071143a176d28465a41a9b818e78e8219421127d3531068be6c732d7c4d8053`.

FileBox pins the resulting fork commit. When updating the SDK, re-check this
patch against upstream and repeat the two-stage link and instruction comparison.
