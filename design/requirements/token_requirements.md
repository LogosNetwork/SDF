#Token Requirements


<table>
  <tr>
   <td>
<h1>Requirements</h1>


   </td>
   <td>
<h1>Notes</h1>


   </td>
  </tr>
  <tr>
   <td><strong>Definitions</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Token Account: The genesis account for a specific token issued by an issuer. It contains important settings of that specific account
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Token Account Settings</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Overview: The token account capability is an extension to the conventional Logos Network's account architecture which allows for token issuance on the Logos Network. The token account contains various important token accounting settings that dictate the allowable operations on the tokens. Some settings are immutable while others can be configured post account opening. For example, if the total supply is set as immutable, then the token issuer cannot issue additional tokens post the initial issuance. The section below describes the detailed requirements for token account settings. Note that all commands are executed only when delegate consensus are reached.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for token issuance on the Logos Network per the message format described in the Logos IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall charge a transaction fee for all successful execution of token related commands to the token account.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any request to change any of the mutability setting of any token account from FALSE to TRUE.
   </td>
   <td>If the mutability flag is toggled to FALSE, then the associated setting and the flag itself can not be changed. However, we can always turn mutability from TRUE to FALSE to lock in a setting.
   </td>
  </tr>
  <tr>
   <td>The core software shall reject additional issuance of the token if the mutability setting of the total supply is set to immutable (FALSE).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any request to change the revocability setting if the mutability setting of revocability is set to immutable (FALSE)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any request to change the freezability setting if the mutability setting of freezability is set to immutable (FALSE)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any request to change the transaction fee setting if the mutability setting of transaction fee setting is set to immutable (FALSE)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any request to change the whitelist setting if the mutability setting of whitelist is set to immutable
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Token Account Operations and Executions</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Overview: Token accounts support many operations to facilitate with activities associated with issued tokens. There is a mutability flag for some of the settings to dictate whether certain operations can be performed on the account. For example, if the revocability mutability setting is set to false, then the token issuer cannot revoke an account’s balance. The section below provides details regarding supported operations and their dependencies.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to update token issuer's information per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a mutability setting for revocability.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a mutability setting for freezability.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a mutability setting for transaction fee.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a mutability setting for issuing additional tokens.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a mutability setting for whitelist.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to change the token account's mutability settings from TRUE to FALSE per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to issue additional tokens of the same type per the IDD if the additional token issuance setting is set to TRUE
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to add/delete token account's controller list.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to configure a token account's controllers privilege per the IDD.
   </td>
   <td>Each controller can be set to have different levels of privilege similar to a Linux file system - revocability. freezability, whitelist capability, burn capability, transaction fee setting and the ability to withdraw tokens/transaction fees. Mutability setting still overrides these privilege settings in the sense that if a mutability setting for revocability is set to FALSE, then even if revocability for a specific controller is set to TRUE, no funds can be revoked.
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to add an account to its whitelist per the IDD if the whitelist setting is enabled.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject the request to add an account to its whitelist if the account itself has not requested to be whitelisted.
   </td>
   <td>For an account to be whitelisted, the whitelist setting must be set to TRUE in the token account first, then a non-token holding account can request whitelisting. After the request is executed, the token account can then whitelist the account by sending a request to the network. Only after both requests are executed, the account will have the token shown as whitelisted in its own account and it is then ready to receive funds.
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to remove an account from the whitelist.
   </td>
   <td>The protocol still allows for sending of tokens if the account is off the whitelist - but it can no longer receive.
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to revoke tokens from an account per the IDD if the revocability setting is set to TRUE.
   </td>
   <td>The command has a destination address in which the revoked balance is sent to.
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to toggle the revocability setting.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to add an account to the freeze list per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall toggle the freeze flag of an account to TRUE if a valid request to freeze the account is executed and the account already holds the corresponding token or the account has been whitelisted.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall add the account to its freeze list within the token account if a valid request to freeze the account is executed and the account exists but does not hold the corresponding token.
   </td>
   <td>This requirement is more design oriented - but it is called out here so that we save storage space. We only add the accounts to a list if its not open or does not hold the specific type token to be frozen.
   </td>
  </tr>
  <tr>
   <td>The core software shall reject freeze list command if the target account does not exist.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to toggle the freezability setting.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to toggle the whitelist setting.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept a request to send tokens if the freezability setting in the token account is set to FALSE even if the account is frozen.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall delete the freeze list from the token account if the freezability setting is set to false and the corresponding mutability setting is also set to FALSE.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to send tokens from the token account account balance per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to send Logos native currency from the token account account balance per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to configure the token transaction fee of the token account per the IDD.
   </td>
   <td>percentage/flat fee.
   </td>
  </tr>
  <tr>
   <td>The core software shall only accept operation requests from the corresponding token account's controllers accounts with sufficient privileges.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to burn tokens from the token account by reducing its current balance by the requested amount.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject all requests to send tokens if the target or source account is frozen.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to send tokens from the token account transaction fee balance per the IDD
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall collect token transaction fee and deposit it into the token account if the token transaction fee setting is enabled.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Expanded Account capability</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Overview: In order to fully support token operations, Logos basic accounts capabilities are expanded to include additional attributes. These attributes help transact tokens on top of Logos Network. The section below describes the details extension to the Logos accounts.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall add the token name and its associated balance to the account when a new token is received.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to send tokens from the token holder's account per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command to request for whitelisting from the token holder's account per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any send tokens command if the targeted account is not whitelisted and the whitelist setting is set to TRUE in the token account.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a status field for each of the token the account owns or after submitting a whitelist request that shows the whitelist status as well as whether the funds are frozen.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject a whitelist request command if the token does not require whitelisting.
   </td>
   <td>
   </td>
  </tr>
</table>

