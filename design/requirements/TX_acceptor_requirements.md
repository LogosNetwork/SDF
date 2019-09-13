#TX_Acceptor Requirements

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
   <td>Precondition, when the core software is initialized to be a standalone TX acceptor, the core software shall provide TCP/IP server access point for a valid delegate to connect.
   </td>
   <td>via configuration file
   </td>
  </tr>
  <tr>
   <td>Precondition, when the core software is initialized to be a standalone TX acceptor, the core software shall provide a handshake to establish the validity of the connected delegate.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Precondition, when the core software is initialized to be a TX acceptor and P2P subsystem then both of these subsystems have independent communication channel.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>A Delegate can connect to multiple TX Acceptors.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a public interface to receive from a client via TCP/IP transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The transaction request shall be an array of single state blocks in json format or binary format. The latter is described per IDD format.
   </td>
   <td>The argument in favour of json is that either binary or json have to be deserialized. Since the performance is affected either way, from client's perspective json is more user friendly.
   </td>
  </tr>
  <tr>
   <td>The core software shall validate that the transaction is not created by the burned account.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall perform validity check on transaction fee amount (lower than the minimum transaction fee defined in the epoch block).
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core Software shall perform validity check on the signature before processing the transaction request.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall send a rejection message with the proper error code if a transaction request cannot be validated.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall forward validated transaction(s) to the connected delegate.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>The core software shall send a heartbeat message to the connected Delegate after inactivity (no transactions received) of 40 seconds.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>A Delegate shall reconnect to TX Acceptor after inactivity of 60 seconds.
   </td>
   <td>
   </td>
  </tr>
</table>
