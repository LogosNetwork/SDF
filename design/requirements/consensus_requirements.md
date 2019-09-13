#Consensus Requirements




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
   <td>delegate: one of the n Logos network delegates within an epoch, where n = 3f+1, and f is the (weighted) number of byzantine faulty delegates that the network can tolerant. A delegate has two weights, one is based on the votes it received when elected, the other is based on its stake. Both of them are capped.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><em>Note: A delegate has two weights, one of them is based on the votes it received, and the other is based on the stake it advertised when elected. For a consensus phase to reach majority, both thresholds must be met.</em>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>two-third majority: > 2f of weighted delegates approved a phase of a consensus session.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>three-fourth majority: >= (3/4)n of weighted delegates approved a phase of a consensus session
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>one-third rejection: > f of weighted delegates explicitly reject a phase of a consensus session.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>int random_timeout(int init_delay, int range){
<p>
int offset, x = rand() % NUM_DELEGATES;
<p>
if (x < 2) offset = 0;
<p>
else if (x < 4) offset = range/2;
<p>
else offset = range;
<p>
return init_delay + offset;}
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
   <td><strong>Blacklisting</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall maintain a blacklist of client addresses that had provable malicious activities.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall remove a blacklist item inserted during epoch_i at the end of epoch_{i+1}.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Note: to prevent someone from blocking (for many epochs) an account that had a provable malicious activity in the past, by replay the proof every epoch, we need a way to identify when the malicious activity happened. Including a timestamp in the proof works, but need extra bytes. Since we already have the Sequence Number in single state blocks, we will use that.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall ignore a provable malicious activity if the sequence number in the request is lower than the sequence number of the last successful request of the account.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall broadcast the provable malicious activities to the network.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, received a message proving a malicious activity of an account, the core software shall verify the proof. If the proof is valid and activity is malicious, the core software shall blacklist the account. Otherwise, the core software shall ignore the message.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Note: to be provable, the evidence must include the account's signature. So a send request with a valid signature and a invalid PoW is a provable malicious activity. But if the signature is invalid, it is not a provable activity. For the detailed format, please reference the IDD
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall add a double-spend send request to the blacklist. If two different and individually valid transactions of the same account use the same previous hash value, then the account double-spends.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall maintain a local gray-list of client addresses that had unprovable malicious activities.
   </td>
   <td>As discussed on the Sep 11, there will be a RPC interface where nodes can input their own rules of graylist.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Initiating Consensus</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core Software shall provide a public interface to receive transaction requests that conform to the Interface Description Document (IDD).
   </td>
   <td>IDD is created and will be updated as needed, which will have the format of all messages that are input/output of our software. Anything that leaves the software or comes in from an outside source, even if it is initiated by the core software itself, needs to be listed in the document.
   </td>
  </tr>
  <tr>
   <td>The core software shall initiate consensus sessions for client send requests that conform to the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>See the epoch_requirement document for other requests for which the core software shall initiate consensus sessions.
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
   <td><strong>Transaction Request</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Overview: When receiving a transaction request, a delegate can immediately process it, queue the request, forward it to another delegate, or reply immediately. In addition, when a delegate sees a request for the first time either as an individual request or in the batchblock of a pre-prepare message, it performs several validity checks.
<p>
Every request has a default primary, whose index can be computed from the ""Previous hash"" of the account, as defined in the IDD. E.g. the last 5 bits of the Previous hash is an integer (evenly distributed) in [0,31], which can be used as an index to find the primary. So based on the Previous hash field in the request (also in the account stored in the local database), every delegate can compute the index of the default primary. When a delegate receives a new request, if it is not the default primary, it will forward the request to the default primary. It also stores the request locally in the secondary waiting list and set a timer. When the timer expires, if the request has not been handled by the default primary, this delegate will handle it by moving the request to its primary waiting list.
<p>
Every delegate has two waiting lists, the primary waiting list and the secondary waiting list. The items in the primary waiting list will be used when creating a new batch state block. The items in the secondary waiting list have timers. When a timer expires, the item is moved from the secondary waiting list to the primary waiting list, so that it could be included in later batch state blocks."
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reply with the post-commit if the transaction request is a duplicate of a previous transaction that is already in a post-commit.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall ignore the new transaction request if the request is already in an ongoing consensus session
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall ignore the new transaction request if the request is already in the local node's waiting list.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>If the local node is not the default primary of a transaction request, the core software shall store the request in its secondary waiting list with a timer. The timeout value is 5 seconds.
   </td>
   <td>Clients should pick delegates based on the previous send hash and then send to 2nd delegate if it doesn’t receive the post-commit within a timeout.
   </td>
  </tr>
  <tr>
   <td>The core software shall monitor the time out status of all transaction requests in the secondary waiting list.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall move all timed out requests from the secondary waiting list to the primary waiting list.
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
   <td><strong>Verification</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject a transaction request if the associated account is blacklisted.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary, the core software shall reject the transaction if it is gray-listed.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core Software shall perform validity checks on the transaction amount before processing the transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup, the core software shall perform validity checks on the time-stamp of the Batch State Block. If the difference between the message time-stamp and the local clock is larger than 20 seconds, then the batch state block shall be rejected.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core Software shall perform validity checks on the transaction target before processing the transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core Software shall perform validity checks on the previous transaction hash value before processing the transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall perform validity checks on transaction fee amount (lower than the minimum transaction fee defined in the epoch block).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary, the core Software shall perform validity checks on the PoW before processing the transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Note of the reservation status: For every account, a delegate can only prepare (reserve) for the same previous hash (equivalent to the PBFT sequence number) once. Once prepared, it cannot be unprepared until the epoch after the next, I.e. > one epoch. If transaction A is prepared, but due to any reason the batch state block cannot be processed, then it is OK to prepare for the same transaction A if found in another batchblock.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core Software shall perform validity checks on the reservation status of the account associated with the transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, when verifying the reservation status of an account, the core Software shall return true if the account is not reserved yet, or is reserved by the same transaction request, or is reserved for more than one epoch (I.e., if an account is reserved in epoch_i, it will be un-reserved at the beginning of epoch_{i+2}, if it has not been un-reserved already.).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>If the reservation status is valid, the core Software shall reserve the account with the current epoch number and the current transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core Software shall perform validity checks on the signature before processing the transaction request.
   </td>
   <td>for performance reason, more expensive validity check should be performed later
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall send a rejection message with the proper error code if a transaction request cannot be validated.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall start the bootstrap protocol if a gap (unknown previous hash) in transaction request is detected.
   </td>
   <td>(1) Similarly if a delegate realizes that it missed some previous messages, it shall bootstrap. (2) Question: will there be too many bootstraps? No, since we put the account causing the bootstrap in the gray list temporarily.
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall add an entry into the blacklist if a provable malicious transaction is detected.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall add an entry into the gray-list if an unprovable malicious transaction is detected.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept a batch of transaction requests as a transaction request.
   </td>
   <td>Sep 10 2018, note that BSB changed to batch of requests
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Summary: the following sections of requirements define the Axios consensus</strong>
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
   <td><strong>Primary delegate consensus</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall initiate the PBFT Axios consensus after the transaction has passed the validation checks.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall have at most one on going primary consensus session.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall propose a pre-prepare message immediately for valid transaction requests in the primary waiting list, if the primary does not have an ongoing primary consensus session.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall accumulate valid transaction requests in the primary waiting list when it has an ongoing primary consensus session.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall sign the proposed pre-prepare message with its own BLS signature private key.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall broadcast the Pre-Prepare message in the format that is called out in the IDD to all the other delegates of the current epoch.
   </td>
   <td>All the other delegates become backup delegate for this consensus session
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall set a pre-prepare timer once the pre-prepare message is broadcasted. The timeout value of the timer is 60 seconds.
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
   <td>As a primary delegate, the core software shall perform validity checks for all the prepare message it receives according to the IDD.
   </td>
   <td>Check the validity of the messages.
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall abort the proposed pre-prepare message if the prepare messages received did not reach two-thirds majority when the pre-prepare timer is expired.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall reject, by sending reject messages with proper error codes to the clients, the subset of the transaction requests which had one-third rejection, once it aborts the last pre-prepare message.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall re-propose a subset of the transaction requests in a new pre-prepare message if the requests did not have one-third rejection, once it aborts the last pre-prepare message. The new pre-prepare message shall use the same previous hash and sequence number as the aborted one.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall broadcast a post-commit message defined in the IDD to the network if the proposed batchblock reached three-fourth majority before the pre-prepare timer is expired.
   </td>
   <td>Note the details on how to sign the messages are intentionally omitted as they should be part of the software design document. Remember, SRS = What is the software doing. SDD = How does it do it and IDD = What format is the data we are operating on for external I/O
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall broadcast a post-prepare message defined in the IDD to all the backup delegates if the proposed batchblock reached two-third majority but not three-fourth majority after the pre-prepare timer is expired.
   </td>
   <td>The primary broadcasts the post-prepare message to all the backup delegates.
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall set a post-prepare timer once the post-prepare message is broadcasted. The timeout value of the timer is 60 seconds.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall broadcast a post-commit message defined in the IDD to the network if the validated commit messages reached two-third majority before the post-prepare timer is expired.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall send a response back to the client if a valid post-commit message has been created or received and the software is not under load per TBD.
   </td>
   <td>Or we use the gossip based block propagation and no direct response to the clients
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall set a recall timer if the pre_prepare timer is timed out and the prepare phase does not have two-third majority.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall set a recall timer if the post_prepare timer is timed out and the commit phase does not have two-third majority.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall initiate the recall protocol once the recall timer is expired and no valid post-commit message has been received.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>After starting, the core software shall initiate a primary consensus session with an empty BatchStateBlock.
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
   <td><strong>Backup delegate consensus</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall verify the validity of a Pre-Prepare message according to the format called out in the IDD once a Pre-Prepare message is received from a valid primary delegate.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall verify that the transaction requests contained within the Pre-Prepare message is a valid candidate for the requesting account.
   </td>
   <td>A little vague here. Perhaps this is what we need? We need to find a good balance between how explicit a requirement should be vs how much it costs to maintain, test and update the requirement in the future.
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall reject any invalid transaction requests within a pre-prepare message.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall send a reject message for all transaction requests that are rejected per the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall allow the same valid request to appear in multiple batch blocks included in different pre-prepare messages.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall send a prepare message in a format that is specified in the IDD back to the primary delegate once the pre-prepare message is validated and accepted.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall start the bootstrap protocol if an unknown previous hash is detected in a transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall start the slash protocol if a provable malicious pre-prepare message is received.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall verify the validity of a Post-Prepare message according to the format called out in the IDD once a post-prepare message is received from the primary delegate.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall send a commit message in a format that is specified in the IDD back to the primary delegate once the post-prepare message is validated and accepted.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>After sending the prepare message of pre-prepare_i containing BatchStateBlock_i, if the local node receives pre-prepare_j containing BatchStateBlock_j from the same delegate, where pre-prepare_i and pre-prepare_j have the same previous hash and the same sequence number and BatchStateBlock_j is a subset of BatchStateBlock_i, then the core software shall drop the backup consensus session for pre-prepare_i and start a backup consensus session for pre-prepare_j
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
   <td><strong>Fallback delegate consensus</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Notes, in the Aug 28, 2018 design meeting, 3 alternatives of fallback consensus were discussed, (A) retry by the client, (B) add the requests in the pre-prepare (individually or as a whole batch) into the secondary waiting list by all the backup delegates, (C) the original broadcast based fallback consensus. The choice (A) will be implemented by the clients. In addition, either (B) or (C) will be implemented by the delegates.
   </td>
   <td>To-discuss, for (B), should the requests be inserted individually or as a whole batch? If individually, the requests could end up in different batch state blocks. If as a batch, then the secondary waiting list and the primary waiting list should be able to accommodate both individual requests and batch state blocks. Also, we have the requirement that once a post-commit is received, the individual request listed on the waiting list should be removed if they are covered by the post-commit. We could extend this requirement so that the batch can be removed too.
   </td>
  </tr>
  <tr>
   <td>TO-KEEP (very likely), the requirements below are for the secondary waiting list based fallback consensus. I.e., the design choice (B)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, the core software shall store the batch of requests in the batch state block of a valid pre-prepare in its secondary waiting list with a timer. The timeout value = random_timeout(10, 20). Once the timer expires, the batch state block shall be moved to the primary waiting list and later be proposed by the local node in a pre-prepare message.
   </td>
   <td>To-discuss, the timer can be randomized with a cap. So that a small number (1 or 2 with high probability) of backup delegates will re-propose the batch state block earlier than other backup delegates.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Post-commit</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, the core software shall remove the requests in waiting lists if they are included in a post-commit just received.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall update their local database once a valid post-commit is received the first time or created locally.
   </td>
   <td>A little vague here. But the node should perform the validation as listed in the "Verification" section.
   </td>
  </tr>
  <tr>
   <td>For the post-commit of a "send" transaction, the core software shall store the post-commit and update the following fields: sender’s balance, target’s balance, sender's send chain, target's receive chain, and transaction fee pool.
   </td>
   <td>
   </td>
  </tr>
</table>


