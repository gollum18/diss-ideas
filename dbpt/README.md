# dbpt - The Distributed B+ Tree Data Structure

## Background

### B/B+ Trees
A B+ tree is a data structure created for fast indexing on large database 
corpora. In the database realm, B+ trees were created in response to the 
increasing demand for fast disk access within the age of rapidly expanding
data storage needs. Based on B trees, B+ trees extend the classical B tree
algorithm with much needed functionality for disk indexing purposes.


B+ trees maintain their keys in sorted order. Each inner node in the tree
is treated as an index node that points to index nodes in the next plie
down the tree. Leaf nodes either contain the data associated with the 
tree or they contain a record of how to access that data. Note that in
B+ trees, there are always K+1 pointers in each node where the K+1th pointer
points to the next sibling node (one of the improvements on B trees).


That said, B+ trees are usually constrained by two factors:
1. The number of keys: K
2. The minimum and maximum fill factors: N, M respectively


N and M are simultaneously employed to enable the tree to dynamically expand
and contract as data items are added and removed from the tree. Specifically,
when the number of elements in a node is less than K * N, the node is collapsed
and its remaining data items are absorbed by its parent (and consequently 
redistributed if necessary). Likewise, then the number of elements in a node
exceeds K * M, the node overflows, and new child nodes are created as necessary
to hold the to-be-redistributed data items.


This dynanicism makes B+ trees perfect for a highly dynamic envrionment such
as a database management system or other file indexing mechanisms, but what 
about in a distributed environment? How would B+ trees fair then?


At a minimum a B+ tree must support three operations:
1. Searching
2. Insertion
3. Deletion


Due to the ordered and linked nature of the tree, (1) is simple to achieve.
To search the tree, the user queries the root. The root then passes the request
down to the correct child who in turn passes the request to the correct child until
the data item is or is not found. Note that because each node at the data level is
linked, one can just pass the search request down the left side of the tree, and 
traverse the last level for the data item in question.


Insertion and deletion are primary concerns in a B+ tree. Any algorithm to perform
either operation must be flexible enough to handle such operations under varying
operating conditions.


### Distributed Systems and P2P Networks
A distributed system is any such collection of mutually independent, networked 
machines that cooperate to solve a common goal or set of goals. Such systems are
necessary to solve complex problems such as approximating PI, finding prime numbers, 
or more recently, accelerating Artificial Intelligence applciations and Machine
Learning.


There are many types of distributed systems but the most prominent today are
what known as Peer-to-Peer (P2P) systems. Common examples of P2P systems include
infamous protocols such as the Bittorrent protocol and the Bitcoin protocol. Such
systems suffer from many types of logistical issues including scaling, indexing,
membership management, etc...


Now, P2P networks may be further divided into two types:
1. Structured networks
2. Unstructured networks


In Structured P2P networks, there is a strong association between each peer in
the network. All peers know how to reach other, either directly, or indirectly 
through another common peer. The Bittorent protocol is an excellent example
of a Structured P2P network. In Bittorent, peers communicate directly through
the use of a tracker peer. This tracker manages major logistics for a given 
file in the Bittorent network. When a peer wishes to download a file through
Bittorent, they first visit the tracker who assigns them a list of peers that
currently have access to all or part of the file. The new peer then uses their
list (which may and often is different than other peers associated with the
same file), to request parts of the file.


In Unstructured P2P networks, most if not all nodes do not know have any direct
lines of communication between them. For a node to join the Bitcoin protocol
it must know the address of a peer already in the system (this address is 
often obtained automatically and opaquely via a Bitcoin client). In Unstructured
networks like Bitcoin the only way for nodes to communicate is via message 
flooding. In simpler, more naive Unstructured P2P networks, this leads to 
massive scalability and bandwidth problems as the number of messages exchanged
in the most naive systems grows exponentially with the number of peers in 
the system. Consequently, much research in this area has been concentrated on
creating efficient gossiping protocols to reduce the aforementioned effects of
message flooding.


### Superpeers
It should be clear now that the major drawback to any distributed system is
scalability. Associated with it are questions such as: Will my messages be
on time? How do I ensure my messages are not tampered with? How do I know the
network is secure? etc... Many of these questions are directly related to the
concept of scalability in distributed systems. The concept of a Superpeer was
proposed to address these issues. 


In systems that utilize Superpeers, groups of peers (often referred to as
committees in most distributed systems literature), propose a peer from 
their group to act as the groups representative in the system. Such a peer does
not carry out the basic tasks of the system as these tasks are performed by
ordinary peers. The Superpeer is responsible for performing routing for that 
group of peers (which we can refer to as an autonomous system). By utilizing a 
Superpeer, network communication is now bifurcated to intra-committee communcation
and inter-committee communication. Respective algorithms exist for both types
of communnication but they will not be discussed here. Those familiar with 
computer networking can utilize the rough analogues of the OSPF and BGP protocols 
for intra-committee and inter-committee communication respectively.


The real question with Superpeers concerns how they are selected. A desirable
Superpeer is one such peer that would satisfy at least the following conditions:
1. Has enough bandwidth available to handle communication for the committee at the 
network-wide level.
2. Is a reliable peer. That is, it has the lowest expected probability of failure
amongst all peers in it's committee.
3. Is an honest peer. That is, the peer once selected as a Superpeer, does not 
attempt to manipulate the committee or go against the intended design and
functionality of the network.


Now, it is important to recognize that these constraints are what would be preferred
in an ideal Superpeer. In reality, such peers are rare. Thankfully however, 
much effort in distributed systems research has been conducted to find ways to 
work within these constraints.


### Leader Election
Leader election is the process of selecting a peer amongst the set of all the
peers in the committee that at least partially satisfies the constraints above.
However, such a task is not easy, as many problems can arise in doing so. One 
such classical problem is the Byzantine Generals problem. 


[REFERENCE GENERALS PROBLEM AND DESCRIPTION]


Attempts at satisfying the Byzantine Generals problem fall into a category of 
algorithms known as 'Consensus algorithms'. Consensus algorithms did not really
play a major part in distributed systems research until the appearance of 
the Blockchain protocol in 2009 by Satoshi Nakamoto (Bitgold by Nick Szabos
is the Blockchain protocols predecessor, and some say, the original Blockchain).


Within the domain of Blockchain, many consensus algorithms exist to balance
the economics of Cryptocurrency with the performance of the network. The first
Cryptocurrency protocol known as Proof-of-Work (POW) requires all nodes to solve
cryptographic puzzles of varying difficulty in order to elect a leader.
The problem with POW is that it requires intense amounts of computational power
that scales with the number of mining peers in the network.


Another consensus algorithm is Proof-of-Stake (POS) and has most recently been 
adopted by the Ethereum Blockchain network. POS splits election into a round or 
number of rounds where interested nodes lock up a portion of their Cryptocurrency
in a mechanism that exchanges Cryptocurrency for stake. A node holding stake may
then vote on a node in the network to act as the leader for the next round.
A popular modification of POS is Byzantine-Fault Tolerant (BFT) POS whereby instead
of a single round, the stake holding peers utilize several rounds to minimize the 
problems raise by the Byzantine Generals problem.


Like many distributed systems, Blockchain suffers from a scalablity problem.
Notable recent attempts at assuaging this problem include: committees, separating
consensus from mining, random oracles, etc... Many of these solutions are derived
from obvious attempts at solving this problem, however, there is a better solution.
Note that the epoch at which the scalability problem becomes apparent to normal
users of the system is implementation dependent. As such a proper solution must
consider several variables including:
1. Which consensus algorithm the network employs.
2. Which gossiping/messaging protocols the network employs.
3. Which mechanisms the network employs to enhance scalability.
4. How resistant the network is to certain security attacks such as denial-of-
service (DOS) and distributed DOS attacks, byzantine attacks, etc...


## Problem Statement
Blockchain systems suffer the distributed system scalability problem wherein the 
performance and efficiency of the network substantially degrades once the network
reaches a certain implementation-dependent size.


## Proposed Solution
The solution proposed by this research involves organizing a P2P network into a 
series of overlay networks similar to DNS that is built ontop of the dynamic B+
tree algorithm. This enables incredible scaling while minimizing so-called
overlay levels.


## Anticipated Challenges
I have identified the following challenges with the proposed solution:
1. Synchronizing the state of the distributed B+ tree across the entire network.
2. Handling dynamic insertion and deletion in a distributed environment.
3. Guaranteeing the network stays responsive when a peer or group of peers becomes
malicious.


### Terminology
Autonomous System (AS): An autonomous system is a collection of disparate nodes
organized into a closed system. The AS is able to automatically respond to feedback
both from within and without the system. The most common autonomous systems can
be found within the modern Internet infrastructure.


B+ Tree: A dynamic data structure often used to minimize searching and disk access
within relational database systems. B+ trees possess the ability to automatically
expand and contract as keys (and potential associated values) are addedd to the 
tree.


Blockchain: A relatively recent technology better known as a distributed ledger.
A blockchain is literally a distributed read-only time-stamping server that
underlies all major cryptocurrency systems. Recent research has shifted towards
inter-blockchain communication (what I refer to as IBC) and improving accessibility
and usability of smart contracts.


BFT: Byzantine Fault Tolerance (BFT) is a desired property of all distributed 
systems. BFT is a classical problem in distributed systems. It represents how
well a distributed system is able to make accurate and correct decisions in
the presence of potentially faulty nodes. Leslie Lamport et al. proved in their
classical paper on the Byzantine Generals problem that for any system with *N* nodes,
a subset *n* must exist from all nodes *N* where |*n*| must be greater than 1/3N 
for the system to continue making honest decisions in the face of *N*-*n* faulty 
nodes.


Committee: A collection of elected nodes in a distributed system that represents
a larger group of nodes. These nodes make collective decisions on behalf of their
collective electorate. They are also often used to minimize communication 
complexity amongst the nodes they represent. Committees largely exist as a 
possible solution to Byzantine Fault Tolerance and the Byzantine Generals problem.


Cryptocurrency: An alternative form of economic exchange. Cryptocurrency is purely
digital and was designed to reduce the overhead in traditional financial markets
by elminating the use of third-party intermediaries in financial transactions.
The role of cryptocurrency as a bargaining chip is only a byproduct of its
original goal. The primary goal of cryptocurrency is to serve as incentive in
blockchain systems. Cryptocurrency is highly speculative (more so than most stocks)
as its value is bound to the performance of the distributed ledger it is built on.


DNS: The Dynamic Name Service (DNS). DNS is a classical distributed system whose
design serves as the basic model for the pursuits of this research. DNS is a
fundamental service in the Internet. Without it, users would need to remember
the IP addresses of the servers/hosts they wish to communicate with instead of a
convenient pnemonic hostname.


Round-Trip Time: The time between sending a message from a sender to
a receiver and getting back the reply. RTT can be used as a simple
form of load-balancing for intermediary routes. For many applications
the classical formula for RTT: $(1 - alpha) * sample + alpha * estimate$.


Routing Table: Usually associated with layer three (Network Layer) devices, a routing
table maps IP addresses to outbound ports. In computer networking when a router 
receives a packet, it strips off the layer three header and looks up the destination
IP address in its routing table. The router will match the IP address with the 
outbound I/O port corresponding to the longest matching IP prefix. How the 
routing table is updated and maintained is outside of the scope of this table.
Interested readers can refer __Computer Networking: A Top-Down Approach"__ by James
Kurose and Keith Ross.


### General Description and Architecture
B+ trees are extensively used in all types of database management systems. Their 
primary purpose is to serve as a key index to accelerate lookups. I propose 
utilizing a modified B+ tree as the basis for a generalized architecture for
a distributed system. The standard overflow and underflow mechanisms of B+ trees
would play a large part in enabling greater scalability in such a system. All 
internal nodes of the system are regarded as directory nodes. From a generic
standpoint, directory nodes haev the sole responsibilty of routing requests through
the network to leaf nodes. Leaf nodes act as worker nodes within the network. The
services offered by the network are performed by leaf nodes. Essentially every 
level of the tree that is not the leaf level serves as an overlay network and is
intended to reduce communication complexity.


Due to the B+ tree architecture of the system, certain systems (dependent on use
case) can leverage the B+ trees internal nodes as a load balancing mechanism. For
instance, if the service offered by the network is a replicated datastore such as
a blockchain, then all leaf nodes hold full chain state regardless. Internal nodes
in such a system only need to maintain state on received and observed feedback 
regarding their immediate children. This information can be utilized to route 
requests appropriately to automatically balance load for the system. Heuristics and
feedback control mechanisms for these types of DBPT are part of this proposal.


In such a system, new entrants would generate a key from a large domain, say N+ 
(or some restricted domain drawn from it). The new entrant would then submit 
their key to the network and a quick lookup would be performed to determine if 
the key is already registered. If not, the entrant enters the network by being 
mapped to the correct place in the B+ tree. Otherwise, a collision resolution 
mechanism is employed to assign the entrant a unique key. This key would then be 
used to uniquely identify a user in the network. Systems that require certain 
leels of anonymity would require different schemas.


Leaving a DBPT system is simple. It largely performs identically to leaving a 
regular B+ tree. There are however, two distinct cases:
1. A leaf (worker) node requests to leave the network.
2. An internal (directory) node requests to leave the network.


In the first case, the leaf node notifies its parent of its intent to leave. Here
parent is defined as the directory node responsible for routing traffic to and from
the leaf node. Once notified, the parent node would just remove the worker node
from its routing table (here, I borrow terminology from networking literature).


In the second case, just as with the first case, the internal node must inform
its parent so it can update its routing table. The internal node must then promote
its left-most worker node to take its place. This first requires notifying the 
worker node so it can handle all pending requests. Next the internal node must 
notify its parent that the worker node is taking its place so the parent node can
update its routing table appropriately. It should never be the case that the 
internal node does not have any children as that would indicate that the internal
is not a directory node but is instead, a worker node.


Note that when a removal, overflow, or underflow occur, the regional subtree where
this occurs must be locked so that its state remains consistent. Doing so is 
actually quite simple. In most cases, only the directory node needs locked in the
request direction (travelling down the tree). Unless the directory node is being
directly manipulated itself (i.e. it is being removed), responses should still be 
allowed to flow back up through the tree.


Given the nature of B+ trees, communication between nodes in the tree is simple to
achieve. Requests from clients arrive into the network at level 0. A level 0 node
then forwards the request to the appropriate level 1 node. This is carried out 
until the requests reaches level N-1 where the target worker node handles the 
request. The reply is then percolated back up to level 0 following the same path
as the request. Once a request reaches level 0, it is then handed back to the 
requesting client. The communication pattern differs if load balancing is
incorporated into the network.


A DBPT network that incorporates load balancing must appropriately choose the 
correct node at each N+1 level to route requests to. Full load balancing in
such a system is only applicable when the system also supports at least some level
of replication. If each worker node implements a different service, then load 
balancing is pointless and should not be utilized. From here on, assume that the
network supports load balancing and has at least some service replication.


In this case, upon receiving a request at level N, the receiving level N node should
first analyze its descendant sub-networks to find nodes that support the requested
service. The node should then perform load analysis on the child nodes who can 
reach the target service nodes. Finally, the request should be routed down the 
child node with the lowest overall load. In this sense, the network tries to minimize
load across all nodes. Replies would thus be treated the same as with a non load
balancing DBPT network.


As an example, assume that the network implements a distributed ledger service. In
such a service, all worker nodes maintain an almost near-consistent state of the 
ledger that get synchronized at each service epoch. In this service, replication
is almost always at 100% so any worker node can provide the same service as any
other worker node. In this case, the directory nodes only need maintain load 
statistics on all of their children that they periodically update using a simple
calculation such as RTT estimation.

### Formal Proof
A formal proof of the architecture presented here is forthcoming.
