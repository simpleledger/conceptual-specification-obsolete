# Simple Ledger Protocol (SLP) Specification

This specification defines a protocol for using a single blockchain address as a ledger for performing accounting and bookkeeping on a blockchain.  Any quantifiable resource allocated between any number of entities can be kept track of using a single blockchain address.  The protocol is designed to be used with the bitcoin OP_RETURN opcode as a human readable payload when decoded from bytes to ASCII. The protocol can be implemented on any blockchain that provides ample room for data entry (e.g., the Bitcoin Cash blockchain provides 220-byte OP_RETURN data allowance as of May 15, 2018).

## Introduction and Purpose
There are many useful business cases where only a single person or group is responsible for keeping track of items, but the data is relied on by other entities.  In these cases having an absolute immutable record of historical events may be just as important as knowing the current allocation state.  For such purposes having a simple protocol for immutable bookkeeping of resources is defined herein.

An important part of any accounting system is to provide a way to resolve descrepancies and errors that exists within the ledger.  The protocol provides a standard way for a validation service provider to check the state of the ledger and notify the address when errors exist.  Using a validation service provider is not a required part of the protocol, but would be essential feature for any serious ledger.  

An exciting feature of the protocol provides a standardized way to associate entities with blockchain addresses so that bookkeeping can be done on actual balances of an address on the blockchain.  A validation service provider can then compare balances of blockchain addresses to resolve differences between the ledger.

### Example Use Cases:
* Keeping tabs between roommates
* Company's stock ledger
* Government accounting & bookkeeping
* Product inventory
* Loan balance
* Tracking and bookkeeping of actual bitcoin transactions

### Terms and Definitions:
| Term     | Definition                                                                             |
|----------|----------------------------------------------------------------------------------------|
| Ledger   | A series of immutable entries with prefixes and fields defined in this protocol        |
| Resource | A quantifiable item or object (e.g., company shares, products) being held by an entity |
| Entity   | A person, place, thing or category that resources are allocated to                     |
| Transfer | Indicates movement of a resource between the defined entities                          |
| Void     | Indicates a previously made transaction should be ignored in the ledger calculation    |
| Update   | Indicates updating of field data for a particular entity or resource                   |
| Implementation| The company validating your transactions comply with the SLP protocol             |
| Comment  | Indicates a note is being made about a transaction on the blockchain                   |
| Password | Indicates a password is to be used for certain data fields in a transaction            |


### Simplest Example

| Entry # | Command  | Arguments                           |
|---------|----------|-------------------------------------|
| 1       | LEDGER   | slpver=1,name=ABC Inventory        |
| 2       | ENTITY   | eid=1,name=CompanyABC               |
| 3       | RESOURCE | rid=1,name=WidgetA,qty=100000,eid=1 |
| 4       | ENTITY   | eid=2,name=XYZ Inc.                 |
| 5       | TRANSFER | rid=1,from=1,to=2,qty=30000         |

#### Resulting ledger snapshot/state calculated by software application

|         | CompanyABC | XYZ Inc. |
|:-------:|:----------:|:--------:|
| WidgetA |    70000   |   30000  |

### Advanced Example

| Entry # | Command  | Arguments                                             |
|---------|----------|-------------------------------------------------------|
| 1       | LEDGER   | slpver=1,name=ABC Inventory                           |
| 2       | ENTITY   | eid=1,name=CompanyABC                                 |
| 3       | ENTITY   | eid=2,name=XYZ Inc.                                   |
| 4       | RESOURCE | rid=1,name=WidgetA,qty=100000,eid=1                   |
| 5       | RESOURCE | rid=2,name=WidgetB,qty=100,eid=1                      |
| 6       | ENTITY   | eid=3,name=Global Inc.                                |
| 7       | RESOURCE | rid=3,name=FooBar,qty=6000,eid=3                      |
| 8       | TRANSFER | rid=1,from=1,to=2,qty=30000                           |
| 9       | TRANSFER | rid=3,from=3,to=1,qty=2000 <--(had bitcoin txn=12345) |
| 10      | TRANSFER | rid=3,from=3,to=1,qty=1000                            |
| 11      | VOIDTRAN | bitcointxnid=12345                                    |
| 12      | TRANSFER | rid=2,from=1,to=3,qty=25                              |
| 13      | UPDATE   | type=entity,id=3,name=BarFoo                          |

#### Resulting ledger snapshot/state calculated by a software application
|         | CompanyABC | XYZ Inc. | Global Inc |
|:-------:|:----------:|:--------:|:----------:|
| WidgetA |    70000   |   30000  |      0     |
| WidgetB |     75     |     0    |     25     |
|  BarFoo |    1000    |     0    |    5000    | 


### Privacy on a Public Blockchain using Encryption
Encryption of fields containing sensitive name or quantity data (i.e., name=, qty= fields) is possible using a combination of assymetric keys and symetric keys.  This feature will allow someone to maintain an encrypted ledger with the ability to share only selected transaction data with third-parties using a passcode.  Everyone will be able to see the command, but the certain field data can be hidden with a password.

### Edit and Renaming capibilities
SLP defines voiding of resource transfers and renaming of entities and resources so that they can be displayed nicely in the ledger's current state.

### Upgrading a ledger's protocol version
Any ledger utilizing a particular SLP version can change to another SLP version by simply using the LEDGER command again with the desired version specified.  Since the blockchain economics and it's protocol capibilities will change over time it will be important to be able to switch to a different SLP version.  For example, if many ledger entries are being made through an API then transactions may be expensive so a much less verbose SLP protocol will be desired to reduce cost.  Another scenario may be that one protocal version may have more features than another.  

### Timestamps
A transaction or transfer of resources that happens in the real world will likely occur at a different time from when the associated ledger entry is made on the blockchain.  For this reason an optional timestamp field will be included for TRANSFER entries.  

## SLP Version 0

The initial version of SLP designed for simple human readible ledger entries within 220-bytes.  

### Commands to be used by a user at a single bitcoin address
| 8-byte Command Prefix | Required Arguments    | Optional Arguments  | Description                                             |
|:---------------------:|:---------------------:|:-------------------:|:-----------------------------------------------------|
| LEDGER                | slpver=               | name=, date=, chain=| Create a new ledger or change to specified version   | 
| ENTITY                | eid=                  | name=, addr=, chain=| Create a new entity in the ledger                    |
| RESOURCE              | rid=, qty=, eid=.     | name=               | Create a new resource in the ledger with initial assignment and quantity allowcation |
| TRANSFER              | from=, to=, rid=, qty=| date=               | Move an allocation of resource from one entity to another entity |
| UPDATE                | type=, id=,           | name=, addr=, chain=| Update at least one of the optional arguments for an entity, resource, or the ledger |

### Argument requirements
| Argument          | Type      | Encoding  | Bytes   | Representation                                                      |
|:-----------------:|:---------:|:---------:|:-------:|:-------------------------------------------------------------------:|
| slpver=           | number    |  byte     | 1 exact | representing up to 256 versions                                     | 
| date=             | number    |  bytes    | 4 exact | representing POSIX time manually set date stamp by user             |
| eid=              | number    |  bytes    | 2 max   | representing up to 65536 possible entity ids in a single ledger     |
| rid=              | number    |  bytes    | 2 max   | representing up to 65536 possible resource ids in a single ledger   |
| qty=              | number    |  bytes    | 4 max   | big enough to hold bit                                              |
| name=, to=, from= | string    |  ascii    | 100 max | large enough to hold a bitcoin cash address as name plus some       |
| chain=            | string    |  ascii    | 10 max  | moniker for ledger or addr's blockchain(e.g. bitcoincash, etherium) |
| addr=             | string    |  ascii    | 100 max | blockchain address associated with the entity                       |
| type=             | string    |  ascii    | 8 max   | can be one of: entity, resource, ledger                             |

### Collision checking between the user's desired values and the arguments that could need to be parsed. This ensures a valid SLP entry
| Argument |  Hex from ASCII | Byte count |  
|:--------:|:---------------:|:----------:|
| slpver=  | 736c707665723d  |     7      |
| name=    | 6e616d653d      |     6      |
| eid=     | 6569643d        |     4      |
| rid=     | 7269643d        |     4      |
| qty=     | 7174793d        |     4      |
| date=    | 646174653d      |     5      |
| from=    | 66726f6d3d      |     5      | 
| to=      | 746f3d          |     3      |
| addr=    | 616464723d      |     5      |
| chain=   | 636861696e3d    |     6      |
| type=    | 747970653d      |     5      |         

### Commands to be sent from a special address maintained by a validation service for the SLP protocol.
| 8-byte Command Prefix | Required Arguments | Optional Arguments  | Description |
|:---------------------:|:------------------:|:-------------------:|:---------------------------------------------------|
| SIMPLELEDGER          | version=           |                     | Mark in the blockchain when the implementation starts accepting the version indicated, sent from implementation address to self |
| TRACKEDBY             | url=               |                     | Transaction sent from an SLP validation service address to a user's address that is being tracked by the implementation's internal database - this also acts as a backup for the validation service in case implementation's database of tracked addresses is lost.  Also, an address may be tracked by multiple entities validating the address data against the SLP protocol. |
| UNTRACKEDBY           | url=               |                     | Notification from validation service address to user's address that the address is no longer being actively tracked |
| MESSAGE               | message=           |                     | Error or other message sent from the implementation address to a user's tracked address |

### Rules
1) After satisfiying an implementation join requirements for tracking and validating your address's SLP ledger you should have a message received from that implementation to indicate the address is actively being tracked for ledger errors and notifications from the service.  This is an optional rule/step, however, without joining a service that actively validates your SLP ledger you will need to do it on your own.
2) Address must have a message to self with LEDGER, where everything afterwards would be part of the ledger parsing

# SLP Version 1 - Future

Include user commenting system and encryption.

