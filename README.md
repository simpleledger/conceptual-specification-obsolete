# Simple Ledger Protocol (SLP) Specification - Draft

## NOTICE: The SLP Tokens Specification is currently being drafted [here](https://docs.google.com/document/d/1GcDGiVUEa87SIEjrvM9QcCINfoBw-R7EPWzNVR4M8EI/edit).  

The following specification needs to be updated to reflect the technical approach of using a [directed acrylic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) to keep track of ledger information in the various SLP types/modes.

## Overview

This specification defines a protocol for using a single Bitcoin Cash address as a data storage device that can be used in a number of interesting modes and applications.  When SLP is used in `numerical` type modes, arbitrarily named quantifiable resources can be created and allocated between a number of named entities; these entities may also be represented by a bitcoin cash address.  When SLP is used in `label` type modes, uniquely named resources can be associated with one or more bitcoin cash addresses or transaction hashes.  In all SLP modes the ledger address is owned and managed by an single entity the holding the private keys for the ledger address.  Implementations would simply construct a ledger's current state by examining the series of a human-readable SLP payloads at a ledger's address.  Some additional modes are described (i.e., `lookup` and `token` modes) which would involve querying for data at the individual addresses listed within the ledger.

Here is a brief description of each of the SLP modes:

* **Numerical Mode (e.g., private company stock ledger):**  In the `numerical` SLP mode named quantifiable resources can be created and transferred between named entities using a series of OP_RETURN transactions outputs from the same Bitcoin Cash address.  That series of transactions can be combined to calculate the ledger's current numerical state.  The SLP ledger can be shared with others for their inspection or audit by simply sharing the ledger's bitcoin address and using a web based SLP implementation.

* **Label Mode (e.g., native wallet labeling system):** In the `label` mode, a uniquely-named resource may be assigned to any number of bitcoin addresses or transaction hashes.  For example, you could have a bitcoin wallet that dedicates one of its own bitcoin cash addresses for storing names and labels that the user uses to easily identify addresses and transactions.  For some scenarios, like in a wallet, the names/labels fields could be encrypted using the SLP ledger address's public key in order to protect the user's privacy.  This would eliminate the need to use cloud file based-storage for labeling like Trezor relies upon, which likely has a propriatary data structure and implementation that is not portable to other wallets.  SLP would provide a pathway to acheive portable address and transaction labeling across wallet softwares that could be restored with only a BIP-32 12 word mnemonic.

* **Moniker Mode (e.g., DNS-like lookup for bitcoin addresses)** The `moniker` SLP mode is a more strict version of `label` mode, where a uniquely-named resource may only be associated to a single entity.  Any software, wallet or otherwise, could implment a simple SLP reader for converting from a unique monikers to a bitcoin address.  This could provide a DNS-like system for maping between a unique moniker and a bitcoin address, like an address book for contacts or trusted oracles.  Uniqueness may be enforced for monikers by using either first-in/first-out (FIFO) or last-in/first-out (LIFO) approach when querying for a moniker record.

* **Lookup Mode (e.g., centralized voting system)**: The `lookup` mode is an extension of `moniker` mode.  In the `lookup` SLP mode the ledger address would be used to store a list of Bitcoin Cash addresses just like the `moniker` mode, and each individual address listed would also be queried on the blockchain for SLP data published by each individual bitcoin address within the OP_RETURN space.  For example, if you are responsible for running a centrally controlled voting event you could first register your voter verified addresses within your SLP ledger, and then voters could each vote using an OP_RETURN output like: `SLP for=<target-slp-ledger-address> vote=<their voting data here>`.  This voting system using SLP would allow a central authority to verify each voter's integrity by registering the voters at the SLP address, but each voter votes using their own bitcoin address.  An SLP ledger in `lookup` mode would also be able to inheret entities from a previously created ledger, so new ledger could be easily created for a very specific particular lookup use case (like a shareholder vote).  Another interesting usage of the `lookup` mode would be to use an SLP ledger as a central DNS datastore for a particular top-level domain (e.g., ".bitcoin"), where the SLP ledger would store a list of bitcoin addresses and the unique domain name; then the IP address for the actual server would be published by each bitcoin address owner using SLP data within an OP_RETURN output like: `SLP for=<target-slp-ledger-address> ip4=<ip4 addr> ip6=<ip6 addr>`.

* **Token Mode (e.g., company stock ledger):** The `token` SLP mode is an extension of the `numerical` mode. In this mode bitcoin cash addresses would be required for each entity.  Each entity would be able to use their own address to provide a written notice to the issuer to indicate desire to purchase or transfer a token.  This would be a **centralized** method to issue and transfer tokens to individual bitcoin addresses, and the token issuer (i.e,. the SLP ledger operator) would need to be trusted by the token holders. The SLP ledger would record all actions related to creation, destruction, and transfer of tokens between addresses holding an asset. Requests to purchase or transfer tokens could either be approved or rejected by the token issuer. If a token holder lost their private keys, a transfer could be made solely by the SLP ledger operator, and this type of transfer would obviously be void of the token owner's written notice.  An excellent use case for this token mode would be for a private company's stock ledger, where the Company is ultimately responsible for maintaining its own stock ledger AND the stockholders **should not** be responsible for proving share ownership.  Private corperations are legally required to maintain their own stock ledger in a controlled and centralized manner, which will not likely change.

As part of the SLP project a public registry will be created and hosted at http://simpleledger.cash as an example SLP ledger implementation.  The website would be used to manage at least one SLP ledger, which would be owned by the website, but the website would allow anyone to add new entries to the SLP ledger.  The first SLP ledger managed by this site will referred to as the "Public BCH Address Registry" and would simply allow anyone to associate a unique moniker with their own bitcoin cash address.  This SLP ledger could then be referenced by any software implementation to find an address from a user provided moniker by implementing a SLP reader.  Of coarse this SLP ledger registry could easily be swapped out with someone else's managed registry, sort of like changing registry channels.

It is possible that a additional registries could be managed by this site. A registry called "Ledger Address Registry", which would simply allow anyone to publish their own SLP-compliant ledger address for public consumption and inspection, may be useful.  The registry itself be an SLP ledger in `lookup` mode to display the SLP ledger's published name, mode, genisis date, and address.  Another useful registry may be used for storing oracles who would like to be publically known.

The registry will allow anyone to associate where SLP ledgers can publish their address if desired and create a unique moniker for their SLP ledger.  This public registry would be stored on the Bitcoin Cash blockchain as an SLP ledger in `moniker` mode.  This public registry may simplify the usage of SLP ledgers for other protocols utilizing one or more SLP ledgers.  and any wallet or web service following SLP can check current state for any SLP asset.  The registry itself will be stored within a SLP ledger. During the registration process each SLP ledger's address will be registered and a unique moniker may be provided.  SLP can work without the registry, but the registry will provide a unique moniker, analogous to a DNS, to protect the holder in case the SLP ledger address is changed.

## Introduction and Purpose
There are many useful business cases where only a single person or group is responsible for keeping track of items, and the data is also relied upon by others.  In these cases, having an immutable record of historical events is often just as important as knowing the current allocation state of resources.  For such purposes having a simple protocol for immutable bookkeeping of resources on a blockchain is defined herein.

A service that implements SLP in order to create SLP transactions, detect SLP transactions, and calculate the current state of a SLP ledger will be referred to as a Validation Service Provider (VSP). Managing a SLP ledger without a VSP would be very difficult, requiring manual entry into OP_RETURN without any validation, so the VSP is a critical component for SLP ledgers.  

The VSP should first provide protocol validation prior to making a SLP ledger transaction, however, post-transaction validation by the VSP should also be performed whenever state is fetched.  If errors are found during a post-transaction validation check (because errors were not properly validated during the pre-transaction check) the VSP should then respond with an appropriate human readable OP_RETURN message so that corrections can be made.  A VSP can be used to examine balances of digital asset addresses and identify differences between actual and expected balances for those digital assets being tracked with an SLP ledger.  

### Example Use Cases (using a single bitcoin address):
* Maintain a company's stock ledger with multiple types/series of stock
* Issuing and holding event tickets
* Track inventory for multiple products
* Voting
* Manage and track balances and payments of multiple loans or accounts receivable
* Store a public registry of various entities and resources

### Obtaining New SLP Tokens
To obtain a new issuance of an SLP token you may simply send BCH (without any OP_RETURN needed) to the SLP address owned by the token of interest.  In return the responsible VSP maintaining the associated SLP ledger will detect your transaction and assign its digital asset to your bitcoin address within its own SLP ledger.  You will be issued digital asset at an exchange rate that is published by the token, VSP, on the public registry, or using a fair market value.  If your transaction does not meet minimum requirements (such as price or there is no new asset available to issue) then the VSP shall return your funds (less fees) and respond back using an appropriate OP_RETURN error message that your SLP compliant web service or wallet can detect.

### Transferring SLP Tokens
To transfer the digital asset, you can use an SLP compliant wallet (e.g., a web service or downloadable wallet).  The SLP compliant wallet will scan SLP ledgers that you have previously interacted with so that it can be aware of your asset holdings.  Once the wallet is aware of your holdings it can use the SLP OP_RETURN commands to transfer your digital asset to any address.  The digital asset's SLP address may even accept a buy back of the digital asset using an exchange rate.  

If a wallet is not SLP compliant but allows manual OP_RETURN message entry, then you can perform a transfer manually by typing the SLP command `TRANSFER chain=<SLP address or public moniker> qty=# to=<recipient addr>` within the OP_RETURN message space.  Using a public moniker that is set at the SLP public registry, analogous to a DNS, will be the recommended method for getting the SLP's correct bitcoin address.

### A Zero-balance Bitcoin Address Can Still Hold SLP Tokens
SLP allows you to hold an infinite number of tokens even with a zero balance at your bitcoin address.  This is because the each SLP compliant asset maintains its own ledger which records and indicates how much token you own.  You only need to maintain a balance at your bitcoin address if you would need to make an SLP token transfer in order to satisfy the underlying bitcoin cash network transaction requirements.

### The Different Roles of a VSP & Security
A VSP can perform multiple functions to make SLP as useable as possible. At the most basic level a VSP just maintains SLP ledgers and understands the SLP protocol enough to prevent mistakes.  

The first role that has been already discussed is providing validation of transactions for an SLP ledger.  This is an important role because the VSP should be the only entity who holds the SLP ledger's private key, so it should be a trusted and responsible business entity for an important ledger.  If the VSP makes mistakes the record of the mistakes will be on the blockchain, and corrections can be made by the VSP if required by customers or the law. 

The second role that VSPs may play is to act as a front-end web interface for an SLP ledger's customers or asset holders.  The VSP can allow potential customers and holders to visit a website or provide an SLP compliant downloadable wallet.  The website or wallet should allow the asset holder to interact with the SLP ledger, check their SLP ledger balances, and make transfers.

The weak part of this SLP system is the VSP holding the private key to the SLP ledger allowing the VSP to totally control an asset's allocation.  However, there will always be a trade-off between security of a system and its simplicity. Therefore, the security of the SLP system may need to be backed by trust in the VSP as a responsible entity.  If the VSPs keys are compromised steps will need to be taken by the VSP to ensure the SLP ledger's address is changed and the malicious or erroneous transactions are voided within the SLP ledger.

### Terms and Definitions:
| Term     | Definition                                                                                                      |
|----------|-----------------------------------------------------------------------------------------------------------------|
| Ledger   | A series of immutable entries with prefixes and fields defined in this protocol                      |
| Resource | A quantifiable item, object, asset, or liability (e.g., company shares, products, digital assets) being held by an entity |
| Entity   | A person, place, thing or category that resources are allocated to                               |
| Transfer | Indicates movement of a resource between the defined entities                                                 |
| Void     | Indicates a previously made transaction should be ignored in the ledger calculation                           |
| Update   | Indicates updating of field data for a particular entity or resource                                          |
| Validation Service Provider | A custom full-node implementation that is validating SLP transactions and oracle service for ledger addresses it owns or manages  |                          


### Example Numerical Mode (Simple)

| Entry/Txn | Command  | Arguments                                     |
|-----------|----------|-----------------------------------------------|
| 1         | LEDGER   | mode=numerical, slpver=1, name=ABC Inventory  |
| 2         | ENTITY   | eid=1,name=CompanyABC                         |
| 3         | RESOURCE | rid=1,name=WidgetA,qty=100000,eid=1           |
| 4         | ENTITY   | eid=2,name=XYZ Inc.                           |
| 5         | TRANSFER | rid=1,from=1,to=2,qty=30000                   |

#### Resulting ledger calculated by a software application

|         | CompanyABC | XYZ Inc. |
|:-------:|:----------:|:--------:|
| WidgetA |    70000   |   30000  |

### Example Numerical Mode (Advanced)

| Entry/Txn| Command | Arguments                                             |
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
| WidgetA |    70000   |   30000  |      -     |
| WidgetB |     75     |     -    |     25     |
|  BarFoo |    1000    |     -    |    5000    | 

### Example Moniker Mode (e.g., Like for your own DNS-like naming system for )
| Entry/Txn | Command  | Arguments                                |
|-----------|----------|------------------------------------------|
| 1         | LEDGER   | mode=moniker, slpver=1, name=MyOracles   |
| 2         | ENTITY   | name=ComanyXYZ, addr=qsdoifjs...         |
| 3         | RESOURCE | txn=2 name=tabs.cash qty=1               |
| 4         | ENTITY   | name=XYZ Inc., addr=qasdddjs...          |
| 5         | RESOURCE | txn=4 name=bets.cash qty=1               |
| 6         | ENTITY   | name=Global Inc, addr=qfsddfjs...        |
| 7         | RESOURCE | txn=6 name=global.com qty=1              |

#### Resulting monkiker ledger calculated by a softwar application
|             | CompanyXYZ | XYZ Inc. | Global Inc |
|:-----------:|:----------:|:--------:|:----------:|
| tabs.cash   |     1      |     -    |      -     |
| bets.cash   |     -      |     1    |      -     |
| global.com  |     -      |     -    |      1     | 

Note: Transaction #5 would replace the address for moniker created at Transaction #2 using LIFO retrieval.
  
### Token Mode Example
To do...

### Resource & Entity Genisis IDs
Identifiers for resources and entities will be implied to be the transaction hash associsted with the genisis of the resource or entity entry.  This will ensure unique ID generation and also elliminate the need to explicitly provide an ID within the OP_RETURN space when generating new resources or entities. If a name, address, or moniker fields are provided within the associated entity's genisis then the GUI may display the name, address, or moniker instead of the entity's transaction hash.  Likewise, if the name field is provided for the associted resource's genisis the GUI may display the name of the resource instead of the resource's genisis transaction hash.

### Encryption for Ledger Privacy And Provability
When encryption is necessary (e.g., private company stock ledger), SLP messages will be signed prior to being encrypted with the ledger's public key. The signature for those unencrypted SLP messages will be stored with the encrypted SLP transaction.  This way integrity of the unencrypted SLP messages will be provable with the signature which is stored on the blockchain.  

### Edit and Renaming capabilities
SLP defines voiding of resource transfers and renaming of entities and resources so that they can be displayed nicely in the ledger's current state.

### Upgrading a ledger's protocol version
Any ledger utilizing a particular SLP version can change to another SLP version by simply using the LEDGER command again with the desired version specified.  Since the blockchain economics and protocol capabilities will change over time it will be important to be able to switch to a different SLP version.  For example, if many ledger entries are being made through an API then transactions may be expensive so a much less verbose SLP protocol will be desired to reduce cost.  Another scenario may be that one protocol version may have more features than another.  

### Timestamps
A transaction or transfer of resources that happens in the real world will likely occur at a different time from when the associated ledger entry is made on the blockchain.  For this reason, an optional timestamp field can be included for TRANSFER entries.

### Simple Ledger Registry (http://simpleledger.cash)
The SLP will eventually provide a public registry of unique monikers, each associated with a bitcoin cash address being used either as a SLP ledger or as a token holder.  This will make it easier to create SLP transactions by using the published SLP moniker instead of the full address.  It will be possible to change the bitcoin cash address associated with a unique moniker if required (e.g., if the private keys of an SLP address have been compromised).  Changing the bitcoin cash address will a special procedure for the registrant to follow in order to help guard against high jacking an SLP ledger's unique moniker.

### Detecting Problems with a Ledger's VSP
The protocol will require that each SLP ledger publish an "is active" OP_RETURN signal at a certain interval so that anyone who is subscribing to updates from the ledger will know if there is a problem with the VSP associated with the ledger.

## SLP Specification

The initial version of SLP designed to be human-readable ledger entries less than 220-bytes.  

### Prefix
The SLP will use a prefix of 0x00534c50.  Following the spirit of human-readability, this hex translates to ASCII as "SLP".  Having a consistent prefix will allow blockchain explorers to simply identify SLP related transactions so they can provide a simple URL which points to http://simpleledger.cash/txn/{txn_id} where the transaction can be displayed in the context of the SLP protocol and transaction.

For the purpose of collision avoidance and detection by BCH blockchain explorers SLP is registered with [Lokad's Terab Project](https://github.com/Lokad/Terab/blob/master/etc/protocols.csv).

### Commands and arguments used by a VSP to maintain a SLP ledger
| 8-byte Command Prefix | Required Arguments    | Optional Arguments  | Description                                          |
|:---------------------:|:---------------------:|:-------------------:|:-----------------------------------------------------|
| LEDGER                | slpver=               | name=, date=, registry= | Create a new ledger or change to specified version   | 
| ENTITY                | eid=                  | name=, addr=        | Create a new entity in the ledger                    |
| RESOURCE              | rid=, qty=, eid=      | name=               | Create a new resource in the ledger with initial assignment and quantity allocation |
| TRANSFER              | from=, to=, rid=, qty=| date=.              | Move an allocation of resource from one entity to another entity |
| UPDATE                | type=, id=,           | name=, addr=, chain=| Update at least one of the optional arguments for an entity, resource, or the ledger |
| COMMENT               | msg=                  | txn=, chain=, replyto=| Create a generic comment about any transaction on any chain, can also reply to a previously made comment using chain's txn id |

### Commands and arguments used by a bitcoin address to interact with a VSP, to trigger SLP ledger entry
| 8-byte Command Prefix | Required Arguments      | Optional Arguments  | Description                                        |
|:---------------------:|:-----------------------:|:-------------------:|:---------------------------------------------------|
| TRANSFER              | to=, qty=, for=         | rid=                | Move an allocation of resource from one entity to another entity |

### Argument requirements
| Argument          | Type      | Encoding  | Bytes   | Representation                                                      |
|:-----------------:|:---------:|:---------:|:-------:|:-------------------------------------------------------------------:|
| slpver=           | number    |  byte     | 1 exact | representing up to 256 versions                                     |
| registry=         | string    |  ascii    | 100 max | the url to a public SLP `moniker` registry                          |
| date=             | number    |  bytes    | 4 exact | representing POSIX time manually set date stamp by user             |
| eid=              | number    |  bytes    | TBD     | representing up to X possible entity ids in a single ledger         |
| rid=              | number    |  bytes    | TBD     | representing up to X possible resource ids in a single ledger       |
| qty=              | number    |  bytes    | TBD     | needs to be big enough to hold bitcoin number value                 |
| name=, to=, from= | string    |  ascii    | 42 max  | large enough to hold a bitcoin cash address                         |
| for=              | string    |  ascii    | 42 max  | moniker or address for SLP ledger                                   |
| ledger=           | string    |  ascii    | 42 max  | moniker or address for SLP ledger                                   |
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
