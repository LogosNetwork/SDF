## Overview of databases


| MDB_dbi Name | Key | Key Size | Value | Value Size | Description |
| --- | ------------- | ----------------- | --- | --- | --- |
| `account_db` | `account_address` | 32 | `account_info` | 160 | Maps an account_address to the account information |
| `reservation_db` | `account_address` | 32 | `reservation_info` | 36 | Maps an account_address to the reservation information |
| `state_db` | `block_hash` | 32 | StateBlock | [231-567] _\(183 + [1-8] * 48\)_ | Maps a block hash to a state block |
| `receive_db` | `block_hash` | 32 | ReceiveBlock | 66 | Maps a block hash to a receive block |
| `batch_db` | `block_hash` | 32 | Post-Commited BatchStateBlock | [203-48,203] _\(203 + [0-1500] * 32\)_ | Maps a block hash to a Post-committed BatchStateBlock, which contains a list of hashs of StateBlocks |
| `batch_tips_db` | `delegate_index` | 1 | `block_hash` | 32 | Maps a delegate index to the Key (block_hash) of the most recent Post-Commited BatchStateBlock |
| `micro_block_db` | `block_hash` | 32 | Post-Commited MicroBlock | 1,230 | Maps a block hash to a micro block |
| `micro_block_tip_db` | 0 | 1 | block_hash | 32 | References the Key (block_hash) of the most recent micro block. This database has only 1 row, so the key of this database is always 0. |
| `epoch_db` | `block_hash` | 32 | Post-Commited Epoch | 4,345 | Maps a block hash to an epoch block | 
| `epoch_tip_db` | 0 | 1 | `block_hash` | 32 | References the Key (block_hash) of the most recent epoch block. This database has only 1 row, so the key of this database is always 0. |

Note all the sizes are in bytes.

## Database Details

### `account_db`

| Value Item | Size | Description |
| --- | --- | --- |
| (send) head | 32 | tip of account send chain |
| receive_head | 32 | tip of account receive chain |
| rep_block | 32 | block where account most recently changed representative (to be implemented) | 
| open_block | 32 | original send block that opened this account | 
| balance | 16 | amount of Logos this account currently owns | 
| modified | 8 | seconds elapsed since unix epoch |
| sequence | 4 | number of send transactions this account has created |
| receive_count | 4 | number of receive transactions this account has received |

### `reservation_db`

| Value Item | Size | Description |
| --- | --- | --- |
| reservation_hash | 32 | hash of the request that reserved this account |
| reservation_epoch | 4 | the epoch when this account was reserved |

### `state_db`

| Value Item | Size | Description | Hash Sequence |
| --- | --- | --- | - |
| account_address | 32 | the account that created this block |  1 |
| previous | 32 | previous block's hash in account (send) chain | 2|
| sequence_number | 4 | sequence number | 3|
| request_type | 1 | type of the request | 4 |
| Count | 2 | Number of transactions, [1-8]| 5 |
| Transactions[Count] | sizeof(Transaction) * Count | Array of transactions (see below) | 6 |
| Transaction Fee | 16 | Transaction Fee | 7 |
| Signature | 64 | EdDSA signature | - |
| BatchHash | 32 | Key (BlockHash) of the post-committed BatchStateBlock containing this StateBlock | - |
| IndexInBatch | 2 | Index of the StateBlock in the post-committed BatchStateBlock | - |


#### Transaction

| Field Name |Size (Byte)| Description |Hash Sequence |
| --- | -------------| ----------------- | - |
| Target Address | 32 | Transaction target address | 1 |
| Amount | 16 | Transaction Amount.| 2 |

### `receive_db`

| Value Item | Size | Description | Hash Sequence |
| --- | --- | --- | - |
| previous | 32 | previous receive's hash in account receive chain | - |
| send_hash | 32 | the original StateBlock corresponding to this receive | 1 |
| transaction_index | 2 | index to the transaction array of the StateBlock | 2 |

### `batch_db`, `micro_block_db`, and `epoch_db`

BatchStateBlocks, MicroBlocks, and Epochs are blocks that must be approved (post-committed) by delegate consensus sessions. All the three kinds of post-committed blocks share some common fields. 

### Structure and common fields of post-committed blocks databases

(Common field size total: 201B)

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | - | 
| MessagePrequel | 8 | Message header (see below) |  1 |
| Primary | 1 | Primary delegate's index |  2 |
| Epoch Number  | 4 | Global epoch number |  3 |
| Sequence Number | 4 | Starting from 0 at the beginning of the epoch | 4 |
| Timestamp | 8 | UTC timestamp (millisecond)| 5 |
| Previous | 32 | Hash of the Previous Consensus Block; <br/>Reference the NULL block if this is the first BatchStateBlock of this delegate. <br/> Reference the genesis Epochblock if this is the first Epochblock. <br/> Reference the Microblock in the genesis Epochblock if this is the first Microblock. | 6 |
| Signature | 32 | Primary's BLS signature | - |
| Consensus Block | Sizeof(Consensus Block) | A multipurpose field. It can be a Batch State Block, a Microblock, or an Epoch. (see below) <br/> The type of the block is in the MessagePrequel.| 7 |
| Participation Map of Prepare | 8 | Bitmap of the delegates that participated in prepare|  - |
| Aggregated Signature| 32 | BLS aggregated signature of prepare | - |
| Participation Map of Commit | 8 | Bitmap of the delegates that participated in commit|  - |
| Aggregated Signature|  32 | BLS aggregated signature of commit | - |
| Next | 32 | Hash of the next Consensus Block | - |

#### Message Prequel

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | - |
| Version | 1 | Logos Core Protocol version|  1 |
| Message Type| 1 | Type of message| - |
| Consensus_Type | 1 | BatchStateBlock, MicroBlock, or Epoch | -|
| MPF | 1 | Multipurpose Field (currently not used) | -|
| Payload size | 4 | Size of the message |-|

#### Batch State Block

| Field Name |Size| Description | Hash Sequence |
| --- | -------------| ----------------- | - |
| Count  | 2 | The number of Single State Blocks included in this batch block | 1 |
| Single State Block Hashes[Count] | 32 * Count | hashes of Single State Blocks | 2 |

#### Micro Block

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | - |
| Last Micro Block Flag | 1 | Last Micro block flat | 1
| Number of Batch Blocks| 4 | Number of batch blocks included in the microblock | 2 |
| Batch Block Tips [32] | 32*32 | Tip of the batch block chain for each delegate| 3 |

#### Epoch

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | - |
| Microblock tip  | 32 | last micro-block's hash | 1 |
| Transaction Fee Pool | 16 | Transaction Fee Pool | 2 |
| Delegate [32] | sizeof(Delegate)*32 | 32 delegates and their election results (see below) | 3 |

#### Delegate

| Field Name |Size | Description |Hash Sequence |
| --- | -------------| ----------------- | -|
| Address | 32 | Account address of the delegate | 1 |
| Publick Key | 64 | Delegate's BLS public key |2|
| Vote | 16 | Votes the delegate received |3|
| Stake | 16 | Stake of the delegate |4|

### `batch_tips_db`

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| tip | 32 | Current tip of the batch block. One per delegate. |

### `micro_block_tip_db`

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| tip | 32 | Current microblock's hash tip. Only one in the network. |

### `epoch_tip_db`

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| tip | 32 | Current epoch block's hash tip. Only one in the network. |

