# Simple Ledger Protocol (SLP) Specification WIP

This specification defines a protocol for using a single blockchain address as a ledger for tracking resources on a blockchain.  Any quantifiable resource allocated between any number of entities can be tracked using a single blockchain address.  The protocol is designed to be used with the Bitcoin Cash OP_RETURN space as a human readable payload when decoded from bytes to ASCII. The protocol can be implemented on any blockchain that provides ample room for data entry (e.g., the Bitcoin Cash blockchain provides 220-byte OP_RETURN data allowance as of May 15, 2018).

SLP can be used as a method to issue and transfer "colored" coins or create and maintain virtual blockchain containing one or more digital assets.  These goals can be accomplished by using an internal SLP ledger to track creation, destruction, and transfer of its own digital assets between holding entities.

As part of the SLP project a public registry will be created to where virtual blockchains and colored coins using SLP can publish their address if desired.  The public registry will promote interoperability between virtual blockchains utilizing SLP.  The registry itself will be a SLP ledger and during the registration process each virtual chain's SLP address will be registered and a unique moniker may be provided.  This public registry will be relied upon by virtual blockchains for promoting interoperability between virtual chains. 

## Introduction and Purpose
There are many useful business cases where only a single person or group is responsible for keeping track of items, and the data is also relied upon by others.  In these cases, having an immutable record of historical events is often just as important as knowing the current allocation state of resources.  For such purposes having a simple protocol for immutable bookkeeping of resources on a blockchain is defined herein.

An important part of any computerized bookkeeping system is the prevention of errors that exist within the ledger's entries.  To that end, SLP provides specific guidelines for a Validation Service Provider (VSP) to check the state and validity of a SLP ledger, and to also make valid SLP ledger transactions.  The VSP is an essential component of ledgers that have any importance.

A virtual blockchain may use SLP as a method to keep track of its own state.  If many virtual blockchains utilize SLP and the public SLP registry the ability for virtual blockchains to interoperate will be enhanced since VSPs can be built to be aware of other virtual chains. will be enh  will need a space to store their  or colored coin may want to build and run it's own VSP implementation in order to handle the customized requirements of that particular chain.  VSPs will either built to be general purpose or highly specialized.

The VSP should first provide protocol validation prior to making a SLP ledger transaction, however, post-transaction validation by the VSP should also be performed whenever state is fetched.  If errors are found during a post-transaction validation check (because errors were not properly validated during the pre-transaction check) the VSP should then send notify the ledger's address with a human readable OP_RETURN message with an appropriate error message so that the ledger can be corrected manually by the user. A VSP can examine balances of digital asset addresses and identify differences between actual and expected balances for those digital assets being tracked with an SLP ledger.

### Example Use Cases (using a single bitcoin address):
* Track multiple types of blockchain assets (e.g., colored coins and virtual chain assets)
* Maintain a company's stock ledger with multiple types/series of stock
* Track inventory for multiple products
* Manage and track balances and payments of multiple loans or accounts receivable
* Store a public registry of various entities and resources

### Obtaining New SLP Digital Assets
To obtain a new issuance of an SLP asset you may simply send BCH to the SLP address owned by the colored coin or virtual blockchain of interest.  In return the responsible VSP maintaining the associated SLP ledger will detect your transaction and assign its digital asset to your bitcoin address within its SLP ledger.  You will be issued digital asset at an exchange rate that is published by the colored coin, VSP, on the public registry, or fair market value.  If your transaction does not meet minimum requirements (such as price or there is no new asset available to issue) then the VSP can respond back using an appropriate OP_RETURN error message that your SLP compliant wallet can detect.

### Transferring SLP Digital Assets
To transfer the digital asset you can use an SLP compliant web service or offline wallet that is aware of your asset holdings (using the public registry to look up SLP ledgers) and will allow you to transfer your digital asset using an OP_RETURN command such as `TRANSFER chain=<SLP ledger address or moniker> to=<recipeint addr>`.

### A Zero-balance Bitcoin Address Still Holds SLP Assets
SLP allows you to hold an infinite number of digital assets even with a zero balance at your bitcoin address.  This is because the each SLP compliant asset maintains its own ledger which records and indicates how much asset you own.  You only need to maintain a balance at your bitcoin address if you would need to make an SLP asset transfer in order to satisfy the underlying bitcoin cash network transaction requirements.  This is a significant improvement over other protocols where colored coins can be lost if the colored coins are used as inputs in a transaction which isn't a transfer transaction, their value is lost, it is not transferred to outputs of this or other transaction. Also the value of EPOBC colored coins might be lost in a malformed transaction.  With SLP a malformed transaction would just simply be ignored by the VSP responsible for maintaining the colored coin's ledger.

### Terms and Definitions:
| Term     | Definition                                                                                                      |
|----------|-----------------------------------------------------------------------------------------------------------------|
| Ledger   | A series of immutable entries with prefixes and fields defined in this protocol                      |
| Resource | A quantifiable item, object, asset, or liability (e.g., company shares, products, digital assets) being held by an entity |
| Entity   | A person, place, thing or category that resources are allocated to                               |
| Transfer | Indicates movement of a resource between the defined entities                                            |
| Void     | Indicates a previously made transaction should be ignored in the ledger calculation                  |
| Update   | Indicates updating of field data for a particular entity or resource          |
| Validation Service Provider | A company that is validating transaction compliance with the SLP protocol. |                          | Colored Coins | Colored coins and Virtual Blockchains are loosely defined terms for systems that issues transferable digital assets on an existing host blockchain. SLP is just one protocol that can be used to build and maintain either of these. |

### Simplest Example

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

### Virtual Chain Registry (https://virtualchain.cash)
The SLP will provide a public registry of identifiers so that various blockchains, virtual blockchains, and colored coins can be referenced efficiently in an SLP transaction.  This public registry itself will be stored at a single bitcoin address using the SLP with it's key maintained by a VSP.  For a small fee anyone will be allowed to register their virtual blockchain's moniker on this registry.

### Inter-ledger communications (still needs deep thought)
SLP requires that any address using SLP shall ignore any OP_RETURN outputs that come from unapproved addresses. In other words, by default only ledger entries that have come from the same address should be included.  Otherwise, someone could easily mess up your ledger by just sending OP_RETURN messages to your address.  Nothing except transaction fees would detour someone from sending OP_RETURN messages to your address if they wanted to.  VSP following SLP will know to ignore addresses that have not been recorded in your ledger with permissions.

## SLP Version 0

The initial version of SLP designed for simple human readable ledger entries within 220-bytes.  

### Commands and arguments used by a VSP to maintain a SLP ledger
| 8-byte Command Prefix | Required Arguments    | Optional Arguments  | Description                                          |
|:---------------------:|:---------------------:|:-------------------:|:-----------------------------------------------------|
| LEDGER                | slpver=               | name=, date=, registry= | Create a new ledger or change to specified version   | 
| ENTITY                | eid=                  | name=, addr=        | Create a new entity in the ledger                    |
| RESOURCE              | rid=, qty=, eid=      | name=               | Create a new resource in the ledger with initial assignment and quantity allocation |
| TRANSFER              | from=, to=, rid=, qty=| date=.              | Move an allocation of resource from one entity to another entity |
| UPDATE                | type=, id=,           | name=, addr=, chain=| Update at least one of the optional arguments for an entity, resource, or the ledger |
| MESSAGE               | msg=, eid=            |                     | Create a message to be sent to an entity's address, handled by a VSP.  VSP should provide confirmation response when complete. |
| COMMENT               | msg=                  | txn=, chain=, replyto=| Create a generic comment about any transaction on any chain, can also reply to a previously made comment using chain's txn id |
| PERMIT                | level=, addr=, chain= |                    | Specify access permissions to this ledger for an address on a particular chain (e.g. one or more VSPs on main chain or even an address from a virtual chain) |

### Commands and arguments used by a bitcoin address to interact with a VSP, to trigger SLP ledger entry
| 8-byte Command Prefix | Required Arguments      | Optional Arguments  | Description                                        |
|:---------------------:|:-----------------------:|:-------------------:|:---------------------------------------------------|
| TRANSFER              | to=, qty=, chain=       | rid=                | Move an allocation of resource from one entity to another entity |

### Argument requirements
| Argument          | Type      | Encoding  | Bytes   | Representation                                                      |
|:-----------------:|:---------:|:---------:|:-------:|:-------------------------------------------------------------------:|
| slpver=           | number    |  byte     | 1 exact | representing up to 256 versions                                     |
| registry=         | string    |  ascii    | 100 max | the url to a virtual blockchain registry                            |
| date=             | number    |  bytes    | 4 exact | representing POSIX time manually set date stamp by user             |
| eid=              | number    |  bytes    | TBD     | representing up to X possible entity ids in a single ledger         |
| rid=              | number    |  bytes    | TBD     | representing up to X possible resource ids in a single ledger       |
| qty=              | number    |  bytes    | TBD     | needs to be big enough to hold bitcoin number value                 |
| name=, to=, from= | string    |  ascii    | 42 max  | large enough to hold a bitcoin cash address                         |
| chain=            | string    |  ascii    | 42 max  | moniker or address for SLP ledger                                   |
| addr=             | string    |  ascii    | 42 max  | blockchain address associated with the entity                       |
| type=             | string    |  ascii    | 8 max   | can be one of: entity, resource, ledger                             |
| level=            | TBD       |  TBD      | TBD     | permissions access level to SLP ledger can be one of: ...           |

### Collision checking between the user's desired values and the arguments that may need parsed. This will ensure a valid SLP entry
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
| level=   | 6c6576656c3d    |     6      |

### Rules WIP
1) After satisfying a validation service's join requirements for tracking and validating your address's ledger you should have a message received from that validation service to confirm your address is actively being validated by the service.  This is an optional step, but without joining a validation service your address will not be actively validated for errors and discrepancies.
2) Your address must have a message to self with the protocol message of "LEDGER", where everything afterwards would be part of the ledger parsing

