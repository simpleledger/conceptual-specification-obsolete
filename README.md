# Simple Ledger Protocol (SLP) Specification WIP

This specification defines a protocol for using a single blockchain address as a ledger for tracking resources on a blockchain.  Any quantifiable resource allocated between any number of entities can be tracked using a single blockchain address.  The protocol is designed to be used with the Bitcoin Cash OP_RETURN space as a human readable payload when decoded from bytes to ASCII. The protocol can be implemented on any blockchain that provides ample room for data entry (e.g., the Bitcoin Cash blockchain provides 220-byte OP_RETURN data allowance as of May 15, 2018).

SLP can be used in three different modes:

* **As a general purpose numerical ledger:**  In this SLP mode the all numerical values transfered between entities can be combined to arrive at a current ledger's numerical state.  This ledger can be shared with others for viewing by simply sharing the ledger's bitcoin address.

* **As a general purpose moniker or labeling ledger (like your own DNS):** In this SLP mode monikers or names can be associated with a particular bitcoin cash address.  This moniker mode could be used to create your own moniker/address datastore, where the datastore is identified by the SLP ledger's address.  Any new protocol or implementation could adopt this method for associating unique monikers with bitcoin addresses.  There may be many SLP ledgers that store monikers associated with bitcoin addresses in the future all stored on chain, not just one.  After a bitcoin address associated with a moniker is found through the SLP ledger, the associated bitcoin address can then be searched for custom field data at that bitcoin address stored via OP_RETURN.  For example if the SLP ledger were being used like a DNS datastore for a particular TDL, the owning bitcoin address for a particular domain name could publish its IP address whenever it would like using: `SLP slp=<target slp ledger address> ip4=<ip4 addr> ip6=<ip6 addr>`.  A valid signature by the moniker's owner can optionally be provided in a signature field (i.e. sig=<chosen moniker message signature here>), and the most recently signed moniker with a particular address would be used by an implementation.  It would be easy for an implementation to determine which bitcoin address to use by finding the most recently validly signed moniker.  
  
* **As a ledger for creating "tokens":** In this SLP mode the ledger can be used as a method to issue and transfer tokens, colored coins or create and maintain a virtual blockchain that contains one or more digital assets.  These goals can be accomplished by using an internal SLP ledger to track creation, destruction, and transfer of digital assets between the public keys or addresses holding an asset.  In a strict digital asset mode, signatures can be required during asset transfers initiated by the holder.

As part of the SLP project a public registry will be created to where SLP ledgers can publish their address if desired.  The public registry should simplify usage of SLP interoperability between assets utilizing SLP and any wallet or web service following SLP can check current state for any SLP asset.  The registry itself will be stored within a SLP ledger. During the registration process each SLP ledger's address will be registered and a unique moniker may be provided.  This public registry will be relied upon by virtual blockchains for promoting interoperability between virtual chains. SLP can work without the registry, but the registry will provide a unique moniker, analogous to a DNS, to protect the holder in case the SLP ledger address is changed.

## Introduction and Purpose
There are many useful business cases where only a single person or group is responsible for keeping track of items, and the data is also relied upon by others.  In these cases, having an immutable record of historical events is often just as important as knowing the current allocation state of resources.  For such purposes having a simple protocol for immutable bookkeeping of resources on a blockchain is defined herein.

A service that implements SLP in order to create SLP transactions, detect SLP transactions, and calculate the current state of a SLP ledger will be referred to as a Validation Service Provider (VSP). Managing a SLP ledger without a VSP would be very difficult, requiring manual entry into OP_RETURN without any validation, so the VSP is a critical component for SLP ledgers.  

A virtual blockchain may use SLP as a method to keep track of its own state.  If many virtual blockchains utilize SLP and the public SLP registry the ability for virtual blockchains to interoperate will be enhanced since VSPs can be built to be aware of other virtual chains. Each virtual blockchain or colored coin may want to build and run its own VSP implementation in order to handle the customized requirements of that particular chain, like interoperability.  VSPs will either be built to be general purpose or highly specialized.  General purpose and basic VSPs may allow someone to simply create their own colored coin and SLP ledger that is managed by a third party VSP.

The VSP should first provide protocol validation prior to making a SLP ledger transaction, however, post-transaction validation by the VSP should also be performed whenever state is fetched.  If errors are found during a post-transaction validation check (because errors were not properly validated during the pre-transaction check) the VSP should then respond with an appropriate human readable OP_RETURN message so that corrections can be made.  A VSP can be used to examine balances of digital asset addresses and identify differences between actual and expected balances for those digital assets being tracked with an SLP ledger.  

### Example Use Cases (using a single bitcoin address):
* Track multiple types of blockchain assets (e.g., colored coins and virtual chain assets)
* Maintain a company's stock ledger with multiple types/series of stock
* Issuing and Holding Event Tickets
* Track inventory for multiple products
* Voting
* Manage and track balances and payments of multiple loans or accounts receivable
* Store a public registry of various entities and resources

### Obtaining New SLP Digital Assets
To obtain a new issuance of an SLP asset you may simply send BCH (without any OP_RETURN needed) to the SLP address owned by the colored coin or virtual blockchain of interest.  In return the responsible VSP maintaining the associated SLP ledger will detect your transaction and assign its digital asset to your bitcoin address within its own SLP ledger.  You will be issued digital asset at an exchange rate that is published by the colored coin, VSP, on the public registry, or using a fair market value.  If your transaction does not meet minimum requirements (such as price or there is no new asset available to issue) then the VSP shall return your funds (less fees) and respond back using an appropriate OP_RETURN error message that your SLP compliant web service or wallet can detect.

### Transferring SLP Digital Assets
To transfer the digital asset, you can use an SLP compliant wallet (e.g., a web service or downloadable wallet).  The SLP compliant wallet will scan SLP ledgers that you have previously interacted with so that it can be aware of your asset holdings.  Once the wallet is aware of your holdings it can use the SLP OP_RETURN commands to transfer your digital asset to any address.  The digital asset's SLP address may even accept a buy back of the digital asset using an exchange rate.  

If a wallet is not SLP compliant but allows manual OP_RETURN message entry, then you can perform a transfer manually by typing the SLP command `TRANSFER chain=<SLP address or public moniker> qty=# to=<recipient addr>` within the OP_RETURN message space.  Using a public moniker that is set at the SLP public registry, analogous to a DNS, will be the recommended method for getting the SLP's correct bitcoin address.

### A Zero-balance Bitcoin Address Can Still Hold SLP Assets
SLP allows you to hold an infinite number of digital assets even with a zero balance at your bitcoin address.  This is because the each SLP compliant asset maintains its own ledger which records and indicates how much asset you own.  You only need to maintain a balance at your bitcoin address if you would need to make an SLP asset transfer in order to satisfy the underlying bitcoin cash network transaction requirements.  This is a significant improvement over protocols where colored coins can be lost if the colored coins are used as inputs in a transaction which isn't a transfer transaction, their value is lost, it is not transferred to outputs of this or other transaction.

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
| Validation Service Provider | Software that is validating transaction compliance with the SLP protocol. |                        
| Colored Coins and Virtual Blockchains | Colored coins and Virtual Blockchains are loosely defined terms for systems that can issue and transferable digital assets on an existing host blockchain. The SLP is just one protocol that can be used to build and maintain either of these. Interoperability can be achieved between virtual chains by using SLP as a common language or interface. |

### Example for Numerical Mode (Simple)

| Entry/Txn | Command  | Arguments                                   |
|---------|----------|-----------------------------------------------|
| 1       | LEDGER   | mode=numerical, slpver=1, name=ABC Inventory  |
| 2       | ENTITY   | eid=1,name=CompanyABC                         |
| 3       | RESOURCE | rid=1,name=WidgetA,qty=100000,eid=1           |
| 4       | ENTITY   | eid=2,name=XYZ Inc.                           |
| 5       | TRANSFER | rid=1,from=1,to=2,qty=30000                   |

#### Resulting ledger calculated by a software application

|         | CompanyABC | XYZ Inc. |
|:-------:|:----------:|:--------:|
| WidgetA |    70000   |   30000  |

### Example for Numerical Mode (Advanced)

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
| WidgetA |    70000   |   30000  |      0     |
| WidgetB |     75     |     0    |     25     |
|  BarFoo |    1000    |     0    |    5000    | 

### Example for Moniker Mode (e.g., Like for your own DNS for BCH addresses)
| Entry/Txn | Command  | Arguments                                          |
|-----------|----------|----------------------------------------------------|
| 1         | LEDGER   | mode=moniker, slpver=1, name=MyOwnBCHDNS           |
| 2         | ENTITY   | name=com.deskviz, addr=qsdoifjs..., sig=aSig...    |
| 3         | ENTITY   | name=com.bitcoin, addr=qasdddjs..., sig=aSig2...   |
| 4         | ENTITY   | name=sx.bch, addr=qfsddfjs..., sig=aSig...         |
| 5         | ENTITY   | name=com.deskviz, addr=newaddr..., sig=aNewSig...  |  
  
Note: Transaction #5 would replace the address for moniker created at Transaction #2.
  
### Colored Coin Mode Example
Coming soon.

### Resource & Entity Genisis IDs
Identifiers for resources and entities will be implied to be the transaction hash associsted with the genisis of the resource or entity entry.  This will ensure unique ID generation and also elliminate the need to explicitly provide an ID within the OP_RETURN space when generating new resources or entities. If a name, address, or moniker fields are provided within the associated entity's genisis then the GUI may display the name, address, or moniker instead of the entity's transaction hash.  Likewise, if the name field is provided for the associted resource's genisis the GUI may display the name of the resource instead of the resource's genisis transaction hash.

### Privacy on a Public Blockchain using Encryption
Encryption of fields containing sensitive name or quantity data (i.e., name=, qty= fields) is possible using a combination of asymmetric keys and symmetric keys.  This feature will allow someone to maintain an encrypted ledger with the ability to share only selected transaction data with third-parties using a passcode.  Everyone will be able to see the command, but the certain field data can be hidden with a password.

### Edit and Renaming capabilities
SLP defines voiding of resource transfers and renaming of entities and resources so that they can be displayed nicely in the ledger's current state.

### Upgrading a ledger's protocol version
Any ledger utilizing a particular SLP version can change to another SLP version by simply using the LEDGER command again with the desired version specified.  Since the blockchain economics and protocol capabilities will change over time it will be important to be able to switch to a different SLP version.  For example, if many ledger entries are being made through an API then transactions may be expensive so a much less verbose SLP protocol will be desired to reduce cost.  Another scenario may be that one protocol version may have more features than another.  

### Timestamps
A transaction or transfer of resources that happens in the real world will likely occur at a different time from when the associated ledger entry is made on the blockchain.  For this reason, an optional timestamp field can be included for TRANSFER entries.  

### Multiple OP_RETURN Spaces
Multiple outputs with OP_RETURN included in their script provides additional data storage.  The protocol will utilize multiple transaction output in some cases as appropriate.

### Simple Ledger Registry (http://simpleledger.cash)
The SLP will eventually provide a public registry of unique monikers, each associated with a bitcoin cash address being used either as a SLP ledger or as a token holder.  This will make it easier to create SLP transactions by using the published SLP moniker instead of the full address.  It will be possible to change the bitcoin cash address associated with a unique moniker if required (e.g., if the private keys of an SLP address have been compromised).  Changing the bitcoin cash address will a special procedure for the registrant to follow in order to help guard against high jacking an SLP ledger's unique moniker.

### Detecting Problems with a Ledger's VSP
The protocol will require that each SLP ledger publish an "is active" OP_RETURN signal at a certain interval so that anyone who is subscribing to updates from the ledger will know if there is a problem with the VSP associated with the ledger.

## SLP Specification

The initial version of SLP designed to be human-readable ledger entries less than 220-bytes.  

### Prefix
The SLP will use a prefix of 0x00534c50.  Following the spirit of human-readability, this hex translates to ASCII as "SLP".  Having a consistent prefix will allow blockchain explorers to simply identify SLP related transactions so they can provide a simple URL pointing to http://simpleledger.cash/txn/{txn_id} where the transaction to be fully parsed and recognized. After the redirect has occurred the Simple Ledger registry can automatically provide a redirect to the SLP implementation's website can be provided.

For collision avoid and BCH blockchain explorers whom want to detect many OP_RETURN protocols SLP is registered with [Lokad's Terab Project](https://github.com/Lokad/Terab/blob/master/etc/protocols.csv).

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

