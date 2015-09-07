
A scalable and eventually consistent image sending backend with no single point failure!


Change log:

Added RaftManager that would take care of Raft leader election and log replication.
State pattern implemented for representing the current state of the nodes (follower, candidate, leader)
Logs are replicated in each server node by the leader and committed once they are passed on to a majority of nodes

Added basic client for transferring images.

———————————————————————

Implemented new mechanism for message delivery:

-> Client sends request to any node 

-> Node (if not the leader) forwards the request to leader (in contrast to previous mechanism where we broadcasted the message) 

-> Leader creates log that includes the ClientMessage 

-> Log gets replicated 

-> log is committed by the leader, Followers imitate the leader 

-> when a node commits a log, it actually delivers the message to client

————————————————————————

Only client requests are delivered across clusters. 

When a cluster receives message from another cluster, it just delivers the message to its own clients, and does not forward the message to other clusters.

________________________

Solution to guarantee if the image has been received before committing logs ( to be implemented, effectively scalable though! )

-> Register clients on each server node in a balanced way using consistent hashing. 

->This hash-table for each client-server_node mapping will be maintained by each server in the system.

-> Each client can still request any server in the cluster, but image data(not the log) will be forwarded only to that server where the receiver client is registered and the log request will be forwarded to the leader as before.

-> When the server receives the data, it notifies the leader to commit the log. This scales the system for 1000s of server nodes in a cluster, since we get rid of forwarding the image data to each node and still prevent data loss.

-> If a node in the cluster fails, consistent hashing will ensure that the clients are re-balanced amongst the remaining nodes of cluster.  
 
