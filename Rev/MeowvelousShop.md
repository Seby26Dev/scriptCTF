# MeowvelousShop

## Description

> `to distrcat your enemy, you must first distrcat yourself`
> `--⚞^. .^⚟`

Remote service:

```bash
nc challs.scriptsorcerers.xyz 10337
```

We are also given the challenge binary, so this is mainly a reversing challenge.

## Solution

I started by running the binary and looking through the available options:

```text
[1] Browse the Cat-alog
[2] Set your membership ID
[3] View your membership ID
[4] Redeem membership rewards
[5] View credits
[6] Earn credits
[7] Buy a cat
[8] View current cat
[9] Exit
```

At first, the credits and cat shop looked like the obvious way to get the flag. There is an expensive plushie in the shop, and the program lets you earn credits, so it looks like you are supposed to somehow get enough money to buy it.

Considering the challenge description literally says to "distrcat" yourself, I assumed this was probably a rabbit hole and started looking at the membership functionality instead.

### Basic reversing

Running `strings` on the binary gives some useful strings:

```bash
strings chall
```

Some interesting ones were:

```text
flag.txt
enter new membership ID:
membership invalid
membership IDs must be 9 characters long
gud try, but no flag for u
maybe buy some plushies?
```

The important part here is that membership IDs have to be exactly 9 characters long, and there is clearly some validation happening before the reward can be redeemed.

I opened the binary in Ghidra and followed the references to strings such as:

```text
membership invalid
```

This leads to the membership validation code.

The validation looks much more complicated than expected because the binary uses C++ exception handling and unwind routines. Some of the imported functions include things like:

```text
_Unwind_Resume
_Unwind_GetLanguageSpecificData
__cxa_throw
__cxa_begin_catch
__cxa_end_catch
```

Instead of checking every character in a normal loop, the program spreads parts of the validation across exception-unwind handlers.

That also explains the flag text later on:

```text
bu5y_c47_unw1nd1ng...
```

### Membership ID check

After following the validation logic, I found that the membership ID is restricted to alphanumeric characters and has a length of 9.

One of the main checks builds a 64-bit state from each character. The relevant operation is basically:

```python
state = rol(state, 13)
state ^= character * 0xff51afd7ed558ccd
```

The state starts at `0`, and after all 9 characters are processed there is another XOR using a rotated `0xa5a5a5a5a5a5a5a5` value.

The final state is compared against:

```text
0xdbe43b2ca91a04d7
```

Instead of trying every possible 9-character string manually, I reproduced the check using Z3.

My solver looked like this:

```python
from z3 import *

MASK = 0xffffffffffffffff

MUL = 0xff51afd7ed558ccd
A5 = 0xa5a5a5a5a5a5a5a5
TARGET = 0xdbe43b2ca91a04d7


def rol(x, n):
    n %= 64
    return ((x << n) | (x >> (64 - n))) & MASK


chars = [BitVec(f"c{i}", 8) for i in range(9)]

s = Solver()

# membership ID is alphanumeric
for c in chars:
    s.add(
        Or(
            And(c >= ord("0"), c <= ord("9")),
            And(c >= ord("A"), c <= ord("Z")),
            And(c >= ord("a"), c <= ord("z")),
        )
    )

state = BitVecVal(0, 64)

for c in chars:
    state = RotateLeft(state, 13)
    state ^= ZeroExt(56, c) * MUL

mix = rol(A5, 5)

s.add(state ^ mix == TARGET)

if s.check() == sat:
    m = s.model()
    membership = "".join(chr(m[c].as_long()) for c in chars)
    print(membership)
```

Running it gives:

```text
N0Fl4gY37
```

Which can be read as:

```text
N0 Fl4g Y37
No Flag Yet
```

So even the correct membership ID is another joke from the challenge.

### Getting the flag

Now I connected to the remote service:

```bash
nc challs.scriptsorcerers.xyz 10337
```

First I selected option `2` to set the membership ID:

```text
> 2

enter new membership ID:
N0Fl4gY37
```

The server accepted it:

```text
updated ≽^•⩊•^≼
```

Then I used option `4` to redeem the membership reward:

```text
> 4
```

The server responded with:

```text
gud try, but no flag for u ≽^╥⩊╥^≼
maybe buy some plushies?
scriptCTF{bu5y_c47_unw1nd1ng_fr0m_h15_5h1f7_@_7h3_5h0p_b057e18ad44d}
```

The plushie message is just one final distraction. The flag is printed immediately after it.

## Flag

```text
scriptCTF{bu5y_c47_unw1nd1ng_fr0m_h15_5h1f7_@_7h3_5h0p_b057e18ad44d}
```

Overall, the main trick was realizing that the shop and credit system were mostly there to waste time. The actual solution was hidden in the membership validation, which was intentionally obfuscated using C++ exception unwinding.
