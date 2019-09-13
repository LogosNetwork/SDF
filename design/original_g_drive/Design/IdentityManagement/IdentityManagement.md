# Identity Management

## Overview

Identity management is a subsystem that provides the overall identification capability for a node's identity. This includes:
- Delegate's private IP/port advertisement
- Tx-Acceptor public IP/port advertisement
- Handshake between Tx-Acceptor and the delegate
- Handshake between the delegates
- P2p commands to request delegates IP/port
- Private key management

## Use cases

![](/home/gregt/Documents/Design/IdentityManagement/identity-use.png)

Figure 1. Identity Management use case diagram.

## Execution concept

Identity manager is driven by the following events:
* Node start up to advertise delegate and tx-acceptor address
* One hour before the epoch start to advertise delegate and tx-acceptor address
* Thirty minutes before the epoch start to advertise delegate and tx-acceptor address
* Every ten minutes starting at one hour before the epoch start to request delegate's addresses
* RPC call to add or delete tx-acceptor
* Unsolicited P2p delegate's address advertisement message

Details of these events are covered in the sections below.

## Delegate and Tx-Acceptor IP/port advertisement

### Class Diagram

Identity management extends existing and implements new classes:
- AddressAd is the delegate's address advertisement message. The message has ip and port encrypted with ECIES 
public key of every delegate except for the sender. The hash of the message is signed with the sender's private BLS key.
- AddressAdTxAcceptor is the delegate's tx acceptor advertisement message. The hash of the message is signed with the sender's private BLS key.
- P2pConsensusHeader precedes P2p consensus message. The header identifies receiver of the consensus message. 
- P2pHeader is the main p2p message header. If app_type is Consensus then it is followed by P2pConsensusHeader, which in turn 
is followed by  corresponding consensus message. 
If app_type is AddressAd then it is followed by AddressAd. If app_type is AddressAdTxAcceptor then it is followed by AddressAddTxAcceptor.
- DelegateIdentityManager handles all identity manager functionality related to advertisement and persistence of the delegate and tx acceptor addresses.

The class diagram is shown on Figure 2.

![](/home/gregt/Documents/Design/IdentityManagement/identity-cls.png) 

Figure 2. Class diagram.

### Delegate Address Encryption

Elliptic curve integrated encryption scheme (ECIES) is used for the delegate's address encryption. ECIES private key is retrieved 
by the delegatate via the private key management capability. ECIES public keys are distributed via the epoch block along with
the BLS public keys. During AddressAd message serialization, IP and port members of the message are encrypted 
with the ECIES public key of every other delegate depending on client/server relationship in the delegate's connection schema.
This reduces by half the number of AdMessage required to be propagated via P2p subsystem.
In the TCP/IP connection process, one delegate is a client (delegate with lower delegate id) and another delegate is a server. For instance,
when delegate 1 and delegate 2 establish connection with each other, delegate 1 is the client and delegate 2 is the server.
Since delegate 2 doesn't need delegate's 1 IP for connection, delegate 1 doesn't have to advertise it's ip/port, which could
be sent instead to delegate 2 as part of the handshake after delegate 1 connects to delegate 2.

### Sequence Diagram Delegate and Tx Acceptor Address Advertisement

On start up the node checks if it is the delegate in the current or next epoch (IdentifyDelegates()). If it is the delegate 
then it advertises via P2p subsystem its encrypted address and all of its tx acceptors' address (Advertise(), AdvertiseTxAcceptor()). 
If the node is the delegate in the next epoch then depending on the current time the node schedules two timers at 1 hour and at 30 min
before the start of the next epoch (ScheduleAdvert()). On timer's timeout (Advert()), the node 
advertises via P2p subsystem its encrypted address and all of its tx acceptors' address.
On EpochTransitionEnd event, if the node is not the delegate in the current or next epoch then it repeats the advertisement sequence (AdvertCheck). 
When a node receives tx acceptor address via RPC call (OnTxAcceptorAdded), it saves the address to the database (UpdateTxAcceptorAddressDB()). 
The address is also stored by DelegateIdentityManager (tx_acceptor_addresses).
If the node is the delegate in the current or next epoch then it advertises tx acceptor address. When a node receives tx acceptor address 
deletion RPC call then the node advertises tx acceptor deletion via P2p subsystem. The node deletes tx acceptor address from the database. 
Upon receiving a recall event, if the node is the delegate in the next epoch, it advertises via P2p subsystem its encrypted 
address and all of its tx acceptors' address. 
Sequence diagram for the address advertisement is shown on Figure 3.

![](/home/gregt/Documents/Design/IdentityManagement/identity-adv-seq.png) 

Figure 3. Sequence diagram, address advertisement

### Sequence Diagram Delegate Address and Tx Acceptor Address Receiving

On start up the node ascertains if it is the delegate in the current or next epoch (IdentifyDelegates()).
If it is the delegate and it doesn't have addresses of all other delegates 
then the delegate requests missing addresses from its peers via P2p subsystem (RequestDelegateAddresses()) and 
schedules 10 min timer (SchedulePeerAddrRequest()). When the delegate receives the response back from its peers (OnPeerAddrRequest()), 
it updates delegates address (UpdateDelegateAddressDB()) and connects to the received peers address (ConnectToPeer()) if 
it is the delegate in the current epoch. Connected peer sends it's AdAddress message as part of the handshake and
the delegate updates the delegates address. On timeout (PeerAddrRequest()) the delegate repeats the sequence (CheckDelegateAddress). 
When a node receives a delegate's or tx acceptor address advertisement (OnAddressAd(), OnAddressAdTxAcceptor()), 
and if the node is the delegate in the current or the epoch then it updates the address database. 
The address is also stored by DelegateIdentityManager (addresses, tx_acceptor_addresses).If the node is the delegate 
in the current epoch then it connects to the received delegate address (ConnectToPeer()). The node also deletes older epoch entries for the given delegate id.
Sequence diagram for the address receiving is shown on Figure 4.

![](/home/gregt/Documents/Design/IdentityManagement/identity-adv-recv-seq.png) 

Figure 4.Sequence diagram, address receiving.

## Handshake between the delegates

High level sequence diagram on Figure 5 shows AdAddress message propagation via P2p subsystem and
handshake that takes place when delegates connect to each other.

![](/home/gregt/Documents/Design/IdentityManagement/identity-distr.png)

Figure 5. Sequence diagram, AdMessage propagation and delegate's handshake.

## Handshake between Tx-Acceptor and the delegate

High level sequence diagram on Figure 6 shows handshake interaction between TxAcceptor and Delegate. Once the handshake
completes, TxAcceptor forwards to TxAcceptor transactions received from the client.

![](/home/gregt/Documents/Design/IdentityManagement/identity-tx-distr.png)

Figure 6. Sequence diagram, TxAcceptor and delegate handshake.


## P2p commands to request delegates IP/Port (TBD)

## Private key management (TBD)

