# Gmem
LLVM IR based SFI, Memory Guard for HyperMoon

## Dependencies

best for llvm-10 and clang-10,

other versions may need to edit the Makefiles.

require at least llvm-5.0+, clang-5.0+. 

(not working with llvm-3.x)

## Usage

Insert AndOr checks or IfElse checks to llvm IR for SFI.

As shown in the test, add `-load /path/to/libGmem-pass.so -gmem` in the opt pass to do SFI.

```shell
clang-10 -Wall -g -O3 -emit-llvm -c -I../ -o obj/test.bc test.c
llvm-link-10 obj/test.bc > obj/test-link.bc

opt-10 -load ../obj/libGmem-pass.so -gmem -Gmem-rw=w -Gmem-check-method=ifelseheap -Gmem-verify-external-call-args=false -Gmem-whitelist-section=safe_functions -O3 < obj/test-link.bc > obj/test-opt.bc

clang-10 -o obj/test obj/test-opt.bc -g -O3
```

Detailed declaration is in [Gmem.h](./Gmem.h)

```shell
-Gmem-rw=   
	What type of memory accesses to protect when doing sfi:
        rw          Reads and writes
        r           Reads only
        w           Writes only
-Gmem-check-method=
    What type of ptr check to use when doing sfi:
        andor         And Or Masking
        ifelseheap    If Else Heap Boundary Check Only on Heap
        ifelseall     If Else Boundary Check on Stack and Heap
-Gmem-verify-external-call-args=true/false
    Add checks to all pointer-type arguments to external functions 
	(make sure uninstrumented libraries cannot use invalid pointers)
-Gmem-whitelist-section=stringref
    Functions in this section are allowed access to the safe region
    Initialized to "Gmem_functions".
```

## Note

1. Any function with its name started with `"_Gmem_"` would be automatically inlined.

2. The allocator provided in [Auxiliary.c](./Auxiliary.c) is not indispensable, replace it as you like.

3. You can directly write to `SFI_MASK` and `SFI_START` (for AndOr methods), or `Heap Lower/Upper Bound` and `Stack Lower/Upper Bound` (for private heap and stack).

4. Without optimization(with `-O0` flag set) the `main()` function would fail on Segmentation Fault. This is becasue main() would first assign `argc` and `**argv` , and then the SFI instructions would check them, which would fail.(these ptrs are not in the range, for SFI paras haven't been set yet before allocating)

5. Ispired by [Memsentry](https://github.com/vusec/memsentry)
