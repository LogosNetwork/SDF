#Epoch Transition Requirements


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
   <td>Delegate-consensus: the Axios consensus protocol as defined by the white paper and the consensus requirement document. A delegate-consensus session is run by a set of N delegates, where N=32 for example.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Microblock-interval: the predefined constant interval between two consecutive microblocks. The default is 10 minutes.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Microblock-time_i: the cut off time for Batch State Blocks (BSBs) that are included in microblock_i. I.e., if the timestamp of a post-committed BSB > microblock-time_{i-1} and <= microblock-time_i, then the BSB <strong><em>should</em></strong> be included in microblock_i. In addition, Microblock-time_{i+1} - Microblock-time_i == Microblock-interval.
   </td>
   <td>Note: the timestamp in a post-committed BSB is its primary's <strong>local</strong> timestamp. However since the timestamp also went through the consensus protocol, it became <strong>global</strong>.
<p>
Note: the last microblock of an epoch is special. It must include all post-committed BSBs of the epoch, which were not included in the previous BSBs.
   </td>
  </tr>
  <tr>
   <td>Microblock-propose-time_i: the time for delegates to propose microblock_i. Microblock-propose-time_i = Microblock-time_i + t*Microblock-interval, where t must be large enough to compensate for clock differences, propagation delay, and possible fallback consensus sessions. For example, t = 1.
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
   <td>Clock-diff-max: the maximum clock difference allowed among delegates. It is 20 seconds.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Epoch-interval: the predefined constant interval between two consecutive epochs. The default is 12 hours.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Epoch-start-time_i: the starting time of the epoch_i.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Epoch-transition-period_i: [Epoch-start-time_i - Clock-diff-max, Epoch-start-time_i + Clock-diff-max]
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Epoch-transition-start-time_i: Epoch-start-time_i - Clock-diff-max
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Epoch-transition-end-time_i: Epoch-start-time_i + Clock-diff-max
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
   <td>Delegate_epoch_i: a delegate of epoch_i.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Delegate-new_epoch_i: a node added to the set of delegates at the beginning of epoch_i. The default is 8 of them.
   </td>
   <td>When an epoch finishes, 1/L (L > 3) of the delegates are replaced with new ones. Since L > 3, so we have to replace less than 1/3 of delegates, given our current default configuration with 32 delegates, we can replace 8 of them. I.e, we set L = 4.
   </td>
  </tr>
  <tr>
   <td>Delegate-retired_epoch_i: a delegate of epoch_{i-1}, but not longer in epoch_i. In fact, the set of Delegate-retired_epoch_i == the set of Delegate-new_epoch_{i-L}
   </td>
   <td>It is replaced by a delegate-new_i.
   </td>
  </tr>
  <tr>
   <td>Delegate-persistent_epoch_i: a delegate in the set of delegate_epoch_i - the set of delegates-new_epoch_i.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>E#_i: epoch number i in consensus messages
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
   <td><strong>Setup</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As Delegate-new_epoch_i, the core software shall make network connections to all other Delegate_epoch_i, S (default 300 >> Clock-diff-max) seconds before epoch-transition-start-time_i.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall synchronize with the NTP server every hour.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall stop all processing and disconnect itself from the network if the drift between its system clock and the NTP server is greater or equal to 20 seconds
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall move all timed out transaction requests from the secondary waiting list to the primary waiting list.
   </td>
   <td>Note that this requirement is duplicated from the one in the consensus requirement document.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Microblock</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Overview: microblocks are proposed by delegates periodically to facilitate the archiving of transaction during epoch block construction. A microblock is a checkpoint that includes all the post-committed BSBs since the previous microblock.
<p>
Microblocks go through the delegate-consensus protocol like other transactions. Similar to the account "send" requests, a microblock also has a previous hash field, which can be used to select the default primary. When it is time to create the next microblock, all the delegates can insert their own microblocks in the secondary waiting list with a timeout (or insert a placeholder for the microblock so that the computation of the Merkel tree root of the batch blocks is delayed). The timeout value could also be randomized and larger than the one for "send" requests.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall initiate delegate-consensus sessions for the microblock requests, where the delegate-consensus session including error handling is defined in the "primary delegate consensus" section and the "backup delegate consensus" section of the consensus requirement document.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall propose microblocks as defined in the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall propose microblock_i at microblock-propose-time_i, where microblock-propose-time_i = microblock-propose-time_{i-1} + 10 minutes.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall include in microblock_i all the post-committed BSBs (1) with timestamps <= microblock-time_i and (2) not included in microblock_{i-1}.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a Delegate_epoch_i, the core software shall propose the last microblock of epoch_{i-1}. For a regular epoch transition, the proposed time is Epoch-start-time_i + Microblock-interval. For a recall epoch transition, the proposed time is new_epoch_start_time + (Microblock-interval * 2 - new_epoch_start_time % Microblock-interval).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall use the microblock_{i-1}'s hash to compute the index of the default primary delegate of microblock_i.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall use the hash of the last microblock of epoch_{j-1} as the previous hash of the first microblock of epoch_j.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the default primary delegate, the core software shall initiate a delegate-consensus session for microblock_i at microblock-propose-time_i according to its local clock.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, when receiving a pre-prepare for microblock_i, the core software shall validate the pre-prepare according to the IDD. If the pre-prepare is valid, the core software shall start a backup consensus session (as defined in the Backup delegate consensus section of the consensus requirement document).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, when receiving an invalid pre-prepare for microblock_i, the core software shall ignore the pre-prepare message.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate that is not the default primary, if has not received a valid pre-prepare for microblock_i at microblock-propose-time_i according to its local clock, as a fallback, the core software shall add a request of microblock_i into its secondary waiting list with a timer. The timeout value = random_timeout(60, 60).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Receiving post-committed microblock_i, the core software shall validate it according to the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>NOTE: Given the current set of requirements, the default primary of a microblock will have the first chance to propose. If its consensus session fails, other delegates will propose later since they will insert the request of the microblock in their secondary waiting list. If all of them fail, recall with be initiated. Please note that nothing other than primary timeout is needed for entering the recall_wait status of the recall protocol. Also note that the propose_time is 10 minutes after the cutoff_time of any microblock. It is more than enough for the honest nodes to have the same view of what should be included in the microblock.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the primary delegate of the consensus session of a microblock, the core software shall set a recall timer if the session fails.
   </td>
   <td>Note that this requirement clearifies the error handling of failed microblock consensus sessions. Also note that the microblock and epoch components of the code does not need to implement this requirement. Because it summarizes two requirements in the consensus requirement "As a primary delegate, the core software shall set a recall timer if the pre_prepare timer is timed out and the prepare phase does not have two-thirds majority." and "As a primary delegate, the core software shall set a recall timer if the post_prepare timer is timed out and the commit phase does not have two-third majority."
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Epoch Block</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall initiate delegate-consensus sessions for the epochblock requests, where the delegate-consensus session including error handling is defined in the "primary delegate consensus" section and the "backup delegate consensus" section of the consensus requirement document.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall propose epochblocks as defined in the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall select the most voted delegate as the default primary delegate.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the default primary delegate, after creating or receiving the last microblock of epoch_i, the core software shall initiate a delegate-consensus session for epochblock_i.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, when receiving the pre-prepare message of an epochblock, the core software shall verify if the message conforms to the IDD. If the pre-prepare is valid, the core software shall start a backup consensus session (as defined in the Backup delegate consensus section of the consensus requirement document).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate, when receiving a invalid pre-prepare message of an epochblock, the core software shall ignore the pre-prepare message.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate that is not the default primary, after creating or receiving the last microblock of epoch_i, as a fallback, the core software shall add a request of epochblock_i into its secondary waiting list with a timer. The timeout value = random_timeout(60, 60).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a representative, after receiving the post-commit message of an epochblock, the core software shall verify it against the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Receiving a validated epoch block, the core software shall split the transaction fee pool to all representatives (including delegates) according to a formula (TBD), and append a block to the receive chain of every representative.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As the primary delegate of the consensus session of an epochblock, the core software shall set a recall timer if the session fails.
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
   <td><strong>Epoch Transition (During Epoch-transition-period_i)</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Overview: When an epoch finishes, D (8 by default) of the delegates are replaced with new ones. The retiring delegates still participate in consensus during Epoch-transition-period_i. At a client's side, it switches to the new epoch at epoch-start-time_i according to its local clock. I.e., after epoch-start-time_i, the client selects primaries from delegate_epoch_i, (with the index computed from the "previous hash").
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Delegate-new_epoch_i</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate-new_epoch_i, the core software shall start proposing pre-prepares only after epoch-transition-start-time_i according to its local clock.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate-new_epoch_i, the core software shall use e#_i in its pre-prepare messages.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Delegate_persistent_epoch_i</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate_persistent_epoch_i, the core software shall reject pre-prepare messages with e#_{>=i} before epoch-transition-start-time_i according to its local clock.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate-persistent_epoch_i, during epoch-transition-period_i, the core software shall start to use e#_i in its pre-prepare messages if (1) the local time >= epoch_start_time_i, (2) received a post-commit message with e#_i, or (3) received more than 1/3 pre-prepare with error code NEW_EPOCH.
   </td>
   <td>The reason for (2): No circular dependency among requests or blocks. I.e., there should not be a chain where an earlier transaction on the chain is post-committed with e#_i, but a later transaction on the chain is post-committed with e#_{i-1}.
<p>
red is added during code review
   </td>
  </tr>
  <tr>
   <td>As a delegate-persistent_epoch_i, after started using e#_i, if received a pre-prepare message with e#_{i-1}, the core software shall (1) reject it by replying a pre-prepare reject message with an error code NEW_EPOCH, and (2), queue the batch of requests in the BSB into its secondary waiting list with a timer. The timeout value = random_timeout(10, 20).
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
   <td>Delegate-retired_epoch_i
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate-retired_epoch_i, the core software shall only use e#_{i-1} in its pre-prepares.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate-retired_epoch_i, the core software shall enter the state of ForwardOnly (1) at Epoch-start-time_i according to its local clock, or (2) received >=1/3 pre-prepare reject messages with error code NEW_EPOCH.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a delegate-retired_epoch_i, after entered the state of ForwardOnly, the core software (1) stop proposing pre-prepares, (2) forward the requests in the local waiting list to their default primary in epoch_i, and (3) forward requests received later to their default primary in epoch_i.
   </td>
   <td>(2) and (3) are deferred
   </td>
  </tr>
  <tr>
   <td>As a delegate-retired_epoch_i, at Epoch-transition-end-time_i according to its local clock, the core software shall stop accepting requests and disconnect the connections to other delegates.
   </td>
   <td>
   </td>
  </tr>
</table>


