# XNU Kernel Segment Overflow

The XNU kernel's stack initialization contains a vulnerability allowing attackers to overflow the kernel stack segment.  
This is possible because the kernel lacks boundary checks on the number of stack pages one can pass to the boot-arguments of the kernel.  
The kernel will then initialize the stack with that number of pages which will overlap into the next segment, at this time resulting in a Denial of Service.

Attackers can abuse this vulnerability only if they manage to prevent the cpu from faulting as result of the bug.  
When exploited an attacker might be able to overlap protected kernel segments which may lead to a potential KTRR (memory protection) defeat at initialization stage.  

More research is needed to point out how much of the segment is overflown and what the exact impact could be.  
If it is possible to overflow into a segment with r-x mapping, it might mean as the stack generally is rw- or r-- that the segments will overlap in such a way that the access rights of the pages become rwx, which is dangerous.  

If the segment after the stack is part of the protected KTRR region it could mean that an attacker will be able to alter critical structures of the kernel as then the KTRR region will be come a KTRWR (Kernel Text ReadWrite Region) for the number of overlapping pages.  
However this all still depends on being able to restore the intialization stage after the overflow.  

A proof-of-concept for the exploit can be found here in exploits/segmentoverflow.sh  
It is very simple, but very dangerous on production devices as it might brick the device into an unusable state.  
Therefore it is recommended to only test it on development device with iboot unlocked.  
The vulnerability exists in the latest version of XNU as seen here at [Latest XNU](https://github.com/UKERN-Developers/darwin-xnu/blob/1527a46c171c66d57c22fa6b233e664d092f81a1/osfmk/kern/stack.c#L119)  
