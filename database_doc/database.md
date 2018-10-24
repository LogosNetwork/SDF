# Database Layout Documentation

This document complements the Logos IDD and specifies the format of data stored by `logos_core` software in LMDB (**L**ightning **M**emory-Mapped **D**ata**b**ase). 

The native database layout is expected to be fully consistent with the format described below at software release time. This documentation is best understood when read together with Logos IDD.

All individual databases below exist within one MDB environment.

| MDB_dbi Name | Key | Key Size | Value | Value Size | Description |
|-------------:|:--- |:-------- | -----:|-----------:|------------:|
| batch_db | block_hash | 32 | **BatchStateBlock**<br>header<br>block_count<br>blocks<br>signature |**48,088**<br>48<br>8<br>48,000<br>32 | Maps block hash to batch block<br>|
| state_db | block_hash | 32 | **state_block**<br>size (size_t)<br>account<br>previous<br>representative<br>amount<br>link<br>signature<br>work<br>batch_hash | **256**<br>8<br>32<br>32<br>32<br>16<br>32<br>64<br>8<br>32 | Maps block hash to state block<br> |
| account_db | account | 32 | **account_info**<br>(send) head<br>receive_head<br>rep_block<br>open_block<br>balance<br>modified<br>block_count<br>receive_count| **168**<br>32<br>32<br>32<br>32<br>16<br>8<br>8<br>8 | Maps account to account information |
| receive_db | block_hash | 32 | **receive_block**<br>send_hash<br>previous | **64**<br>32<br>32 | Maps block hash to receive block|
| batch_tips_db | delegate_id | 1 | block_hash | 32 | Maps delegate id to hash of most recent batch block |
| micro_block_db | block_hash | 32 | **MicroBlock**<br>header<br>delegate<br>epoch_number<br>micro_block_number<br>(is_)last_micro_block<br>tips<br>number_batch_blocks<br>signature | **1,162**<br>48<br>32<br>4<br>2<br>1 *+1*<br>1,024<br>4 *+4*<br>32 | Maps block hash to micro block|
| micro_block_tip_db | "microblocktip"<br>(key value of 0) | 1 | block_hash | 32 | References micro block tip (only one key) |
| epoch_db | logos::block_hash | 32 | **Epoch**<br>header<br>(proposer) account<br>epoch_number<br>micro_block_tip<br>delegates, each with<br>(*account*)<br>(*vote*)<br>(*stake*)<br>transaction_fee_pool<br>signature | **1,696**<br>48<br>32<br>4 *+4*<br>32<br>1,536<br>(*32*)<br>(*8*)<br>(*8*)<br>8<br>32 | Maps block hash to epoch|
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

##### Header Layout

| Value | Size |
|-------|------|
| version<br>(message_)type<br>consensus_type<br>manual padding<br>timestamp<br>previous<br>| 1<br>1<br>1<br>5<br>8<br>32<br> |

Notes & pending fixes:

1. All sizes are in bytes, assuming a 64-bit architecture. Padding and sub-struct members are explictly shown.

2. Fixes
   + batch_db & state_db: change current implementation (batch contains list of full blocks)
   + state_db: remove timestamp?
   + receive_db: trim down

3. Proposed changes: 
   + add `next` field in message header (modified retroactively in DB) to enable doubly-linked list. Actual message sent over the wire obviously doesn't need to contain this field.
   + same for dual account chains
