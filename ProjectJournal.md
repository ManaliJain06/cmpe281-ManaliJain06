# cmpe281-ManaliJain06

This Journal is for CMPE281 Personal project of testing partitiion tolerance of NOSQL Database(Can be any)

References link-
1) https://www.infoq.com/articles/jepsen
2) https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed
3) https://www.annashipman.co.uk/jfdi/the-cap-theorem-and-mongodb.html

# Agenda-

- Getting to know NOSQL databases, CAP theorm.

- Finding out AP and CP databases.

- Selection of the databases for testing partition tolerance

- Design of the project.

- AWS EC2 configuration to host Go API and test our partition tolerance.


# Week 1 (10/07/2018) to (10/13/2018)

## Plan
1)  Study CAP theorm in detail
2) Getting to know AP and CP databases and select databases for testing

## Status
1) CAP theorm- It states that it is Impossible for a distributed system to achieve Consistency, availability and partition tolerance all at the same time. A distributed system can only achieve either consistency or availability during network partition tolerance. 

2) My project NoSQL database selection

 * CP - MongoDB

   MongoDB is highly consistent database because it works on replica set where we have one 					  primary which is mainly for writes and the replicas are secondary for reads. If primary is down then new primary will be elected and then system is unavilable for some time

* AP - Riak

  Riak is highly avaialable database during netwrok partition because it uses clustering where servers are grouped together so even in failure of one cluster other can respond for read and writes. It is also known as masterless architecture.

  Avaialable at - http://basho.com/posts/technical/relational-to-riak-part-1-high-availability/
