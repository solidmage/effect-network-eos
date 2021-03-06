# Effect Token Contract

## Specification

##### `token::create (issuer: name | maximum_supply: asset)`
Create the token with an issuer, given supply and symbol on the authority of the account on which the contract is deployed. The following example creates one million EFX tokens with a precision of four. The issuer is able to issue the initial tokens to other accounts. The RAM allocation for the different types of tokens is payed by the contract owner.

```bash
cleos push action effect.token create '[ "effect", "1000000.0000 EFX" ]' -p effect.token@active
```

##### `token::issue (to: name | quantity: asset | memo: string)`
The issue action issues a certain amount of tokens to an account. Each time this action is performed, it raises the supply of the token. When the maximum supply is reached, it is impossible to issue more tokens. This action must be performed with the authority of the issuer account specified when the token was created.

```bash
cleos push action effect.token issue '[ "peter", "50000.0000 EFX", "m" ]' -p effect@active
```

##### `token::transfer (from: name | to: name | quantity: asset | memo: string)`
The transfer action transfers tokens from account "from" to account "to". This action must be performed with the authority of account "from". Notice that if account "to" does not possess any of these tokens, this action creates a corresponding accounts table in the scope of "to". the RAM allocation for this table is payed by "from".

```bash
cleos push action effect.token transfer '[ "peter", "josh", "23.0000 EFX", "m" ]' -p peter@active
```

##### `token::retire (quantity: asset | memo: string)`
Allows the issuer of a token to remove tokens from circulation. The tokens must be owned by the issuer at the time they are retired. So it does not allow an issuer to retire tokens out of other people's wallets. Retired tokens will decrease the supply, allowing them to be issued by the issuer again at a later stage.

```bash
cleos push action effect.token retire '[ "2000.0000 EFX", "m" ]' -p effect@active
```

##### `token::open (owner: name | symbol: symbol&, ram_payer: name)`
Initiates an account with a balance of 0. The symbol consists of the precision of the currency (number of zero's after the period `.`). The RAM is payed by the person opening the account, which must be validated.

```bash
cleos push action effect.token open '[ "josh", "4,EFX", "eosio" ]' -p effect@active
```

##### `token::close (owner: name | symbol: symbol&)`
Closes the account by removing the account entry. This can only be done by the owner and the balance must be zero.

```bash
cleos push action effect.token close '[ "josh", "4,EFX" ]' -p josh@active
```

### Future Functions

These actions are being considered and are net yet merged in the main code.

##### `token::approve (owner: name | spender: name | quantity: asset)`
The approve action is used when the owner whishes to give spending priveleges to the spender. Sets the amount `quantity` that `spender` can send on behalf of `owner`. The amount that can be spent is stored in a seperate table `allowance`. The RAM allocated for these costs are payed by the `owner` and must be validated as such.

```bash
cleos push action effect.token approve '[ "peter", "josh", "50 EFX" ]' -p peter@active
```

##### `token::transferfrom (from: name | to: name | spender: name | quantity: asset | memo: string)`
The transferfrom action sends the tokens on your behalf from another account to someone else. There must be sufficient allowance provided to the originator. The command fails unless the "from" account has deliberately authorized the "spender" via the approve method. Note that if account "to" does not already possess a balance in the specified tokens, an accounts table must be created in its scope. In this case, the RAM allocation costs are payed by the spender.

```bash
cleos push action effect.token transferfrom '[ "peter", "sarah", "josh", "10.0000 LSD", "memo" ]' -p josh@active
```

##### `token::lock (from: name | to: name | quantity: asset | time: uint64_t)`
Send `quantity` of tokens to a user that are locked until `time`. The locked amount is stored in a seperate table. Any additional calls that have the same time should be added to the same record.

Note:
- If you have allowance, can you also use them to lock tokens in the name of someone else?

##### `token::unlock (to: name | time: uint64_t)`
Unlock all tokens locked for `to` at given `time`.
