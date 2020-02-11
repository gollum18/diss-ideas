# dbpt - The Distributed B+ Tree Data Structure

## Background

### B+ Trees

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
series of overlay networks similar to DNS. This solution will organize each overlay
network into a single distributed B+ tree.

## Anticipated Challenges
I have identified the following challenges with the proposed solution:
1. Synchronizing the state of the distributed B+ tree across the entire network.
2. Handling dynamic insertion and deletion in a distributed environment.
3. Guaranteeing the network stays responsive when a peer or group of peers becomes
malicious.

### Operational Algorithms

#### Hashing

#### Searching

#### Insertion

#### Deletion
