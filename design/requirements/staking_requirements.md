<h1>Staking Requirements</h1>


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
   <td>Requirements for staking. The staking subsystem defines how accounts can lock up a portion of their funds, making those funds unspendable, in exchange for a portion of inflation rewards, as well as an increase in elections voting power (if the account is a representative) and an increase in consensus weight (if the account is a delegate). Representatives and delegates each have minimum staking requirements.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Staking comes in 3 forms. Accounts may stake to themselves; this form of staking will mainly be used by delegates and representatives. Delegates and representatives each must have a minimum amount of self stake to be a delegate or representative. Self staking also increases a representatives voting power and a delegates weight within consensus. The second form of staking is locked proxy; this form of staking will mainly be used by non-representative/non-delegate accounts, who will stake funds to a representative, via the proxy command. These funds are locked (unspendable) and increase the reps voting power by the amount staked. The third form of staking is unlocked proxy; in this form of staking, the available account balance of non-representative/non-delegate accounts (available account balance does not include locked proxy funds or staked funds) are counted towards the representatives voting power, multiplied by a dilution factor. The dilution factor is less than 1, so unlocked proxied funds count less towards a rep's voting weight than the same amount of staked/locked proxied funds. These funds are not actually staked: accounts are free to spend these unlocked proxied funds, and the activity will affect the representative's voting weight. See the definitions for more detail.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Staking is handled by 3 different requests: Stake, Unstake and Proxy. Stake and Unstake are used by delegates and representatives to adjust their self stake. Proxy allows a non-representative account to choose a representative, and simultaneously allows an account to increase or decrease the amount of locked proxied stake.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>One thing to note: reps and delegates must always stake to themselves (cannot proxy) and the delegate stake, used within consensus to approve or reject request blocks, does not include proxied stake. See the definitions for more details
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
   <td>staked: (state) funds that are staked are unspendable and are used to calculate elections voting power and delegate_stake. Funds are staked <strong><em>to</em></strong> an account.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>thawing: (state) funds that are thawing are unspendable but are not used to calculate elections voting power and delegate_stake
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>available: (state) funds that are available are free to be spent by the holding account
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>frozen: (state) funds that are frozen are unspendable but are not thawing and are not used to calculate elections voting power and delegate_stake. When a delegate submits an Unstake request, the funds are frozen until the delegate has finished their current term, after which those funds begin thawing
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>thawing_period: length of time that funds are in the thawing state. Parameter of the system that defaults to 3 weeks.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>target_account: property of staked, thawing or frozen funds. the target_account of staked funds is the account whose election voting power has increased due to the staked funds. the target_account of frozen or thawing funds is the account those funds were staked to immediately prior to entering the thawing or frozen state.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>source_account: property of staked, thawing or frozen funds. the source_account of staked, thawing or frozen funds is the account that issued the request to move those funds to the staked, thawing or frozen state, and the account that will be able to spend those funds 1 thawing period after those funds move to the thawing state
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>origin: the account that signed a request. Typically the sending account
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Stake: (request) move funds to a staked state from a thawing, frozen or available state. target_account is the origin of the request. Contains a field specifying amount to stake
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Unstake: (request) move staked funds, with a target_account equal to origin, to a thawing state, with a possible intermediate state of frozen. Contains a field specifying the amount to unstake
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Proxy: (request) select an account to be a representative for the origin account. Optionally specify an amount of funds to lock proxy to the newly selected representative, where the newly selected representative will be the target_account of the staked funds.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>liable: (property of staked, thawing or frozen funds) funds that are liable are liable for a specific account, A, (or multiple accounts) because those funds are or were previously staked to A. If I locked proxy to A, the locked proxy funds are liable for A. If I locked proxy to A, then reproxy those same funds to B, those funds are liable for A and B. If I locked proxy to A, then adjust my locked proxied amount to 0, those thawing funds are liable for A.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>liability_period: length of time that funds are liable for a specific account after funds are no longer staked to that account. Equal to the thawing_period
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>self_stake_amount: per each account. the amount of staked funds for which this account is the target_account and the source_account
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>locked proxy: a type of staked funds where the target_account is different than the source_account. Also, to stake funds such that those funds will be locked proxied funds
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>representative: an account, A, chosen by another account, B, where A is voting during elections on B's behalf.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>locked_proxy_amount: per each representative. the amount of staked funds for which this account is the target_account but not the source_account
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>unlocked_proxy_amount: per each representative R. the sum of all account balances (available funds) for each account that has chosen R as a representative via Proxy request
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>dilution_factor: 0.25. When accounts choose a representative, that representative's voting power is increased by (accounts available balance * dilution_factor)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>total_stake_amount: self_stake_amount + locked_proxy_amount + unlocked_proxy_amount * dilution_factor
   </td>
   <td>Note rep_stake_amount is already multiplied by dilution_factor
   </td>
  </tr>
  <tr>
   <td>minimum_rep_stake: the minimum self_stake_amount a representative must have to be eligible to vote
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>minimum_delegate_stake: the minimum self_stake_amount a delegate must have to be eligible for candidacy
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>delegate_stake_raw: the self_stake_amount of a candidate during the epoch that they were elected.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>delegate_stake: an amount used within consensus to approve or reject request blocks. A request block must be approved by a minimum amount of total stake, where each delegate that approves the request block adds their own delegate_stake to the total stake. Delegate_stake is delegate_stake_raw after redistribution
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>candidacy_action_tip: the latest post-committed request of an account that is a candidacy request (AnnounceCandidacy, RenounceCandidacy or StopRepresenting). See elections requirements and design
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>representative_action_tip: the latest post-committed request of an account that is a representative request (StartRepresenting, AnnounceCandidacy or StopRepresenting). See elections requirements and design
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>stake_subfield: field of StartRepresenting or AnnounceCandidacy, which specifies how much to stake as a representative or delegate. Is optional
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>unstake_subfield: field of RenounceCandidacy or StopRepresenting, which specifies how much to unstake upon post-commit. Is optional
   </td>
   <td>stake_subfield and unstake_subfield are both optional as an account may already have enough staked to become a candidate or representative, and an account may also not wish to unstake anything when RenouncingCandidacy
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Requests</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a request (Proxy) for an account to select a representative
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a request (Stake) for an account to stake funds to themselves or increase the amount of funds staked to themselves
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a request (Unstake) for an account to move a specified amount of funds staked to themselves to the thawing state (or frozen state)
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a request (Proxy) for an account to lock proxy to a representative or adjust the amount of funds locked proxied to a representative
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept Stake requests that conform to the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept Unstake requests that conform to the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall accept Proxy requests that conform to the IDD.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall ensure rep_stake_amount, locked_proxy_amount and unlocked_proxy_amount reflect all and only requests that were post-committed prior to the current epoch.
   </td>
   <td>Stake, Unstake and Proxy don't take effect until the next epoch. Requests that affect unlocked_proxy_amount don't alter unlocked_proxy_amount until start of next epoch.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Thawing</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall move funds from the thawing state to the available state 1 thawing period after the end of the epoch during which the request (Proxy or Unstake) that moved those funds to the thawing state was post-committed
   </td>
   <td>Thawing period starts when the epoch ends.
   </td>
  </tr>
  <tr>
   <td>The core software shall allow for x different funds to be thawing concurrently per account, where x is a system parameter.
   </td>
   <td>X can be set to 32 in initial implementation.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Liability</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing a Stake request, make the staked funds liable for the origin of the request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing an Unstake request, make the thawing of frozen funds liable for the origin of the request for 1 liability period after the end of the epoch during which the Unstake request was post-committed
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing a Proxy request that specifies an amount of funds to lock proxy, make the locked proxied funds liable for the representative specified in the request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon moving locked proxied funds to thawing state (via Proxy or Stake request), make those funds liable for the target_account of the funds for 1 liability period after the end of the epoch during which the Proxy or Stake request was post-committed
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon changing the target_account of locked proxied funds (via Proxy or Stake request), make those funds liable to the original target_account for 1 liability period after the end of the epoch during which the Proxy or Stake request was post-committed
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall allow funds to be liable for at most 2 accounts simultaneously.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall allow for x liabilities to exist concurrently per account, where x is a system parameter.
   </td>
   <td>X can default to 32
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Selection of Funds to Stake</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing a Stake request or a Proxy request, first draw the specified amount of funds to stake, or lock proxy, from any locked proxied funds that are liable for 1 account, then from any thawing funds liable for 1 account which is not the origin of the request (ordered by furthest from finishing thawing) then from available funds. Any remaining staked funds (including locked proxied funds) are moved to the thawing state.
   </td>
   <td>Note, Stake and Proxy do not draw from any funds staked to self, or any thawing funds previously staked to self. Note, this requirement ensures that an account only ever has one active proxy, and that staking to self eliminates any proxies.
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Proxy</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing a Proxy request that does not specify an amount of funds to lock proxy, move any previously locked proxied funds to the thawing state.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing a Proxy request that does specify an amount of funds to lock proxy, locked proxy the specified amount of funds to the new representative, selecting the funds as described in "Selection of Funds to Stake" section.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject Proxy requests where the representative_action_tip of the origin is StartRepresenting or AnnounceCandidacy
   </td>
   <td>Can't select a representative if you are a representative
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td><strong>Representatives and Delegates</strong>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall weight election votes by a representative's total_stake_amount
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject Unstake requests that would update the self_stake_amount to less than the minimum_delegate_stake from an account whose candidacy_action_tip is AnnounceCandidacy
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject Unstake requests that would update the self_stake_amount to less than the minimum_rep_stake from accounts whose representative_action_tip is StartRepresenting or AnnounceCandidacy
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall calculate delegate_stake based on delegate_stake_raw according to the formula described in the design document.
   </td>
   <td>delegate_stake will be capped to eliminate any one delegate controlling a majority of stake within consensus, creating a single point of failure. Also, delegate_stake is only dependent on delegate_stake_raw, which is the self_stake_amount at election time
   </td>
  </tr>
  <tr>
   <td>The core software shall reject StartRepresenting requests where the origin's self_stake_amount is less than the minimum_rep_stake and the stake subfield is empty
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject StartRepresenting requests where the stake_subfield is not empty and origin's self_stake_amount would be less than the minimum_rep_stake after staking to self the amount specified in stake_subfield
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject AnnounceCandidacy requests where the origin's self_stake_amount is less than the minimum_delegate_stake and the stake subfield is empty
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject AnnounceCandidacy requests where the stake_subfield is not empty and origin's self_stake_amount would be less than the minimum_delegate_stake after staking to self the amount specified in stake_subfield
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing AnnounceCandidacy or StartRepresenting, if the stake_subfield is not empty, move the specified amount of funds to the staked state with a target_account equal to origin
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall reject RenounceCandidacy requests where the unstake_subfield is not empty and the origin's self_stake_amount would be less than the minimum_rep_stake after unstaking the amount specified in the unstake_subfield
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing StopRepresenting or RenounceCandidacy, unstake the amount of funds specified in the unstake_subfield, if unstake_subfield is not empty
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall, upon post-committing StartRepresenting or AnnounceCandidacy, change the representative of the origin account to null
   </td>
   <td>Can't have a representative if you are a representative
   </td>
  </tr>
  <tr>
   <td>The core software shall ensure, if a delegate or delegate-elect submits an Unstake request, or a StopRepresenting/RenounceCandidacy request with an unstake_subfield, the staked funds will move to the frozen state in the next epoch, and then to the thawing state at the end of the delegate's current term
   </td>
   <td>Thawing period doesn't begin until delegates finish their term
   </td>
  </tr>
  <tr>
   <td>The core software shall ensure, if a candidate submits an unstake command, or StopRepresenting/RenounceCandidacy request with an unstake subfield, and the candidate is elected, the staked funds will move to the frozen state in the next epoch, and then to the thawing state at the end of the candidates delegate term
   </td>
   <td>
   </td>
  </tr>
</table>


