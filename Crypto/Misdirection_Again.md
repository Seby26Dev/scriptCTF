# Misdirection Again

**Category:** Crypto

<img width="489" height="358" alt="image" src="https://github.com/user-attachments/assets/e9daf67d-759c-408c-92e6-a87c7385f0c4" />

---

## Challenge Description

**Attachments:**
- [enc.txt](https://scriptctf-2026-wave2-randomchars4919-7d6b4f3a2b.s3.us-east-1.amazonaws.com/Crypto/Misdirection-Again/enc.txt)

Contents of `enc.txt`:

```
111101001000111101110000101110000110000111100000010011011111000110001010000100011101001101000101000101011001001101010110110011000110111011011101011100001111010001010110111000110111000110101001111010011110011011010100111101100101000010110010001111011010011101101101001010100101
```

---

## Analysis

At first glance, this looks like the same kind of setup as the previous "Misdirection" challenge. Binary that should be grouped and converted to text. But every standard grouping attempt fails to produce anything readable:

----
- Grouping into 8-bit chunks -> garbage
- Grouping into 7-bit chunks -> garbage
- Grouping into 6-bit chunks -> garbage

----

## The actual misdirection: 
The binary string isn't meant to be split into fixed-size groups at all. Instead, it should be treated as a single large binary number, converted to its decimal representation, and then that decimal string should be parsed as a sequence of ASCII codes of variable length 2 or 3 digits per character

---

## Solution

### Convert the binary string to a decimal integer

Treating the entire 276-bit string as one big binary number and converting it to base 10:

```python
n = int(s, 2)
d = str(n)
```

This produces an 84-digit decimal number:

```
115991141051121166784701231094911510049114519911649481109510249110521089598485353125
```

### Parse the decimal digits as variable-length ASCII codes

Reading through the decimal string, greedily trying 3-digit groups first, then falling back to 2-digit groups whenever the value is outside the printable ASCII range , reconstructs the message character by character:

| Digits | ASCII | Char | | Digits | ASCII | Char |
|--------|-------|------|---|--------|-------|------|
| 115 | 115 | `s` | | 51  | 51  | `3` |
| 99  | 99  | `c` | | 99  | 99  | `c` |
| 114 | 114 | `r` | | 116 | 116 | `t` |
| 105 | 105 | `i` | | 49  | 49  | `1` |
| 112 | 112 | `p` | | 48  | 48  | `0` |
| 116 | 116 | `t` | | 110 | 110 | `n` |
| 67  | 67  | `C` | | 95  | 95  | `_` |
| 84  | 84  | `T` | | 102 | 102 | `f` |
| 70  | 70  | `F` | | 49  | 49  | `1` |
| 123 | 123 | `{` | | 110 | 110 | `n` |
| 109 | 109 | `m` | | 52  | 52  | `4` |
| 49  | 49  | `1` | | 108 | 108 | `l` |
| 115 | 115 | `s` | | 95  | 95  | `_` |
| 100 | 100 | `d` | | 98  | 98  | `b` |
| 49  | 49  | `1` | | 48  | 48  | `0` |
| 114 | 114 | `r` | | 53  | 53  | `5` |
| 51  | 51  | `3` | | 53  | 53  | `5` |
| 99  | 99  | `c` | | 125 | 125 | `}` |
| 116 | 116 | `t` | |     |     |     |
| 49  | 49  | `1` | |     |     |     |
| 48  | 48  | `0` | |     |     |     |

Reading it all in order gives the flag directly.

### Full solve script

```python
s = "111101001000111101110000101110000110000111100000010011011111000110001010000100011101001101000101000101011001001101010110110011000110111011011101011100001111010001010110111000110111000110101001111010011110011011010100111101100101000010110010001111011010011101101101001010100101"

#Bin -> int
n = int(s, 2)
#int -> dec.str
d = str(n)

flag = ''
i = 0
while i < len(d):
    for w in (3, 2):
        if i + w > len(d):
            continue
        v = int(d[i:i+w])
        if 32 <= v <= 126:
            flag += chr(v)
            i += w
            break
    else:
        i += 1

print(flag)
```

---

## Flag

```
scriptCTF{m1sd1r3ct10n_f1n4l_b055}
```
