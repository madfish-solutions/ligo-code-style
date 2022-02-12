# Table of Contents

- [Introduction](#introduction)
- [Code organization](#code-organization)
  - [Subfolders](#subfolders)
    - [Contracts](#contracts)
    - [Builds](#builds)
    - [Migrations](#migrations)
    - [Scripts](#scripts)
    - [Tests](#tests)
  - [File's naming](#files-naming)
- [Scripts](#scripts)
- [Formatting](#formatting)
- [Names](#mames)
- [Comments](#comments)
- [Types](#types)
- [Control flow structures](#control-flow-structures)
- [Functions](#functions)
- [Exceptions](#exceptions)
- [Values of complex types](#values-of-complex-types)
- [Constants](#constants)

# Introduction

Formatting and structuring issues are the main sticking point in the way of creating human-readable and maintainable solutions. Despite the Ligo code-base is quickly growing there is no common code style which makes it harder to involve new people in project development, review, or audit processes.

This document provides tips for writing clean Ligo code. In spite of most of the subjects are covered, the specification isn't final and can be improved anytime. Anyone can add the rule with the **(suggestion)** mark at the beginning. The mark can be removed after discussion and approval of the majority.

# Code organization

To make the navigation easier, the followed project structure is suggested:

- [contracts](#contracts): smart contracts, it can be both Ligo and Michelson code.
- [builds](#builds): compiled contracts, storage, expressions etc.
- [migrations](#migrations): scripts to deploy the contracts to the network.
- [scripts](#scripts): helpful scripts.
- [test](#test): code for testing smart contracts and ensuring their validity.

## Subfolders

### Contracts

All the smart contracts code is placed in the `contracts`. There are 2 approaches to organize the subfolder based on the project's complexity.

1. In the case of the small solutions where every contract can be written in the separate files, the `contracts` should contain all the files directly.

   **Example**:

   ```jsx
   ├──  contracts/
   ├──────── example.ligo
   ├──────── fa2_token.ligo
   ```

2. In the case of the big ones with a bunch of dependencies, files, and imports, the subfolder should be divided into few subfolders:

   1. `main` : core parts of the smart contract that represent the contracts and contain `main` function.
   2. `partial` : the smart contracts code parts, grouped by functionality, relation to the smart contract etc; internal structure is optional.
   3. `vendor`(\*) : the code of the external libraries, projects and other dependencies.
   4. `interfaces`(\*) : the entrypoints and storage types of the contracts.

   **Example**:

   ```jsx
   ├──  contracts/
   ├──────── main/
   ├──────────── fa2_token.ligo
   ├──────────── dex.ligo
   ├──────────── farm.ligo
   ├──────── vendor/
   ├──────────── oracle.ligo
   ├──────── partial/
   ├──────────── fa2_types.ligo
   ├──────────── fa2_methods.ligo
   ├──────────── dex_types.ligo
   ├──────────── dex_methods.ligo
   ├──────────── farm_types.ligo
   ├──────────── farm_methods.ligo
   ├──────────── farm_lambdas.ligo
   ├──────────── common_helpers.ligo
   ```

### Builds

Once contracts are compiled their michelson code is stored in the builds folder in the `.json` and/or `.tz` format.

**Example:**

```jsx
├──  builds/
├──────── fa2_token.json
├──────── dex.json
├──────── farm.json
├──────── example.tz
```

### Migrations

The migrations should be properly ordered by starting the file name with `XX_`, where `XX` is the migration index starting from `00`. Each migration file normally contains the migrations for the only contract.

**Example:**

```jsx
├──  migrations/
├──────── 00_token_migration.js
├──────── 01_farm_migration.js
├──────── 02_dex_migration.js
```

### Scripts

The folder contains utility scripts that can be organized in an arbitrary way. They are mostly used for compilation, migrations, infrastructure needs, and so on.

**Example:**

```jsx
├──  scripts/
├──────── cli.js
├──────── helpers.js
├──────── managers.js
```

### Test

The subfolder contains the code that verifies the correctness of the smart contracts. The code can be organized in a preferred way, however, it is recommended to start the file names with the test scenario index to ensure the order won't be changed if new tests are added.

**Example**:

```jsx
├──  test/
├──────── 00_fa2_methods.js
├──────── 01_swap.js
├──────── 02_invest.js
├──────── 03_divest.js
├──────── 04_stake.js
├──────── 05_unstake.js
```

## File's naming

There are few rules for file and folder naming, some of which were already mentioned in the Subfolders paragraph:

- The names of files and folders should be in the snake case.
- The files that contain the compiled smart contract should be called by the name of the smart contract.
- The files that contain the compiled parts of the smart contract can have arbitrary names.
- The files that contain the main function of the smart contract should be named by the name of the contract.
- The files that contain the parts of the smart contracts should have the postfix that describes their content:
  - `_types` : for files with contract types.
  - `_methods`: for files with contract methods that are called be the entrypoints directly.
  - `_lambdas`: for files with contract lambdas that are usually intended to be stored in the contract storage.
  - `_helpers`: for files with different util functions used by the smart contract.
  - `_views`: for files with views (Hangzhou protocol update, view functions) cause views can't be saved in the storage as the lambdas (\*\*Tezos.call_view\*\* can't find the view function in the storage) and view functions shouldn't be mentioned in the main function.
- Migrations names are started by the index in the form `XX_` (`00_[<name>].[<type>]` is the first migration).
- Test names are started by the index in the form `XX_` (`00_[<name>].[<type>]` is the first test to be executed).

# Scripts

In the case of developing smart contracts as an npm project, the followed scripts must be supported:

- `compile`: the command compiles all the smart contracts along with their dependencies; i.e., if the smart contract requires some other internal and external stuff(lambdas, test contracts, etc) to be compiled it should happen at this step.
- `start-sandbox`: the command should run the sandbox node that is suitable for tests.
- `migrate`: the contract deploys the smart contracts to the network; `--network` option should be supported; if network isn't provided, the contracts should be originated to the development network.
- `test`: the command should run the tests in the development environment.

# Formatting

There a few general rules for Ligo code formatting:

- The indent is 2 spaces.

  **Example:**

  ```jsx
  if is_staked
  then {
    user.stake := abs(user.stake - fee);
    s.fees := s.fees + fee;
  }
  else skip;
  ```

- The max line length is 120 symbols.
- The names are written in the snake case (exceptions: the first later of the variant types and modules).
- There should be no space after `(` and before `)` brackets.

**Wrong**:

```jsx
| Transfer (params)          -> transfer (s, params)
| Update_operators (params)  -> (no_operations, update_operators (s, params))
| Balance_of (params)        -> (get_balance_of (s, params), s)
```

**Correct**:

```jsx
| Transfer(params)          -> transfer(s, params)
| Update_operators(params)  -> (no_operations, update_operators(s, params))
| Balance_of(params)        -> (get_balance_of(s, params), s)
```

- After the `[` and before `]` the space is used, except the case when `[]` means getting or setting the value by the key of the map or big map.
- When the expression is wrapped to the next line, it must be indented by 2 spaces.

**Wrong**:

```jsx
case (Tezos.get_entrypoint_opt(
"%wrap",
contract_address) : option(contract(wrap_t))) of
| Some(contr)           -> contr
| None                  -> failwith("Example/wrong-contract")
end;
amount_ := abs(amount_ * accuracy - s.fee_balances.dev -
   s.fee_balances.withdraw_burn) / accuracy;
```

**Correct**:

```jsx
case (Tezos.get_entrypoint_opt(
  "%wrap",
  contract_address) : option(contract(wrap_t))) of
| Some(contr)           -> contr
| None                  -> failwith("Example/wrong-contract")
end;

amount_ := abs(amount_ * accuracy - s.fee_balances.dev -
  s.fee_balances.withdraw_burn) / accuracy;
```

- The big numbers should be formatted with `_`.
  **Example:**
  ```jsx
  const precision: nat = 1_000_000_000n;
  ```

In case the project contains the code on other languages, the project either should follow default [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) config or contain the configuration file for formatting.

# Names

The variables, constants, types and functions names are written in the snake case. The only exception is the first laters of the variant types and modules which are written in the upper case due to compiler limitations.

The other rules:

- the max length of the variable and type name is 15 symbols;

**Wrong**:

```jsx
const balance_of_params : (address * nat) = (Tezos.sender * 100n);
```

**Correct**:

```jsx
const balance_request : (address * nat) = (Tezos.sender * 100n);
```

- the function name must start with the verb;

**Wrong**:

```jsx
function typed_transfer(
  const value           : nat;
  var s                 : storage_t)
                        : return_t is
  block {
  ...
```

**Correct**:

```jsx
function wrap_transfer(
  const value           : nat;
  var s                 : storage_t)
                        : return_t is
  block {
  ...
```

- the variables that contains the values multiplied by accuracy should have postfix `_f` to improve readability and make it easier to understand when normalisation with precision is required;

**Wrong**:

```jsx
(* wrong *)
const liquidity : nat = abs(token.total_liquid
  + token.total_borrows - token.total_reserves);

const redeem_amount : nat =
  if params.amount = 0n
  then user_info.balance * liquidity / token.total_supply / precision
  else params.amount;

if redeem_amount * precision > token.total_liquid
then failwith("Example/low-liquidity")
else skip;

var burn_tokens : nat := redeem_amount * precision *
  token.total_supply / liquidity;
if user_info.balance < burn_tokens
then failwith("Example/low-balance")
else skip;
```

**Correct**:

```jsx
(* correct *)
const liquidity_f : nat = abs(token.total_liquid_f
  + token.total_borrows_f - token.total_reserves_f);

const redeem_amount : nat =
  if params.amount = 0n
  then user_info.balance_f * liquidity_f / token.total_supply_f / precision
  else params.amount;

if redeem_amount * precision > token.total_liquid_f
then failwith("Example/low-liquidity")
else skip;

var burn_tokens_f : nat := redeem_amount * precision *
  token.total_supply_f / liquidity_f;
if user_info.balance < burn_tokens_f
then failwith("Example/low-balance")
else skip;
```

- the type names must have the postfix `_t`.
  **Example:**

  ```jsx
  type fees_t             is [@layout:comb] record [
    reinvest                : nat;
    dev                     : nat;
  ]

  type token_t            is [@layout:comb] record[
    address                 : address;
    id                      : nat;
  ]

  type token_standard_t   is
  | Fa12                     of address
  | Fa2                      of token_t

  type reinvest_t         is unit
  type change_fee_t       is fees_t
  ```

# Comments

There are two types of comments: block (\* \*) comments and line // comments.

It is recommended to use block comments in all cases.

In general, the code should be self-documented and only really needed explanatory comments should be provided: complicated values descriptions, formulas, dev notes, etc.

**Wrong**:

```jsx
(* get user info *)
const user_info : balance_info = get_map_info(
  user_account.balances,
  token_id
);

(* get token info *)
const token : token_info = get_token_info(token_id, param.s);

(* verify token is updated *)
verify_token(token);

(* calculated collateral value *)
param.res := param.res + ((user_info.balance * token.last_price
  * token.collateral_factor_f) * (abs(token.total_liquid_f
  + token.total_borrows_f - token.total_reserves_f)
  / token.total_supply_f) / precision);
```

**Correct**:

```jsx
const user_info : balance_info = get_map_info(
  user_account.balances,
  token_id
);
const token : token_info = get_token_info(token_id, param.s);

verify_token_updated(token);

(* collateral += collateral_factor * exchange_rate * collateral_price * lp_balance *)
param.collateral := param.collateral + ((user_info.balance * token.last_price
  * token.collateral_factor_f) * (abs(token.total_liquid_f
  + token.total_borrows_f - token.total_reserves_f)
  / token.total_supply_f) / precision);
```

The comment shouldn't break the type, if the comment is too long try to shorten it and write it in the same line. Ensure the comment is really needed

**Wrong**:

```jsx
type fees_t             is [@layout:comb] record [
  (* the % of reinvested amount goes to the reinvest caller *)
  reinvest                : nat;
  (* the % of reinvested amount that is converted to $TOKEN and burnt *)
  reinvest_burn           : nat;
  (* the % of withdrawed amount that is converted to $TOKEN and burnt  *)
  withdraw_burn           : nat;
  (* the % of withdrawed amount that is goes to dev fund  *)
  withdraw_dev            : nat;
]
```

**Correct**:

```jsx
(*
    Reinvest fees applied on the reinvested amount in reward token;
    Withdrawal fees applied on the withdrawal amount in deposit token;
*)
type fees_t             is [@layout:comb] record [
  reinvest_bounty         : nat;
  reinvest_burned         : nat;
  withdraw_burned         : nat;
  withdraw_dev            : nat;
]
```

# Types

Types are usually defined in the separate file according to the followed rules:

- Each type definition is written as `type [<name>]_t is [<type_definition>]`, where `is` are always 25th and 26th symbols in the line.

**Wrong**:

```jsx
type token_t is
[@layout:comb] record[
  address : address;
  id : nat;
]
type approve_token_t is
| Fa12_ of approve_fa12_token_t
| Fa2_ of approve_fa2_token_t

type token_standard_t is
| Fa12 of address
| Fa2 of token_t

type reinvest_t is nat
type change_fee_t is fees_t
```

**Correct**:

```jsx
type token_t            is [@layout:comb] record[
  address                 : address;
  id                      : nat;
]

type approve_token_t    is
| Fa12_                   of approve_fa12_t
| Fa2_                    of approve_fa2_t

type token_standard_t   is
| Fa12                    of address
| Fa2                     of token_t

type reinvest_t         is nat
type change_fee_t       is fees_t
```

- In case of the record type `[@layout:comb]` is highly recommended. `[@layout:comb] record [` is written on the same line as the type name, the fields and their types are written on the followed lines (pair field-type on each line) and `]` is written on the last line separately. The type definition `: [<type>]` always starts from the 27th symbol.

**Wrong**:

```jsx
type divest_t           is [@layout:comb] record [
  pair_id : nat;
  min_amounts_out : map(nat, nat);
  shares : nat;
]

type reserves_t is
[@layout:comb]
record [
  receiver                : contract(map(nat, nat));
  pair_id                 : nat;
]

type total_supply_t     is record [
  receiver                : contract(nat);
  pair_id                 : nat;
]
```

**Correct**:

```jsx
type divest_t           is [@layout:comb] record [
  pair_id                 : nat;
  min_amounts_out         : map(nat, nat);
  shares                  : nat;
]

type reserves_t         is [@layout:comb] record [
  receiver                : contract(map(nat, nat));
  pair_id                 : nat;
]

type total_supply_t     is [@layout:comb] record [
  receiver                : contract(nat);
  pair_id                 : nat;
]
```

- In case of the variant type, subtypes definition starts with the pipe. The subtypes with no arguments aren't followed by the type. The type definition `of [<type>]` in other cases always starts from the 27th symbol.

**Wrong**:

```jsx
type parameter_t        is
  | Deploy                of deploy_t
  | Change_owner          of address
  | Reset                 of unit

type parameter_t is
   Deposit of nat
 | Withdraw of nat
 | Reinvest of reinvest_t
```

**Correct**:

```jsx
type action_t           is
| Deploy                  of deploy_t
| Change_owner            of address
| Reset

type parameter_t        is
| Deposit                 of nat
| Withdraw                of nat
| Reinvest                of reinvest_t
```

# Control flow structures

The `if-else-then` are written in at least 3 lines: `if [<condition>]` on the first line, `then [<actions>]` starts from the new line as well as then `[<actions>]`. `if`, `then` and `else` start from the same intend.

**Wrong**:

```jsx
if account.balance < shares * accuracy
  then failwith("Example/low-balance");
  else skip;

if value = 10n then {
  value := 9n;
  s.winner := Tezos.sender;
} else s.score := abs(s.score - 1n);
```

**Correct**:

```jsx
if account.balance < shares * accuracy
then failwith("Example/low-balance");
else skip;

if value = 10n
then {
  value := 9n;
  s.winner := Tezos.sender;
}
else s.score := abs(s.score - 1n);
```

In case of the single actions block (i.e., when one of the options is `skip`), the condition should be built in the way where the action block is placed in the block after `then` and `else` will be followed by `skip`.

**Wrong**:

```jsx
if is_owner
then skip
else check_permision(sig);
if not is_outdated
then skip
else update(unit);
```

**Correct**:

```jsx
if not is_owner
then check_permision(sig)
else skip;
if is_outdated
then update(unit)
else skip;
```

# Functions

[@inline] should be written in the same line with the var or functions declaration.

**Wrong:**

```jsx
[@inline]
const a : nat = 1n;

[@inline]
function transfer_token(...)
```

**Correct:**

```jsx
const a : nat = 1n;

[@inline] function transfer_token(...)
```

The function declaration is split into the few lines. The first line contains the keyword and the function name in the forms `function [<name>](`. The next few lines describes the function arguments in the form `const|var [<name>] : type;` where the type definition `: [<type>]` always starts from the 25th symbol. The line with the last argument contains the close bracket. The last line contains the return type `: [<type>] is` definition that is intended and starts from the 25th symbol.

**Wrong**:

```jsx
function vote(const votes : nat; const storage : storage_t) : storage_t is
  block {
    storage.total_votes := storage.total_votes + votes;
  } with storage

function update_reward(
  const multiplier : nat;
  var storage : timestamp) : timestamp is
block {
  const owner : address = params.sender;
  var future_reward : nat := 0n;
  storage := Tezos.now;
} with storage
```

**Correct**:

```jsx
function vote(
  const votes           : nat;
  const storage         : storage_t)
                        : storage_t is
  block {
    storage.total_votes := storage.total_votes + votes;
  } with storage

function update_reward(
  const multiplier      : nat;
  var storage           : timestamp)
                        : timestamp is
  block {
    const owner : address = params.sender;
    var future_reward : nat := 0n;
    storage := Tezos.now;
  } with storage
```

# Exceptions

There is 3 approaches to raise an error in Ligo:

1. `failwith`
2. `assert_with_error` and `assert_some_with_error`
3. `assert` ad `assert_some`

Only the first two types should be used because they require an error message as an argument, which is helpful in understanding the reason. Among these two error types `assert_with_error` and `assert_some_with_error` are preferred because they make the code tidier.

An error message should be short in the followed form `[<contract_name>]/[<error_message>]`.

**Wrong**:

```jsx
if is_owner
then failwith("Example/not-permitted")
else skip;
assert_some(account_info);
assert_with_error(is_updated, "The X parameter isn't updated yet; check the update time in the storage");
```

**Correct**:

```jsx
assert_with_error(is_owner, "Example/not-permitted");
assert_some_with_error(account_info, "Example/unknown-account");
assert_with_error(is_updated, "Example/outdated-X");
```

# Values of complex types

When values of option or variant types are unwrapped, the first line contains `case [<name>] of`, the next few lines contains the variant - actions pairs in the form `| [<variant>] -> [<actions>]`, where all actions are indented to the beginning of the furtherest `->`. If the variant or option subtype wraps unit type, such arguments should be ommited.

**Wrong**:

```jsx
case token of
  Fa12(_) -> Tezos.transaction(
    (sender_,
    (receiver, amount_)),
    0mutez,
    get_contract(token_address))
| Fa2(token_) -> transfer_fa2(
    sender_,
    receiver,
    amount_,
    token_.id,
    token_address)
end;

case s.accounts[addr] of
| None(_) -> record[balance = 0n; permits = (set[]: set(address));]
| Some(account) -> account
end;

case (Tezos.get_entrypoint_opt(
  "%wrap",
  contract_address) : option(contract(wrap_t))) of
| Some(contr)           -> contr
| None                  -> failwith("Example/wrong-contract")
end;
```

**Correct**:

```jsx
case token of
| Fa12(_)     -> Tezos.transaction(
    (sender_,
    (receiver, amount_)),
    0mutez,
    get_contract(token_address))
| Fa2(token_) -> transfer_fa2(
    sender_,
    receiver,
    amount_,
    token_.id,
    token_address)
end;

case s.accounts[addr] of
| None          -> record[
    balance  = 0n;
    permits  = (set[]: set(address));
  ]
| Some(account) -> account
end;

case (Tezos.get_entrypoint_opt(
  "%wrap",
  contract_address) : option(contract(wrap_t))) of
| Some(contr) -> contr
| None        -> failwith("Example/wrong-contract")
end;
```

# Constants

The constant variables are defined in the module `Constants`.

**Example**:

```jsx
module Constants {
  const precision : nat = 1_000_000n;
}
```

```jsx
module Constants {
  const precision : nat = 1_000_000n;
}
```

Errors are defined in **_errors.ligo_** file in the module with the contract name this errors belongs to. And just use the modules to organize the code.

**Example:**

```jsx
module DexCore is {
  [@inline] const err_unknown_func    : nat = 0n;
  [@inline] const err_func_set        : nat = 1n;
  [@inline] const err_high_func_index : nat = 2n;
}

module TezStore is {
  [@inline] const err_not_dex_core : nat = 0n;
}

module Common is {
  [@inline] const err_not_admin : nat = 0n;
}
```
