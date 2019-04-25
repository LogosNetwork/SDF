

### Request

The Request is a single unit of operation that can be performed on an account. The table below depicts the format of a Request:

| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Type | 1 | Request type | Yes |
| Origin  | 32 | Account address | Yes |
| Previous | 32 | Previous block hash on account| Yes |
| Fee | 16 | Transaction Fee | Yes |
| SQN  | 4 | Sequence Number, the number of sends, increment only | Yes |
| Request | depend on the Request Type | Request | Yes |
| Signature | 64 | EdDSA signature | - |
| HasWork  | 1 | If Proof of Work is included |- |
| Work      | 8 | Proof of Work. It is included in the message if HasWork is true | -|

Note that Signature = Sign(Hash). Also note that the "Work" field is not covered by the signature. This field is sent from the client to the TxAcceptor. After verifying it, it is discarded. So it will not in the database.

#### Request Type
| Request Type | Code | Description |
| --- | ----------------- | ----------------- |
| Native_Send | 0 | Send multiple transaction from the native Logos account |
| Native_Change  | 1 | Change the representative of the native Logos account |
| Token_Issuance | 2 | Issue a new token |
| Token_Issuance_Addition | 3 | Issue more of an existing token |
| Token_Change_Setting | 4 | Change one token account setting |
| Token_Immute_Setting | 5 | Make one token account setting immutable |
| Token_Revoke | 6 | Revoke some amount of tokens from one account and deposit the tokens in another account |
| Token_Adjust_User | 7 | Adjust a client token account |
| Token_Fee_Rate | 8 | Change the transaction fee rate of a token |
| Token_Issuer_Info | 9 | Update the issuer's information |
| Token_Controller | 10 | Add or remove a controller, or change the privilege of an existing controller |
| Token_Burn | 11 | Burn some amount of tokens from the token account |
| Token_Account_Distribute | 12 | Send some amount of tokens from the token account |
| Token_Account_withdraw_Fee | 13 | Withdraw some amount of tokens from the transaction fee pool |
| Withdraw Logos | 14 | Withdraw logos from the token account |
| Token_send | 15 | Send multiple transactions from an user account |
| Election_Vote | 16 | Cast vote for current epoch's election |
| Announce_Candidacy | 17 | Declare delegate candidacy for upcoming elections |
| Renounce_Candidacy | 18 | Undeclare delegate candidacy for upcoming elections |
| Start_Representing | 19 | Become Representative on the network |
| Stop_Representing  | 20 | Stop being a representative on the network |
| Claim_Reward | 21 | Claim all available rewards |

Note that open and receive transactions of accounts are inferred from the send (any type of send) transaction request.

#### Transaction
| Field Name |Size (Byte)| Description |
| --- | -------------| ----------------- |
| Address | 32 | Transaction target address |
| Amount | 16 | Transaction Amount.|

#### Native_Send
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- | -- |
| Count | 1 | Number of requests, [1-8]| - |
| Transaction[]   | sizeof Transaction * Count | Array of Transactions | Yes |

#### Native_Change_Representative
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- | -- |
| Client | 32 | to be deleted | - |
| Address | 32 | Representative address | Yes |

#### Token_Issuance
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |-- |
| Token_ID | 32 | Token_ID | Yes |
| Symbol | 8 | The symbol of the token. It could be less than 8 bytes. Only English letters and numbers are allowed. Other characters are ignored. | Yes |
| Name | 32 | The name of the token. It could be less than 32 bytes. Only English letters and numbers are allowed. Other characters are ignored. | Yes |
| Total_Supply | 16 | The total amount of token issued | Yes |
| Fee_Type | 1 | percentage or absolute | Yes |
| Fee_Rate | 16 | [0, 100] if Fee_Type is percentage | Yes |
| Settings | 8 | A set of settings as shown in the Token Setting table | Yes |
| Controller_Count | 1 | The number of controllers, in [0, 10] | - |
| Controllers[] | Controller_Count * sizeof Controller | An array of controllers. The format of controllers is defined in the Controller table. | Yes |
| Issuer Info | <= 512 | Optional field for the issuer to put its information. It is suggested to be json encoded with fields such as email, homepage, location, etc. | Yes |

Note that tokens always have 3 less decimals than the native Logos.

##### Token_ID
Every token account has a unique identifier that is computed as:
Token_ID = Hash(Issuer's Previous-hash, Issuer_Address, Symbol||Name), where || means concatenation.

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
| Privilege | 8 | The set of privileges that the controller has, as defined in the Controller Privilege table. |

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
| 10 | Issue_Additional_Token | If the controller can issue additional tokens |
| 11 | Revoke | If the controller can tokens from an account |
| 12 | Freeze | If the controller can freeze an account |
| 13 | Adjust_Fee | If the controller can change the transaction fee rate |
| 14 | Whitelist | If the controller can whitelist an account |
| 15 | Update_IssuerInfo | If the controller can update Issuer information |
| 16 | Update_Controller | If the controller can update another controller |
| 17 | Burn | If the controller can burn tokens from the token account balance |
| 18 | Distribute | If the controller can withdraw tokens from the token account balance |
| 19 | Withdraw_Fee | If the controller can withdraw tokens from the token account transaction fee pool |

Note that the _mutable settings can not be changed from mutable (True) to immutable (False).

#### Token_Issuance_Addition
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |---- |
| Token_ID | 32 | Unique ID of the token | Yes |
| Amount | 16 | The amount of additional tokens | Yes |

#### Token_Change_Setting
| Field Name |Size (Byte)| Description |Hash |
| --- | -------------| ----------------- |---- |
| Token_ID | 32 | Unique ID of the token | Yes |
| Setting_Index | 1  | Index to the token settings bit array. Must be in [0,2,4,6,8]. |Yes |
| Value | 1 | The new value of the setting, if 0 false, else true |Yes |

#### Token_Immute_Setting
| Field Name |Size (Byte)| Description |Hash |
| --- | -------------| ----------------- |---- |
| Token_ID | 32 | Unique ID of the token |Yes |
| Setting  | 1  | Index to the token settings bit array. Must be in [1,3,5,7,9]. |Yes |

#### Token_Revoke
| Field Name |Size (Byte)| Description |Hash |
| --- | -------------| ----------------- |---- |
| Token_ID | 32 | Unique ID of the token |Yes |
| From | 32 | The account that token is revoked from |Yes |
| To | 32 | The account that the token is deposited to |Yes |
| Amount | 16 | The amount of token to revoke | Yes |

#### Token_Adjust_User
| Field Name |Size (Byte)| Description |Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token |Yes |
| Address | 32 | The account address |Yes |
| Status | 1 | See the table below |Yes |

##### Token_User_Status
| Status | value |
| --- | -------------|
| Frozen | 0 |
| Unfrozen | 1 |
| Whitelisted | 2 |
| NotWhitelisted | 3 |

#### Token_Fee_Rate
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| PercentageOrFlat | 1 | If 0, the rate will be a percentage, else the rate is flat | Yes |
| Rate | 16 | New rate, [0, 100] if percentage | Yes |

#### Token_Issuer_Info
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| Info | <= 512 | New information | Yes |

#### Token_Update_Controller
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| AddOrRemove | 1 | If 0, add, else remove | Yes |
| Controller | sizeof Controller | The controller as defined in the Controller table | Yes |

#### Token_Burn
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| Amount | 16 | The amount of token to burn | Yes |

#### Token_Distribute
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| To | 32 | The account that the token is deposited to | Yes |
| Amount | 16 | The amount of token to send | Yes |

#### Token_Account_withdraw_Fee
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| To | 32 | The account that the token is deposited to | Yes |
| Amount | 16 | The amount of token to send | Yes |

#### Withdraw Logos
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| To | 32 | The account that the token is deposited to | Yes |
| Amount | 128 | The amount of logos to send | Yes |

#### Token_Send
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Token_ID | 32 | Unique ID of the token | Yes |
| Count | 1 | The number of single transactions | - |
| Transaction[] | Count * sizeof Transaction | An array of transactions | Yes |
| Fee | 16 | The transaction fee in tokens | Yes |

#### Election_Vote
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Count | 1 | The number of entries in vote array [1-8] | - |
| Vote[] | Count * sizeof Vote | An array of votes | Yes |
| Epoch_Number | 4 | Epoch number request was issued in | Yes |

#### Vote
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Account_Address | 32 | Public key of candidate | Yes |
| Num_Votes | 1 | The number of votes for specified account address [1-8] | Yes |

Note: The sum of all the Num_Votes fields in an ElectionVote must be no greater than 8

#### Start_Representing
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Stake | 16 | Amount of logos to stake as representative | Yes |
| Epoch_Number | 4 | Epoch number request was issued in | Yes |

#### Stop_Representing
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Epoch_Number | 4 | Epoch number request was issued in | Yes |

#### Announce_Candidacy
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Stake | 16 | Amount of logos to stake as delegate (optional, can use existing stake as representative) | Yes |
| BLS_Key | 64 | BLS key used as a delegate | Yes |
| Epoch_Number | 4 | Epoch number request was issued in | Yes |
| Identity_Encryption_Key | 32 | Key used to encrypt IP | Yes |

#### Renounce_Candidacy
| Field Name |Size (Byte)| Description | Hash |
| --- | -------------| ----------------- |--|
| Epoch_Number | 4 | Epoch number request was issued in | Yes |

#### Claim_Reward
No additional fields
