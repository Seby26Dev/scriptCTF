# wpm-game2


<img width="480" height="430" alt="image" src="https://github.com/user-attachments/assets/47f200a5-be71-4889-b5bb-f0854127e5a6" />


This challenge used the same restricted Python eval() idea as the first WPM
challenge, but Werkzeug debug mode was disabled. Exceptions only returned a
generic HTTP 500 page, so the flag could not be leaked directly.

I reused the small-alphabet payload technique to construct `/app/flag.txt` from
byte values and read it with:

```python
next(open(path))
```

For extraction, I compared each flag character against a guessed character. The
guess was generated without quotes using `bytes()` and the type of the flag:

```python
type(flag)(bytes([candidate]))[2]
```

The comparison controlled an expensive exponentiation:

```python
(1+1+1)**((2**22) * (flag[index] is candidate))
```

A wrong guess returned in roughly 0.4 seconds, while a correct guess took about
1.9 seconds. Testing the candidate alphabet in pairs recovered the flag one
character at a time

## Flag

```text
scriptCTF{r3v3ng3_1337_0626be7e5a1e}
```
