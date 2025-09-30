# Bomb solutions: Phases 1-5

## Overview
This guide documents the reverse engineering process and solutions for the first five phases of the CMU Bomb Lab. I could not solve phase 6, so I put it aside for another time to think about it. Overall, this was a very fun assignment.

---

## phase 1

### key lines
```assembly
0x400ee4 <+4>:   mov    $0x402400,%esi
0x400ee9 <+9>:   call   0x401338 <strings_not_equal>
0x400eee <+14>:  test   %eax,%eax
0x400ef0 <+16>:  je     0x400ef7 <phase_1+23>
0x400ef2 <+18>:  call   0x40143a <explode_bomb>
```

### analysis
for this phase, I simply examined the address and found the target string.

### strategy
```bash
(gdb) disas phase_1
(gdb) x/s 0x402400
```

### solution
`Border relations with Canada have never been better.`

---

## phase 2

### key lines
```assembly
0x400f05 <+9>:   call   0x40145c <read_six_numbers>
0x400f0a <+14>:  cmpl   $0x1,(%rsp)
0x400f17 <+27>:  mov    -0x4(%rbx),%eax
0x400f1a <+30>:  add    %eax,%eax
0x400f1c <+32>:  cmp    %eax,(%rbx)
```

### analysis
these lines are key. at a high level, it's a loop that loads the input from the register, doubles it, and compares it with the previous digit. based on the line at +14, the first input should be 1.



### Solution
`1 2 4 8 16 32`

---

## Phase 3

### key lines
```assembly
0x400f6a <+39>:  cmpl   $0x7,0x8(%rsp)
0x400f71 <+46>:  mov    0x8(%rsp),%eax
0x400f75 <+50>:  jmp    *0x402470(,%rax,8)
0x400f7c <+57>:  mov    $0xcf,%eax
0x400f83 <+64>:  mov    $0x2c3,%eax
...
0x400fbe <+123>: cmp    0xc(%rsp),%eax
```

### solution process
through debugging and examining registers after a case executes, I found the value stored in `%eax`. for example, at case 7, stepping through the code reveals `%eax` contains 327.

### Solution
`7 327` (case 7 loads 327 into %eax, which must match the second input)

Note: Several solutions exist (0-7 for the first number, with a corresponding second number for each case).

---

## Phase 4

### Key Lines (phase_4)
```assembly
0x40102e <+34>:  cmpl   $0xe,0x8(%rsp)
0x401033 <+39>:  jbe    0x40103a <phase_4+46>
0x40103a <+46>:  mov    $0xe,%edx
0x40103f <+51>:  mov    $0x0,%esi
0x401044 <+56>:  mov    0x8(%rsp),%edi
0x401048 <+60>:  call   0x400fce <func4>
0x40104d <+65>:  test   %eax,%eax
0x401051 <+69>:  cmpl   $0x0,0xc(%rsp)
```

### Key Lines (func4)
```assembly
0x400fd2 <+4>:   mov    %edx,%eax
0x400fd4 <+6>:   sub    %esi,%eax
0x400fd8 <+10>:  shr    $0x1f,%ecx
0x400fdb <+13>:  add    %ecx,%eax
0x400fdd <+15>:  sar    $1,%eax
0x400fdf <+17>:  lea    (%rax,%rsi,1),%ecx
0x400fe2 <+20>:  cmp    %edi,%ecx
0x400fe9 <+27>:  call   0x400fce <func4>
0x400fee <+32>:  add    %eax,%eax
```



### Solution Approach
The second number must be zero (verified by the comparison at line +69). The first number should be a value that makes func4 return zero. this goal is satisfied by several numbers. The process resembles a binary search.

solutions include: `0 0`, `1 0`, `3 0`, `7 0`

I believe these numbers work because they appear as midpoints during the binary search traversal of the range (0, 14). other numbers don't appear as exact midpoints and cause the recursion to return non-zero values.

---

## Phase 5: Character Encoding Table

This was very hard!

### key lines
```assembly
0x40107a <+24>:  call   0x40131b <string_length>
0x40107f <+29>:  cmp    $0x6,%eax
0x40108b <+41>:  movzbl (%rbx,%rax,1),%ecx
0x401096 <+52>:  and    $0xf,%edx
0x401099 <+55>:  movzbl 0x4024b0(%rdx),%edx
0x4010a0 <+62>:  mov    %dl,0x10(%rsp,%rax,1)
0x4010b3 <+81>:  mov    $0x40245e,%esi
0x4010bd <+91>:  call   0x401338 <strings_not_equal>
```

### analysis
the input should be 6 characters. First, I examined the address `0x40245e`, which contained the string "flyers". In the previous lines, there is a byte loading operation (+41), and the next line extracts the lower 4 bits. I suspected that the input characters should have lower 4 bits that match the indices in the lookup table needed to reconstruct "flyers".

### Lookup Table
```
 0:m  1:a  2:d  3:u  4:i  5:e  6:r  7:s
 8:n  9:f 10:o 11:t 12:v 13:b 14:y 15:l
```

With Claude's help, I found characters whose lower 4 bits match the required indices:

**Target: "flyers"**
- 'f' is at index 9 → need char with lower 4 bits = 9 (e.g., 'i', 'y', '9')
- 'l' is at index 15 → need char with lower 4 bits = 15 (e.g., 'o', '?', 'O')
- 'y' is at index 14 → need char with lower 4 bits = 14 (e.g., 'n', '.', 'N')
- 'e' is at index 5 → need char with lower 4 bits = 5 (e.g., 'e', '5', 'E')
- 'r' is at index 6 → need char with lower 4 bits = 6 (e.g., 'f', '6', 'F')
- 's' is at index 7 → need char with lower 4 bits = 7 (e.g., 'g', '7', 'G')

### Example Solution
`ionefg`



