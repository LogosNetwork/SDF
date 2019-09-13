#Identity Management Requirements


<table>
  <tr>
   <td>
<h1>Requirements</h1>


   </td>
   <td>
   </td>
   <td>
<h1>Notes</h1>


   </td>
  </tr>
  <tr>
   <td><strong>Overview</strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The Identity Management (IM) subsystem provides the overall identification capability for a node. It guides the Logos node software to
<p>
1. ascertain its role within the Logos Ecosystem,
<p>
2. keep track of key communication endpoints, namely delegates’ consensus and transaction-acceptor (TxA) addresses and ports, and
<p>
3. as a consensus participant, communicate its local endpoint(s) to the Logos network, as well as establish connections with peer delegates and their TxA's.
<p>
The main additional component introduced by IM is the concept of a “sleeve” (see definition below) within the core software. A sleeve can be in various states (defined below), which affects the types of role(s) the software can assume.
<p>
Additionally, action settings (defined below) impact the actual behavior the node exhibits in the network, including if/when it participates in consensus, if/when it connects to standalone TxA’s, as well as if/when it advertises its communication endpoints. The software provides an interface for user to modify and view the node identity states and action settings.
   </td>
   <td>
   </td>
   <td>communication endpoints refer to both consensus and TxA endpoints
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>State Definitions</strong>
<p>
<strong><em>locked, unlocked, sleeved</em> are three mutually exclusive states; the software must be at one of the three, and can transition from one to another</strong>
<p>
<strong>See diagram to the right for transitions. I, L, U, and S stand for init (initial state), locked, unlocked, and sleeved, respectively</strong>
   </td>
   <td>
   </td>
   <td>see usage in context in Requirements below
   </td>
  </tr>
  <tr>
   <td><strong>sleeve</strong>
<p>
a container within the software that stores and manages the node's governance identity ("governance ID")
   </td>
   <td>
   </td>
   <td>in other words, if the node is a regular p2p node, the sleeve would be empty
<p>
See below for governance identity definition
   </td>
  </tr>
  <tr>
   <td><strong><em>locked</em></strong>
<p>
state of the software where the sleeve (hence the node's governance ID) cannot be modified or assume the role of a delegate, and the software cannot perform actions associated with such role
   </td>
   <td>
   </td>
   <td><em>locked</em> is equivalent to the software being a "p2p" / regular "full" node. Its identity to the rest of the network is essentially NULL. Note that the sleeve database may contain encrypted keys while <em>locked</em>, but the keys are nevertheless inaccessible.
   </td>
  </tr>
  <tr>
   <td><strong><em>unlocked</em></strong>
<p>
state where the sleeve can be modified, but contains no content (hence no notion of governance ID), and the software cannot perform delegate actions
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong><em>sleeved</em></strong>
<p>
state where the sleeve can be modified, is managing one particular governance ID, and the software can perform delegate actions
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Action Setting Definitions</strong>
<p>
<strong>(settings are not mutually exclusive)</strong>
   </td>
   <td>
   </td>
   <td>see usage in context in HLR below
   </td>
  </tr>
  <tr>
   <td><strong><em>activated</em></strong>
<p>
setting mode that allows the software to perform delegate actions while in <em>sleeved</em> state if set to "<em>true</em>", and prevents the software to perform such actions if "<em>false</em>"
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong><em>auto broadcast enabled</em></strong>
<p>
setting mode that, if "<em>true</em>", instructs the software to automatically broadcast its consensus endpoint(s) and transaction acceptor endpoint(s) if in <em>sleeved</em> state as prescribed in the requirements below, and disables automatic broadcast if "<em>false</em>"
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Other definitions</strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Governance Identity</strong>
<p>
<strong>A governance identity is uniquely associated with the BLS key used for delegate election.</strong>
<p>
<strong>Specifically, given an BLS private key, the software can calculate its public key, query the epoch database, and ascertain its corresponding governance identity. If the BLS key is not matched with a recent database entry, then the node is a regular p2p one</strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>incumbent</strong>
<p>
The BLS public key's associated ID is a current delegate, i.e., the BLS key belongs to the elected delegate set recorded in epoch block with epoch number from i-2 to i-5 (if currently in epoch i)
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Requirements</strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>General Settings</strong>
<p>
<strong><em>describes how node ID action and TxA settings are configured and stored</em></strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall enable or disable modifying and viewing node identity states and action settings based on user configuration, in the form of a new "enable identity control" flag
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall attempt to load in node identity action settings ('activated') from user-provided configuration on startup
   </td>
   <td>
   </td>
   <td>setting/command parameter names are in single quotes
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to modify node identity action settings, with an option to specify a later epoch in which the new setting will begin taking effect
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall immediately apply the new setting when it receives an action setting modification command without a specified epoch
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall apply the new setting at epoch transition start of the provided epoch when it receives an action setting modification command with a specified epoch. For a specified epoch i, the new setting shall be applied at EPOCH_TRANSITION_START before the EPOCH_PROPOSAL_TIME for `i`
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall attempt to load in peer connectivity settings ('initial peers' and 'disable new connections') from user-provided configuration on startup
   </td>
   <td>
   </td>
   <td>This is more closely related to p2p component, but helpful to include here since it impacts governance node setup
   </td>
  </tr>
  <tr>
   <td>The software shall attempt to load in transaction acceptor (TxA) settings from user-provided configuration on startup
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall have two p2p commands to query from its peers other delegates' consensus endpoints and TxA endpoints
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Sleeve Access Management</strong>
<p>
<strong><em>describes how the sleeve is managed and accessed. note this section doesn't cover the sleeve's internal content</em></strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall be in <em>locked</em> state on startup if an existing sleeve database is detected; otherwise, it initializes a new sleeve DB with an empty password and starts in <em>unlocked</em> state
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to initialize the sleeve with a password ("<strong>update password</strong>"), with an option to 'overwrite' an existing sleeve. The software shall enter <em>unlocked</em> state after initialization
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to reset an existing sleeve ("<strong>reset sleeve</strong>")
   </td>
   <td>
   </td>
   <td>in case the user has no access to the old sleeve and wishes to re-initialize
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to unlock the sleeve with a password ("<strong>unlock</strong>" command)
   </td>
   <td>
   </td>
   <td>designated command names are in double quotes
   </td>
  </tr>
  <tr>
   <td>The software shall transition to <em>unlocked</em> upon receiving valid "unlock" while <em>locked</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to lock the sleeve ("<strong>lock</strong>" command)
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall transition to <em>locked </em>when it receives "lock" while <em>unlocked</em> or <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Sleeve Identity Management</strong>
<p>
<strong><em>describes how the sleeve content is managed</em></strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to store keys ("<strong>store keys</strong>" command) with BLS and ECIES private keys, with an 'overwrite' option
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall reject "store keys" while <em>locked</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall persist private keys in encrypted format locally and transition to <em>sleeved</em> when it is <em>unlocked</em> and receives "store keys"
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall persist private keys in encrypted format locally (clearing previously existing keys) when it is <em>sleeved</em> and receives "store keys" with 'overwrite' set to true
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall reject "store keys" without 'overwrite' set to true while <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>not handling the case where sleeved ID is incumbent. Specifically, there's no stopping an incumbent delegate from getting swapped out during the term, however it will only be to the delegate's detriment
   </td>
  </tr>
  <tr>
   <td>The software shall further transition to <em>sleeved</em> immediately after transitionining to <em>unlocked</em> if locally persisted governance keys are detected
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to clear the sleeve content ("<strong>unsleeve</strong>" command)
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall erase locally persisted governance keys and transition to <em>unlocked</em> when it receives "unsleeve" while <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>not handling the case where sleeved ID is incumbent. similar reasoning to note above
   </td>
  </tr>
  <tr>
   <td><strong>Delegate Connectivity Management</strong>
<p>
<strong><em>describes how the software as a delegate 1) manages TxA and consensus connections, and 2) broadcasts its own endpoints</em></strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to connect to TxA's ("<strong>txa connect</strong>") with a list of endpoint(s)
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall reject "txa connect" if not <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>Because not <em>sleeved</em> means the software has no BLS key to sign handshake messages with
   </td>
  </tr>
  <tr>
   <td>The software shall attempt to connect to the provided TxA endpoint(s) when it receives "txa connect" while <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall write the new endpoint(s) to configuration file (if not already present) after successfully connecting to TxA('s)
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall provide a command to delete TxA's ("<strong>txa delete</strong>") with a list of endpoint(s)
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall reject "txa delete" if not <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall disconnect from provided endpoints immediately when it 1) receives "txa delete" 2) while <em>sleeved</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall advertise endpoint deletion after disconnecting from TxA(s) through "txa delete"
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall sever any existing TxA connections after after transitioning from <em>sleeved</em> to either <em>unlocked</em> or <em>locked</em>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall perform delegate actions after transitioning to <em>sleeved</em> if 1) the governance ID is of an incumbent delegate and 2) 'activated' is currently set to "true"
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the delegate in the next epoch, one hour before the epoch transition, the core software shall broadcast consensus endpoint advertisement messages every 30 minutes via p2p
<p>
Each consensus endpoint advertisement message shall include the delegat's consensus endpoint (ip + port) and current time stamp, and shall be encrypted with another delegate’s public ECIES key and shall be signed with the sender delegate's private BLS key
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the delegate in the next epoch, one hour before the epoch transition, the core software shall broadcast TxA endpoint advertisement messages every 30 minutes via p2p
<p>
Each TxA endpoint advertisement message shall include the IPs and ports of the delegate's tx-acceptors.
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the delegate in the current/next epoch, the core software shall broadcast received TxA and consensus endpoints via p2p
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall attempt to connect to TxA's stored in current settings after transitioning to <em>sleeved</em>
<p>
The core software shall send to tx-acceptor a handshake message consisting of the current time-stamp
<p>
The message shall be signed with the delegate’s private bls key.
   </td>
   <td>
   </td>
   <td>the software does not need to be a delegate in order to connect to TxA. It does, however, in order to broadcast TxA endpoints
   </td>
  </tr>
  <tr>
   <td>As the delegate in the next epoch, the core software shall request every 10 minutes from its peers list other delegates' consensus endpoints that the delegate doesn't have
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the delegate in the current/next epoch, the core software shall send a handshake message upon connecting to another delegate
<p>
The message shall contain software version, current epoch, delegate’s id, type of the delegate’s set (current (old delegate set) or transitioning (new delegate set)), and its consensus endpoint
<p>
The message is signed with the delegate’s private bls key
   </td>
   <td>
   </td>
   <td>the handshake can take place for a delegate in current epoch if it comes online late / reconnects after crashing;
<p>
it takes place for a delegate in next epoch during the EPOCH_DELEGATES_CONNECT period before epoch start
   </td>
  </tr>
  <tr>
   <td>As the delegate, if the delegate handshake message can not be authenticated, the core software shall close the connection
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Endpoint Record-Keeping</strong>
<p>
<strong><em>describes how the core software (regardless of role) keeps track of key endpoint information in the network</em></strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall receive consensus endpoint advertisement messages via p2p and persist them in the database.
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>After epoch transition end event, the software shall remove from the database consensus endpoint advertisement messages from epochs more than one term length ago
   </td>
   <td>
   </td>
   <td>"one term length" since endpoint advertisements remain useful for the broadcaster delegate's upcoming term. Advertisement messages in epoch `i` shall be removed in epoch `i+5` (since they are useful in epochs `i+[1-4]`
   </td>
  </tr>
  <tr>
   <td>The software shall receive TxA endpoint advertisement messages via p2p and persist them in the database.
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>After epoch transition end event, the software shall remove from the database TxA endpoint advertisement messages from epochs more than one term length ago
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>TxA Behavior</strong>
<p>
<strong><em>describes how the transaction acceptor communicates with the core node</em></strong>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the tx-acceptor, the core software shall have the delegate's public BLS key included in the configuration file
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the tx-acceptor, the core software shall close the connection if the handshake message can’t be authenticated.
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Low Level Requirements (LLR)</strong>
<p>
<strong><em>unfinished, ignore for now</em></strong>
   </td>
   <td>Derived? (If not, link to HLR)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall, on startup, attempt to load in 'enable id control' from the configuration file
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall set 'enable id control' to "false" by default if this field is not defined in the configuration file or if the provided value is not true/false
   </td>
   <td>YES
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall store the values for 'enable id control' in memory
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall reject <strong>all</strong> node identity adjustment commands from the user if 'enable id control' is "false"
   </td>
   <td>
   </td>
   <td>"node identity adjustment commands" refer to all user commands in this requirements document
   </td>
  </tr>
  <tr>
   <td>The software shall, on startup, set 'activated' to "false" by default if this field is not defined in the configuration file or if the provided value is not true/false
   </td>
   <td>YES
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall, on startup, set 'auto broadcast enabled' to "false" by default if this field is not defined in the configuration file or if the provided value is not true/false
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall store the setting values for 'activated' and 'auto broadcast enabled' in memory
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall read in 'initial peers' from configuration file if such field is present
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall query from Logos DNS peer discovery service if 'initial peers' field is not present in configuration file
   </td>
   <td>YES
   </td>
   <td>Exact peer discovery source to be discussed
   </td>
  </tr>
  <tr>
   <td>The software shall store the value for 'initial peers' in memory
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall, on startup, read in 'disable new connections' from configuration file if such field is present
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall set 'disable new connections' to "false" by default if such field is not defined in the configuration file or if the provided value is not true/false
   </td>
   <td>YES
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The software shall store the values for 'disable new connections' in memory
   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
</table>

