---
layout: post
title: Configuring a Raspberry Pi 5
subtitle: Changes to default Raspberry Pi OS
tags: [raspberrypi]
comments: true
---

I got a Raspberry Pi 5 8GB and installed Raspberry Pi OS on an sdcard to try it
out. Here are some of the issues I found.

### Building a kernel

I found issues with the kernel and so followed the [Raspberry Pi building
guide](https://www.raspberrypi.com/documentation/computers/linux_kernel.html)

It is a shame just using an upstream kernel wouldn't work.

### BPF

BTF missing in the kernel. Tools like BCC and `perf trace` use the
[CO-RE](https://nakryiko.com/posts/bpf-core-reference-guide/) to create portable
BPF programs often embedded into the loading program using BPF skeletons.
Fixing the missing kernel BTF is done by adding `CONFIG_DEBUG_INFO_BTF=y`.

The warning message about BTF not being present in the kernel could be better so
some [libbpf patches were
added](https://lore.kernel.org/lkml/20240125231840.1647951-1-irogers@google.com/).

### Additional PMUs?

I was hoping that the Cortex-A76 in the Raspberry Pi 5 would have some PMU
features to play with like Coresight and SPE. Sadly no, but working this out is
some trouble. Something that would have helped prior to investigating this was a
list of the extra features implemented by the Raspberry Pi 5's ARM CPU. I came
across [arm-cpusysregs](https://github.com/lelegard/arm-cpusysregs) that is a
program and kernel module that solves this problem. Its output for the Raspberry
Pi 5 is below:

```shell
FEAT_AA32BF16 ............ no
FEAT_AA32HPD ............. yes
FEAT_AA32I8MM ............ no
FEAT_AArch32 ............. yes
FEAT_ABLE ................ no
FEAT_ADERR ............... no
FEAT_AdvSIMD ............. yes
FEAT_AES ................. yes
FEAT_AFP ................. no
FEAT_AIE ................. no
FEAT_AMUv1 ............... no
FEAT_AMUv1p1 ............. no
FEAT_ANERR ............... no
FEAT_B16B16 .............. no
FEAT_BBM ................. no
FEAT_BF16 ................ no
FEAT_BRBE ................ no
FEAT_BRBEv1p1 ............ no
FEAT_BTI ................. no
FEAT_CCIDX ............... no
FEAT_CLRBHB .............. no
FEAT_CMOW ................ no
FEAT_CONSTPACFIELD ....... no
FEAT_CRC32 ............... yes
FEAT_CSSC ................ no
FEAT_CSV2 ................ yes
FEAT_CSV2_1p1 ............ no
FEAT_CSV2_1p2 ............ no
FEAT_CSV2_2 .............. no
FEAT_CSV2_3 .............. no
FEAT_CSV3 ................ yes
FEAT_D128 ................ no
FEAT_Debugv8p1 ........... yes
FEAT_Debugv8p2 ........... yes
FEAT_Debugv8p4 ........... no
FEAT_Debugv8p8 ........... no
FEAT_Debugv8p9 ........... no
FEAT_DGH ................. no
FEAT_DIT ................. no
FEAT_DotProd ............. yes
FEAT_DoubleFault ......... no
FEAT_DoubleFault2 ........ no
FEAT_DoubleLock .......... yes
FEAT_DPB ................. yes
FEAT_DPB2 ................ no
FEAT_E0PD ................ no
FEAT_EBEP ................ no
FEAT_EBF16 ............... no
FEAT_ECBHB ............... no
FEAT_ECV ................. no
FEAT_EPAC ................ no
FEAT_ETE ................. no
FEAT_ETEv1p1 ............. no
FEAT_ETEv1p2 ............. no
FEAT_ETEv1p3 ............. no
FEAT_ETMv4 ............... no
FEAT_ETMv4p1 ............. no
FEAT_ETMv4p2 ............. no
FEAT_ETMv4p3 ............. no
FEAT_ETMv4p4 ............. no
FEAT_ETMv4p5 ............. no
FEAT_ETMv4p6 ............. no
FEAT_ETS ................. no
FEAT_EVT ................. no
FEAT_ExS ................. no
FEAT_F32MM ............... no
FEAT_F64MM ............... no
FEAT_FCMA ................ no
FEAT_FGT ................. no
FEAT_FGT2 ................ no
FEAT_FHM ................. no
FEAT_FlagM ............... no
FEAT_FlagM2 .............. no
FEAT_FP .................. yes
FEAT_FP16 ................ yes
FEAT_FPAC ................ no
FEAT_FPACCOMBINE ......... no
FEAT_FRINTTS ............. no
FEAT_GCS ................. no
FEAT_GICv3 ............... no
FEAT_GICv4 ............... no
FEAT_GICv4p1 ............. no
FEAT_GTG ................. no
FEAT_HAFDBS .............. yes
FEAT_HAFT ................ no
FEAT_HBC ................. no
FEAT_HCX ................. no
FEAT_HPDS ................ yes
FEAT_HPDS2 ............... yes
FEAT_HPMN0 ............... no
FEAT_I8MM ................ no
FEAT_IDST ................ no
FEAT_IESB ................ yes
FEAT_ITE ................. no
FEAT_JSCVT ............... no
FEAT_LOR ................. yes
FEAT_LPA ................. no
FEAT_LPA2 ................ no
FEAT_LRCPC ............... yes
FEAT_LRCPC2 .............. no
FEAT_LRCPC3 .............. no
FEAT_LS64 ................ no
FEAT_LS64_ACCDATA ........ no
FEAT_LS64_V .............. no
FEAT_LSE ................. yes
FEAT_LSE128 .............. no
FEAT_LSE2 ................ no
FEAT_LSMAOC .............. no
FEAT_LVA ................. no
FEAT_LVA3 ................ no
FEAT_MEC ................. no
FEAT_MOPS ................ no
FEAT_MPAM ................ no
FEAT_MPAMv0p1 ............ no
FEAT_MPAMv1p0 ............ no
FEAT_MPAMv1p1 ............ no
FEAT_MTE ................. no
FEAT_MTE2 ................ no
FEAT_MTE3 ................ no
FEAT_MTE4 ................ no
FEAT_MTE_CANONICAL_TAGS .. no
FEAT_MTE_NO_ADDRESS_TAGS . no
FEAT_MTE_PERM ............ no
FEAT_MTE_STORE_ONLY ...... no
FEAT_MTE_TAGGED_FAR ...... no
FEAT_MTPMU ............... no
FEAT_NMI ................. no
FEAT_nTLBPA .............. no
FEAT_NV .................. no
FEAT_NV2 ................. no
FEAT_PACIMP .............. no
FEAT_PACQARMA3 ........... no
FEAT_PACQARMA5 ........... no
FEAT_PAN ................. yes
FEAT_PAN2 ................ yes
FEAT_PAN3 ................ no
FEAT_PAuth ............... no
FEAT_PAuth2 .............. no
FEAT_PFAR ................ no
FEAT_PMULL ............... yes
FEAT_PMUv3 ............... yes
FEAT_PMUv3_EDGE .......... no
FEAT_PMUv3_ICNTR ......... no
FEAT_PMUv3_SS ............ no
FEAT_PMUv3_TH ............ no
FEAT_PMUv3p1 ............. yes
FEAT_PMUv3p4 ............. no
FEAT_PMUv3p5 ............. no
FEAT_PMUv3p7 ............. no
FEAT_PMUv3p8 ............. no
FEAT_PMUv3p9 ............. no
FEAT_PRFMSLC ............. no
FEAT_RAS ................. yes
FEAT_RASv1p1 ............. no
FEAT_RASv2 ............... no
FEAT_RDM ................. yes
FEAT_RME ................. no
FEAT_RNG ................. no
FEAT_RNG_TRAP ............ no
FEAT_RPRES ............... no
FEAT_RPRFM ............... no
FEAT_S1PIE ............... no
FEAT_S1POE ............... no
FEAT_S2FWB ............... no
FEAT_S2PIE ............... no
FEAT_S2POE ............... no
FEAT_SB .................. no
FEAT_SCTLR2 .............. no
FEAT_SEBEP ............... no
FEAT_SEL2 ................ no
FEAT_SHA1 ................ yes
FEAT_SHA256 .............. yes
FEAT_SHA3 ................ no
FEAT_SHA512 .............. no
FEAT_SM3 ................. no
FEAT_SM4 ................. no
FEAT_SME ................. no
FEAT_SME2 ................ no
FEAT_SME2p1 .............. no
FEAT_SME_F16F16 .......... no
FEAT_SME_F64F64 .......... no
FEAT_SME_FA64 ............ no
FEAT_SME_I16I64 .......... no
FEAT_SPE ................. no
FEAT_SPE_CRR ............. no
FEAT_SPE_FDS ............. no
FEAT_SPECRES ............. no
FEAT_SPECRES2 ............ no
FEAT_SPEv1p1 ............. no
FEAT_SPEv1p2 ............. no
FEAT_SPEv1p3 ............. no
FEAT_SPEv1p4 ............. no
FEAT_SPMU ................ no
FEAT_SSBS ................ yes
FEAT_SSBS2 ............... no
FEAT_SVE ................. no
FEAT_SVE2 ................ no
FEAT_SVE2p1 .............. no
FEAT_SVE_AES ............. no
FEAT_SVE_BitPerm ......... no
FEAT_SVE_PMULL128 ........ no
FEAT_SVE_SHA3 ............ no
FEAT_SVE_SM4 ............. no
FEAT_SYSINSTR128 ......... no
FEAT_SYSREG128 ........... no
FEAT_TCR2 ................ no
FEAT_THE ................. no
FEAT_TIDCP1 .............. no
FEAT_TLBIOS .............. no
FEAT_TLBIRANGE ........... no
FEAT_TME ................. no
FEAT_TRBE ................ no
FEAT_TRF ................. no
FEAT_TTCNP ............... yes
FEAT_TTL ................. no
FEAT_TTST ................ no
FEAT_TWED ................ no
FEAT_UAO ................. yes
FEAT_VHE ................. yes
FEAT_VMID16 .............. yes
FEAT_VPIPT ............... no
FEAT_WFxT ................ no
FEAT_XNX ................. yes
FEAT_XS .................. no
```

The `lscpu` information is always nice to have too:

```shell
Architecture:            aarch64
  CPU op-mode(s):        32-bit, 64-bit
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               ARM
  Model name:            Cortex-A76
    Model:               1
    Thread(s) per core:  1
    Core(s) per cluster: 4
    Socket(s):           -
    Cluster(s):          1
    Stepping:            r4p1
    CPU(s) scaling MHz:  62%
    CPU max MHz:         2400.0000
    CPU min MHz:         1500.0000
    BogoMIPS:            108.00
    Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimd
                         rdm lrcpc dcpop asimddp
Vulnerabilities:         
  Gather data sampling:  Not affected
  Itlb multihit:         Not affected
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Mmio stale data:       Not affected
  Retbleed:              Not affected
  Spec rstack overflow:  Not affected
  Spec store bypass:     Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:            Mitigation; __user pointer sanitization
  Spectre v2:            Mitigation; CSV2, BHB
  Srbds:                 Not affected
  Tsx async abort:       Not affected
```

Another data source is the [ARM Main ID Register
(midr)](https://www.kernel.org/doc/html/v6.4-rc7/arm64/cpu-feature-registers.html)
whose function is similar to the x86 cpuid instruction:
```shell
$ cat /sys/devices/system/cpu/cpu0/regs/identification/midr_el1
0x00000000414fd0b1
```

### Comparison with a Raspberry Pi 400

[arm-cpusysregs](https://github.com/lelegard/arm-cpusysregs) reported features:

```shell
FEAT_AA32BF16 ............ no
FEAT_AA32HPD ............. no
FEAT_AA32I8MM ............ no
FEAT_AArch32 ............. yes
FEAT_ABLE ................ no
FEAT_ADERR ............... no
FEAT_AdvSIMD ............. yes
FEAT_AES ................. no
FEAT_AFP ................. no
FEAT_AIE ................. no
FEAT_AMUv1 ............... no
FEAT_AMUv1p1 ............. no
FEAT_ANERR ............... no
FEAT_B16B16 .............. no
FEAT_BBM ................. no
FEAT_BF16 ................ no
FEAT_BRBE ................ no
FEAT_BRBEv1p1 ............ no
FEAT_BTI ................. no
FEAT_CCIDX ............... no
FEAT_CLRBHB .............. no
FEAT_CMOW ................ no
FEAT_CONSTPACFIELD ....... no
FEAT_CRC32 ............... yes
FEAT_CSSC ................ no
FEAT_CSV2 ................ no
FEAT_CSV2_1p1 ............ no
FEAT_CSV2_1p2 ............ no
FEAT_CSV2_2 .............. no
FEAT_CSV2_3 .............. no
FEAT_CSV3 ................ no
FEAT_D128 ................ no
FEAT_Debugv8p1 ........... no
FEAT_Debugv8p2 ........... no
FEAT_Debugv8p4 ........... no
FEAT_Debugv8p8 ........... no
FEAT_Debugv8p9 ........... no
FEAT_DGH ................. no
FEAT_DIT ................. no
FEAT_DotProd ............. no
FEAT_DoubleFault ......... no
FEAT_DoubleFault2 ........ no
FEAT_DoubleLock .......... yes
FEAT_DPB ................. no
FEAT_DPB2 ................ no
FEAT_E0PD ................ no
FEAT_EBEP ................ no
FEAT_EBF16 ............... no
FEAT_ECBHB ............... no
FEAT_ECV ................. no
FEAT_EPAC ................ no
FEAT_ETE ................. no
FEAT_ETEv1p1 ............. no
FEAT_ETEv1p2 ............. no
FEAT_ETEv1p3 ............. no
FEAT_ETMv4 ............... no
FEAT_ETMv4p1 ............. no
FEAT_ETMv4p2 ............. no
FEAT_ETMv4p3 ............. no
FEAT_ETMv4p4 ............. no
FEAT_ETMv4p5 ............. no
FEAT_ETMv4p6 ............. no
FEAT_ETS ................. no
FEAT_EVT ................. no
FEAT_ExS ................. no
FEAT_F32MM ............... no
FEAT_F64MM ............... no
FEAT_FCMA ................ no
FEAT_FGT ................. no
FEAT_FGT2 ................ no
FEAT_FHM ................. no
FEAT_FlagM ............... no
FEAT_FlagM2 .............. no
FEAT_FP .................. yes
FEAT_FP16 ................ no
FEAT_FPAC ................ no
FEAT_FPACCOMBINE ......... no
FEAT_FRINTTS ............. no
FEAT_GCS ................. no
FEAT_GICv3 ............... no
FEAT_GICv4 ............... no
FEAT_GICv4p1 ............. no
FEAT_GTG ................. no
FEAT_HAFDBS .............. no
FEAT_HAFT ................ no
FEAT_HBC ................. no
FEAT_HCX ................. no
FEAT_HPDS ................ no
FEAT_HPDS2 ............... no
FEAT_HPMN0 ............... no
FEAT_I8MM ................ no
FEAT_IDST ................ no
FEAT_IESB ................ no
FEAT_ITE ................. no
FEAT_JSCVT ............... no
FEAT_LOR ................. no
FEAT_LPA ................. no
FEAT_LPA2 ................ no
FEAT_LRCPC ............... no
FEAT_LRCPC2 .............. no
FEAT_LRCPC3 .............. no
FEAT_LS64 ................ no
FEAT_LS64_ACCDATA ........ no
FEAT_LS64_V .............. no
FEAT_LSE ................. no
FEAT_LSE128 .............. no
FEAT_LSE2 ................ no
FEAT_LSMAOC .............. no
FEAT_LVA ................. no
FEAT_LVA3 ................ no
FEAT_MEC ................. no
FEAT_MOPS ................ no
FEAT_MPAM ................ no
FEAT_MPAMv0p1 ............ no
FEAT_MPAMv1p0 ............ no
FEAT_MPAMv1p1 ............ no
FEAT_MTE ................. no
FEAT_MTE2 ................ no
FEAT_MTE3 ................ no
FEAT_MTE4 ................ no
FEAT_MTE_CANONICAL_TAGS .. no
FEAT_MTE_NO_ADDRESS_TAGS . no
FEAT_MTE_PERM ............ no
FEAT_MTE_STORE_ONLY ...... no
FEAT_MTE_TAGGED_FAR ...... no
FEAT_MTPMU ............... no
FEAT_NMI ................. no
FEAT_nTLBPA .............. no
FEAT_NV .................. no
FEAT_NV2 ................. no
FEAT_PACIMP .............. no
FEAT_PACQARMA3 ........... no
FEAT_PACQARMA5 ........... no
FEAT_PAN ................. no
FEAT_PAN2 ................ no
FEAT_PAN3 ................ no
FEAT_PAuth ............... no
FEAT_PAuth2 .............. no
FEAT_PFAR ................ no
FEAT_PMULL ............... no
FEAT_PMUv3 ............... yes
FEAT_PMUv3_EDGE .......... no
FEAT_PMUv3_ICNTR ......... no
FEAT_PMUv3_SS ............ no
FEAT_PMUv3_TH ............ no
FEAT_PMUv3p1 ............. no
FEAT_PMUv3p4 ............. no
FEAT_PMUv3p5 ............. no
FEAT_PMUv3p7 ............. no
FEAT_PMUv3p8 ............. no
FEAT_PMUv3p9 ............. no
FEAT_PRFMSLC ............. no
FEAT_RAS ................. no
FEAT_RASv1p1 ............. no
FEAT_RASv2 ............... no
FEAT_RDM ................. no
FEAT_RME ................. no
FEAT_RNG ................. no
FEAT_RNG_TRAP ............ no
FEAT_RPRES ............... no
FEAT_RPRFM ............... no
FEAT_S1PIE ............... no
FEAT_S1POE ............... no
FEAT_S2FWB ............... no
FEAT_S2PIE ............... no
FEAT_S2POE ............... no
FEAT_SB .................. no
FEAT_SCTLR2 .............. no
FEAT_SEBEP ............... no
FEAT_SEL2 ................ no
FEAT_SHA1 ................ no
FEAT_SHA256 .............. no
FEAT_SHA3 ................ no
FEAT_SHA512 .............. no
FEAT_SM3 ................. no
FEAT_SM4 ................. no
FEAT_SME ................. no
FEAT_SME2 ................ no
FEAT_SME2p1 .............. no
FEAT_SME_F16F16 .......... no
FEAT_SME_F64F64 .......... no
FEAT_SME_FA64 ............ no
FEAT_SME_I16I64 .......... no
FEAT_SPE ................. no
FEAT_SPE_CRR ............. no
FEAT_SPE_FDS ............. no
FEAT_SPECRES ............. no
FEAT_SPECRES2 ............ no
FEAT_SPEv1p1 ............. no
FEAT_SPEv1p2 ............. no
FEAT_SPEv1p3 ............. no
FEAT_SPEv1p4 ............. no
FEAT_SPMU ................ no
FEAT_SSBS ................ no
FEAT_SSBS2 ............... no
FEAT_SVE ................. no
FEAT_SVE2 ................ no
FEAT_SVE2p1 .............. no
FEAT_SVE_AES ............. no
FEAT_SVE_BitPerm ......... no
FEAT_SVE_PMULL128 ........ no
FEAT_SVE_SHA3 ............ no
FEAT_SVE_SM4 ............. no
FEAT_SYSINSTR128 ......... no
FEAT_SYSREG128 ........... no
FEAT_TCR2 ................ no
FEAT_THE ................. no
FEAT_TIDCP1 .............. no
FEAT_TLBIOS .............. no
FEAT_TLBIRANGE ........... no
FEAT_TME ................. no
FEAT_TRBE ................ no
FEAT_TRF ................. no
FEAT_TTCNP ............... no
FEAT_TTL ................. no
FEAT_TTST ................ no
FEAT_TWED ................ no
FEAT_UAO ................. no
FEAT_VHE ................. no
FEAT_VMID16 .............. no
FEAT_VPIPT ............... no
FEAT_WFxT ................ no
FEAT_XNX ................. no
FEAT_XS .................. no
```

The `lscpu` information:

```shell
$ lscpu
Architecture:            aarch64
  CPU op-mode(s):        32-bit, 64-bit
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               ARM
  Model name:            Cortex-A72
    Model:               3
    Thread(s) per core:  1
    Core(s) per cluster: 4
    Socket(s):           -
    Cluster(s):          1
    Stepping:            r0p3
    CPU(s) scaling MHz:  33%
    CPU max MHz:         1800.0000
    CPU min MHz:         600.0000
    BogoMIPS:            108.00
    Flags:               fp asimd evtstrm crc32 cpuid
Caches (sum of all):     
  L1d:                   128 KiB (4 instances)
  L1i:                   192 KiB (4 instances)
  L2:                    1 MiB (1 instance)
Vulnerabilities:         
  Gather data sampling:  Not affected
  Itlb multihit:         Not affected
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Mmio stale data:       Not affected
  Retbleed:              Not affected
  Spec rstack overflow:  Not affected
  Spec store bypass:     Vulnerable
  Spectre v1:            Mitigation; __user pointer sanitization
  Spectre v2:            Vulnerable
  Srbds:                 Not affected
  Tsx async abort:       Not affected
```

MIDR data:

```shell
$ cat /sys/devices/system/cpu/cpu0/regs/identification/midr_el1
0x00000000410fd083
```

### Comparison with the Orange Pi R1 LTS

[arm-cpusysregs](https://github.com/lelegard/arm-cpusysregs) needs kernel build
information and so I need to do more work to get a build and gather that data.

The `lscpu` information:

```shell
$ sudo lscpu
[sudo] password for ian: 
Architecture:            aarch64
  CPU op-mode(s):        32-bit, 64-bit
  Byte Order:            Little Endian
CPU(s):                  4
  On-line CPU(s) list:   0-3
Vendor ID:               ARM
  Model name:            Cortex-A53
    Model:               4
    Thread(s) per core:  1
    Core(s) per cluster: 4
    Socket(s):           -
    Cluster(s):          1
    Stepping:            r0p4
    CPU max MHz:         1296.0000
    CPU min MHz:         408.0000
    BogoMIPS:            48.00
    Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
NUMA:                    
  NUMA node(s):          1
  NUMA node0 CPU(s):     0-3
Vulnerabilities:         
  Itlb multihit:         Not affected
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Spec store bypass:     Not affected
  Spectre v1:            Mitigation; __user pointer sanitization
  Spectre v2:            Not affected
  Srbds:                 Not affected
  Tsx async abort:       Not affected
```

MIDR data:

```shell
$ cat /sys/devices/system/cpu/cpu0/regs/identification/midr_el1
0x00000000410fd034
```
