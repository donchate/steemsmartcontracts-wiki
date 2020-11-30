Documentation written by [donchate](https://github.com/donchate)

**Note:** this smart contract is only available on Hive Engine, there is no Steem Engine equivalent.

# Table of Contents

* [Introduction](#introduction)
* Actions:
  * [create](#create)
  * [update](#update)
  * [setActive](#setactive)
  * [apply](#apply)
  * [resign](#resign)
  * [vote](#vote)
  * [unvote](#unvote)
  * [deposit](#deposit)
* [Tables available](#tables-available)
  * [params](#params)
  * [batches](#batches)

# Introduction

This contract allows a decentralized organization (i.e DAO) to collect and
distribute payments to a number of internal roles, each containing a configurable number of
primary and secondary candidates. The two classes of candidates determine the payout
calculation in a manner analagous to the Hive DPOS witness system.
The portion of payment allocated to primary candidates is divided evenly among each, while the secondary
candidates are ranked and paid according to the staked voting weight they have received.

A voter's weight is calculated as ```staked balance + pending unstake balance + delegations in```

The distribution batch only accepts tokens defined in its configuration, and is limited to one staked token
per instance. The distribution can be deactivated (suspended), during which time
new deposits are rejected and payouts are not made. If no eligible candidates are available, the contract will hold the token value until the next deposit and determination of voting weights.

# Actions available

### create:
Create a distribution batch to represent your roles and number of candidates per payout class.

A fee of 750 BEE is required.

Pre-requisites:
1. Determine the number of roles and their names
1. Determine the portion of the incoming deposits assigned to each role (```roles[].pct```), totalling 100% among roles.
1. Determine how many primary candidates each role will have (```roles[].primary```). Minimum 1.

Request:
* requires active key: yes
* can be called by: anyone
* parameters:
  * roles (list): List of accepted tokens
    * roles[].name (string <= 255): Role title
    * roles[].description (string <= 255): Brief description of role and/or requirements
    * roles[].pct (integer >= 1 and <= 100): Percentage of incoming deposit to allocate to this role
    * roles[].primary (integer >= 1 and <= 40): Number of selectable primary candidates. Any additional become classified as secondary.
  * stakeSymbol (string): Valid token symbol to use for deposits, staking and payout

* example request data:
```
{
  "roles": [
    {
      "name": "President", 
      "description": "Very important executive role", 
      "pct": 25,
      "primary": 1
    }, 
    {
      "name": "Vice President", 
      "description": "Important executive role", 
      "pct": 25, 
      "primary": 2
    },
    {
      "name": "Treasurer", 
      "description": "Important financial role",
      "pct": 25, 
      "primary": 2
    },    
    {
      "name": "Board Member", 
      "description": "Member at Large", 
      "pct": 25, 
      "primary": 4
    }
  ], 
  "stakeSymbol": "TKN", 
}
```

### update
The instance creator may update the role definitions and properties using this action.
Be cautious when using this action as any roles removed will also purge the associated candidates and votes from the database. If the role is later re-created, candidates will have to apply and obtain votes again.

A fee of 500 BEE is required.

Request:
* requires active key: yes
* can be called by: anyone
* parameters:
  * id (integer): ID of the distribution instance
  * roles (list): List of accepted tokens
    * roles[].name (string <= 255): Role title
    * roles[].description (string <= 255): Brief description of role and/or requirements
    * roles[].pct (integer >= 1 and <= 100): Percentage of incoming deposit to allocate to this role
    * roles[].primary (integer >= 1 and <= 40): Number of selectable primary candidates. Any additional become classified as secondary.

* example request data:
```
{
  "id": 1,
  "roles": [
    {
      "name": "President", 
      "description": "Very important executive role", 
      "pct": 25,
      "primary": 1
    }, 
    {
      "name": "Vice President", 
      "description": "Important executive role", 
      "pct": 25, 
      "primary": 2
    },
    {
      "name": "Treasurer", 
      "description": "Important financial role",
      "pct": 25, 
      "primary": 2
    },    
    {
      "name": "Board Member", 
      "description": "Duties and expectations", 
      "pct": 25, 
      "primary": 5
    }
  ]
}
```

### setActive
For enabling and disabling the distribution contract. When disabled, no payments will be made
and no new deposits are accepted. Newly created distributions need to be activated.
No fee required.

* requires active key: yes
* can be called by: contract creator
* parameters:
  * id (int): ID of distribution instance
  * active (boolean): true to activate, false to deactivate

* activate a distribution:
```
{
  "id": 1, 
  "active": true, 
  "isSignedWithActiveKey": true
}
```
* deactivate a distribution:
```
{
  "id": 1, 
  "active": false, 
  "isSignedWithActiveKey": true
}
```

### apply
Accounts that wish to register themselves for a role must apply using this function to receive votes.
No fee required.

* requires active key: no
* can be called by: anyone
* parameters:
  * id (int): ID of distribution instance
  * role (string): role name to apply for (from ```roles[]```)

* example request data:
```
{
  "id": 1, 
  "role": "President"
}
```

### resign
Accounts may resign from a role using this action. Upon resignation, vote history will be cleared.
No fee required.

* requires active key: no
* can be called by: anyone
* parameters:
  * id (int): ID of distribution instance
  * role (string): role name to resign from (from ```roles[]```)

* example request data:
```
{
  "id": 1, 
  "role": "President"
}
```

### vote
Accounts with staked tokens can vote for candidates and roles using this action.
No fee required.

* requires active key: no
* can be called by: anyone
* parameters:
  * id (int): ID of distribution instance
  * role (string): role name to vote for (from ```roles[]```)
  * to (string): candidate account to vote for (from ```candidates[]```)

* example request data:
```
{
  "id": 1, 
  "role": "President",
  "to": "donchate"
}
```

### unvote
Accounts with staked tokens can remove their vote for candidates and roles using this action.
No fee required.

* requires active key: no
* can be called by: anyone
* parameters:
  * id (int): ID of distribution instance
  * role (string): role name that vote was for (from ```roles[]```)
  * to (string): candidate account that vote was for for (from ```candidates[]```)

* example request data:
```
{
  "id": 1, 
  "role": "President",
  "to": "donchate"
}
```

### deposit
The contract will perform a transfer of the configured ```stakeSymbol``` token using this action. A payment distribution will be attempted immediately following a valid deposit.
No fee required.

* requires active key: yes
* can be called by: anyone
* parameters:
  * id (int): ID of distribution instance
  * symbol (string): Symbol of token being deposited
  * quantity (string): string decimal up to staked token max precision

* example request data:
```
{
  "id": 1, 
  "symbol": "TKN", 
  "quantity": "1000"
}
```

# Tables available

## params:
contains internal contract parameters
* fields
  * distCreationFee = the cost in BEE to create an instance
  * distUpdateFee = the cost in BEE to update an instance

## batches:
contains the instance configurations
* fields
  * _id = MongoDB internal primary key
  * roles = JSON object of configured roles
  * candidates = JSON object of candidates who have applied for a role and their achieved vote weight
  * votes = JSON object of each vote recorded
  * voters = JSON object of accounts that voted in this contract instance and their staking weight
  * stakeSymbol = Token symbol for deposits and payout
  * dustWeight = Calculated minimum threshold for candidate eligiblity (currently set to 1% of token supply)
  * active = boolean whether distribution is enabled or not
  * creator = account name of distribtuion creator