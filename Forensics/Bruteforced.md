# Bruteforced

**Category:** Forensics

<img width="486" height="413" alt="image" src="https://github.com/user-attachments/assets/2b75e22c-6a68-43cc-b3cc-b7d9aaed9bd5" />

---

## Challenge Description

 Help! Our website got bruteforced. Hopefully the attacker did not leak anything.

**Attachments:**
- [log.pcap](https://scriptctf-2026-wave1-randomchars-4f7d3a6b.s3.us-east-1.amazonaws.com/Forensics/Bruteforced/log.pcap)

---

## Analysis

The description tells us straight away what happened: the site was bruteforced. Meaning an attacker likely sent a huge number of http requests trying different endpoint names until one of them worked.

Opening log.pcap in Wireshark confirms this: the capture contains over 100,000 packets, which is consistent with an automated endpoint/directory bruteforce scan

Since a bruteforce attack against endpoints means most requests will fail with http 404, and only the correct endpoint will return a valid response, the goal becomes finding the one request that didn't get a 404.

---

## Solution

### Filter the traffic in Wireshark

Apply a display filter that shows only http traffic excluding 404 responses:

```
http && http.response.code != 404
```

Out of 100,004 total packets, this filter narrows the results down to exactly one matching packet

### Inspect the packet

The single result is an "HTTP/1.1 200 OK" response. Expanding the" Hypertext Transfer Protocol" layer in the packet details pane reveals the request that triggered it:

```
[Request URI: /flag_4919]
[Full request URI: http://ctf.scriptsorcerers.xyz/flag_4919]
```

So out of all the bruteforced paths, the endpoint /flag_4919 was the one that returned a 200 OK instead of a 404, this is the endpoint the attacker successfully found

### Visit the discovered endpoint

Navigating to:

```
https://ctf.scriptsorcerers.xyz/flag_4919
```

Returns the flag directly in the page content , but it isn't visible at first glance, since the text is rendered in black font on a black background. Selecting the page content reveals the hidden flag

---

## Flag

```
scriptCTF{7h3_h1dd3n_3ndp01n7_g0t_l34k3d}
```

