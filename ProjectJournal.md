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

## Challenges

None faced

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

**Step 1: Launch Private EC2 Free-Tier Instance**

```
Name: 			 node1
    	AMI:             Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
    	Instance Type:   t2.micro
    	VPC:             cmpe281
    	Network:         private subnet (us-west-1c)
    	Auto Public IP:  disable
    	Security Group:  mongodb-cluster 
    	SG Open Ports:   22, 27017
    	Key Pair:        cmpe281-us-west-1
```

**Step 2: Launch Public EC2 Free-Tier Instance for Jumpbox**

		Name: 			 Jumpbox
		AMI:             Ubuntu Server 16.04 LTS (HVM)
		Instance Type:   t2.micro
		VPC:             cmpe281
		Network:         public subnet (us-west-1c)
		Auto Public IP:  Enable
		Security Group:  jumbox-security-group
		SG Open Ports:   22
		Key Pair:        cmpe281-us-west-1

**Step 3: Install mongodb in the private instance using Jumbox**

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



## Challenges

None faced 

## Mistakes

1) Mongo shell was not not running - 

When you do mongo after installing and starting mongodb, then it gives an error - Connection Refused.

**Solution**- 

* Check whether your mongod service is up or not by - ```sudo service mongod status ``` If the service is up and running it will give actively running

* Check for mongod.service and mongod.conf parameters. Check for the name and indentations in the files


# Week 3 (10/21/2018) to (10/27/2018)

**Mongo Week**

## Plan

1) To do replication on the Mongo cluster by following the design 

2) Research about Riak for AP configuration

## Status

Now that as we have our Mongo setup for one node is done we will create other Nodes for replication and to test the CP properties for Mongo

**Step 4: Create AMI of the priavte EC2 instance**
​	Now we have the private instance with all the mongo DB setup done we will now create the AMI to   create other 4 clusters.

**Step 5: Launch new Instances using the AMI created**		

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

**Step 6: Replace Host Names with Public IP or DNS Names.**

	        sudo vi /etc/hosts   
	            10.0.1.163 primary
	            10.0.1.73 secondary1
	            10.0.1.109 secondary2
	            10.0.3.191 secondary3
	            10.0.3.107 secondary4

**Step 7: Intializing the replica set**

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

**Step 8: Insert data into the monog using primary** 

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
			
			#Create database burger and collection restaurant
			use burger;
		
			db.restaurant.insert({"id": "1","restaurantName": "Burger Place","zipcode":"950012","phone":"669-456-7675","address":"34 Green Ave","email":"king@gmail.com"});
	
			db.restaurant.find({}).pretty()
			
			#This further insertions will be used for testing
			db.restaurant.insert({"id": "3","restaurantName": "GO Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});
	
			db.restaurant.insert({"id": "4","restaurantName": "Milli Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});

  

## Challenges

None Faced

## Mistakes

1) Hostname is not getting set in the /etc/hosts file
**Solution**- I fixed this problem by doing reboot on the instense

2) Replicas initiation was giving error as connection refused

I was getting the below error while initiating the replica set for mongodb​	

```
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
{
	"ok" : 0,
	"errmsg" : "replSetInitiate quorum check failed because not all proposed set members responded affirmatively: secondary1:27017 failed with Error connecting to secondary1:27017 (10.0.1.73:27017) :: caused by :: Connection refused, secondary2:27017 failed with Error connecting to secondary2:27017 (10.0.1.109:27017) :: caused by :: Connection refused, secondary3:27017 failed with Error connecting to secondary3:27017 (10.0.3.191:27017) :: caused by :: Connection refused, secondary4:27017 failed with Error connecting to secondary4:27017 (10.0.3.107:27017) :: caused by :: Connection refused",
	"code" : 74,
	"codeName" : "NodeNotFound"
}	
```



**Solution**- Jumpbox into each of your MongoDB instances(i.e primary and all secondary) and make sure that all instances have mongodb service up.

You can start mongodb by - ```sudo service mongod restart```



# Week 4 (10/28/2018) to (10/03/2018)

**Mongo Week**

## Plan

1) Test for CP properties of Mongodb setup that we have done earlier
2) Write Test Cases

## Status

Below are the test cases created for the Consistency of MongoDB during Partition Tolerance

- [x] **Test 1 : Replication Test**

  **Test Plan-** Secondary nodes should be able to replicate the data inserted into primary node 

  **Expected Outcome-** Should be able to query secondary nodes and be able to read data from secondary

  **Actual Outcome-** Able to read the data from secondary node showing that replication from Master to slave is working properly.

  **Test Exceution-**

1. do rs.status() to see which node is primary 

2. Excecute the command ```mongo``` in your primary node EC2 instance and insert some data in your burger database.

   ```
   db.restaurant.insert({"id": "1","restaurantName": "Burger Place","zipcode":"950012","phone":"669-456-7675","address":"34 Green Ave","email":"king@gmail.com"});
   ```

3. Jumpbox into your seconday EC2 instances and then execute below query to see whether you are able to read the previous data you inserted in primary.

   ```
   db.restaurant.find({}).pretty()
   ```

   **Test Result-** 

   Image- https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test1_Result(1).png



- [x] Test 2 : API test reading and updating data**

  **Test Plan-** Update the data in primary and see whether secondary is getting the updated data. 

  **Expected Outcome-** Secondary node should be up to date with primary

  **Actual Outcome-** Able to read the data from secondary node showing that replication from Master to slave is working properly.

  **Test Exceution-**

   1. In your primary mongodb node execute the command ```mongo```

   2. Insert more data into your primary.

      ```
      db.restaurant.insert({"id": "2","restaurantName": "GO Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});
      ```

   3. Excecute the command ```mongo``` in your all secondary nodes EC2 instance and insert some data in your burger database

      You should be able to see both the records being inserted.

      ```
      db.restaurant.find({}).pretty()
      ```


  **Test Result-** 

  Image-



- [ ] 

- [x] **Test 3 Made a partition by disconnecting a secondary node i.e. node 2 with private IP 10.0.1.73 from all other nodes** **and read stale data**

  **Test Plan-** Make a partition tolerance by disconnecting one node(secondary) and observing behavioiur. 

  **Expected Outcome-** When secondary node is disconnected then that disconnected node will have stale data and we should be able to read it

  **Actual Outcome-** Able to read stale data from the secondary disconneced node. Inserted a new data in primary and other secondary nodes got updated but the one which was disconnected gives the stale data to read.

  **Test Exceution-**

1. Go to any of your secondary node Ec2 instance and execute the below commands to disconnect that node from the other nodes (Here I am disconnecting node2)

```
DROP connection
	# drop ipaddress
	sudo iptables -A INPUT -s 10.0.1.163 -j DROP      (primary node)
	sudo iptables -A INPUT -s 10.0.1.109 -j DROP
	sudo iptables -A INPUT -s 10.0.3.107 -j DROP
	sudo iptables -A INPUT -s 10.0.3.191 -j DROP
```

2. Then insert some data in your primary node

```
db.restaurant.insert({"id": "3","restaurantName": "GO Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});
```

3. The login into your disconnected secondary and you will see that we are getting the stale records    and not the currenlty updated one.

```
db.restaurant.find({}).pretty()
```

 4. Login into your secondary mongodb nodes which are not disconnected and you will see that connected secondary nodes are up to date and getting the newly insterted data.

```
db.restaurant.find({}).pretty()
```

​       ​     

​	**Test Result-** 

​	Image-



- [x] **Test 4: Connect the secondary (node2) again and then disconnect the primary from the other 3 secondary mongodb instances**		

  **Test Plan-** Make a partition tolerance by disconnecting primary node and observe **leader election**. 

  **Expected Outcome-** When primary node is disconnected then the new primary must be elected from the other secondary nodes

  **Actual Outcome-** New Primary node is elected from the other secondary nodes and the previous primary node is now secondary.

  **Test Exceution-**

  1. Go to your primary node Ec2 instance and execute the below commands to disconnect that node from the other secondary nodes.

  ```
  #Disconnecting primary from other 3 secondary
  
  		sudo iptables -A INPUT -s 10.0.1.109 -j DROP
  		sudo iptables -A INPUT -s 10.0.3.107 -j DROP
  		sudo iptables -A INPUT -s 10.0.3.191 -j DROP
  ```

  2. Then do ```rs.status()``` and you will see that the node has error message as disconnected as it is disconnected from the netowork

  3. Do ```rs.status()``` in any of the secondary node and you will see that a new primary is elected which was earlier secondary.

  4. You can again connect the disconnecterd nodes by below commands

     ```
           sudo iptables -D INPUT -s 10.0.1.109 -j DROP
     	  sudo iptables -D INPUT -s 10.0.3.107 -j DROP
     	  sudo iptables -D INPUT -s 10.0.3.191 -j DROP
     ```



  **Test Result-** 

  Image-


- [x] **Test 5 Stop Primary mongodb to show leader selection and recovery**

  This test is in continuation of the Test4

  **Test Plan-** Observer **partition recovery**

  **Expected Outcome-** When the disconnected node is connected again then it will try to sync up with the network and get the new records or changes being done during partition tolerance

  **Actual Outcome-** Node after connection is working as per other secondary nodes with latest recrords being udpdated.

  **Test Exceution-**

  1. Go to the node that you disconnected in the Test4 and run below commands to connect it back to the network.

     ```
     sudo iptables -D INPUT -s 10.0.1.109 -j DROP
     sudo iptables -D INPUT -s 10.0.3.107 -j DROP
     sudo iptables -D INPUT -s 10.0.3.191 -j DROP
     ```

  2. Then do ```rs.status``` and see that the node is live again the network and sending heartbeats

  3. Run the below commands in Mongo shell to see the upadted records being fetched.

     ```
     db.restaurant.find({}).pretty()
     ```



## Challenges

1) MongoDB was not allowing to read from secondary nodes when you do ```db.restaurant.find({}).pretty()``` and giving error as ''not master and slaveOk=false"

**Solution**- use this command ```rs.slaveOk()``` to set tell mongo shell that you want to read from the secondary nodes and are allowing reads.



# Week 5 (11/04/2018) to (11/10/2018)

**MongoDB Sharding Week**

## Plan

1) Configure MongoDB to support two data shards

2) Design for MongoDB sharding

3) Using MongoDB Bios Collection demonstrate sharding and select a shard key



## Status

Sharding steps

1. **Setting up Config servers**

**Step1- First launch 3 EC2 isntancefrom the Mongodb AMI we had created earlier**

1. ```
     For each of the config server ubuntu instance do the following (using Command or      Configuration)
   
      1) Using commands-
    
      		mongod --configsvr --replSet cmpe281 --dbpath /var/lib/mongodb --port 27017. --logpoath /var/log/mongodb/mongod.log
   
      		ps -aux | grep mongod   // to see the mongod service running
   
      2) By configuration parameters-
   
   	   # change the configuration parameters by
   	   sudo vi /etc/mongod.conf
   	   
   	   	1) net:
   	   			port: 27019
   	   
   	   	2) replication 
   	   			replSetname: cmpe281
   
   	   	3) Sharding
   	   			clusterRole: configsvr
   
   	   # make the host file changes
   		 sudo vi /etc/hosts  
   			<your public IP of the instance> host name
   			E.g.
   			54.183.98.65 configServer1
   			54.241.174.222 configServer2
   			54.183.90.178 configServer3
   ```



**Step2- Now connect the mongo shell to one of the config server members by specifying the port number**

1. 		Enable Mongo Service
   			sudo systemctl enable mongod.service
   	
   		Restart MongoDB to apply our changes
   			sudo service mongod restart
   			sudo service mongod status
   	
   		Login to Mongo 
   			mongo -port 27019


**Step3- Now inititalize the replia set for the config servers**

1. 		rs.initiate( {_id : "cmpe281",configsvr: true,members: [{ _id: 0, host: "configServer1:27019" },{ _id: 1, host: "configServer2:27019" },{ _id: 2, host: "configServer3:27019" }]})
   	
   		rs.status()   // this will show you the status of the nodes

