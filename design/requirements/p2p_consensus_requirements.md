#P2P Backup Consensus Requirements


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
   <td>P2p backup consensus is used by the primary delegate when direct consensus to the backup delegates times out. In this case the request is propagated to the backup delegates via P2p subsystem. The backup delegate responds to the request via both P2p subsystem and direct communication channel. The primary delegate keeps on using both communication channels for configured period of time to send consensus messages. P2p backup consensus protocol is compliant and follows all specified consensus requirements.
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
   <td><strong>Primary delegate</strong>
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
   <td>As a primary delegate, the core software shall re-propose the batch state block via both P2p subsystem and direct connection if the recall timer set during direct connection consensus attempt is timed out.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate, the core software shall set 60 seconds P2p timer.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Once P2p consensus protocol is activated, as the primary delegate the core software shall continue using both P2p subsystem and direct connection for consensus messages.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate the core software shall count messages received via P2p subsystem and direct connection.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a primary delegate the core software shall discontinue using P2p subsystem after P2p timer times out if the number of weighted direct connections to the delegates reaches the quorum.
   </td>
   <td>What is the ratio?
   </td>
  </tr>
  <tr>
   <td>If the number of weighted direct connections doesn't reach the quorum then the core software shall set the 60 seconds P2p timer and continue using P2p subsystem and direct connection for consensus messages.
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
   <td><strong>Backup delegate</strong>
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
   <td>Once a backup delegate receives a consensus message from the primary delegate, as the backup delegate the core software shall start using both P2p subsystem and direct connection for consensus messages.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate the core software shall set 60 seconds P2p timer.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate the core software shall count messages received via P2p subsystem.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>As a backup delegate the cores software shall discontinue using P2p subsystem after P2p timer times out if there is no consensus message received via P2p subsystem.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>If the the number of received P2p messages is 0 then the core software shall discontinue using P2p subsystem.
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>If the the number of received P2p messages is not 0 then the core software shall set the 60 seconds P2p timer and continue using P2p subsystem and direct connection for consensus messages.
   </td>
   <td>
   </td>
  </tr>
</table>

