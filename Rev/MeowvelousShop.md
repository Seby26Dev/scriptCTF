
# MeowvelousShop

## Description

> `to distrcat your enemy, you must first distrcat yourself`  
> `--⚞^. .^⚟`

Remote service:

```bash
nc challs.scriptsorcerers.xyz 10337
```

## Solution

The binary contains a membership ID validation routine designed to look more complicated than it is due to C++ exception unwinding and other distracting logic.

Reversing the validation gives the valid membership ID:

```text
N0Fl4gY37
```

Connect to the service and choose:

```text
2
N0Fl4gY37
4
```

Option 2 sets the membership ID, and option 4 redeems the membership reward.

The service responds with:

```text
gud try, but no flag for u ≽^╥⩊╥^≼
maybe buy some plushies?
scriptCTF{bu5y_c47_unw1nd1ng_fr0m_h15_5h1f7_@_7h3_5h0p_b057e18ad44d}
```

## Flag

```text
scriptCTF{bu5y_c47_unw1nd1ng_fr0m_h15_5h1f7_@_7h3_5h0p_b057e18ad44d}
```
