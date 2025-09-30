# data lab solutions
almost solve them, except the last three functions for float operations, maybe another time...



### 1. bitXor 
**goal:** Implement XOR using only `~` and `&`

**solution:**
```c
int a = x & ~y;      
int b = ~x & y;      
return ~(~a & ~b);   
```

**approach:**
its obvious, just write it on paper and think about demorgan's laws :)

---

### 2. tmin 
**goal:** return minimum two's complement integer

**solution:**
```c
return ~(0x7FFFFFFF);
```

**approach:** TMAX and TMIN are bitwise complements of each other. By flipping all bits of the maximum positive value, we get the minimum negative value. 

---

### 3. isTmax 
**goal:** Check if x equals TMAX (0x7FFFFFFF)

**solution:**
```c
return (!((x + 1) & 0x0FFFFFFF)) & (((x+1) >> 31) & 1);
```

**approach** adding 1 to TMAX produces a unique testable pattern which is TMIN.

---

### 4. allOddBits
**goal:** Return 1 if all odd-numbered bits (1, 3, 5, ..., 31) are set

**solution:**
```c
return !((x & 0xAAAAAAAA) ^ 0xAAAAAAAA);
```

**approach:**
- mask `0xAAAAAAAA` and isolate odd bits
- XOR with mask checks for equality (0 if equal) and after that negate for returning 1


---

### 5. negate 
**goal:** Return -x

**solution:**
```c
return ~x + 1;
```

**Approach:**
its sky blue!

---

### 6. isAsciiDigit 
**goal:** Check if `0x30 ≤ x ≤ 0x39` (ASCII codes for '0'-'9')

**Solution:**
```c
int bound_lower = x + (~0x30 + 1);      
int bound_upper = 0x39 + (~x + 1);      
int lower = !(bound_lower >> 31);       
int upper = !(bound_upper >> 31);      
return lower & upper;
```

**approach:**
- compute both bounds using two's complement subtraction and then extract sing bits, if its negative, its failed


---

### 7. conditional
**goal:** Implement `x ? y : z` without conditionals

**solution:**
```c
int flag = !x;                    
int mask = ~flag + 1;             
return (mask & z) | ((~mask) & y);
```

**approach:**

**convert x to boolean**
- `flag = !x` converts any value to 0 or 1
- If x is non-zero (truthy): `flag = 0`
- If x is zero (falsy): `flag = 1`

**selection mask**
- `mask = ~flag + 1` uses two's complement to create all 0s or all 1s
- If `flag = 0`: `~0 + 1 = 0xFFFFFFFF + 1 = 0x00000000` (all 0s)
- If `flag = 1`: `~1 + 1 = 0xFFFFFFFE + 1 = 0xFFFFFFFF` (all 1s)

**choose y or z using masks**
- When x is **non-zero** (want y):
  - `mask = 0x00000000` (all 0s)
  - `mask & z = 0` (z is zeroed out)
  - `~mask & y = 0xFFFFFFFF & y = y` (y passes through)
  - result: `0 | y = y` 

- When x is **zero** (want z):
  - `mask = 0xFFFFFFFF` (all 1s)
  - `mask & z = z` (z passes through)
  - `~mask & y = 0x00000000 & y = 0` (y is zeroed out)
  - result: `z | 0 = z` 


---

### 8. isLessOrEqual 
**goal:** Return 1 if `x ≤ y`, handling overflow

**solution:**
```c
int sub = !((y + (~x + 1)) >> 31 & 1);  
int sign = ((y >> 31) & 1) ^ ((x >> 31) & 1);  

int same_sign_mask = !sign;
int diff_sign_mask = sign;
int diff_sign_result = (x >> 31) & 1; 

return (same_sign_mask & sub) | (diff_sign_mask & diff_sign_result);
```

**approach:**
- Use `y - x` and check if non-negative
- if x negative and y positive → true; otherwise false
- masks are here to select appropriate result
- we should take care of overflows as well, it only happens with different signs


---

### 9. logicalNeg 
**goal:** Implement `!x` without using `!`

**solution:**
```c
return ((x | (~x + 1)) >> 31) + 1;
```


---

### 10. howManyBits
**goal:** Return minimum bits needed to represent x in two's complement

**solution:**
```c
int sign = x >> 31;
int magnitude = (x ^ sign) + (sign & 1);  
int count = 0;
int temp;

temp = !!(magnitude >> 16);
count += temp << 4;
magnitude >>= temp << 4;

temp = !!(magnitude >> 8);
count += temp << 3;
magnitude >>= temp << 3;

temp = !!(magnitude >> 4);
count += temp << 2;
magnitude >>= temp << 2;

temp = !!(magnitude >> 2);
count += temp << 1;
magnitude >>= temp << 1;

temp = !!(magnitude >> 1);
count += temp;

return count + 1 + !!x;
```

**approach:**
so its a binary search, at first i am converting negative to positive, then do binary search, 16 bit, 8 bit, ..., at each step
if bits exist in the upper half, i added range size and shift focus, and at the end added one for sign bit
- Convert negative to positive magnitude: XOR with sign mask, adjust by 1



