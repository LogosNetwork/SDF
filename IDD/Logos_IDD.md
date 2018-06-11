
#Interface Description 

This document details the format of the exchanged data over the network. The native memory layout, database layout and intermediate data format can be different from the format depicted in this document.  TBD: Figure how if NANO uses network order/big endian for encoding/transmission

## Transaction Request

### Single State Block

The *single state* block is a single unit of operation that can be performed on an account. The table below depicts the format of a state block:

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte| Logos Core Protocol version|
| Message Type| 1 - Byte| Type of message|
| Account  | 32-Byte | Account address |
| Previous | 32-Byte | Previous head block on account; 0 if *open* block|
| Target Address [0..7] | 32-Byte | Multipurpose Field, up to an array of 8 addresses; <br/>See Table below for additional clarification<br/>Must have the same number of elements as transaction amount |
| Transaction Amount [0..7]   | 16-Byte | Amount to send, up to an array of 8 elements. <br/>Must have the same number of elements as target address |
| Transaction Fee | 8 - Byte| Transaction Fee for this transaction|
| Signature | 32 - Byte | BLS signature |
| Work      | 8-Byte  |Proof of Work Nonce |

Depending on the transaction intent, the "target" field is multipurpose:

| Type | Value Description | 
| --- | ----------------- | 
| Send | Destination address | 
| Change | Representative address |


The following table depicts the message type definition:

| Type | Value  | 
| --- | ----------------- | 
| Pre-Prepare | 0  | 
| Prepare | 1 |
| Post-Prepare | 2 | 
| Commit | 3 |
| Post-Commit | 4 |
| Pre-Prepare Reject | 5 |
| Post-Prepare Reject | 6 |
| Single State Block | 7 |

Note that open and receive transactions are inferred from the send transaction request. Change requests must have a value of 0 to indicate it is a change request.


## Pre-Prepare

### Batch State Block


The *batch state* block is one or more single state blocks with additional metadata. Batch State Block is used for Logos' consensus.

At any given time, the primary delegate can only have one batch transaction active for consensus at a time. 


| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte| Logos Core Protocol version|
| Message Type| 1 - Byte| Type of message|
| Number of Single State Blocks  | 1-Byte | The number of elements included in this batch block |
| Epoch Sequence Number  | 4-Byte | Starting from 0 at the beginning of the epoch |
| Previous Batch State Block | 32-Byte | Previous Batch state block; <br/>Reference the current epoch block if this is the first batch block of the epoch|
| Single State Block [...] | Variable |Number of single blocks specified in the number field |
| Timestamp | 8 - bytes | UTC timestamp (millisecond)|
| Signature | 32 - Byte | BLS signature |
| Hash  | 32 - Byte | Hash digest of the batch block |


### Prepare (Pre-Prepare Acceptance)

This message is used by the back up delegate to accept the pre-prepare initiated by the primary delegate

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte| Logos Core Protocol version|
| Message Type| 1 - Byte| Type of message|
| Hash| 32 - Byte | Hash digest of the batch block from pre-prepare|
| Timestamp | 8 - Byte | UTC timestamp (millisecond)|
| Signature | 32 - Byte | BLS signature |



### Pre-Prepare Reject

TBD This should include error code on why it is reject and this will trigger error handling.

### Post-Prepare

This message is used by the back up delegate to accept the pre-prepare initiated by the primary delegate

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte| Logos Core Protocol version|
| Message Type| 1 - Byte| Type of message|
| Hash| 32 - Byte | Hash digest of the batch block from pre-prepare|
| Timestamp | 8 - bytes | UTC timestamp (millisecond)|
| Participation Map | 8 - bytes | Bitmap of the delegates that participated in this round of signing| 
| Aggregated Signature| 32 - Byte | BLS aggregated signature

### Commit

This message is used by the primary delegate to perform final proof aggregation and database update of transactions

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte| Logos Core Protocol version|
| Message Type| 1 - Byte| Type of message|
| Hash| 32 - Byte | Hash digest of the batch block from pre-prepare|
| Timestamp | 8 - bytes | UTC timestamp (millisecond)|
| Signature | 32 - Byte | BLS signature |

### Commit Reject 

TBD This should include error code on why it is reject and this will trigger error handling.

### Post-Commit

This message is used as the final proof to perform database update for the transactions contained within the block.

| Field Name |Size| Description |
| --- | -------------| ----------------- |
| Version | 1 - Byte| Logos Core Protocol version|
| Message Type| 1 - Byte| Type of message|
| Hash| 32 - Byte | Hash digest of the batch block from pre-prepare|
| Timestamp | 8 - bytes | UTC timestamp (millisecond)|
| Participation Map | 8 - bytes | Bitmap of the delegates that participated in this round of signing| 
| Aggregated Signature| 32 - Byte | BLS aggregated signature