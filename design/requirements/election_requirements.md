<h1>Election Requirements</h1>


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
   <td><strong>Overview</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>This is the requirements document for elections, which concerns delegate candidacy and voting. Elections are on a per epoch basis. Each epoch has a set of candidates that representatives can vote on. Each epoch we are electing a set of delegates that will replace some but not all of the current delegates. In our current configuration, we replace 8 of 32 delegates each epoch. However, delegates are allowed to run for reelection, and we could end up in a situation where the 8 elected candidates are actually current delegates, and the delegate set does not actually change.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Running for election takes place over several epochs. A candidate will announce in the first epoch, be voted on in the second epoch, learn the results of the election in the third epoch, and then actually be a delegate in the fourth, if they were elected. Reference the below visual. Delegates serve for a fixed length term, currently 4 epochs, though they may run for reelection and remain in office.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Candidates that were not reelected remain candidates until they issue a command to renounce their candidacy. Renouncing candidacy takes one epoch to take effect, just like announcing candidacy. Note, this creates a situation where if you renounce candidacy but are elected in that epoch, you must serve. You won't be added to subsequent elections, but you are forced to serve one term
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Announcing candidacy, and renouncing candidacy, are done by creating requests very similar to a send transaction and go through delegate consensus. The same goes for voting, as well as commands related to representative status.
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
   <td>L: term length, in epochs, of delegates.When an epoch finishes, 1/L (L > 3) of the delegates are retired and replaced with newly elected delegates. Note however that delegates can run for reelection and remain in office, which means no delegates are actually replaced. Since L > 3, we have to replace less than 1/3 of delegates, given our current default configuration with 32 delegates, we can replace 8 of them. I.e, we set L = 4. Assume L evenly divides N.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>epoch_i: the 12 hour time period, or shorter in the event of recall, in which the delegate set is fixed.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Epoch_block_i: the epoch block created at the end of epoch_i, to be post-committed through delegate consensus, and propagated via p2p in epoch_{i+1}. Described further in the epochblock requirements document
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>candidacy request: a block that serves to announce the delegate candidacy of a representative for upcoming elections. Signed by the representative announcing candidacy
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>vote request: a block signed by a representative issuing their votes for the current epoch
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>candidates_epoch_i: the set of candidates listed in the database for the current_epoch. This is the list of candidates that representatives can vote for during epoch_i
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>delegate-elects_epoch_i: the elected N/L candidates that received the most weighted votes during epoch_{i-1}. Note, delegate-elects_epoch_i is a superset of delegates-new_epoch_{i+1}. Also, delegate-elects_epoch_i is a subset of candidates_epoch_{i-1}
   </td>
   <td>these are nodes that were candidates in the previous epoch and received the most votes. They are going to be full delegates next epoch. Note, the members of this set could be current delegates that were reelected
   </td>
  </tr>
  <tr>
   <td>Delegates-new_epoch_i: the set of delegates of epoch_i - delegates_epoch_{i-1}
   </td>
   <td>Note, this does not include delegates that remained in office because of reelection
   </td>
  </tr>
  <tr>
   <td>Delegates-persistent_epoch_i: a delegate in the set of delegates_epoch_i - delegates-new_epoch_i.
   </td>
   <td>Note, this includes delegates that were reelected
   </td>
  </tr>
  <tr>
   <td>Delegates-retired_epoch_i: the set of delegates of epoch_{i-1} - delegates_epoch_i. Note, the number of delegates retired == the number of new delegates
   </td>
   <td>Note, this is the set of delegates that actually do get retired. if you are reelected in time to not get retired, you never join this set
   </td>
  </tr>
  <tr>
   <td>Delegates_epoch_i: delegates of epoch_i
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>renounce_candidacy request: a block issued in epoch_i that attempts to remove a candidate from candidates_epoch_{i+1}
   </td>
   <td>Note the 1 epoch lag. Revoking candidacy takes effect in the next epoch. Note, this will have no effect if you end up being elected during epoch_i.
   </td>
  </tr>
  <tr>
   <td>minimum_delegate_stake: the minimum stake required to be a delegate. currently 0.01% of total supply
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>minimum_representative_stake: the minimum stake required to be a representative. currently 0.001% of total supply
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>candidacy_action: announce_candidacy request or renounce_candidacy request
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>candidacy_action_tip: the latest post-commited candidacy_action of an account. could be nothing if account has never sent a candidacy_action
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>representative_action: announcing yourself as a representative or renouncing yourself as a representative
   </td>
   <td>Note this does not include change representative. This only includes commands that affect your own status as a representative
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Candidacy</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for a representative to announce candidacy to the network via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for a representative to renounce candidacy in the network via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall write to the database when a candidacy action is post-committed via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall add all representatives for which an announce_candidacy request was post-committed in epoch_i to candidates_epoch_{i+1}
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall add all members of candidates_epoch_i to candidates_epoch_{i+1} that satisfy the following requirement: are not members of delegate-elects_epoch_{i+1} and candidacy_action_tip is not a renounce_candidacy request
   </td>
   <td>This says you remain a candidate if you were not elected and did not renounce your candidacy.
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any candidacy_action during epoch_i for a representative for which a candidacy_action was already post-committed during epoch_i
   </td>
   <td>A node can only submit one of announce_candidacy request or renounce_candidacy request during a single epoch. One cannot submit an announce_candidacy request followed by a renounce_candidacy request, or vice versa, in a single epoch. One cannot submit two announce_candidacy request's or two renounce_candidacy request's
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any renounce_candidacy request whose candidacy_action_tip is not an announce_candidacy request
   </td>
   <td>You can only renounce candidacy if you are a current candidate, or will be auto added as a candidate in a future epoch transition
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any announce_candidacy request for a representative whose candidacy_action_tip is an announce_candidacy request
   </td>
   <td>You can't announce candidacy if you are already a candidate or will be auto added as a candidate in a future epoch transition.
   </td>
  </tr>
  <tr>
   <td>The core software shall add all members of delegates_epoch_i to candidates_epoch_{i+1} whose current term ends after epoch_{i+2} and whose candidacy_action_tip is not a renounce_candidacy request
   </td>
   <td>this ensures continuity. if a delegate is set to retire, they are added to the candidate list at the proper time such that they will remain a delegate
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any candidacy_action for a representative that has not staked at least the minimum_delegate_stake
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any candidacy_action from an account that is not also a representative
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept candidacy_action that conform to the IDD
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
   <td><strong>Representative</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for an account to become a representative in the network via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for an account to change their representative in the network via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for an account to renounce being a representative via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any representative_action during epoch_i from a representative for which a representative_action was already post-committed during epoch_i
   </td>
   <td>Can only do one of announce or renounce per epoch. Can change representative multiple times
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for a representative to delegate their entire voting power to a different account. This other account will issue votes on behalf of the representative
   </td>
   <td>This allows a representative who would normally issue their own votes to issue their votes from a different account. The votes from that different account will be treated as if they came from the representatives account. This is done to minimize exposure of the private key that is guarding the staked funds.
   </td>
  </tr>
  <tr>
   <td>The core software shall designate a representative of a new account to be the representative of the account sending that new account the initial funds
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject a command to become a representative for an account that has not staked the minimum_representative_stake
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept a command to become a representative that conforms to the IDD
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
   <td><strong>Election</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a command for a representative announce their delegate candidate votes to the network via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall write to the database when a vote request is post-committed via delegate consensus
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject vote requests during epoch_i that contain votes for representatives not listed in candidates_epoch_i
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject a vote request that does not contain exactly N/L votes
   </td>
   <td>Reps have to use all of their votes
   </td>
  </tr>
  <tr>
   <td>The core software shall reject vote requests that contain votes for more than N/L different candidates
   </td>
   <td>Cannot split votes
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any vote request during epoch_i from a representative for which a vote request has already been post-committed during epoch_i
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall weight each vote according to the formula described in the design document
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall weight each vote that occurred during epoch_i based on representative_actions and change representative requests that happened prior to epoch_i
   </td>
   <td>This says that any change representative request during epoch_i does not affect the weighting of votes issued during epoch_i. The stake for each representative is constant for each epoch, and only changes on epoch_transition. This also says announcing to be a representative during epoch_i does not allow you to vote until epoch_{i+1}, and renounce in epoch_i does not take effect until epoch_{i+1}. Meaning if I renounce my representative status in epoch_i, I can still vote in epoch_i
   </td>
  </tr>
  <tr>
   <td>The core software shall determine delegate voting power, to be used during delegate consensus, according to the formula described in the design document
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall populate the delegates field of epoch_block_i as follows: the N/L candidates who received the most weighted votes during epoch_i (this is delegate-elects_epoch_{i+1}), along with all members of delegates_epoch_{i+1} that are not members of delegate-elects_epoch_{i-3}
   </td>
   <td>Anyone who is a member of delegate-elects_epoch_{i-3} is serving their last term during epoch_{i+1}. The delegates listed in epoch_block_i are the delegates for epoch_{i+2}
   </td>
  </tr>
  <tr>
   <td>As a backup delegate, upon receiving a pre-prepare for epoch_block_i, the core software shall validate the N/L candidates who received the most weighted votes during epoch_i
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall break vote ties while calculating delegate-elects_epoch_i by most stake, followed by most epochs spent as delegate, followed lastly by greatest value of delegate address
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall add a representative to delegate-elects_epoch_i even if candidacy_action_tip is a renounce_candidacy request
   </td>
   <td>Note, renounce_candidacy only works if you were not elected.
   </td>
  </tr>
  <tr>
   <td>The core software shall reject any vote request from an account that is not also a representative
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept vote requests that conform to the IDD
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
   <td>Note: Yellow requirements have been moved to epoch requirements
   </td>
   <td>
   </td>
  </tr>
</table>



