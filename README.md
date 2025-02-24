# Compiler for Windows NT PowerPC

This is a fork of [Retro68](https://github.com/autc04/Retro68)'s GCC 9.1, heavily modified to emit asm that the official NTPPC assembler `PAS.EXE` (aka cross-assembler `PASM.EXE` as included in [old VC6-era Windows CE 2.x for PowerPC toolchain](https://archive.org/download/VS6WCE/VS6WCE.ISO/VC60_WINCETK%2FVCCE%2FWCE%2FWCE210%2FBIN%2FPASM.EXE)) can build. Combine with VC++4.x's linker (any later will not work) to make a full toolchain.

## Known bugs

* `-O2` and higher is broken, without disabling problematic optimisations: at least `-fno-align-functions -fno-align-labels -fno-align-jumps -fno-align-loops` 
* No SEH support
* Function prologues + epilogues / stack frames are technically not correct for PPC NT, this only really causes issues with some exceptions in kernel mode leading to `PANIC_STACK_SWITCH` bugcheck (instead of the correct one) due to exception handling related code in NT itself raising an exception
 * C++ support is completely untested.
 * No libgcc, if you need math related functions from there use [arith64.c](https://github.com/glitchub/arith64) (or manually use `Rtl*` functions from ntdll/etc)

There may be other issues.
  
## Building

```
git clone https://github.com/Wack0/peppc
mkdir peppc-build
mkdir peppc-build/gcc-build
mkdir peppc-build/toolchain
cd peppc-build/gcc-build
../../peppc/configure --target=powerpcle-pe-winnt --prefix=/e/peppc-build/toolchain --
enable-languages=c,c++ --disable-libssp --disable-lto --src=../../peppc MAKEINFO=missing
make # add -j as appropriate
# libgcc build will fail, this is expected.
cd gcc
make install
```

## Usage

Compile, assemble, link like so:
```
powerpcle-ppc-winnt-gcc -S -O1 -o - file.c | powerpcle-ppc-winnt-cpp - -P -w -o file.asm # add compiler switches as appropriate
pasm.exe -o file.obj file.asm # ran through wine if needed
link.exe /OUT:file.exe file.obj libs... # ran through wine if needed
```
