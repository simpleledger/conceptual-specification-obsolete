 # Simple Ledger Protocol (SLP) Specification

This specification defines a protocol for using a single blockchain address as a ledger for performing accounting and bookkeeping on a blockchain.  Any quantifiable resource allocated between any number of entities can be kept track of using a single blockchain address.  The protocol is designed to be used with the bitcoin OP_RETURN opcode as a human readable payload when decoded from bytes to ASCII. The protocol can be implemented on any blockchain that provides ample room for data entry (e.g., the Bitcoin Cash blockchain provides 220-byte OP_RETURN data allowance as of May 15, 2018).

## Introduction and Purpose
There are many useful business cases where only a single person or group is responsible for keeping track of items, but the data is relied upon by other entities.  In these cases, having an immutable record of historical events is often just as important as knowing the current allocation state of resources.  For such purposes having a simple protocol for immutable bookkeeping of resources is defined herein.

An important part of any accounting system is to provide a way to resolve discrepancies and errors that exists within the ledger.  The protocol provides rules for a Validation Service Provider (VSP) to check the state of such a ledger and notify the ledger's address when errors do exist.  Using a VSP is not a required part of the protocol but it will be an essential feature for ledgers that have importance.  

Digital assets and non-digital assets (i.e., resources) can be tracked using SLP, but more powerful protocol features will exist for digital assets that exist on some blockchain.  The protocol will allow an entity to be associated with digital asset addresses for any blockchain, virtual blockchain, or colored coin protocol.  A validation service provider can then examine balances of blockchain addresses and identify differences between actual and expected balances for the digital assets.

### Example Use Cases:
* Track blockchain assets
* Maintain a company's stock ledger
* Perform government accounting & bookkeeping
* Track inventory for any product
* Manage and track multiple loan balances

### Terms and Definitions:
| Term     | Definition                                                                                                                |
|----------|---------------------------------------------------------------------------------------------------------------------------|
| Ledger   | A series of immutable entries with prefixes and fields defined in this protocol                                           |
| Resource | A quantifiable item, object, asset, or liability (e.g., company shares, products, digital assets) being held by an entity |
| Entity   | A person, place, thing or category that resources are allocated to                                                        |
| Transfer | Indicates movement of a resource between the defined entities                                                             |
| Void     | Indicates a previously made transaction should be ignored in the ledger calculation                                       |
| Update   | Indicates updating of field data for a particular entity or resource                                                      |
| Validation Service Provider | A company that is validating transaction compliance with the SLP protocol                              |

### Simplest Example (non-digital resources)

| Entry # | Command  | Arguments                           |
|---------|----------|-------------------------------------|
| 1       | LEDGER   | slpver=1,name=ABC Inventory         |
| 2       | ENTITY   | eid=1,name=CompanyABC               |
| 3       | RESOURCE | rid=1,name=WidgetA,qty=100000,eid=1 |
| 4       | ENTITY   | eid=2,name=XYZ Inc.                 |
| 5       | TRANSFER | rid=1,from=1,to=2,qty=30000         |

#### Resulting ledger calculated by a software application

|         | CompanyABC | XYZ Inc. |
|:-------:|:----------:|:--------:|
| WidgetA |    70000   |   30000  |

### Advanced Example (non-digital resources)

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

#### Resulting ledger calculated by a software application
|         | CompanyABC | XYZ Inc. | Global Inc |
|:-------:|:----------:|:--------:|:----------:|
| WidgetA |    70000   |   30000  |      0     |
| WidgetB |     75     |     0    |     25     |
|  BarFoo |    1000    |     0    |    5000    | 

### Privacy on a Public Blockchain using Encryption
Encryption of fields containing sensitive name or quantity data (i.e., name=, qty= fields) is possible using a combination of asymmetric keys and symmetric keys.  This feature will allow someone to maintain an encrypted ledger with the ability to share only selected transaction data with third-parties using a passcode.  Everyone will be able to see the command, but the certain field data can be hidden with a password.

### Edit and Renaming capabilities
SLP defines voiding of resource transfers and renaming of entities and resources so that they can be displayed nicely in the ledger's current state.

### Upgrading a ledger's protocol version
Any ledger utilizing a particular SLP version can change to another SLP version by simply using the LEDGER command again with the desired version specified.  Since the blockchain economics and protocol capibilities will change over time it will be important to be able to switch to a different SLP version.  For example, if many ledger entries are being made through an API then transactions may be expensive so a much less verbose SLP protocol will be desired to reduce cost.  Another scenario may be that one protocol version may have more features than another.  

### Timestamps
A transaction or transfer of resources that happens in the real world will likely occur at a different time from when the associated ledger entry is made on the blockchain.  For this reason, an optional timestamp field can be included for TRANSFER entries.  

### Multiple OP_RETURN Spaces
Multiple outputs with OP_RETURN included in their script provides additional data storage.  The protocol will utilize multiple transaction output in some cases as appropriate.

### Blockchain Registry
The SLP will provide a registry of identifiers so that various blockchains, virtual blockchains, and colored coins can be referenced efficiently in an SLP transaction.  

### Inter-ledger communications
TODO

## SLP Version 0

The initial version of SLP designed for simple human readable ledger entries within 220-bytes.  

### Commands to be used by a user at a single bitcoin address
| 8-byte Command Prefix | Required Arguments    | Optional Arguments  | Description                                             |
|:---------------------:|:---------------------:|:-------------------:|:-----------------------------------------------------|
| LEDGER                | slpver=               | name=, date=, chain=| Create a new ledger or change to specified version   | 
| ENTITY                | eid=                  | name=, addr=, chain=| Create a new entity in the ledger                    |
| RESOURCE              | rid=, qty=, eid=      | name=               | Create a new resource in the ledger with initial assignment and quantity allocation |
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
1) After satisfying a validation service's join requirements for tracking and validating your address's ledger you should have a message received from that validation service to confirm your address is actively being validated by the service.  This is an optional step, but without joining a validation service your address will not be actively validated for errors and discrepancies.
2) Your address must have a message to self with the protocol message of "LEDGER", where everything afterwards would be part of the ledger parsing

# SLP Version 1 - Future

Include user commenting system and encryption.

