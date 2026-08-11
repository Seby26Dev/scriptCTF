# RecoverMyPet

**Category:** Forensics

---

### Challenge Description

this is all i have left.

---

### Analysis and solution

Started by extracting the archive and running `exiftool` on each image. Each one had a weird description about a `cat` that was turned into a donut. I then googled for these following keywords: `cat donut algorithm` which google redirected me to `Arnold's cat map`. After that, I tried to understand what the numbers and the fractions from the name and description are used for. They were used as parameters for applying the inverse transformation for each photo square. From the number of photos and the fraction in the description I concluded that the final image is a 6X6 from the tiles.
I then applied the inverse cat map on each tile and merged the photos and the image which contained the flag was formed.

### Flag

A bit hard to read, but here it is:

```
scriptCTF{w@t_4_cu71e_p@too1$}
```
