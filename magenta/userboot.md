Starting the init process.

The userboot image is embedded in the kernel the same way as VDSO.
The start address of the ELF image is ```userboot_image``` defined in ```kernel/lib/userboot/userboot-image.S```. For more information see VDSO discussion https://github.com/slavaim/riscv-notes/blob/master/magenta/VDSO.md .

The ```libuserboot.so``` library is included in the kernel image through ```kernel/lib/userboot/userboot-image.S``` with a generated ```build-magenta-pc-x86-64/kernel/lib/userboot/userboot-code.h```

```
#define USERBOOT_FILENAME "./build-magenta-pc-x86-64/system/core/userboot/libuserboot.so"
#define USERBOOT_DATA_START_dynsym 0x0000000000000298
#define USERBOOT_DATA_END_dynsym 0x00000000000002b0
#define USERBOOT_CODE_START 0x0000000000003000
#define USERBOOT_ENTRY 0x0000000000003a40
#define USERBOOT_ENTRY_SIZE 0x00000000000009df
#define USERBOOT_CODE_END 0x000000000000d000
```

The init process address space initialisation.

```
#2  0xffffffff80004690 in arch_mmu_init_aspace (aspace=0xffffffff81168310, base=16777216, size=274861125632, flags=0) at kernel/arch/riscv/mmu.cpp:73
#3  0xffffffff8004bb64 in VmAspace::Init (this=0xffffffff811681c0) at kernel/kernel/vm/vm_aspace.cpp:142
#4  0xffffffff8004bdb0 in VmAspace::Create (flags=0, name=0x0) at kernel/kernel/vm/vm_aspace.cpp:180
#5  0xffffffff8008f7e0 in ProcessDispatcher::Initialize (this=0xffffffff81167eb0) at kernel/lib/magenta/process_dispatcher.cpp:147
#6  0xffffffff8008ef30 in ProcessDispatcher::Create (job=..., name=..., flags=0, dispatcher=0xffffffff81146e48, rights=0xffffffff81146e3c, root_vmar_disp=0xffffffff81146e40, 
    root_vmar_rights=0xffffffff81146e38) at kernel/lib/magenta/process_dispatcher.cpp:73
#7  0xffffffff80032b84 in attempt_userboot () at kernel/lib/userboot/userboot.cpp:283
#8  0xffffffff800330ec in userboot_init (level=720895) at kernel/lib/userboot/userboot.cpp:357
#9  0xffffffff80005da8 in lk_init_level (required_flag=LK_INIT_FLAG_PRIMARY_CPU, start_level=655360, stop_level=720895) at kernel/top/init.c:86
#10 0xffffffff80005e4c in lk_primary_cpu_init_level (start_level=655360, stop_level=720895) at kernel/include/lk/init.h:51
#11 0xffffffff800060fc in bootstrap2 (arg=0x0) at kernel/top/main.c:136
#12 0xffffffff8000a78c in initial_thread_func () at kernel/kernel/thread.c:84
#13 0xffffffff8000a74c in init_thread_struct (t=0xffffffff81144be0, name=0x0) at kernel/kernel/thread.c:72
```

VDSO allocation and mapping:
```
#0  0xffffffff8003f3a4 in VmAddressRegion::CreateSubVmarInternal (this=0xffffffff81168538, offset=18446744071580183472, size=18446744071580183440, align_pow2=255 '\377', vmar_flags=2165599120, vmo=..., 
    vmo_offset=18446744071580183472, arch_mmu_flags=4294967295, name=0xffffffff80103bd0 "vDSO constants", out=0xffffffff81146c70)
#1  0xffffffff8003fd40 in VmAddressRegion::CreateVmMapping (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, mapping_offset=0, size=4096, align_pow2=0 '\000', 
    vmar_flags=48, vmo=..., vmo_offset=12288, arch_mmu_flags=24, name=0xffffffff80103bd0 "vDSO constants", out=0xffffffff81146d50) at kernel/kernel/vm/vm_address_region.cpp:268
#2  0xffffffff800afcd4 in (anonymous namespace)::KernelVmoWindow<vdso_constants>::KernelVmoWindow (this=0xffffffff81146d50, name=0xffffffff80103bd0 "vDSO constants", vmo=..., offset=13632)
    at kernel/lib/vdso/vdso.cpp:46
#3  0xffffffff800aee64 in VDso::Create () at kernel/lib/vdso/vdso.cpp:226
#4  0xffffffff80032c90 in attempt_userboot () at kernel/lib/userboot/userboot.cpp:294
#5  0xffffffff800330ec in userboot_init (level=720895) at kernel/lib/userboot/userboot.cpp:357
#6  0xffffffff80005da8 in lk_init_level (required_flag=LK_INIT_FLAG_PRIMARY_CPU, start_level=655360, stop_level=720895) at kernel/top/init.c:86
#7  0xffffffff80005e4c in lk_primary_cpu_init_level (start_level=655360, stop_level=720895) at kernel/include/lk/init.h:51
#8  0xffffffff800060fc in bootstrap2 (arg=0x0) at kernel/top/main.c:136
#9  0xffffffff8000a78c in initial_thread_func () at kernel/kernel/thread.c:84
#10 0xffffffff8000a74c in init_thread_struct (t=0xffffffff81144be0, name=0x0) at kernel/kernel/thread.c:72
```

Traversing VA tree to locate a gap to map VDSO:
```
#0  mxtl::WAVLTree<unsigned long, mxtl::RefPtr<VmAddressRegionOrMapping>, mxtl::DefaultKeyedObjectTraits<unsigned long, VmAddressRegionOrMapping>, VmAddressRegionOrMapping::WAVLTreeTraits, mxtl::tests::intrusive_containers::DefaultWAVLTreeObserver>::sentinel (this=0xffffffff80154780 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar+104>) at system/ulib/mxtl/include/mxtl/intrusive_wavl_tree.h:948
#1  0xffffffff800441a4 in mxtl::WAVLTree<unsigned long, mxtl::RefPtr<VmAddressRegionOrMapping>, mxtl::DefaultKeyedObjectTraits<unsigned long, VmAddressRegionOrMapping>, VmAddressRegionOrMapping::WAVLTreeTraits, mxtl::tests::intrusive_containers::DefaultWAVLTreeObserver>::end (this=0xffffffff80154780 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar+104>)
    at system/ulib/mxtl/include/mxtl/intrusive_wavl_tree.h:179
#2  0xffffffff800427e8 in VmAddressRegion::LinearRegionAllocatorLocked (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, size=4096, align_pow2=12 '\f', 
    arch_mmu_flags=24, spot=0xffffffff81146ba0) at kernel/kernel/vm/vm_address_region.cpp:765
#3  0xffffffff80041254 in VmAddressRegion::AllocSpotLocked (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, size=4096, align_pow2=0 '\000', arch_mmu_flags=24, 
    spot=0xffffffff81146ba0) at kernel/kernel/vm/vm_address_region.cpp:522
#4  0xffffffff8003f6e4 in VmAddressRegion::CreateSubVmarInternal (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, offset=0, size=4096, align_pow2=0 '\000', 
    vmar_flags=48, vmo=..., vmo_offset=12288, arch_mmu_flags=24, name=0xffffffff80103bd0 "vDSO constants", out=0xffffffff81146c70) at kernel/kernel/vm/vm_address_region.cpp:150
#5  0xffffffff8003fd40 in VmAddressRegion::CreateVmMapping (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, mapping_offset=0, size=4096, align_pow2=0 '\000', 
    vmar_flags=48, vmo=..., vmo_offset=12288, arch_mmu_flags=24, name=0xffffffff80103bd0 "vDSO constants", out=0xffffffff81146d50) at kernel/kernel/vm/vm_address_region.cpp:268
#6  0xffffffff800afcd4 in (anonymous namespace)::KernelVmoWindow<vdso_constants>::KernelVmoWindow (this=0xffffffff81146d50, name=0xffffffff80103bd0 "vDSO constants", vmo=..., offset=13632)
    at kernel/lib/vdso/vdso.cpp:46
#7  0xffffffff800aee64 in VDso::Create () at kernel/lib/vdso/vdso.cpp:226
#8  0xffffffff80032c90 in attempt_userboot () at kernel/lib/userboot/userboot.cpp:294
#9  0xffffffff800330ec in userboot_init (level=720895) at kernel/lib/userboot/userboot.cpp:357
#10 0xffffffff80005da8 in lk_init_level (required_flag=LK_INIT_FLAG_PRIMARY_CPU, start_level=655360, stop_level=720895) at kernel/top/init.c:86
#11 0xffffffff80005e4c in lk_primary_cpu_init_level (start_level=655360, stop_level=720895) at kernel/include/lk/init.h:51
#12 0xffffffff800060fc in bootstrap2 (arg=0x0) at kernel/top/main.c:136
#13 0xffffffff8000a78c in initial_thread_func () at kernel/kernel/thread.c:84
#14 0xffffffff8000a74c in init_thread_struct (t=0xffffffff81144be0, name=0x0) at kernel/kernel/thread.c:72
```

A gap region is found:
```
#0  0xffffffff8004103c in VmAddressRegion::CheckGapLocked (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, prev=..., next=..., pva=0xffffffff81146ba0, search_base=0, 
    align=4096, region_size=4096, min_gap=0, arch_mmu_flags=24) at kernel/kernel/vm/vm_address_region.cpp:486
#1  0xffffffff8004282c in VmAddressRegion::LinearRegionAllocatorLocked (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, size=4096, align_pow2=12 '\f', 
    arch_mmu_flags=24, spot=0xffffffff81146ba0) at kernel/kernel/vm/vm_address_region.cpp:769
#2  0xffffffff80041254 in VmAddressRegion::AllocSpotLocked (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, size=4096, align_pow2=0 '\000', arch_mmu_flags=24, 
    spot=0xffffffff81146ba0) at kernel/kernel/vm/vm_address_region.cpp:522
#3  0xffffffff8003f6e4 in VmAddressRegion::CreateSubVmarInternal (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, offset=0, size=4096, align_pow2=0 '\000', 
    vmar_flags=48, vmo=..., vmo_offset=12288, arch_mmu_flags=24, name=0xffffffff80103bd0 "vDSO constants", out=0xffffffff81146c70) at kernel/kernel/vm/vm_address_region.cpp:150
#4  0xffffffff8003fd40 in VmAddressRegion::CreateVmMapping (this=0xffffffff80154718 <VmAspace::KernelAspaceInitPreHeap()::_kernel_root_vmar>, mapping_offset=0, size=4096, align_pow2=0 '\000', 
    vmar_flags=48, vmo=..., vmo_offset=12288, arch_mmu_flags=24, name=0xffffffff80103bd0 "vDSO constants", out=0xffffffff81146d50) at kernel/kernel/vm/vm_address_region.cpp:268
#5  0xffffffff800afcd4 in (anonymous namespace)::KernelVmoWindow<vdso_constants>::KernelVmoWindow (this=0xffffffff81146d50, name=0xffffffff80103bd0 "vDSO constants", vmo=..., offset=13632)
    at kernel/lib/vdso/vdso.cpp:46
#6  0xffffffff800aee64 in VDso::Create () at kernel/lib/vdso/vdso.cpp:226
#7  0xffffffff80032c90 in attempt_userboot () at kernel/lib/userboot/userboot.cpp:294
#8  0xffffffff800330ec in userboot_init (level=720895) at kernel/lib/userboot/userboot.cpp:357
#9  0xffffffff80005da8 in lk_init_level (required_flag=LK_INIT_FLAG_PRIMARY_CPU, start_level=655360, stop_level=720895) at kernel/top/init.c:86
#10 0xffffffff80005e4c in lk_primary_cpu_init_level (start_level=655360, stop_level=720895) at kernel/include/lk/init.h:51
#11 0xffffffff800060fc in bootstrap2 (arg=0x0) at kernel/top/main.c:136
#12 0xffffffff8000a78c in initial_thread_func () at kernel/kernel/thread.c:84
#13 0xffffffff8000a74c in init_thread_struct (t=0xffffffff81144be0, name=0x0) at kernel/kernel/thread.c:72

(gdb) p/x real_gap_beg
$63 = 0xffffffc002e01000
(gdb) p/x real_gap_end
$64 = 0xffffffff7fffffff
```

Switching address space to a user process

```
#3  0xffffffff80006108 in arch_mmu_context_switch (old_aspace=0x0, aspace=0xffffffff81169510) at kernel/arch/riscv/mmu.cpp:713
#4  0xffffffff80067ac4 in vmm_context_switch (oldspace=0x0, newaspace=0xffffffff811693c0) at kernel/kernel/vm/vmm.cpp:35
#5  0xffffffff80067b00 in vmm_context_switch (oldspace=0x0, newaspace=0xffffffff811693c0) at kernel/kernel/vm/vmm.cpp:40
#6  0xffffffff8000cb90 in _thread_resched_internal () at kernel/kernel/thread.c:809
#7  0xffffffff8000a58c in sched_reschedule () at kernel/kernel/sched.c:340
#8  0xffffffff8000b81c in thread_resume (t=0xffffffff8116a7b8) at kernel/kernel/thread.c:295
#9  0xffffffff800a0ad0 in UserThread::Start (this=0xffffffff8116a560, entry=176882478496, sp=123530141696, arg1=786048145, arg2=176882556928, initial_thread=true)
    at kernel/lib/magenta/user_thread.cpp:279
#10 0xffffffff8002e920 in ThreadDispatcher::Start (this=0xffffffff8116a208, pc=176882478496, sp=123530141696, arg1=786048145, arg2=176882556928, initial_thread=true)
    at kernel/lib/magenta/include/magenta/thread_dispatcher.h:26
#11 0xffffffff8003458c in attempt_userboot () at kernel/lib/userboot/userboot.cpp:339
#12 0xffffffff80034678 in userboot_init (level=720895) at kernel/lib/userboot/userboot.cpp:362
#13 0xffffffff80006b1c in lk_init_level (required_flag=LK_INIT_FLAG_PRIMARY_CPU, start_level=655360, stop_level=720895) at kernel/top/init.c:86
#14 0xffffffff80006b74 in lk_primary_cpu_init_level (start_level=655360, stop_level=720895) at kernel/include/lk/init.h:51
#15 0xffffffff80006d74 in bootstrap2 (arg=0x0) at kernel/top/main.c:136
#16 0xffffffff8000b2b8 in initial_thread_func () at kernel/kernel/thread.c:84
#17 0xffffffff8000b278 in init_thread_struct (t=0xffffffff8114ac60, name=0x0) at kernel/kernel/thread.c:72
```
