## Overview of databases


| MDB_dbi Name | Key | Key Size | Value | Value Size | Description |
| --- | ------------- | ----------------- | --- | --- | --- |
| `account_db` | `account_address` | 32 | `account_info` | 219+ | Maps an account_address to the account information |
| `reservation_db` | `account_address` | 32 | `reservation_info` | 36 | Maps an account_address to the reservation information |
| `state_db` | `block_hash` | 32 | Request | ToBeUpdated after election,  was [231-567] _\(183 + [1-8] * 48\)_ | Maps a block hash to a request |
| `receive_db` | `block_hash` | 32 | ReceiveBlock | 66 | Maps a block hash to a receive block |
| `token_user_status_db` | `token_user_id` | 32 | TokenUserStatus | 2 | Maps a token_user_id to a  TokenUserStatus|
| `batch_db` | `block_hash` | 32 | Post-Commited BatchStateBlock | ToBeUpdated after election,  was [203-48,203] _\(203 + [0-1500] * 32\)_ | Maps a block hash to a Post-committed BatchStateBlock, which contains a list of hashs of StateBlocks |
| `batch_tips_db` | `delegate_index+epoch_number` | 5 | `block_hash` | 32 | Maps a (delegate index, epoch number) combination to the Key (block_hash) of the most recent Post-Commited BatchStateBlock in a given number for a given delegate index |
| `micro_block_db` | `block_hash` | 32 | Post-Commited MicroBlock | 1,230 | Maps a block hash to a micro block |
| `micro_block_tip_db` | 0 | 1 | block_hash | 32 | References the Key (block_hash) of the most recent micro block. This database has only 1 row, so the key of this database is always 0. |
| `epoch_db` | `block_hash` | 32 | Post-Commited Epoch | 4,345 | Maps a block hash to an epoch block |
| `epoch_tip_db` | 0 | 1 | `block_hash` | 32 | References the Key (block_hash) of the most recent epoch block. This database has only 1 row, so the key of this database is always 0. |
| `representative_db` | `account_address` | 32 | `representative_info` | 112 | Maps an account address of a representative to info about that representative |
| `candidacy_db` | `account_address` | 32 | `candidacy_info` | 100 | Maps an account address of a delegate candidate to info about that candidate |
| `leading_candidacy_db` | `account_address` | 32 | `candidacy_info` | 100 | The current leaders of the election (top 8) |
| `remove_candidates_db` | 0 | 1 | `account_address` | 32 | The candidates to remove from candidacy_db on epoch transition. This db uses duplicate keys |
| `remove_reps_db` | 0 | 1 | `account_address` | 32 | The representatives to remove from representative_db on epoch transition. This db uses duplicate keys |
| `voting_power_db` | `account_address` | 32 | `voting_power_info` | 100 | Maps rep account address to info about a representative's voting power |
| `voting_power_fallback_db` | `account_address` | 32 | `voting_power_fallback` | 32 | Maps rep account address to info about a representative's voting power | 
| `epoch_rewards_db` | `account_address` + `epoch_number` | 36 | `epoch_rewards_info` | 49 | Maps the account address of a representative combined with the epoch number to the rewards earned by the rep and its supporters during that epoch. |
| `global_epoch_rewards_db | `epoch_number` | 4 | `global_epoch_rewards_info` | 48 | Maps epoch number to aggregate info about all reps who voted in that epoch |
| `master_liabilities_db` | `liability_hash`  | 32 | `liability` | 85 | Maps a liability_hash to a liability |
| `rep_liabilities_db` | `account_address` | 32 | `liability_hash` | 32 | Maps a rep address to liability_hash. This db uses duplicate keys | 
| `secondary_liabilities_db` | `account_address` | 32 | `liability_hash` | 32 | Maps an account address to liability_hash. This db uses duplicate keys |
| `staking_db`   | `account_address` | 32 | `staked_funds` | 80 | Maps account address to staked_funds. Funds may be locked proxied or staked to self |
| `thawing_db`   | `account_address` | 32 | `thawing_funds` | 84 | Maps account address to thawing_funds. This db uses duplicate keys |
| `rep_rewards_db` | `block_hash` | 32 | `rep_rewards` | TBD | Maps the account address of a representative combined with the epoch number to the rewards earned by the rep and its supporters during that epoch. |


Note all the sizes are in bytes.

Note: whenever a field is a 1 byte bool flag, 0 represents false and 1 represents true

Note: remove_candidates_db and remove_reps_db use duplicate keys: every account address has key 0, but there are many such pairs.

Note: rep_liability_db uses duplicate keys. There may be many liability_hashes associated with a given representative.
Any liability with hash H and target R adds a corresponding R -> H mapping in rep_liability_db.

Note: thawing_db uses duplicate keys. There may be many sets of thawing funds associated with a given account. However, the thawing funds for a specific account are sorted by expiration time, with the thawing funds expiring the soonest coming first.

Note: liabilities are stored in master_liability_db and hashes of those liabilities are stored in rep_liability_db, secondary_liability_db
and with any associated staked_funds or thawing_funds.

## Database Details

### `account_db`

| Value Item | Size | Description |
| --- | --- | --- |
| type | 1 | native account (0) or token account (1) |
| balance | 16 | amount of Logos this account currently owns |
| modified | 8 | seconds elapsed since unix epoch |
| (send) head | 32 | tip of account send chain |
| sequence | 4 | number of send transactions this account has created |
| receive_head | 32 | tip of account receive chain |
| receive_count | 4 | number of receive transactions this account has received |
| type depended information |  | see the tables below |

#### native account additional information
If the account type is native, the account include the following addtional information

| Value Item | Size | Description |
| --- | --- | --- |
| staking_subchain_head | 32 | hash of request where account most recently changed the amount locked proxied or staked| 
| rep | 32 | address of account's representative (set to 0 if account has no rep or is rep) |
| open_block | 32 | original send block that opened this account | 
| token_count | 2 | number of different kinds of tokens |
| token_entries | token_count * 50 | Array of token_entries, see the table below |
| epoch_thawing_updated | 4 | Epoch number thawing was most recently pruned |
| epoch_secondary_liabilities_updated | 4 | Epoch number secondary liabilities were most recently pruned |
| available_balance | 16 | amount of Logos this account is able to spend |

Note, staking_subchain_head is the head of a chain of requests issued by this account
that affect the amount of funds this account has locked proxied or staked to self.
To calculate amount of funds proxied or staked at any given time in the past, traverse this subchain.
See IDD for more details. 

Note: available balance = (balance) - (amount staked, locked proxied and thawing)




##### token_entry
| Value Item | Size | Description |
| --- | --- | --- |
| token_id | 32 | token ID |
| whitelisted | 1 | if the user account is whitelisted |
| frozen | 1 | if the user account is frozen|
| balance | 16 | balance of this kind of token |

Please refer the Request IDD for token_id computation.

#### token account additional information
If the account type is token, the account include the following addtional information

| Value Item | Size | Description |
| --- | --- | --- |
| total_supply | 16 | The total amount of token issued |
| token_balance | 16 | remaining balance hold by the token account |
| token_fee_balance | 16 | fees collected, unwithdrawed |
| Fee_Type | 1 | percentage or absolute |
| Fee_Rate | 16 | [0, 100] if Fee_Type is percentage |
| Symbol | <=8 | The symbol of the token. |
| Name | <=32 | The name of the token. |
| Issuer Info | <= 512 | Optional field for the issuer to put its information.|
| Controller_Count | 1 | The number of controllers, in [0, 10] |
| Controllers[] | Controller_Count * sizeof Controller | An array of controllers. The format of controllers is defined in the Controller table. |
| Settings | 8 | A set of settings as shown in the Token Setting table |

##### Controller
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Address | 32 | The account address of the controller |
| Privilege | 8 | The set of privileges that the controller has, as defined in the Controller Privilege table. |

### `reservation_db`

| Value Item | Size | Description |
| --- | --- | --- |
| reservation_hash | 32 | hash of the request that reserved this account |
| reservation_epoch | 4 | the epoch when this account was reserved |

### `state_db`

| Value Item | Size | Description | Hash Sequence |
| --- | --- | --- | --- |
| Type | 1 | Request type | 1 |
| Origin  | 32 | Account address | 2 |
| Previous | 32 | Previous block hash on account| 3 |
| Fee | 16 | Transaction Fee | 4 |
| SQN  | 4 | Sequence Number, the number of sends, increment only | 5 |
| BatchHash | 32 | Key (BlockHash) of the post-committed BatchStateBlock containing this StateBlock | - |
| IndexInBatch | 2 | Index of the StateBlock in the post-committed BatchStateBlock | - |
| Request | depend on the Request Type | Request | 6 |
| Signature | 64 | EdDSA signature | - |
| HasWork  | 1 | If Proof of Work is included. Always false for database | - |
| Next      | 32 | Next is the hash of the next Request. It is a database only field | - |

Please refer the Request IDD for Request details.

#### Transaction

| Field Name |Size (Byte)| Description |Hash Sequence |
| --- | -------------| ----------------- | --- |
| Target Address | 32 | Transaction target address | 1 |
| Amount | 16 | Transaction Amount.| 2 |

### `receive_db`

| Value Item | Size | Description | Hash Sequence |
| --- | --- | --- | --- |
| previous | 32 | previous receive's hash in account receive chain | - |
| send_hash | 32 | the original StateBlock corresponding to this receive | 1 |
| transaction_index | 2 | index to the transaction array of the StateBlock | 2 |

### `token_user_status_db`
| Value Item | Size | Description |
| --- | --- | --- |
| whitelisted | 1 | if the user account is whitelisted |
| frozen | 1 | if the user account is frozen|

#### `token_user_id`
A token_user_id = Hash(token_id, account_address). Please refer the Request IDD for token_id computation.

### `batch_db`, `micro_block_db`, and `epoch_db`

BatchStateBlocks, MicroBlocks, and Epochs are blocks that must be approved (post-committed) by delegate consensus sessions. All the three kinds of post-committed blocks share some common fields.

### Structure and common fields of post-committed blocks databases

(Common field size total: 201B)

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | --- |
| MessagePrequel | 8 | Message header (see below) |  1 |
| Primary | 1 | Primary delegate's index |  2 |
| Epoch Number  | 4 | Epoch number to which the block belongs |  3 |
| Delegate Epoch Number  | 4 |  Epoch number of the delegates running the consensus session |4 |
| Sequence Number | 4 | Starting from 0 at the beginning of the epoch | 5 |
| Timestamp | 8 | UTC timestamp (millisecond)| 6 |
| Previous | 32 | Hash of the Previous Consensus Block; <br/>Reference the NULL block if this is the first BatchStateBlock of this delegate. <br/> Reference the genesis Epochblock if this is the first Epochblock. <br/> Reference the Microblock in the genesis Epochblock if this is the first Microblock. | 7 |
| Signature | 32 | Primary's BLS signature | - |
| Consensus Block | Sizeof(Consensus Block) | A multipurpose field. It can be a Batch State Block, a Microblock, or an Epoch. (see below) <br/> The type of the block is in the MessagePrequel.| 8 |
| Participation Map of Prepare | 8 | Bitmap of the delegates that participated in prepare|  - |
| Aggregated Signature| 32 | BLS aggregated signature of prepare | - |
| Participation Map of Commit | 8 | Bitmap of the delegates that participated in commit|  - |
| Aggregated Signature|  32 | BLS aggregated signature of commit | - |
| Next | 32 | Hash of the next Consensus Block | - |

#### Message Prequel

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | --- |
| Version | 1 | Logos Core Protocol version|  1 |
| Message Type| 1 | Type of message| - |
| Consensus_Type | 1 | BatchStateBlock, MicroBlock, or Epoch | -|
| MPF | 1 | Multipurpose Field (currently not used) | -|
| Payload size | 4 | Size of the message |-|

#### Batch State Block

| Field Name |Size| Description | Hash Sequence |
| --- | -------------| ----------------- | --- |
| Count  | 2 | The number of Single State Blocks included in this batch block | 1 |
| Single State Block Hashes[Count] | 32 * Count | hashes of Single State Blocks | 2 |

#### Micro Block

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | --- |
| Last Micro Block Flag | 1 | Last Micro block flat | 1
| Number of Batch Blocks| 4 | Number of batch blocks included in the microblock | 2 |
| Batch Block Tips [32] | 32*32 | Tip of the batch block chain for each delegate| 3 |

#### Epoch

| Field Name |Size | Description | Hash Sequence |
| --- | -------------| ----------------- | --- |
| Microblock tip  | 32 | last micro-block's hash | 1 |
| Transaction Fee Pool | 16 | Transaction Fee Pool | 2 |
| Delegate [32] | sizeof(Delegate)*32 | 32 delegates and their election results (see below) | 3 |

#### Delegate

| Field Name |Size | Description |Hash Sequence |
| --- | -------------| ----------------- | ---|
| Address | 32 | Account address of the delegate | 1 |
| Publick Key | 64 | Delegate's BLS public key |2|
| Vote | 16 | Votes the delegate received |3|
| Stake | 16 | Stake of the delegate |4|

### `batch_tips_db`

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| tip | 40 | Current Tip of the request block. One per `(delegate_id, epoch_number)` combination (although at most only the current epoch and previous are stored in database). |

### `micro_block_tip_db`

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| tip | 40 | Current microblock's Tip. Only one in the network. |

### `epoch_tip_db`

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| tip | 40 | Current epoch block's Tip. Only one in the network. |

#### tip
| Field Name |Size | Description |Hash Sequence |
| --- | -------------| ----------------- | ---|
| epoch number| 4 | epoch number of the block | 1 |
| sequence number| 4 | sequence number of the block | 2 |
| digest | 32 | hash of the block | 3 |

### `representative_db`

| Value Item | Size | Description |
| --- | --- | ---|
| candidacy_action_tip | 32 | hash of most recent post-committed candidacy action request (Announce or Renounce) |
| representative_action_tip | 32 | hash of most recent post-committed representative action request (StartRepresenting or StopRepresenting) |
| election_vote_tip | 32 | hash of most recent post-committed election vote |
| stake | 16 | amount staked as representative |

### `candidacy_db`

| Value Item | Size | Description |
| --- | --- | --- |
| votes_received_weighted | 16 | Total votes received so far this epoch, already weighted by stake of caster |
| bls_key | 64 | bls key to be used as delegate to encrypt consensus messages |
| cur_stake | 16 | Self stake for current epoch |
| next_stake | 16 | Self stake for next epoch |
| epoch_modified | 4 | Which epoch this record was most recently modified | 
| levy_percentage | 1 | Levy percentage used for reward calculation |

### `leading_candidacy_db`

| Value Item | Size | Description |
| --- | --- | --- |
| votes_received_weighted | 16 | Total votes received so far this epoch, already weighted by stake of caster |
| bls_key | 64 | bls key to be used as delegate to encrypt consensus messages |
| cur_stake | 16 | Self stake for current epoch |
| next_stake | 16 | Self stake for next epoch |
| epoch_modified | 4 | Which epoch this record was most recently modified | 
| levy_percentage | 1 | Levy percentage used for reward calculation |


Note, leading_candidacy_db is simply the current top 8 candidates from the candidacy_db. It is continuously kept up to date based on votes cast

### `remove_candidates_db`
| Value Item | Size | Description |
| --- | --- | --- |
| account_address | 32 | account address of candidate to be removed |

### `remove_reps_db`
| Value Item | Size | Description |
| --- | --- | --- |
| account_address | 32 | account address of representative to be removed |

### `voting_power_db`
| Value Item | Size | Description |
| --- | --- | --- |
| current | 48 | voting_power_snapshot for the current epoch |
| next | 48 | voting_power_snapshot for the next epoch |
| epoch_modified | 4 | which epoch this record was most recently modified |

Note, current is used to weight votes, and next is continuously modified during the epoch.
The first time this record is modified in a given epoch, current is replaced by next.
See staking/elections design doc for more details.

Note, a rep is only purged from voting_power_db when that rep's voting power goes to 0.

#### `voting_power_snapshot`
| Value Item | Size | Description |
| --- | --- | --- |
| locked_proxied_amount | 16 | Amount of funds locked proxied to this representative |
| unlocked_proxied_amount | 16 | Amount of funds unlocked proxied to this representative |
| self_stake_amount | 16 | Amount of funds this representative has staked to themselves |

#### `voting_power_fallback_db`
| Value Item | Size | Description |
| --- | --- | --- |
| power | 16 | Voting power (sum of components, with unlocked proxy diluted)
| total_stake | 16 | Sum of locked proxied and self stake |

Note, voting_power_fallback_db is used to alleviate a certain race condition
that arises when voting near the epoch boundary. See Elections and Staking design doc for more
details. A given rep will only have a record in this db if they did not vote in
the previous epoch.

### `epoch_rewards_db`
| Value Item | Size | Description |
| --- | --- | --- |
| levy_percentage | 1 | Levy_percentage in effect for rep for given epoch |
| total_stake | 16 | amount of self stake + amount of locked proxied funds proxied to this rep |
| remaining_reward | 16 | Amount of rewards left to be claimed |
| total_reward | 16 | Total rewards before claiming |

Note, epoch_rewards_db is keyed by concatenating representative account address with epoch number.


### `global_epoch_rewards_db`
| Value Item | Size | Description |
| --- | --- | --- |
| total_stake | 16 | Sum of self stake and lock proxied of all representatives who voted |
| remaining_reward | 16 | Amount of rewards left to be claimed for all reps who voted |
| total_reward | 16 | Total reward before claiming |

Note, global_epoch_rewards_db is keyed only by epoch number

### staking_db
| Value Item | Size | Description |
| --- | --- | --- |
| target | 32 | AccountAddress that funds are staked/locked proxied to |
| amount | 16 | amount of funds staked |
| liability_hash | 32 | Hash of associated liability |

### thawing_db
| Value Item | Size | Description |
| --- | --- | --- |
| expiration_epoch | 4 | Epoch number when funds become available |
| target | 32 | Account Address that funds were staked to prior to thawing |
| amount | 16 | Amount of funds thawing |
| liability_hash | 32 | Hash of associated liability | 

Note, an expiration of 0 means those funds are frozen. Frozen funds are funds
that were unstaked by a delegate or candidate that was elected. Frozen funds
will have their expiration set when the delegate finishes their current term.

Note, in the db, the expiration_epoch is serialized as (max_32_bit_int - expiration_epoch).
The conversion is done during serialization and deserialization. LMDB orders records
of duplicate keys by lexicographical ordering, so this "inversion" ensures that
the thawing funds furthest from expiring are stored first.


#### liability
| Value Item | Size | Description | Hash Sequence |
| --- | --- | --- | --- |
| target | 32 | Account Address that funds are liable for | 1 |
| source | 32 | Account Address that owns the funds | 2 |
| amount | 16 | Amount of funds liable for | - |
| expiration_epoch | 4 | Epoch number when liability expires | 3 |
| is_secondary | 1 | Bool flag set to true if liability is secondary liability | 4 |

Note, an expiration of 0 means the liability has no expiration.
Currently staked funds are liable for the account those funds are staked to, and
a liability expiration is not set until those funds are staked to a different account
or begin thawing.

### `rep_rewards_db`
| Value Item | Size | Description |
| --- | --- | --- |
| rep_rewards | TBD | Rewards earned by a representative and its supporters during a particular epoch period |
