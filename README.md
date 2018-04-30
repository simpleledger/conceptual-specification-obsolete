# Simple Ledger Protocol (SLP) Specificaion

The following specification defines a protocol for accounting of quantifiable Resources that will be allocated and transfered between various Entities using a single address on a blockchain. It was designed to be used with the bitcoin OP_RETURN opcode as a human readable payload when decoded from bytes to ASCII.  

## Introduction and Purpose
There are many business cases where only a single person or group is responsible for keeping track of items, but the data is relied on by other entities.  In these cases having an absolute immutable record of historical events may be just as important as the current allocation state.  

### Protocol Definitions:
| Term     | Definition                                                                             |
|----------|----------------------------------------------------------------------------------------|
| Ledger   | A series of immutable entries with prefixes and fields defined in this protocol        |
| Resource | A quantifiable item or object (e.g., company shares, products) being held by an entity |
| Entity   | A person, place, thing or category that resources are allocated to                     |
| Transfer | Signifies movement of a resource between the defined entities                          |

### Simplest Example SLP Entries on a blockchain (one blockchain data transaction per line):

| Entry # | Command  | Arguments                           |
|---------|----------|-------------------------------------|
| 1       | LEDGER   | version=1,name=ABC Inventory        |
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
| 1       | LEDGER   | version=1,name=ABC Inventory                          |
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

#### Resulting ledger snapshot/state calculated by a software application
|         | CompanyABC | XYZ Inc. | Global Inc |
|:-------:|:----------:|:--------:|:----------:|
| WidgetA |    70000   |   30000  |      0     |
| WidgetB |     75     |     0    |     25     |
|  FooBar |    1000    |     0    |    5000    | 


### Privacy on a Public Blockchain using Encryption
Encryption of fields containing sensitive name or quantity data (i.e., name=, qty= fields) is possible using a combination of assymetric keys and symetric keys.  This feature will allow someone to maintain an encrypted ledger with the ability to share only selected transaction data with third-parties using a passcode.

### Edit and Renaming capibilities
The protocol provide flexibility for the user by allowing for voiding of resource transfers and renaming of entities and resources so that they can be displayed nicely in the ledger's current state.
