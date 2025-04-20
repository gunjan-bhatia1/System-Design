* **Split brain:** Due to network partition or failure system is divided in two or more parts. Each of them think that they are only active part of network.
    * This will lead to inconsistency in data, duplicate and loss of data,
* **Distributed consensus algorithm**: It will help prevent split brain issue
    * Leader based protocols(RAFT):
        * Only one leader is allowed at a time who takes the decision.
        * If there is partition tolerance new leader is one with majority.
        * If there is no quorum(majority win) then no leader is elected until partition heals. Preventing conflicts
        * It also maintains logs and follower replicate. Logs are maintained so that we can replay each action if needed.
        * Here we have leader(manager), follower(replicate logs and listen) and candidate(if follower don't hear from leader they can elect).
        * Timeout and heartbeat: Listen to heartbeat to ensure they are active, in case we don't hear from leader for some time, and it times out we elect new
    * Quorum based(ZAB(Zookeeper atomic broadcast), Paxos):
        * Here also majority play the role.
        * In zookeeper also we have a coordinator which handles everything.
            * Also in zookeeper action is done in order from leader to follower.
        * Paxos we don't have any leader.
            * We have proposers(who propose value), acceptors(who accepts), leaders(who learn value is proposed).
            * As a result there will be to and fro before majority come on conclusion.
    * **Eventual consistency(Dynamo DB, Cassandra):** There are system that achieve eventual consistency. It allows split brain until partition is resolved.
*  In layman ZAB is a coordinator in a club making sure decisions are shared quickly and consistently whereas Raft is  a bookkeeper writing everything in a shared ledger, and everyone following the book exactly
