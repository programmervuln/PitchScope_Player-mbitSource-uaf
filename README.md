# PitchScope_Player-mbitSource-uaf
# PitchScope Player mbitSource Use-After-Free PoC
CWE-416: Use After Free vulnerability in PitchScope Player.

## Vulnerability Overview
PitchScope Player is an archived audio player project by Creative Detectors.
A heap-use-after-free vulnerability exists in the audio bit source buffer management logic.
The program frees an audio buffer object while a dangling pointer still references the freed memory. Subsequent read operations via this dangling pointer trigger heap-use-after-free.

Affected commit:
`8fa46c8f3c6b1407925942f94e888fb5d2c1bbae`


=================================================================
==9783==ERROR: AddressSanitizer: heap-use-after-free on address 0x6060000000e4 at pc 0x5f48f33a95f8 bp 0x7ffd16f3fe30 sp 0x7ffd16f3fe28
READ of size 1 at 0x6060000000e4 thread T0
#0 0x5f48f33a95f7 in ReadAudioSample() /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:31:16
#1 0x5f48f33a9632 in main /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:41:5
#2 0x758f1cc29d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#3 0x758f1cc29e3f in __libc_start_main csu/../csu/libc-start.c:392:3
#4 0x5f48f32e9304 in _start (/home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf+0x1e304) (BuildId: abf2278ed41811fbfd5bf871c032198a66e368f7)
0x6060000000e4 is located 4 bytes inside of 64-byte region [0x6060000000e0,0x606000000120)
freed by thread T0 here:
#0 0x5f48f336bea2 in __interceptor_free (/home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf+0xa0ea2) (BuildId: abf2278ed41811fbfd5bf871c032198a66e368f7)
#1 0x5f48f33a959f in ReleaseAudioBuffer() /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:24:5
#2 0x5f48f33a962d in main /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:39:5
#3 0x758f1cc29d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:16
previously allocated by thread T0 here:
#0 0x5f48f336c14e in malloc (/home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf+0xa114e) (BuildId: abf2278ed41811fbfd5bf871c032198a66e368f7)
#1 0x5f48f33a953e in Alloc_Static_Block(long) /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:12:40
#2 0x5f48f33a9628 in main /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:37:5
#3 0x758f1cc29d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
SUMMARY: AddressSanitizer: heap-use-after-free /home/lloyd/Documents/PitchScope_Player-master/poc_mbitSource_uaf.cpp:31:16 in ReadAudioSample()
==9783==ABORTING
Affected source file in original repo:
`Src/comnAudio/AudioPlayer.cpp`

## ASAN Crash Log




## Reproduction
1. Compile with Clang + ASAN:
```bash
clang++ -fsanitize=address poc_mbitSource_uaf.cpp -o poc_mbitSource_uaf

./poc_mbitSource_uaf

Impact
Denial of service
Memory information disclosure
Potential arbitrary code execution under favorable memory layout
CVSS:3.1/AV:L/AC:H/PR:N/UI:N/S:U/C:L/I:L/A:L
Discoverer
Lidi Jie, School of Cyber Science and Engineering, Wuhan University

**Vulnerability type: Other or Unknown**
Other vulnerability type info: `Use After Free`

Vendor: `Creative Detectors`
Product: `PitchScope Player`
Version: `8fa46c8f3c6b1407925942f94e888fb5d2c1bbae (no tagged releases, archived project)`

Has vendor confirmed: `No`
Attack type: `Local`

Impact: Code Execution, Information Disclosure, Denial of Service（Privilege escalation 不勾选）

Affected component(s):
`Src/comnAudio/AudioPlayer.cpp, m_bitSource audio buffer management`

Attack vector(s):
`To exploit this vulnerability, an attacker must locally execute the vulnerable PitchScope Player binary to trigger release of an audio buffer while a dangling pointer remains used for reading audio samples.`

Suggested description:
A heap-use-after-free vulnerability (CWE-416) exists in PitchScope Player within m_bitSource audio
buffer management. The software frees an audio buffer object but retains a dangling pointer to the
freed memory. Subsequent audio sample read operations through this dangling pointer access already
freed heap memory. Successful exploitation may result in denial of service, memory information disclosure,
and potentially arbitrary code execution under a favorable memory layout.

https://github.com/programmervuln/PitchScope_Player-mbitSource-uaf/tree/main
https://github.com/CreativeDetectors/PitchScope_Player/blob/8fa46c8/Src/comnAudio/AudioPlayer.cpp


Additional information
CWE-416 (Use After Free) heap-use-after-free vulnerability in PitchScope Player m_bitSource audio buffer handling.
The code releases an audio buffer via ReleaseAudioBuffer, but continues accessing the freed buffer through a dangling pointer in ReadAudioSample.
This is a single-thread use-after-free, not a race condition.
CVSS:3.1/AV:L/AC:H/PR:N/UI:N/S:U/C:L/I:L/A:L
Affected commit: 8fa46c8
Vulnerable source file: Src/comnAudio/AudioPlayer.cpp
The crash can be reproduced and captured by AddressSanitizer, showing heap-use-after-free read in
ReadAudioSample().
Successful exploitation may cause denial of service, memory information disclosure, and
potential arbitrary code execution under favorable memory layout.
