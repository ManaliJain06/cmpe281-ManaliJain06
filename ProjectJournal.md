# cmpe281-ManaliJain06

**This Journal is for CMPE281 Personal project of testing partitiion tolerance of NOSQL Database(Can be any)**

References link-
1) https://www.infoq.com/articles/jepsen
2) https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed
3) https://www.annashipman.co.uk/jfdi/the-cap-theorem-and-mongodb.html
4) https://www.simplilearn.com/replication-and-sharding-mongodb-tutorial-video
5) https://docs.mongodb.com/manual/core/sharding-shard-key/

# Agenda-

- Getting to know NOSQL databases, CAP theorm.
- Finding out AP and CP databases.
- Selection of the databases for testing partition tolerance
- Design of the project.
- AWS EC2 configuration to host Go API and test our partition tolerance.
- Testing partition tolerance and system behaviour 
- Demonstrating AP and CP peoperties of the database
- Sharing of MongoDB database


# Week 1 (10/07/2018) to (10/13/2018)

## Plan
1)  Study CAP theorm in detail
2) Getting to know AP and CP databases and select databases for testing

## Status
1) CAP theorm- It states that it is Impossible for a distributed system to achieve Consistency, availability and partition tolerance all at the same time. A distributed system can only achieve either consistency or availability during network partition tolerance. 

2) My project NoSQL database selection

 * CP - MongoDB

   MongoDB is highly consistent database because it works on replica set where we have one 					  primary which is mainly for writes and the replicas are secondary for reads. If primary is down then new primary will be elected.

* AP - Riak

  Riak is highly avaialable database during netwrok partition because it uses clustering where servers are grouped together so even in failure of one cluster other can respond for read and writes. It is also known as masterless architecture.

  Avaialable at - http://basho.com/posts/technical/relational-to-riak-part-1-high-availability/



# Week 2 (10/14/2018) to (10/20/2018)

**Mongo Week**

##Plan 
1) Research on sharding for MongoDB

2) Design for MongoDB replication

3) AWS setup for Mongo

## Status

**Sharding**

As the database grows it is impossible for one server to store and manage the data. So, we need to distribute the data into different servers. Sharding is basically doing horizontally scaling where data from one collection can be present in two different shard servers. Sharding also help in achieving scalability.

Sharding is done by shard key. It defines the collection data distribution among the shard clusters. Shard key is immutable once defined.

Sarding stratergy
1) Hash Based Sharding- Data is partitioned in the form of chunks, then based on the shard key the chunk is divided.

2) Range Sharding- Here the chunks are made based on the range [minimun key, maximum key].

**Sharded cluster**- It has three main components-
1) Config serveres- It stores cofiguration data about the shards

2) Mongos - It works as a query router for aggregating query data and return JSON

3) Sharding server- The shard cluster where the data is stored



**AWS Mongo Setup**

Step 1: Launch Private EC2 Free-Tier Instance
​    
    	Name: 			 node1
    	AMI:             Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
    	Instance Type:   t2.micro
    	VPC:             cmpe281
    	Network:         private subnet (us-west-1c)
    	Auto Public IP:  disable
    	Security Group:  mongodb-cluster 
    	SG Open Ports:   22, 27017
    	Key Pair:        cmpe281-us-west-1


Step 2: Launch Public EC2 Free-Tier Instance for Jumpbox

		Name: 			 Jumpbox
		AMI:             Ubuntu Server 16.04 LTS (HVM)
		Instance Type:   t2.micro
		VPC:             cmpe281
		Network:         public subnet (us-west-1c)
		Auto Public IP:  Enable
		Security Group:  jumbox-security-group
		SG Open Ports:   22
		Key Pair:        cmpe281-us-west-1

Step 3: Install mongodb in the private instance using Jumbox

```
1) Login to Jumbox ec2-isntance by -
	ssh -i "cmpe281-us-west-1.pem" ubuntu@<jumpbox public IP or public DNS> using your AWS 	    pem file
	e.g ssh -i "cmpe281-us-west-1.pem" ubuntu@ec2-13-57-31-201.us-west-		1.compute.amazonaws.com

2) Now to login to your private EC2 to install mongo on it we need to transfer the pem 		file from your local onto the jumbox to use it to 	login to node1 ec2 isntance
	Open a new terminal then do - scp -i cmpe281-us-west-1.pem <pem file to transfer> 		ubuntu@<public IP for jumbox>:/tmp
	
	Now your file is being tranferred to the Jumbox temp folder. Use that file to login to 	   the private node1 isntance by doing ssh 

3) Install mongoDB on ubuntu
	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 					9DA31620334BD75D9DCB49F368818C72E52529D4

	echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-	org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb.list

	sudo apt update
	sudo apt install mongodb-org

4) MongoDB Keyfile
		openssl rand -base64 741 > keyFile
		sudo mkdir -p /opt/mongodb
		sudo cp keyFile /opt/mongodb
		sudo chown mongodb:mongodb /opt/mongodb/keyFile
		sudo chmod 0600 /opt/mongodb/keyFile

5) Config mongod.conf
		sudo vi /etc/mongod.conf
		 
		a) remove or comment out bindIp: 127.0.0.1
		    replace with bindIp: 0.0.0.0 (binds on all ips) 

		  	network interfaces
		  	net:
		        port: 27017
		        bindIp: 0.0.0.0
		 
		b) Uncomment security section & add key file
		 
			Make Make sure there is two spaces in front of KeyFile
		 
			security:
		    	keyFile: /opt/mongodb/keyFile
		 
		c) Uncomment Replication section. Name Replica Set = cmpe281
		 
			Make Make sure there is two spaces in front of replSetName
			replication:
		    	replSetName: cmpe281

	6) Create mongod.service

		sudo vi /etc/systemd/system/mongod.service

		[Unit]
		    Description=High-performance, schema-free document-oriented database
		    After=network.target

		[Service]
		    User=mongodb
		    ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

		[Install]
		    WantedBy=multi-user.target

	7) Enable Mongo Service
		sudo systemctl enable mongod.service

	8) Restart MongoDB to apply our changes
		sudo service mongod restart
		sudo service mongod status
```



# Week 3 (10/21/2018) to (10/27/2018)

**Mongo Week**

## Plan

1) To do replication on the Mongo cluster by following the design 

2) Research about Riak for AP configuration

## Status

Now that as we have our Mongo setup for one node is done we will create other Nodes for replication

Step 4: Create AMI of the priavte EC2 instance
​	Now we have the private instance with all the mongo DB setup done we will now create the AMI to   create other 4 clusters.

Step 5: Launch new Instances using the AMI created		

```
		1) go to launch ec2 isntance
		2) Select the AMI created
		3) Instance Type:   t2.micro
			Numbner of Instances: 2
			VPC:             cmpe281
			Network:         private subnet (us-west-1c)
			Auto Public IP:  disable
			Security Group:  mongodb-cluster 
			SG Open Ports:   22, 27017
			Key Pair:        cmpe281-us-west-1

		4) Instance Type:   t2.micro
			Numbner of Instances: 2
			VPC:             cmpe281
			Network:         private subnet (us-west-1a)
			Auto Public IP:  disable
			Security Group:  mongodb-cluster 
			SG Open Ports:   22, 27017
			Key Pair:        cmpe281-us-west-1
```

Step 6: Replace Host Names with Public IP or DNS Names.

			10.0.1.163 primary
			10.0.1.73 secondary1
			10.0.1.109 secondary2
			10.0.3.191 secondary3
			10.0.3.107 secondary4

Step 7: Intializing the replica set

			Go to one of your private mongodb instance and make a replica set there
	
			1) start mongo shell
				mongo
	
			2) replica set
				rs.initiate( {
			       _id : "cmpe281",
			       members: [
			          { _id: 0, host: "primary:27017" },
			          { _id: 1, host: "secondary1:27017" },
			          { _id: 2, host: "secondary2:27017" },
			          { _id: 3, host: "secondary3:27017" },
			          { _id: 4, host: "secondary4:27017" }
			       ]
			    })


Step 8: Insert data into the monog using primary 

		1) Create user
			Create an admin user to access the database.
	
				mongo
	
			Select admin database.
	
				use admin
	
			Create admin account.
	
			    db.createUser( {
			        user: "admin",
			        pwd: "cmpe281",
			        roles: [{ role: "root", db: "admin" }]
			    });
	
		2) Login to Primary as Admin:
			mongo -u admin -p cmpe281 --authenticationDatabase admin
			use burger;
		
			db.restaurant.insert({"id": "1","restaurantName": "Burger Place","zipcode":"950012","phone":"669-456-7675","address":"34 Green Ave","email":"king@gmail.com"});
	
			db.restaurant.find({}).pretty()
			
			db.restaurant.insert({"id": "3","restaurantName": "GO Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});
	
			db.restaurant.insert({"id": "4","restaurantName": "Milli Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});

  		