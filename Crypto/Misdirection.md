# Misdirection

**Category:** Crypto

---

## Challenge Description

 It is not what it is.

**Attachments:**
- [`enc.txt`](https://scriptctf-2026-wave1-randomchars-4f7d3a6b.s3.us-east-1.amazonaws.com/Crypto/Misdirection/enc.txt)

Contents of enc.txt:

```
1000100010100000100001110100100001010010001010110001101100101010000111000001001001000100101000100100001000101110001
```

---

## Solution

### Decoding with CyberChef

Pasting the ciphertext into [CyberChef](https://gchq.github.io/CyberChef/) and using its Magic, it automatically identifies the data as a Baconian cipher. Adding the Bacon Cipher Decode operation manually with:

- **Alphabet:** `Standard (I=J and U=V)`
- **Translation:** `0/1`

produces the plaintext directly:

```
SCRIPTCTFNOTWHATITSEEMS
```


## Flag

```
scriptCTF{SCRIPTCTFNOTWHATITSEEMS}
```
