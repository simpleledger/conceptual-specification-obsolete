# Simple Ledger Protocol (SLP) Specification

This specification defines a protocol for accounting of resources to be allocated and transferred between various entities using a single address on a blockchain. It was originally designed to be used with the bitcoin OP_RETURN opcode as a human readable payload when decoded from bytes to ASCII. The protocol can be implemented on any blockchain that provides ample room for data entry (e.g., the Bitcoin Cash blockchain provides 220-byte OP_RETURN data allowance as of May 15, 2018).  

## Introduction and Purpose
There are many useful business cases where only a single person or group is responsible for keeping track of items, but the data is relied on by other entities.  In these cases having an absolute immutable record of historical events may be just as important as knowing the current allocation state.  For such purposes having a simple protocol for immutable bookkeeping of resources is defined herein.

### Example Use Cases:
* Company stock ledger
* Product inventory
* Loan balance

### Terms Definitions:
| Term     | Definition                                                                             |
|----------|----------------------------------------------------------------------------------------|
| Ledger   | A series of immutable entries with prefixes and fields defined in this protocol        |
| Resource | A quantifiable item or object (e.g., company shares, products) being held by an entity |
| Entity   | A person, place, thing or category that resources are allocated to                     |
| Transfer | Indicates movement of a resource between the defined entities                          |
| Void     | Indicates a previously made transaction should be ignored in the ledger calculation    |
| Rename   | Indicates resetting of the current given name of a particular entity or resource       |
| Comment  | Indicates a note is being made about a transaction on the blockchain                   |
| Password | Indicates a password is to be used for certain data fields in a transaction            |

### Simplest Example SLP Entries on a blockchain (one blockchain data transaction per line):

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

### Advanced Example SLP Entries (multiple resources, voided transactions, and renaming of entities)

| Entry # | Command  | Arguments                                             |
|---------|----------|-------------------------------------------------------|
| 1       | LEDGER   | slpver=1,name=ABC Inventory                          |
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
| 13      | RENAME   | type=entity,id=3,name=BarFoo                          |

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
Any ledger utilizing a particular SLP version can change to another SLP version by simply using the LEDGER command again with the desired version specified.  Since the blockchain economics and it's protocol capibilities will change over time it will be important to be able to switch to a different SLP version.  For example, if many ledger entries are being made through an API then transactions may be expensive so a much less verbose SLP protocol will be desired to reduce cost.  Another scenario may be that one protocal version may have more features than another.  For each manually set timestamp is desired with each ledger's entries in order to set an "effective date" which may be included in one protocol that doesn't exist in another.

## SLP Version 0

The initial version of SLP designed for simple human readible ledger entries within 220-bytes

### Commands to be used by a user at a single bitcoin address
| 8-byte Command Prefix | Required Arguments | Optional Arguments  | Description |
|:------------------:|:--------------------------:|:--------------------:|:----------------------------------------------|
| LEDGER     | slpver=          |    name=, date=            | Create a new ledger or change to specified version | 
| ENTITY     | eid=            |     name=                 | Create a new entity in the ledger                  |
| RESOURCE   | rid=, qty=, eid=    |     name=            | Create a new resource in the ledger with initial assignment and quantity allowcation |
| TRANSFER   | from=, to=, rid=, qty=,    |  date=               | Move an allocation of resource from one entity to another entity |

### command arguments with specific byte allocation requirements
| argument  | max value bytes | value representation |
|:---------:|:---------------:|:---------------------|
| slpver=   | 1               | representing up to 128 versions | 
| date=     | 8               | representing UTC time in seconds for manually set date stamp by user |
| eid=      | 1               | representing up to 128 identifiable entities in a single ledger |
| rid=      | 1               | representing up to 128 identifiable resources in a single ledger |


### Hex to be used for checking for collisions between the user's entered values and the arguments that could need to be parsed. This ensures a valid SLP entry
| argument |  hex from ascii |  
|:--------:|:---------------:|
| slpver=  | 736c707665723d  |
| name=    | 6e616d653d      |
| eid=     | 6569643d        |
| rid=     | 7269643d        |
| qty=     | 7174793d        |
| date=    | 646174653d      |
| from=    | 66726f6d3d      |
| to=      | 746f3d          |

### Optional: Commands to be sent from a special address maintained by an implementation of the protocol.
| 8-byte Command Prefix | Required Arguments | Optional Arguments  | Description |
|:------------------:|:--------------------------:|:--------------------:|:---------------------------------------------------|
| SIMPLELEDGER       | version=          |              | Mark in the blockchain when the implementation starts tracking the version indicated

# SLP Version 1 - Future

Include commenting system and encryption.

