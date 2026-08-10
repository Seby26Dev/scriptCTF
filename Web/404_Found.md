# 404 Found

**Category:** Web

<img width="479" height="544" alt="image" src="https://github.com/user-attachments/assets/1d27404e-a60e-4a29-8340-236d68c87de2" />

---

## Challenge Description

Please don't hack my shopping cart!


## Solution

### Basic endpoint enumeration

Whenever a web challenge gives no obvious leads, one of the first things to check is whether the site exposes a robots.txt file. This file is meant to tell search engine crawlers which paths they should not index , but it often ends up revealing hidden or unlisted endpoints in the process

Navigating to:

```
https://0adeb329-a0d9-4c2f-bf42-1d77fba343d8.challs.scriptsorcerers.xyz/robots.txt
```

returns:

```
User-agent: *
Disallow: /the-best-robot
```

The Disallow rule points directly at a hidden path: /the-best-robot

### Visit the disallowed endpoint

Since robots.txt only tells crawlers not to index a path it doesn't actually restrict access to it we can simply visit it ourselves:

```
https://0adeb329-a0d9-4c2f-bf42-1d77fba343d8.challs.scriptsorcerers.xyz/the-best-robot
```

The page returns the flag directly in its content.

---

## Flag

```
scriptCTF{r0b07s_4r3_t4k1ng_0v3r_4280c9cf77f9}
```
