# Insanity Check

**Category:** Misc

**Difficulty:** Easy

---

## Challenge Description

The challenge itself gives almost nothing to work with just a title " Insanity Check "

<img width="502" height="218" alt="image" src="https://github.com/user-attachments/assets/af339c04-6aeb-451a-80bc-2612fc3bcbd5" />

---

## Solution

### Inspect the challenge page

Opening the browser Inspector on the challenge page and looking through the challenge-desc element reveals a hidden html comment:

```html
<!--visit /vibecheck ;) !-->
```

This comment is not visible in the rendered page it only shows up when reading the raw html and points us toward an additional endpoint: /vibecheck.

### Navigate to /vibecheck

Following the hint, we browse to:

```
https://play.scriptsorcerers.xyz/vibecheck
```

The rendered page itself doesn't display anything useful. The next step is was to inspect this new page's html as well

### Flag

Viewing the page source of /vibecheck, we find the flag hidden inside an HTML comment within the <main> element:

```html
<main role="main">
  <div class="container">
    <!-- scriptCTF{v1b3_ch3ck_p4ss3d!} -->
  </div>
</main>
```

---

## Flag

```
scriptCTF{v1b3_ch3ck_p4ss3d!}
```
