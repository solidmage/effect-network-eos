# effect-network-eos
EOS smart contract for the Effect.AI token

## Compiling

The token contract is implemented in C++. To compile and deploy the contract the following prerequisites are required:
- eosio.cdt v1.5.0
- nodeos v1.6.1
- keosd v1.6.1
- cleos

First compile the contract:
```bash
cd contracts/effect.token
eosio-cpp -o effect.token.wasm effect.token.cpp --abigen
```

Create an account to deploy the contract to. In your development environment this would be:
```bash
cleos create account eosio effect.token EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
```

Deploy the contract:
```bash
cleos set contract effect.token contracts/effect.token -p effect.token@active
```

## Action Specification

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

##### `token::lock (from: name | to: name | quantity: asset | date: uint64_t)`
Send `quantity` of tokens to a user that are locked until `time`. The locked amount is stored in a seperate table on the context of `to` with an incrementing primary key. The RAM is payed by `from`. The date must be `time_point_sec` compatible, meaning it should be an ISO formatted UTC string without timezone information. `to` can be a non existent account, which can be claimed once the account is created and the `time` is expired.

```bash
cleos push action effect.token lock '[ "peter", "sarah", "10.0000 EFX", "2019-03-14T11:55:23" ]' -p peter@active
```

##### `token::unlock (to: name | lockId: uint64_t)`
Unlock the locked tokens for `to` at a given `lockId`. This will remove the record from the locked table, freeing up RAM payed for by the `from` in the `lock` action.

```bash
cleos push action effect.token unlock '[ "sarah", 0 ]' -p sarah@active

# All locked tokens for `to` can be requested with cleos:
cleos get table effect.token sarah locks
```

### Considerations
Transfer by id, get the RAM back when a transfer is fulfilled.

Instead of giving others an allowance, could it be worth it to create a payment request. With the following options to be groomed:
- To be payed by a single person, or by everyone.
- To expire the request in a given time, or to be valid infinitely.
- To remove the payment request.
- To allow the sender to send a different value other than the one in the payment request
  - Allow higher values
  - Allow lower values
  - Allow any value > 0
  - When a lower/different value is sent, should the payrequest be
    - deleted
    - modified to be lowered
- Hell you can make an entire debt system like this.
- Add interest rates over time because why not.

Karma implements a lock that is true by default. This locks any transactions until these are unlocked by the issuer, is this something that we want?

Who pays for the RAM after spending on someone elses behalf?

In the basic contract, the account is removed when it's quantity is set to zero. With allowance that's what we want, but for the account maybe not. We could build the option to let other people pay the RAM for an account by opening a transfer request. If we close the account automatically when it becomes zero then we can only keep sending tokens by transfer request.
That becomes cumbersome if want to use allowance to create a transfer request. What?
