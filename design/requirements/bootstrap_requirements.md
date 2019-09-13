<h1>Bootstrap Requirements</h1>


<table>
  <tr>
   <td>
<h1>Requirements</h1>


   </td>
  </tr>
  <tr>
   <td>
<h3>Networking</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall manage connections in the bootstrap code.
   </td>
  </tr>
  <tr>
   <td>The core software shall have two types of connections, client and server.
   </td>
  </tr>
  <tr>
   <td>The core software shall make these connections available to the appropriate client/server to satisfy requests, (tips, pull).
   </td>
  </tr>
  <tr>
   <td>The core software shall allow for connections to be pooled so as to be re-used.
   </td>
  </tr>
  <tr>
   <td>The core software shall close sockets in the client/server destructors, such that after the connection is no longer needed, we do not leak the resource.
   </td>
  </tr>
  <tr>
   <td>The core software shall create client connections based on outstanding pull requests, either allocating a new connection for a pull, or reusing from existing connections.
   </td>
  </tr>
  <tr>
   <td>The core software shall track the blocks received and compute block rate.
   </td>
  </tr>
  <tr>
   <td>The core software shall disable connections that are not performing.
   </td>
  </tr>
  <tr>
   <td>The core software shall set the socket recv/send buffer size as appropriate.
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
  <tr>
   <td>
<h3>Initialization</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall request tips from its peer.
   </td>
  </tr>
  <tr>
   <td>The core software shall have tips request/response in a loop to periodically check if the process is in sync.
   </td>
  </tr>
  <tr>
   <td>The core software shall not issue new pull requests provided the system is still processing requests.
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
  <tr>
   <td>
<h3>Pulls</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall decide to pull if its behind its peer for a given delegate.
   </td>
  </tr>
  <tr>
   <td>The decision is made due to a comparison of either timestamps or sequence numbers (preferred).
   </td>
  </tr>
  <tr>
   <td>The core software shall request epoch and micro blocks first, followed by batch state blocks.
   </td>
  </tr>
  <tr>
   <td>The core software shall allow for downloads in parallel using threads.
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
  <tr>
   <td>
<h3>Thread Pool</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall after making a decision to pull, make a request using a structure that is passed to a thread pool for processing.
   </td>
  </tr>
  <tr>
   <td>The core software shall use a structure that has a start and an end hashes for epoch, micro, and batch blocks.
   </td>
  </tr>
  <tr>
   <td>The core software shall implement a generic request for a pull which can pull either of the three supported blocks in any combination.
   </td>
  </tr>
  <tr>
   <td>The core software shall have the thread pool process a request at a time off the queue.
   </td>
  </tr>
  <tr>
   <td>The core software shall process the request by making a client request to the peer's server asking for the desired blocks.
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
  <tr>
   <td>
<h3>Traversing blocks</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall have it's peer server send each requested block sequentially iterating from start to end using a next pointer.
   </td>
  </tr>
  <tr>
   <td>The core software shall handle network errors in this transmission by closing the socket and logging the error.
   </td>
  </tr>
  <tr>
   <td>The core software shall re-issue a request for the missing blocks again using the tips service.
   </td>
  </tr>
  <tr>
   <td>The core software shall continue in a loop periodically checking its peer to see if it is behind.
   </td>
  </tr>
  <tr>
   <td>The core software shall run in an on-going bootstrap loop which is initiated from the top-level.
   </td>
  </tr>
  <tr>
   <td>The core software shall not do pulls if the system is in sync with its peers.
   </td>
  </tr>
  <tr>
   <td>The core software shall wait in the case pulls are on-going.
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
  <tr>
   <td>
<h3>API</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall provide an api to initiate the process.
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a method to initiate the process with no peer specified.
   </td>
  </tr>
  <tr>
   <td>The core software shall provide a method to initiate the process with a specific peer specified.
   </td>
  </tr>
  <tr>
   <td>
   </td>
  </tr>
  <tr>
   <td>
<h3>Validation</h3>


   </td>
  </tr>
  <tr>
   <td>The core software shall transfer blocks to the client.
   </td>
  </tr>
  <tr>
   <td>The core software shall queue arriving blocks.
   </td>
  </tr>
  <tr>
   <td>The core software shall sort the arriving blocks when the queue is filled with n items.
   </td>
  </tr>
  <tr>
   <td>The core software shall then validate and apply to the database the blocks.
   </td>
  </tr>
  <tr>
   <td>The core software shall then remove the applied blocks from the queue, leaving those which could not be validated until more blocks arrive from the network.
   </td>
  </tr>
  <tr>
   <td>The core software shall implement three queues, one for each block type (epoch, micro, batch state block).
   </td>
  </tr>
  <tr>
   <td>The core software shall use these queues to perform validation/apply.
   </td>
  </tr>
  <tr>
   <td>The core software shall use the validation/apply methods of each respective block type.
   </td>
  </tr>
  <tr>
   <td>The core software shall reach n batch state blocks, and then try to validate, if none can be validated, it will wait for the next arrivals.
   </td>
  </tr>
  <tr>
   <td>The core software queues can increase when waiting for more arrivals.
   </td>
  </tr>
  <tr>
   <td>The core software queues can decrease after blocks are validated/applied.
   </td>
  </tr>
  <tr>
   <td>The core software shall check to see if we reached a micro block tip, then validate and apply the micro block.
   </td>
  </tr>
  <tr>
   <td>The core software shall keep track of each delegate's tips in the database.
   </td>
  </tr>
  <tr>
   <td>The core software shall check the tips, either from what was received in memory, or from the database, to see if we reach the next micro block.
   </td>
  </tr>
  <tr>
   <td>The core software shall then validate and apply the micro block.
   </td>
  </tr>
  <tr>
   <td>The core software shall reach an epoch block tip, it will validate and apply the epoch block.
   </td>
  </tr>
  <tr>
   <td>The core software shall keep track of the micro block tip.
   </td>
  </tr>
  <tr>
   <td>The core software shall check the tip is reached that matches the next epoch block, and validate/apply the epoch block.
   </td>
  </tr>
</table>
