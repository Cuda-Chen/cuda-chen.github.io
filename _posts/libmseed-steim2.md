---
layout: post
title: libmseed steim2
category:
tags:
---

measurement
- CPU:
- OS:
- compiler: GCC 11.3.0
- TA_MSTX__B_H_Z.2010.mseed
- time user time five times

O0
(15.481 + 15.145 + 15.279 + 15.168 + 15.256) / 5

O2
(6.320 + 6.435 + 6.156 + 6.284 + 6.171) / 5

original (5515eec)

decode data store branch
#if 0
          if (outputptr == output) /* Ignore first difference, instead store X0 */
                          *outputptr = X0;
                    else /* Otherwise store difference from previous sample */
                                    *outputptr = *(outputptr - 1) + diff[idx];
#endif
#if 1
          if (outputptr != output) /* Store difference from previous sample */
            *outputptr = *(outputptr - 1) + diff[idx];
          else /* Otherwise ignore first difference, instead store X0 */
            *outputptr = X0; 
#endif

BEFORE

49.76%  lm_parse  lm_parse              [.] msr_decode_steim2

stalled-cycles-frontend:u
            if (outputptr == output) /* Ignore first difference, instead store X0 */                                                                              
  0.79 │a69:   mov     -0x80(%rbp),%rax                                                                                                                               
  6.86 │       cmp     -0xd0(%rbp),%rax                                                                                                                               
       │     ↓ jne     a84                                                                                                                                            
       │     *outputptr = X0;                                                                                                                                         
       │       mov     -0x80(%rbp),%rax                                                                                                                               
  0.01 │       mov     -0xac(%rbp),%edx                                                                                                                               
       │       mov     %edx,(%rax)                                                                                                                                    
  0.00 │     ↓ jmp     aa2                                                                                                                                            
       │     else /* Otherwise store difference from previous sample */                                                                                               
       │     *outputptr = *(outputptr - 1) + diff[idx];                                                                                                               
  4.26 │a84:   mov     -0x80(%rbp),%rax                                                                                                                               
  0.35 │       sub     $0x4,%rax                                                                                                                                      
  0.09 │       mov     (%rax),%edx                                                                                                                                    
  6.23 │       mov     -0x94(%rbp),%eax

time
real    0m19.581s
user    0m16.672s
sys     0m2.636s

real    0m18.094s
user    0m15.676s
sys     0m2.262s

real    0m17.869s
user    0m15.686s
sys     0m2.177s

AFTER

48.87%  lm_parse  lm_parse              [.] msr_decode_steim2
      │     #if 1                                                                                                                                                    
       │     if (outputptr != output) /* Store difference from previous sample */                                                                                     
  0.67 │a69:   mov     -0x80(%rbp),%rax                                                                                                                               
  3.12 │       cmp     -0xd0(%rbp),%rax                                                                                                                               
       │     ↓ je      a96                                                                                                                                            
       │     *outputptr = *(outputptr - 1) + diff[idx];                                                                                                               
  1.40 │       mov     -0x80(%rbp),%rax                                                                                                                               
  1.08 │       sub     $0x4,%rax                                                                                                                                      
  0.78 │       mov     (%rax),%edx                                                                                                                                    
  6.21 │       mov     -0x94(%rbp),%eax                                                                                                                               
  0.54 │       cltq                                                                                                                                                   
  0.19 │       mov     -0x70(%rbp,%rax,4),%eax                                                                                                                        
  2.10 │       add     %eax,%edx                                                                                                                                      
  4.62 │       mov     -0x80(%rbp),%rax                                                                                                                               
  0.41 │       mov     %edx,(%rax)                                                                                                                                    
  3.45 │     ↓ jmp     aa2                                                                                                                                            
       │     else /* Otherwise ignore first difference, instead store X0 */                                                                                           
       │     *outputptr = X0;                                                                                                                                         
       │a96:   mov     -0x80(%rbp),%rax                                                                                                                               
  0.01 │       mov     -0xac(%rbp),%edx                                                                                                                               
  0.00 │       mov     %edx,(%rax)                                                                                                                                    
       │     #endif                                                                                                                                                   
       │                                                                                                                                                              
       │     samplecount--;

time
real    0m17.869s
user    0m15.686s
sys     0m2.177s

real    0m16.538s
user    0m14.350s
sys     0m2.185s

real    0m16.650s
user    0m14.575s
sys     0m2.069s

switch case of nibble -> if-else nibble
branch-misses:u

BEFORE
94.74%  lm_parse  lm_parse              [.] msr_decode_steim2
AFTER
95.22%  lm_parse  lm_parse              [.] msr_decode_steim2

real    0m16.451s
user    0m14.342s
sys     0m2.101s

real    0m16.207s
user    0m14.121s
sys     0m2.080s

real    0m16.060s
user    0m14.068s
sys     0m1.989s

switch case of dnib -> if-else dnib

no performance gain

switch case of dnib (w/ default for handling error)

no obvious performance gain, but with more self-explain code

real    0m16.540s
user    0m14.170s
sys     0m2.347s

real    0m15.976s
user    0m13.783s
sys     0m2.192s

real    0m16.034s
user    0m13.966s
sys     0m2.064s

if nibble == 0 unconditional jump

reduce stalled-cycles-frontend but increase execution time

introduce LM_EXTRACT_DIFF

increase execution time a little bit but with more self-explain code

frameidx == 0 -> frameidx != 0

no obvious performance gain, but with unified code style

use macro expansion to replace for loop of diff[idx]

(13.114 + 13.152 + 12.759 + 12.806 + 12.441) / 5
(13.857 + 15.044 + 13.293 + 14.104 + 12.271) / 5

remove unnecessary if

**after expansion macro, this increase execution time**

if (diffcount > 0)
      for (idx = 0; idx < diffcount && samplecount > 0; idx++, outputptr++)
       {
       }

(12.894 + 12.800 + 12.773 + 12.847 + 12.812) / 5
reduce execution time but with ugly macro, need expansion macro

final
O0
(12.824 + 12.735 + 12.846 + 12.605 + 13.851) / 5

O2
(6.544 + 5.832 + 5.892 + 5.720 + 5.956) / 5
