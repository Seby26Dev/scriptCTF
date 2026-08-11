# Oops

**Category:** Crypto

---

## Challenge Description

I am from the future! I accidentally forgot to link chall.zip! Surely you can find it and solve it right?

---

## Solution

### Getting the archive

Getting the archive was pretty easy. When solving `Misdirection` I observed that the resources URL is eassy to guess

``` URL format
https://scriptctf-2026-wave1-randomchars-4f7d3a6b.s3.us-east-1.amazonaws.com/Crypto/Misdirection/enc.txt
```

For getting the arvhive, I just changed the challenge name and file name.

### Solving the content

In the archive, there was a script and an encrypted message. The script was used to encrypt the flag. The script uses `random.seed(int(time.time()))` to generate the key.

To decrypt the flag, we could create the key back by setting the time when the flag was encoded. To get that time I read the metadata from the encrypted flag file in order to extract the `last modified` date which is the date when the file was created.
 
## Flag

```
scriptCTF{mY_buck37_1s_l34k1ng!}}
```
