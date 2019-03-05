# Database Layout Documentation

This document complements the Logos IDD and specifies the format of data stored by `logos_core` software in LMDB (**L**ightning **M**emory-Mapped **D**ata**b**ase). 

The native database layout is expected to be fully consistent with the format described below at software release time. This documentation is best understood when read together with Logos IDD.

All individual databases below exist within one MDB environment.

## Overview of databases

| MDB_dbi Name | Key | Key Size | Value | Value Size | Description |
|-------------:|:--- |:-------- | -----:|-----------:|------------:|
| batch_db | block_hash | 32 | **BatchStateBlock** |**48,088** | Maps block hash to batch block<br>|
| state_db | block_hash | 32 | **state_block** | **256** | Maps block hash to state block<br> |
| account_db | account | 32 | **account_info**| **168** | Maps account to account information |
| receive_db | block_hash | 32 | **receive_block** | **64** | Maps block hash to receive block|
| batch_tips_db | delegate_id | 1 | block_hash | 32 | Maps delegate id to hash of most recent batch block |
| micro_block_db | block_hash | 32 | **MicroBlock** | **1,162** | Maps block hash to micro block|
| micro_block_tip_db | "microblocktip"<br>(key value of 0) | 1 | block_hash | 32 | References micro block tip (only one key) |
| epoch_db | block_hash | 32 | **Epoch** | **1,696** | Maps block hash to epoch|
| epoch_tip_db | "epochtip"<br>(key value of 0) | 1 | block_hash | 32 | References epoch tip<br>(only one key) |
| frontiers | | |  | | _Deprecated_ |
| accounts | | |  | | _Deprecated_ |
| state_blocks | | | | | _Deprecated_ |
| pending | | | | | _Deprecated_ |
| blocks_info | | | | | _Deprecated_ |
| representation | | | | | _Deprecated / not implemented_ |
| unchecked | | | | | _Deprecated_ |
| checksum | | | | | _Deprecated_ |
| vote | | |  |  | _Deprecated / not implemented_ |
| meta | | | | | _Deprecated_ |

## Database Details
### batch_db

Detailed composition of each database value:

| Value Item | Size | Description |
| ---------- | ---- | ----------- |
| header | 48 | *see header layout* |
| block_count | 8 | number of blocks in batch |
| blocks | 48,000 <br> (32 x 1,500) | list of block hashes |
| signature | 32 | proposer delegate's signature |

### state_db

Detailed composition of each database value:

| Value Item | Size | Description |
| ---------- | ---- | ----------- |
| size (size_t) | 8 | size of state block |
| account | 32 | account that created this block |
| previous | 32 | previous block's hash in account (send) chain |
| representative | 32 | account's designated representative |
| amount | 16 | amount of Logos sent in this block |
| link | 32 | destination account |
| signature | 64 | account signature |
| work | 8 | PoW for anti-spamming |
| batch_hash | 32 | batch state to which it belongs |

### account_db

Detailed composition of each database value:

| Value Item | Size | Description |
| ---------- | ---- | ----------- |
| (send) head | 32 | tip of account send chain |
| receive_head | 32 | tip of account receive chain |
| rep_block | 32 | block where account most recently changed representative (to be implemented) |
| open_block | 32 | original send block that opened this account |
| balance | 16 | amount of Logos this account currently owns |
| modified | 8 | Seconds elapsed since unix epoch (*?*) |
| block_count | 8 | Number of send transactions this account has created |
| receive_count | 8 | Number of receive transactions this account has created |

### receive_db

Detailed composition of each database value:

| Value Item | Size | Description |
| ---------- | ---- | ----------- |
| send_hash | 32 | the original send transaction corresponding to this receive |
| previous | 32 | previous block's hash in account receive chain |

### micro_block_db

Detailed composition of each database value:

| Value Item | Size | Description |
| ---------- | ---- | ----------- |
| header | 48 | *see header layout* |
| delegate | 32 | public account key of proposer delegate |
| epoch_number | 4 | epoch to which it belongs |
| micro_block_number | 2 | index number within this epoch |
| (is_)last_micro_block | 1 *+1* | boolean flag indicating if it is the last micro block of an epoch |
| tips | 1,024 <br> (32 x 32) | list of batch block tips for each delegate |
| number_batch_blocks | 4 *+4* | number of batch blocks contained in micro block |
| signature | 32 | proposer delegate's signature (*?*) |


### epoch_db

Detailed composition of each database value:

| Value Item | Size | Description |
| ---------- | ---- | ----------- |
| header | 48 | *see header layout* |
| (proposer) account | 32 | public account key of proposer delegate |
| epoch_number | 4 *+4* | number by which it identifies |
| micro_block_tip | 32 | hash of the last micro block belonging to this epoch |
| delegates, each with<br>(*account*)<br>(*vote*)<br>(*stake*) | 1,536<br>(*32*)<br>(*8*)<br>(*8*) | list of delegate information |
| transaction_fee_pool | 8 | *@Greg?* |
| signature | 32 | signature (*?*) |


### Header Layout

| Value | Size |
|-------|------|
| version<br>(message_)type<br>consensus_type<br>manual padding<br>timestamp<br>previous<br>| 1<br>1<br>1<br>5<br>8<br>32<br> |

## Notes and pending fixes:

1. All sizes are in bytes, assuming a 64-bit architecture. Padding and sub-struct members are explictly shown.

2. Fixes
   + batch_db & state_db: change current implementation (batch contains list of full blocks)
   + state_db: remove timestamp?
   + receive_db: trim down

3. **Proposed changes**: 
   + **add `next` field in message header (modified retroactively in DB) to enable doubly-linked list. Actual message sent over the wire obviously doesn't need to contain this field.**
   + **same for dual account chains**
