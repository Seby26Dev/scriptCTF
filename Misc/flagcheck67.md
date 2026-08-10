# flagcheck67

**Category:** Misc

**Difficulty:** Easy - Medium

---

## Challenge Description

The challenge says that I need to guess the password.

i heard my lil bro's password was \[READACTED\]. just connect to the server and enter it for me, would u?

---

## Solution

### Inspect the challenge connection

Opening the `nc` connection, I saw an output very similar to the `pow-solver.py` script.

#### nc connection

```
sha256(hu5a1cmzTUdqU5xf + ???) == 0000000000000000000000(22 leading zero bits)...
???: 
```

#### Pow\_solver locally

```
Solution: 6690206
Input: hu5a1cmzTUdqU5xf6690206
SHA256: 0000019f331302c15f127384873635a77bfd99df802e6a8660b5b9aa1e9a8e6f
```

### Guessing the password

To guess the password, I had to run the script locally with the `prefix` from the nc connection. The resulting number was put as answer and it was correct.

### The 67 part...

After trying many times, I reached the conclusion that you cannot have all the conditions from the `check.py` script met at the same time. Then I thought about why all the conditions are written on the same line and then it hit me. If you cause an error in the print function, the error code will also include the flag. I introduced a very big number in the `67` format and then caused the error.

```
sha256(0H0s8XZHI7Sukola + ???) == 0000000000000000000000(22 leading zero bits)...
???: 2822397
Proof-of-work correct! Continuing to the challenge...

67676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767676767
Traceback (most recent call last):
  File "/app/main.py", line 31, in <module>
    print('wrong' if num < 67676767 or 676767676767676767%(6767676767676767676767676767/num) == 676767.67 or random.randint(676767676767, 6767676767676767676)%num or num > 6767676767676767 or num//676767*676767==num or pow(67,67)//67676767676767==num or num+676767==67676767676767 else 'scriptCTF{ch47_g3n_4lph4_15_50_c00k3d}')
                                       ~~~~~~~~~~~~~~~~~~^^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ZeroDivisionError: float modulo
```

### Flag

```
scriptCTF{ch47_g3n_4lph4_15_50_c00k3d}
```

