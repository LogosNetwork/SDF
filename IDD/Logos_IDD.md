
#Interface Description 

This document details the format of the exchanged data over the network. The native memory layout, database layout and intermediate data format can be different from the format depicted in this document. 

## Message Header

| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Version | 1 | Logos Core Protocol version|
| Message Type| 1 | Type of message|
| Consensus Type| 1 | Type of consensus|
| MPF | 1 | Multipurpose Field |
| Payload size | 4 | Size of the message - size of the header|

The MPF field in the header is a multipurpose field, currently not used.

### message type:

| Type | Value  | 
| --- | ----------------- | 
| Pre-Prepare | 0  | 
| Prepare | 1 |
| Post-Prepare | 2 | 
| Commit | 3 |
| Post-Commit | 4 |
| Key_Advert | 5 |
| Pre-Prepare Reject | 6 |
| Heart Beat | 7 |
| Post_Committed_Block | 8 |
| Unknown | 0xFF |

### Consensus Block Type
Consensus Block Type is included in every consensus protocol message. It is used to dispatch the received messages to the respective consensus logic.

| Type | Value |
|------|-------|
| BatchStateBlock | 0 |
| MicroBlock | 1 |
| Epoch | 2 |
| Any | 0xFF |

## Transaction Request

### Single State Block

The *single state* block is a single unit of operation that can be performed on an account. The table below depicts the format of a state block:

| Field Name |Size (byte) | Description |
| --- | -------------| ----------------- |
| Account  | 32 | Account address | 
| Previous | 32 | Previous block hash on account| 
| Sequence Number  | 4 | Number of sends, increment only | 
| Request Type | 1 | Request type |
| Count | 2 | Number of requests, [1-8]| 
| Transactions[Count]   | sizeof(Transaction) * Count | Array of transactions |
| Transaction Fee | 16 | Transaction Fee |
| Signature | 64 | EdDSA signature |
| Work      | 8  |Proof of Work Nonce |

Note that Signature = Sign(Hash(Account, Previous, Sequence Number, Request Type, Transaction Fee, Count, Transactions))
Note that the "Work" field is not covered by the signature. This field is sent from the client to the primary delegate. After verifying it, the primary does not send it to the backup delegates.

#### Request Type
| Request Type | Value | 
| --- | ----------------- | 
| Send | 0 | 
| Change | 1 |

Note that open and receive transactions are inferred from the send transaction request. 
[//]: # "Change requests must have a value of 0 to indicate it is a change request."

#### Transaction
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Target Address | 32 | Transaction target address |
| Amount | 16 | Transaction Amount.|

Note that the Amount is 0 if the request type is "Change" representative.

## Consensus Messages

### Pre-Prepare
A primary delegate proposes a pre-prepare message to start a consensus session, which either Approves or Rejects the Consensus Block contained in the pre-prepare message. 

| Field Name |Size (Byte) | Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Primary | 1 | Primary delegate's index |
| Epoch Number  | 4 | Global epoch number |
| Sequence Number | 4 | Starting from 0 at the beginning of the epoch |
| Timestamp | 8 | UTC timestamp (millisecond)|
| Previous | 32 | Hash of the Previous Consensus Batch; <br/>Reference the NULL block if this is the first Batch State Block of this delegate. <br/> Reference the genesis Epochblock if this is the first Epochblock. <br/> Reference the Microblock in the genesis Epochblock if this is the first Microblock. |
| Signature | 32 | BLS signature | 
| Consensus Block | Sizeof(Consensus Block) | A multipurpose field. It can be a Batch State Block, a Microblock, or a Epochblock. <br/> The type of the block is in the message header (Consensus Type).|

Note that Signature = Sign(Pre-Prepare Hash), where the pre-prepare hash is depicted in the next table.

#### Pre-Prepare Hash 

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Pre-Prepare Hash | 32 | Hash(Header.Version, Primary, Epoch Number, Sequence Number, Timestamp, Previous, Consensus Block)|

#### Batch State Block
The *batch state* block is one or more single state blocks with additional metadata. 

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Count  | 2 | The number of Single State Blocks included in this batch block |
| Single State Block Hashes[Count] | 32 * Count | hashes of Single State Blocks |
| Appendix | sum of all Single State Blocks | Single State Blocks included in this batch |

Note that the Single State Blocks are attached to the pre-prepare message as the Appendix field. The PoW field should not be sent. 

#### Batch State Block Tip
Current tip of the batch block for each delegate. Keyed by the delegate's number in this epoch.

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Batch block tip | 32 | Current tip of the batch block |

#### Micro-Block

Micro-block is created periodically at time T = Epoch / N, where N is the desired number of micro-blocks created during each Epoch. Micro-blocks are used for checkpointing and bootstrapping. Micro block is keyed in the database by the micro block's hash.

| Field Name |Size (Byte) | Description |
| --- | -------------| ----------------- |
[//]: # "| Merkle Tree Root | 32 | Merkle tree root of the batch blocks included in this micro block|"
| Number of Batch Blocks| 4 | Number of batch blocks included in the microblock |
| Last Micro Block Flag | 1 | Last Micro block flat [0|1] |
| Batch Block Tips [32] | 32*32 | Tip of the batch block chain for each delegate|

[//]: # "Merkle tree root construction: Merkle tree is constructed from BatchStateBlock hashes. BatchStateBlock's are globally ordered by design within each delegate. Therefore, all BatchStateBlock's included in the MicroBlock can be viewed as one globally ordered logical array of concatenated delegates BatchStateBlock chain, ordered by delegates. To select the BatchStateBlock's included in the Merkle Tree, the algorithm traverses each delegate's BatchStateBlock's chain starting with the delegate's current BatchStateBlock's tip and ending when it reaches the BatchStateBlock's tip included in the previous MicroBlock.  If the BatchStateBlock's timestamp is less than the previous MicroBlock timestamp plus MicroBlock_interval (currently 10 minutes), then the BatchStateBlock is included in the MicroBlock. Note, that the timestamp of the last genesis MicroBlock is 0. Therefore when constructing the very first MicroBlock, the oldest timestamp of all existing BatchStateBlock's is found first. Then this timestamp is used in place of the previous MicroBlock timestamp and the algorithm proceeds as described above. Once hashes of all BatchStateBlock's that should be included in the MicroBlock are collected, the Merkle Tree Root is calculated."

#### Micro-Block tip

Current micro block tip.

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Tip | 32 | Current microblock's hash tip |

#### Epoch Block
A epoch block is proposed after every epoch to summarize the epoch. It includes the summary of all the successful delegate consensus sessions. The table below depicts the format of an epoch block:

| Field Name |Size (Byte) | Description |
| --- | -------------| ----------------- |
| Microblock tip  | 32 | last micro-block's hash |
| Transaction Fee Pool | 16 | Transaction Fee Pool |
| Delegate [32] | sizeof(Delegate)*32 | 32 delegates and their election results |

#### Epoch-Block tip

Current Epoch block tip.

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Tip | 32 Bytes | Current epoch block's hash tip |

#### Delegate
An election result entry, i.e., a delegate with it stake, and the votes it received. 

| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Address | 32 | Account address of the delegate |
| Public Key | 64 | BLS public key of the delegate |
| Vote | 16 | Votes the delegate received |
| Stake | 16 | Stake of the delegate |

### Prepare (Pre-Prepare Acceptance)

This message is used by the back up delegate to accept the pre-prepare initiated by the primary delegate

| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Pre-prepare Hash| 32 | Pre-prepare Hash |
| Signature | 32 | BLS signature |

Note that Signature = Sign(Pre-prepare Hash), where the Pre-prepare Hash is depicted in the Pre-prepare Hash table.

### Pre-Prepare Reject

This message is used by the back up delegate to reject the pre-prepare message of Batch State Blocks initiated by the primary delegate. The only active rejection message is for double spend. Note that the invalid Microblocks and Epochblocks are rejected by silence.

| Field Name | Size (Byte) | Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Pre-prepare Hash | 32 | Pre-prepare Hash |
| Rejection Reason | 1 | reason for rejection |
| Count  | 2 | The number of Single State Blocks in the pre-prepare |
| Approval bitmap | ceiling(Count/8) | One bit per Single State Block to either approve or reject |
| Signature | 32 | BLS signature |

Note that Signature = Sign(Hash(Header, Pre-prepare Hash, Count, Approval bitmap))
Note that the accounts' reservation statuses are not changed, if the pre-prepare is rejected.

#### Rejection Reason:

| Reason | Value  | 
| --- | ----------------- | 
| Clock_Drift | 0  | 
| Contains_Invalid_Request | 1 |
| Bad_Signature | 2 | 
| Invalid_Previous_Hash | 3 |
| Wrong_Sequence_Number | 4 |
| Invalid_Epoch | 5 |
| New_Epoch | 6 |
| Void | 7 |

### Post-Prepare

This message is used by the back up delegate to accept the pre-prepare initiated by the primary delegate

| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Pre-prepare Hash | 32 | Pre-prepare Hash |
| Participation Map | 8 | Bitmap of the delegates that participated in this round of signing | 
| Aggregated Signature| 32 | BLS aggregated signature of the prepare phase |

Note that the Aggregated Signature is aggregated from the pre-prepare signature and the prepare signatures.
Note that Signature = Sign(Post-Prepare Hash), where the post-prepare hash is depicted in the next table.

#### Post-Prepare Hash 

| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Post-Prepare Hash | 32 | Hash(Pre-prepare Hash, Participation Map, Aggregated Signature) |

### Commit

This message is used by the primary delegate to perform final proof aggregation and database update of transactions

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Pre-prepare Hash | 32 | Pre-prepare Hash |
| Signature | 32 | BLS signature |

Note that Signature = Sign(Post-Prepare Hash)

### Post-Commit

This message is used as the final proof to perform database update for the transactions contained within the block.

| Field Name |Size (Byte) | Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Pre-prepare Hash| 32 - Byte | Pre-prepare Hash |
| Participation Map | 8 - bytes | Bitmap of the delegates that participated in commit|" 
| Aggregated Signature| 32 - Byte | BLS aggregated signature of commit |

### Post-Committed Block
A primary delegate proposes a pre-prepare message to start a consensus session, which either Approves or Rejects the Consensus Block contained in the pre-prepare message. 

| Field Name |Size (Byte) | Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Primary | 32 | Primary delegate's address |
| Epoch Number  | 4 | Global epoch number |
| Sequence Number | 4 | Starting from 0 at the beginning of the epoch |
| Timestamp | 8 | UTC timestamp (millisecond)|
| Previous | 32 | Hash of the Previous Consensus Batch; <br/>Reference the NULL block if this is the first Batch State Block of this delegate. <br/> Reference the genesis Epochblock if this is the first Epochblock. <br/> Reference the Microblock in the genesis Epochblock if this is the first Microblock. |
| Signature | 32 | primary BLS signature | 
| Consensus Block | Sizeof(Consensus Block) | A multipurpose field. It can be a Batch State Block, a Microblock, or a Epochblock. <br/> The type of the block is in the message header (Consensus Type).|
| Participation Map of Prepare | 8 - bytes | Bitmap of the delegates that participated in prepare| 
| Aggregated Signature| 32 - Byte | BLS aggregated signature of prepare |
| Participation Map of Commit | 8 - bytes | Bitmap of the delegates that participated in commit|
| Aggregated Signature| 32 - Byte | BLS aggregated signature of commit |

## Blacklist

The blacklist is used to deny service for malicious/deficient accounts. It contains cryptographically signed proof of malicious behaviors of the accounts. A delegate would only propagate newly added entries when a new black list entry is added to the list.

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte | Logos Core Protocol version|
| Number of Entries | 2 - Byte | Number of blacklisted entries|
| Blacklist reason| 1 - Byte | See blacklist entry type|
| Blacklist entry content | varies | See blacklist entry type|
|...|
| Blacklist reason n| 1 - Byte | See blacklist entry type|
| Blacklist entry content n | varies | See blacklist entry type|

### Blacklist Entry Type

The section below describes the type of blacklist entries and its associated format

| Type | Value  | 
| --- | ----------------- | 
| Double Spend | 0 |

### Blacklist Entry Layout

#### Double Spend

This section corresponds to the "Blacklist entry content" field of the blacklist data structure

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Single state block | sizeof (state_block) | Conflicting block #1 |
| Single state block | sizeof (state_block) | Conflicting block #2 |How do you convert DEC to hexadecimal?
Steps:
Divide the decimal number by 16. Treat the division as an integer division.
Write down the remainder (in hexadecimal).
Divide the result again by 16. Treat the division as an integer division.
Repeat step 2 and 3 until result is 0.
The hex value is the digit sequence of the remainders from the last to first.