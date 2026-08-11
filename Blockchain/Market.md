

# Market

<img width="483" height="513" alt="image" src="https://github.com/user-attachments/assets/572b441b-0cc5-4c8d-a74f-a627dcc36678" />


The challenge is a Solana program implementing a small shop. The goal is to
replace the owner stored in the CONFIG PDA with the public key given to us by
the server.

## Bug

In buy(), the program derives the expected holding PDA:

```python
let (holding_pda, holding_expected_bump) =
    Pubkey::find_program_address(&[user.key.as_ref(), b"HOLDING", item.key.as_ref()], program);
```


However, it never checks that the supplied holding account is actually equal
to holding_pda. If the supplied account is already initialized, it is treated
as a Holding and these fields are overwritten:

```
holding_data.owner = *user.key;
holding_data.item = *item.key;
holding_data.quantity += 1;
```

Config and Holding have compatible Borsh layouts:

```
Config:  Pubkey owner, Pubkey treasury, u64 count
Holding: Pubkey owner, Pubkey item,     u64 quantity
```

This means I can pass the CONFIG PDA as both system_config and holding.
The purchase then rewrites Config.owner to my user public key.

# Exploit
My uploaded solver program performs three CPIs in one transaction:

```
Call InitializeUser to create the user PDA.
Deposit 5 SOL into it.
Buy the shell (item_id = 4919), passing the CONFIG PDA as the holding account.
```

The important account list for the final CPI is:

```c
vec![
    user,
    user_config,
    config,
    treasury,
    config,
    shell,
    system_program,
]
```

After uploading solve.so and sending the required account metas, the server
confirmed that the current owner was my generated user and returned the flag.

# Flag
```
scriptCTF{w41t_4_s3c0nd_wh0_4r3_y0u???_60aff1437947}
```
