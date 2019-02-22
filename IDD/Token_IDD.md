
## Note to reviewer: Single State Block is expanded to contain token related commands

### Single State Block

The *single state* block is a single unit of operation that can be performed on an account. The table below depicts the format of a state block:

| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Header | 8 | Message header|
| Account  | 32 | Account address | 
| Previous | 32 | Previous block hash on account| 
| SQN  | 4 | Sequence Number, the number of sends, increment only | 
| Type | 1 | Request type |
| Request | depend on the Request Type | Request |
| Fee | 16 | Transaction Fee |
| Signature | 64 | EdDSA signature |
| Work      | 8 |Proof of Work Nonce |

Note that Signature = Sign(Hash(Header.Version, Account, Previous, SQN, Type, Request, Transaction Fee))

Note that the "Work" field is not covered by the signature. This field is sent from the client to the primary delegate. After verifying it, the primary does not send it to the backup delegates.

#### Request Type
| Request Type | Code | Description |
| --- | ----------------- | ----------------- |
| Native_Multi_Send | 0 | Send multiple transaction from the native Logos account |
| Native_Change_Representative  | 1 | Change the representative of the native Logos account |
| Token_Issuance | 2 | Issue a new token | 
| Token_Issuance_Addition | 3 | Issue more of an existing token |
| Token_Change_Setting | 4 | Change one token account setting | 
| Token_Immute_Setting | 5 | Make one token account setting immutable |
| Token_Revoke | 6 | Revoke some amount of tokens from one account and deposit the tokens in another account | 
| Token_Freeze | 7 | Freeze a token account |
| Token_Fee_Rate | 8 | Change the transaction fee rate of a token |
| Token_Whitelist_Request | 9 | Request to be whitelisted of a token |
| Token_Whitelist | 10 | Whitelist an account |
| Token_Issuer_Info | 11 | Update the issuer's information |
| Token_Controller | 12 | Add or remove a controller, or change the privilege of an existing controller |
| Token_Burn | 13 | Burn some amount of tokens from the token account |
| Token_Account_Send | 14 | Send some amount of tokens from the token account |
| Token_Account_withdraw_Fee | 15 | Withdraw some amount of tokens from the transaction fee pool |
| Token_multi_send | 16 | Send multiple transactions from an user account |

Note that open and receive transactions of accounts are inferred from the send (any type of send) transaction request. 

#### Native_Single_Send
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Address | 32 | Transaction target address |
| Amount | 16 | Transaction Amount.|

#### Native_Multi_Send
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Count | 1 | Number of requests, [1-8]| 
| Native_Single_Send[]   | sizeof Native_Single_Send * Count | Array of Native_Single_Send requests |

#### Native_Change_Representative
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Address | 32 | Representative address |

#### Token_Issuance
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Symbol | 8 | The symbol of the token. It could be less than 8 bytes. Only English letters and numbers are allowed. Other characters are ignored. |
| Name | 32 | The name of the token. It could be less than 32 bytes. Only English letters and numbers are allowed. Other characters are ignored. |
| Issuer_Address | 32 | The Issuer's account address |
| Total_Supply | 16 | The total amount of token issued |
| Settings | 2 | A set of settings as shown in the Token Setting table |
| Controller_Count | 1 | The number of controllers, in [0, 10] |
| Controllers[] | Controller_Count * sizeof Controller | An array of controllers. The format of controllers is defined in the Controller table. |
| Issuer Info | <= 512 | Optional field for the issuer to put its information. It is suggested to be json encoded with fields such as email, homepage, location, etc. |

Note that tokens always have 3 less decimals than the native Logos. 

##### Token Setting
Every token account has a set of settings stored in a bit array, and each setting also have a mutability setting, as the table below shows.

| Index | Setting | Description |
| --- | ----------------- | ----------------------------- |
| 0 | Allow_Additional_Token | If additional token could be issued |
| 1 | Allow_Additional_Token_Mutable | If the Allow_Additional_Token setting is mutable |
| 2 | Allow_Revoke | If tokens in any account could be revoked |
| 3 | Allow_Revoke_Mutable | If the Allow_Revoke setting is mutable |
| 4 | Allow_Freeze | If any account could be frozen |
| 5 | Allow_Freeze_Mutable | If the Allow_Freeze setting is mutable |
| 6 | Allow_Adjust_Fee | If the transaction fee rate could be changed |
| 7 | Allow_Adjust_Fee_Mutable | If the Allow_Adjust_Fee setting is mutable |
| 8 | Need_Whitelist | If an account needs to be whitelisted before receiving tokens |
| 9 | Need_Whitelist_Mutable | If the Need_Whitelist setting is mutable |

##### Controller
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Address | 32 | The account address of the controller |
| Privilege | 3 | The set of privileges that the controller has, as defined in the Controller Privilege table. |

##### Controller Privilege
Every controller has a set of privileges, which is stored in a bit array as shown in the table below. 

| Index | Privilege | Description |
| --- | ----------------- | ----------------------------- |
| 0 | Allow_Additional_Token_Setting | If the controller can change the Allow_Additional_Token setting in the token account | 
| 1 | Allow_Additional_Token_Mutable_Setting | If the controller can change the Allow_Additional_Token_Mutable setting in the token account |
| 2 | Allow_Revoke_Setting | If the controller can change the Allow_Revoke setting in the token account |
| 3 | Allow_Revoke_Mutable_Setting | If the controller can change the Allow_Revoke_Mutable setting in the token account |
| 4 | Allow_Freeze_Setting | If the controller can change the Allow_Freeze setting in the token account |
| 5 | Allow_Freeze_Mutable_Setting | If the controller can change the Allow_Freeze_Mutable setting in the token account |
| 6 | Allow_Adjust_Fee_Setting | If the controller can change the Allow_Adjust_Fee setting in the token account |
| 7 | Allow_Adjust_Fee_Mutable_Setting | If the controller can change the Allow_Adjust_Fee_Mutable setting in the token account |
| 8 | Need_Whitelist_Setting | If the controller can change the Need_Whitelist setting in the token account |
| 9 | Need_Whitelist_Mutable_Setting | If the controller can change the Need_Whitelist_Mutable setting in the token account |
| 10 | Promote_Controller | If the controller can promote another account to be a controller and assign privileges to new or existing controllers |
| 11 | Issue_Additional_Token | If the controller can issue additional tokens |
| 12 | Revoke | If the controller can tokens from an account |
| 13 | Freeze | If the controller can freeze an account |
| 14 | Adjust_Fee | If the controller can change the transaction fee rate | 
| 15 | Whitelist | If the controller can whitelist an account |
| 16 | Burn | If the controller can burn tokens from the token account balance |
| 17 | Withdraw_Balance | If the controller can withdraw tokens from the token account balance |
| 18 | Withdraw_Fee | If the controller can withdraw tokens from the token account transaction fee pool |

Note that the _mutable settings can not be changed from mutable (True) to immutable (False).

##### Token_ID 
Once a Token_Issuance request is post-committed, a unique token account identifier hash is computed:
Token_ID = Hash(Symbol, Name, Issuer_Address, Primary Delegate Address, Pre-prepare SQN, Pre-prepare timestamp).

#### Token_Issuance_Addition
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Amount | 16 | The amount of additional tokens |

#### Token_Change_Setting
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Setting_Index | 1  | Index to the token settings bit array. Must be in [0,2,4,6,8]. |
| Value | 1 | The new value of the setting, if 0 false, else true |

#### Token_Immute_Setting
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Setting  | 1  | Index to the token settings bit array. Must be in [1,3,5,7,9]. |

#### Token_Revoke
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| From | 32 | The account that token is revoked from |
| To | 32 | The account that the token is deposited to |
| Amount | 16 | The amount of token to revoke | 

#### Token_Freeze
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Address | 32 | The account address |
| Action | 1 | If 0 freeze, else unfreeze |

#### Token_Fee_Rate
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| PercentageOrFlat | 1 | If 0, the rate will be a percentage, else the rate is flat |
| Rate | 16 | New rate, [0, 100] if percentage |

#### Token_Whitelist_Request
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |

#### Token_Whitelist
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Count | 2 | Number of accounts to be whitelisted |
| Addresses | 32*Count | The accounts to be whitelisted |

#### Token_Issuer_Info
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Info | <= 512 | New information |

#### Token_Controller
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| AddOrRemove | 1 | If 0, add, else remove |
| Controller | sizeof Controller | The controller as defined in the Controller table |

#### Token_Burn
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Amount | 16 | The amount of token to burn |

#### Token_Account_Send
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| To | 32 | The account that the token is deposited to |
| Amount | 16 | The amount of token to send |

#### Token_Account_withdraw_Fee
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| To | 32 | The account that the token is deposited to |
| Amount | 16 | The amount of token to send |

#### Token_Single_Send
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| To | 32 | The account that the token is deposited to |
| Amount | 16 | The amount of token to send |
| Fee | 16 | The transaction fee in tokens |

#### Token_Multi_Send
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Token_ID | 32 | Unique ID of the token |
| Count | 2 | The number of single transactions |
| Txn[] | Count * sizeof Token_Send_Txn | An array of transactions Token_Send_Txn, as defined below.
| Fee | 16 | The transaction fee in tokens |

#### Token_Send_Txn
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| To | 32 | The account that the token is deposited to |
| Amount | 16 | The amount of token to send |

### Rejection
The existing method of rejection (via Process_Result objects) will be extended to include more error codes.
