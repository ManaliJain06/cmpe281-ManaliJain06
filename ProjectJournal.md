# cmpe281-ManaliJain06

**This Journal is for CMPE281 Personal project of testing partitiion tolerance of NOSQL Database(Can be any)**

References link-
1) https://www.infoq.com/articles/jepsen
2) https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed
3) https://www.annashipman.co.uk/jfdi/the-cap-theorem-and-mongodb.html
4) https://www.simplilearn.com/replication-and-sharding-mongodb-tutorial-video
5) https://docs.mongodb.com/manual/core/sharding-shard-key/
6) https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html
7) https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html
8) https://docs.aws.amazon.com/eks/latest/userguide/EKS_IAM_user_policies.html


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

## Plan
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

**Mongo Replication Design**
![MongoDB design](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/Monogodb_CP_architecture.png)

**AWS Mongo Setup**

**Step 1: Launch Private EC2 Free-Tier Instance**

```
Name: node1
AMI:             Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
Instance Type:   t2.micro
VPC:             cmpe281
Network:         private subnet (us-west-1c)
Auto Public IP:  disable
Security Group:  mongodb-cluster (22, 27017)
SG Open Ports:   22, 27017
Key Pair:        cmpe281-us-west-1
```

**Step 2: Launch Public EC2 Free-Tier Instance for Jumpbox**

```
Name:Jumpbox
AMI:             Ubuntu Server 16.04 LTS (HVM)
Instance Type:   t2.micro
VPC:             cmpe281
Network:         public subnet (us-west-1c)
Auto Public IP:  Enable
Security Group:  jumbox-security-group (22)
SG Open Ports:   22
Key Pair:        cmpe281-us-west-1
```

**Step 3: Install mongodb in the private instance using Jumbox**

```
1) Login to Jumbox ec2-isntance by -
ssh -i "cmpe281-us-west-1.pem" ubuntu@<jumpbox public IP or public DNS> using your AWS pem file.
e.g ssh -i "cmpe281-us-west-1.pem" ubuntu@ec2-13-57-31-201.us-west-1.compute.amazonaws.com

2) Now to login to your private EC2 to install mongo on it we need to transfer the pem file from your local onto the jumbox to use it to login to node1 ec2 isntance
Open a new terminal then do -
scp -i cmpe281-us-west-1.pem <pem file to transfer> ubuntu@<public IP for jumbox>:/tmp

Now your file is being tranferred to the Jumbox temp folder. Use that file to login to the private node1 isntance by doing ssh

3) Install mongoDB on ubuntu
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

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

	a) remove or comment out bindIp: 127.0.0.1.
	Replace with bindIp: 0.0.0.0 (binds on all ips)

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

## Mistakes encountered

1) Mongo shell was not not running - When you do mongo after installing and starting mongodb, then it gives an error - Connection Refused.

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
	Security Group:  mongodb-cluster (22, 27017)
	SG Open Ports:   22, 27017
	Key Pair:        cmpe281-us-west-1

4) Instance Type:   t2.micro
	Numbner of Instances: 2
	VPC:             cmpe281
	Network:         private subnet (us-west-1a)
	Auto Public IP:  disable
	Security Group:  mongodb-cluster (22, 27017)
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
```

**Step 8: Insert data into the mongo using primary**

```
1) Create user
    Create an admin user to access the database
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
```
![MongoDB EC2 instances](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_EC2_Instances.png)

## Challenges

None Faced

## Mistakes encountered

1. Hostname is not getting set in the /etc/hosts file

**Solution**- I fixed this problem by doing reboot on the instense

2.  Replicas initiation was giving error as connection refused

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

## Test 1 : Replication Test

  **Test Plan-** Secondary nodes should be able to replicate the data inserted into primary node

  **Expected Outcome-** Should be able to query secondary nodes and be able to read data from secondary

  **Actual Outcome-** Able to read the data from secondary node showing that replication from Master to slave is working properly.

  **Test Exceution-**

1. do rs.status() to see which node is primary

2. Excecute the command ```mongo -u admin -p cmpe281 --authenticationDatabase admin``` in your primary node EC2 instance and insert some data in your burger database.

   ```
   db.restaurant.insert({"id": "1","restaurantName": "Burger Place","zipcode":"950012","phone":"669-456-7675","address":"34 Green Ave","email":"king@gmail.com"});
   ```

3. Jumpbox into your seconday EC2 instances and then execute below query to see whether you are able to read the previous data you inserted in primary.

   ```
   db.restaurant.find({}).pretty()
   ```

  **Test Result-**
![MongoDB test1 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test1_Result(1).png)

![MongoDB test1 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test1_Result(2).png)

## Test 2 : Test updating data

  **Test Plan-** Update the data in primary and see whether secondary is getting the updated data.

  **Expected Outcome-** Secondary node should be up to date with primary

  **Actual Outcome-** Able to read the data from secondary node showing that replication from Master to slave is working properly.

  **Test Exceution-**

   1. In your primary mongodb node execute the command ```mongo -u admin -p cmpe281 --authenticationDatabase admin```

   2. Insert more data into your primary.

      ```
      db.restaurant.insert({"id": "2","restaurantName": "Zip Burger","zipcode":"950030","phone":"408-456-7675","address":"101 Oak Ave","email":"goto@gmail.com"});
      ```

   3. Excecute the command ```mongo -u admin -p cmpe281 --authenticationDatabase admin``` in your all secondary nodes EC2 instance and insert some data in your burger database

      You should be able to see both the records being inserted.

      ```
      db.restaurant.find({}).pretty()
      ```

  **Test Result-**
  ![MongoDB test2 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test2_Result.png)

## Test 3 Made a partition by disconnecting a secondary node i.e. node2 with private IP 10.0.1.73 from all other nodes and read stale data

  **Test Plan-** Make a partition tolerance by disconnecting one node(secondary) and observing behavioiur.

  **Expected Outcome-** When secondary node is disconnected then that disconnected node will have stale data and we should be able to read it

  **Actual Outcome-** Able to read stale data from the secondary disconneced node. Updated data in primary and other secondary nodes got updated but the one which was disconnected gives the stale data to read.

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

2. Then update some data in your primary node

	```
	db.restaurant.update(
   { "id": "1" },
   { $set: { "restaurantName": "GO Burger" } })
	```

3. Login into your disconnected secondary and you will see that we are getting the stale records and not the currenlty updated one.

	```
	db.restaurant.find({}).pretty()
	```

 4. Login into your secondary mongodb nodes which are not disconnected and you will see that connected secondary nodes are up to date and getting the newly insterted data.

	```
	db.restaurant.find({}).pretty()
	```

**Test Result-**
![MongoDB test3 Partition Tolerance](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test3_PartitionTolerance.png)
![MongoDB test3 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test3_Result.png)

## Test 4: Connect the secondary (node2) again and check partition recovery

  **Test Plan-** Connect Secondary node again to observe partition recovery.

  **Expected Outcome-** When secondary node is connected again then we will see that the secondary node is updated witht the latest data

  **Actual Outcome-** Secondary node is eventually consistent and partition recovery is done.

  **Test Exceution-**

  1. Connect back your secondary again -
```
sudo iptables -D INPUT -s 10.0.1.163 -j DROP
sudo iptables -D INPUT -s 10.0.1.109 -j DROP
sudo iptables -D INPUT -s 10.0.3.107 -j DROP
sudo iptables -D INPUT -s 10.0.3.191 -j DROP

check the list by-
sudo iptables -L
```
 2. Login to the mongo shell of the secondary you just connected again and run below query
	```db.restaurant.find({}).pretty()```

	You will see that the data is upated from the previous test

**Test Result-**
![MongoDB test4 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test4_Result.png)

## Test 5 Disconnect the primary from the other secondary mongodb instances to test Leader Election

  **Test Plan-** Observer Leader Election

  **Expected Outcome-** After the primary is disconnected from all nodes then a new leader should be selected.

  **Actual Outcome-** Leader election is taking place with Secondary4 node elected as new Primary Node.

  **Test Exceution-**

1. Go to your primary node Ec2 instance and execute the below commands to disconnect primary node from the other secondary nodes.
```
Disconnecting primary from other 4 secondary nodes
	sudo iptables -A INPUT -s 10.0.1.73 -j DROP
	sudo iptables -A INPUT -s 10.0.1.109 -j DROP
	sudo iptables -A INPUT -s 10.0.3.107 -j DROP
	sudo iptables -A INPUT -s 10.0.3.191 -j DROP
```
 2. Then do ```rs.status()``` and you will see that the node has error message as disconnected as it is disconnected from the netowork

3. Do ```rs.status()``` in any of the secondary node and you will see that a new primary is elected which was earlier secondary.

4. The disonnected primary will now become a secondary node

**Test Result-**
![MongoDB test5 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/MongoDB_Test5_Result.png)


## Mistakes encountered

1) MongoDB was not allowing to read from secondary nodes when you do ```db.restaurant.find({}).pretty()``` and giving error as "not master and slaveOk=false"

**Solution**- use this command ```rs.slaveOk()``` to set tell mongo shell that you want to read from the secondary nodes and are allowing reads.

# Assignment Questions for CP
1. How does the system function during normal mode (i.e. no partition)
- During normal mode the system works fine i.e. it is allowing insertion of data and replicating it to all the nodes. In Mongo you can write only on master as slaves are for read. Insert/update/delete on master was wroking fine and is replicating to slaves.

2. What happens to the nodes during a partition?
- When the partion happens the disconnected node was avaialble for read. As slaves are not available for writes in mongo we were having consistency in the data because no new records are getting inserted or updated on the disconnected node.

3. Can stale data be read from a node during a partition?
- Yes, stale data can be read from the node during partition. The new data that is inserted into master after partiton is not getting replicated on the disconnected node but the data before partition was readable.

4. What happens to the system during partition recovery?
- During partiton recovery the system was getting eventually consistent with the updated data from the master. The new documents added to the master are getting replicated on the reconnected node.


# Week 5 (11/04/2018) to (11/10/2018)

**MongoDB Sharding Week**

## Plan

1) Configure MongoDB to support two data shards

2) Design for MongoDB sharding

3) Using MongoDB Bios Collection demonstrate sharding and select a shard key


## Status

**Sharding steps**
Avaialabe at- https://www.linode.com/docs/databases/mongodb/build-database-clusters-with-mongodb/

**Sharding Architecture** - ![MongoDB Sharding Design](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/Shard_Architecture.png)

**Preconfiguration before performing sharding**

1. With your previous mongodb AMI created launch 9 more EC2 instances.
```
Name: Give name respectively for Shards and Config servers
AMI:             Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
Instance Type:   t2.micro
VPC:             cmpe281
Network:         public subnet (us-west-1c)
Auto Public IP:  enable
Security Group:  mongodb-sharding
SG Open Ports:   22, 27017-27019
Key Pair:        cmpe281-us-west-1
```
2. These instances will be used for demonstrating sharding for mongodb
3. 3 of them will be our Configuration servers, 3 will be sharding cluster1 and other 3 will be sharding cluster2. So name them accordingly.
4. In each of your EC2 instances change the host file and set the hostname with private IP of your instances.
sudo vi /etc/hosts  
```
<private IP> node/instance name
like this-
10.0.0.182 mongo-config1
10.0.0.154 mongo-config2
10.0.0.82 mongo-config3
10.0.0.146 mongo-shardA1
10.0.0.118 mongo-shardA2
10.0.0.19 mongo-shardA3
10.0.0.99 mongo-shardB1
10.0.0.74 mongo-shardB2
10.0.0.123 mongo-shardB3
10.0.0.110 mongos-query-router
```

# 1. Setting up Config servers

**Step 1- SSH in one of your config servers to set mongodb user**
```
When we created the AMI we had replication parameter setup in the mongo.conf file. To create a monogdb user we first need to comment that parameter and start mongodb by below commands-
No need of setting replication and sharding parameter when you create a mongodb user.
sudo systemctl enable mongod.service

- Restart MongoDB to apply our changes
	sudo service mongod restart
	sudo service mongod status

	mongo
	use admin
	db.createUser(
		{user: "mongo-admin",
		pwd: "cmpe281",
		roles:[{role: "root", db: "admin"}]
		})
```
**Step 2- SSH in each of your config servers to change mongod.conf**
```
For each of the config server ubuntu instance do the following
change the configuration parameters by
   	sudo vi /etc/mongod.conf

	1) net:
		port: 27019
		bindIp: <private IP of your instance> e.g. 10.0.0.182

	2) replication
		replSetname: configReplica

	3) Sharding
		clusterRole: "configsvr"

Enable mongod service-
	sudo systemctl enable mongos.service

Restart your monogdb instance by -
	sudo systemctl restart mongod
	sudo systemctl status mongod
```
**Step 3- Inintialize replica set on the config serversOn one of your config servers preformNow connect the mongo shell to one of the config server members by specifying the port number**
```
On one of your config servers perform the below steps

- Login into the mongo shell by
mongo mongo-config1:27019 -u mongo-admin -p --authenticationDatabase admin

- enter password that you set earlier on prompt

- Inititalize replica set

rs.initiate(
	{_id: "configReplica",
	configsvr: true,
	members: [
		{ _id: 0, host: "mongo-config1:27019" },
		{ _id: 1, host: "mongo-config2:27019" },
		{ _id: 2, host: "mongo-config3:27019" } ]
	} )

- you will get- configReplica:PRIMARY>
                configReplica:SECONDARY> like this
```
![Config server setup](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/config-server-setup.png)

# 2. Setting up Query Router (Mongos)

**Step 1- Launch a new Public EC2 instance for Mongos**
```
EC2configuration
Name: mongos-query-router
AMI:             Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
Instance Type:   t2.micro
VPC:             cmpe281
Network:         public subnet (us-west-1c)
Auto Public IP:  enable
Security Group:  mongodb-cluster
SG Open Ports:   22, 27017-27019
Key Pair:        cmpe281-us-west-1
```
**Step 2- Configure host file with IP of each instances**
```
change the host file- sudo vi /etc/hosts
10.0.0.182 mongo-config1
10.0.0.154 mongo-config2
10.0.0.82 mongo-config3
10.0.0.146 mongo-shardA1
10.0.0.118 mongo-shardA2
10.0.0.19 mongo-shardA3
10.0.0.99 mongo-shardB1
10.0.0.74 mongo-shardB2
10.0.0.123 mongo-shardB3
10.0.0.110 mongos-query-router
```
**Step 3- Setup Mongodb on Mongos instance**
```
1) Download mongodb
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb.list

sudo apt update
sudo apt install mongodb-org

2) MongoDB Keyfile

openssl rand -base64 741 > keyFile
sudo mkdir -p /opt/mongodb
sudo cp keyFile /opt/mongodb
sudo chown mongodb:mongodb /opt/mongodb/keyFile
sudo chmod 0600 /opt/mongodb/keyFile

3) Config mongos.conf
sudo vi /etc/mongos.conf

		# where to write logging data.
		systemLog:
		  destination: file
		  logAppend: true
		  path: /var/log/mongodb/mongos.log

		# network interfaces
		net:
		  port: 27017
		  bindIp: 10.0.0.110

		security:
		  keyFile: /opt/mongodb/keyFile

		sharding:
  		  configDB: configReplica/mongo-config1:27019,mongo-config2:27019,mongo-config3:27019

No need to do replication here.
We define a configDB parameter with the replica set name of config servers followed by the config serveer's hostname and port number.

4) Create a new systemd file -

	sudo vi /lib/systemd/system/mongos.service

	[Unit]
		Description=Mongo Cluster Router
		After=network.target

	[Service]
		User=mongodb
		Group=mongodb
		ExecStart=/usr/bin/mongos --config /etc/mongos.conf
		# file size
		LimitFSIZE=infinity
		# cpu time
		LimitCPU=infinity
		# virtual memory size
		LimitAS=infinity
		# open files
		LimitNOFILE=64000
		# processes/threads
		LimitNPROC=64000
		# total threads (user+kernel)
		TasksMax=infinity
		TasksAccounting=false

	[Install]
		WantedBy=multi-user.target

7) Make sure that mongod is stopped- This is because the mongos sercice need the data lock. So if mongod service is running then the lock will not be accuried . So stop mongod service before enabling mongos sercice by executing below command
 sudo systemctl stop mongod

8) Enable Mongos Service
sudo systemctl enable mongos.service

9) Restart Mongos to apply our changes
sudo systemctl start mongos
sudo systemctl status mongos
sudo systemctl stop mongos

when successfully run then it will give the output as-
● mongos.service - Mongo Cluster Router
   Loaded: loaded (/lib/systemd/system/mongos.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-11-22 08:05:17 UTC; 3s ago
 Main PID: 15616 (mongos)
    Tasks: 19
   Memory: 11.2M
      CPU: 69ms
   CGroup: /system.slice/mongos.service
           └─15616 /usr/bin/mongos --config /etc/mongos.conf

```
![Mongos setup](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/query-router-setup.png)

# 3. Setting up Sharding servers

**Step 1- SSH in one of your config servers to set mongodb user**
```
When we created the AMI we had replication parameter setup in the mongo.conf file. To create a monogdb user we first need to comment that parameter and stat mongodb by below commands-
No need of setting replication and sharding parameter when you create a mongodb user.

sudo systemctl enable mongod.service

- Restart MongoDB to apply our changes
	sudo service mongod restart
	sudo service mongod status

	mongo
	use admin
	db.createUser(
		{user: "mongo-admin",
		pwd: "cmpe281",
		roles:[{role: "root", db: "admin"}]
		})
```
**Step 2- Shard Replicas setupConfigure mongod.conf in each of your sharding replicas**
```
We have two sharding cluster with 3 EC2 instances in each one of them.

For each shard cluster do the below steps (1-3)- (make note of changing replSetname of instances according to the cluster)

1) change mongod.conf

sudo vi /etc/mongod.conf  
	1) net:
		port: 27017
		bindIp: <private IP of your instance> e.g. 10.0.0.182

	2) replication
		replSetname: shard1   //this name should change according to cluster

	3) Sharding
		clusterRole: shardsvr

2) enable mongod service-
sudo systemctl enable mongod.service

3) Restart monogod
sudo service mongod restart
sudo service mongod status

4) Login to mongo shell

For Shard Cluster 1
mongo mongo-shardA1:27017 -u mongo-admin -p --authenticationDatabase admin
enter password that you set earlier

rs.initiate( {
	_id: "shard1",
	members: [
		{ _id: 0, host: "mongo-shardA1:27017" },
		{ _id: 1, host: "mongo-shardA2:27017" },
		{ _id: 2, host: "mongo-shardA3:27017" } ]
	} )

For Shard Cluster 2

mongo mongo-shardB1:27017 -u mongo-admin -p --authenticationDatabase admin
enter password that you set earlier

rs.initiate( {
	_id: "shard2",
	members: [
		{ _id: 0, host: "mongo-shardB1:27017" },
		{ _id: 1, host: "mongo-shardB2:27017" },
		{ _id: 2, host: "mongo-shardB3:27017" } ]
	} )
```
![Shard1 setup](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/shardCluster1_setup.png)

![Shard2 setup](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/shardCluster2_setup.png)

**Step 3- Adding shards to our clusters - SHARDING STEPS**
```
1) In your query router connect to mongos by-

mongo mongos-query-router:27017 -u mongo-admin -p --authenticationDatabase admin

2) In the mongos shell now add shards

sh.addShard( "shard1/mongo-shardA1:27017,mongo-shardA2:27017,mongo-shardA3:27017" )
sh.addShard( "shard2/mongo-shardB1:27017,mongo-shardB2:27017,mongo-shardB3:27017" )

3) Sharding at Database Level
mongo mongos-query-router:27017 -u mongo-admin -p --authenticationDatabase admin

	use cmpe281DB
	sh.enableSharding("cmpe281DB")

4) Sharding at collection level

	use cmpe281DB

	db.bios.ensureIndex( { _id : "hashed" } )

	sh.shardCollection( "cmpe281DB.bios", { "_id" : "hashed" } )

	insert two document from BIOS collection
	db.bios.insert({
	    "name" : {
		"first" : "Guido",
		"last" : "van Rossum"
	    },
	    "birth" : ISODate("1956-01-31T05:00:00Z"),
	    "contribs" : [
		"Python"
	    ],
	    "awards" : [
		{
		    "award" : "Award for the Advancement of Free Software",
		    "year" : 2001,
		    "by" : "Free Software Foundation"
		},
		{
		    "award" : "NLUUG Award",
		    "year" : 2003,
		    "by" : "NLUUG"
		}
	    ]  })

	    db.bios.insert({
		    "name" : {
			"first" : "Kristen",
			"last" : "Nygaard"
		    },
		    "birth" : ISODate("1926-08-27T04:00:00Z"),
		    "death" : ISODate("2002-08-10T04:00:00Z"),
		    "contribs" : [
			"OOP",
			"Simula"
		    ],
		    "awards" : [
			{
			    "award" : "Rosing Prize",
			    "year" : 1999,
			    "by" : "Norwegian Data Association"
			},
			{
			    "award" : "Turing Award",
			    "year" : 2001,
			    "by" : "ACM"
			},
			{
			    "award" : "IEEE John von Neumann Medal",
			    "year" : 2001,
			    "by" : "IEEE"
			}
		]} )

	//insert more data in your collection and the data will get shard.

Check the shard distribution by-

db.bios.getShardDistribution()

Output will be like-
Shard shard1 at shard1/mongo-shardA1:27017,mongo-shardA2:27017,mongo-shardA3:27017
 data : 508B docs : 1 chunks : 2
 estimated data per chunk : 254B
 estimated docs per chunk : 0

Shard shard2 at shard2/mongo-shardB1:27017,mongo-shardB2:27017,mongo-shardB3:27017
 data : 407B docs : 1 chunks : 2
 estimated data per chunk : 203B
 estimated docs per chunk : 0

Totals
 data : 915B docs : 2 chunks : 4
 Shard shard1 contains 55.51% data, 50% docs in cluster, avg obj size on shard : 508B
 Shard shard2 contains 44.48% data, 50% docs in cluster, avg obj size on shard : 407B
```
 Image for Shard disribution for 10 Bios Collection -
 ![MongoDB Sharidng distributionb for 10 Bios documents](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/ShardDistributionFor_10BiosDocuments.png)

**Detailed screenshots of all the cluster setup is attached in the pdf in MongoDB-Sharding folder on below link-**

https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/MongoDB-Sharding/MongoDB_ShardingSteps.pdf



## Mistakes encountered
1) No such process on starting mongos
```
ubuntu@ip-10-0-0-110:~$ sudo systemctl start mongos
ubuntu@ip-10-0-0-110:~$ systemctl status mongos

● mongos.service - Mongo Cluster Router
   Loaded: loaded (/lib/systemd/system/mongos.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Thu 2018-11-22 07:35:23 UTC; 7s ago
  Process: 15440 ExecStart=/usr/bin/mongos --config /etc/mongos.conf (code=exited, status=217/USER)
 Main PID: 15440 (code=exited, status=217/USER)

Nov 22 07:35:23 ip-10-0-0-110 systemd[1]: Started Mongo Cluster Router.
Nov 22 07:35:23 ip-10-0-0-110 systemd[15440]: mongos.service: Failed at step USER spawning /usr/bin/mongos: No such process
Nov 22 07:35:23 ip-10-0-0-110 systemd[1]: mongos.service: Main process exited, code=exited, status=217/USER
Nov 22 07:35:23 ip-10-0-0-110 systemd[1]: mongos.service: Unit entered failed state.
Nov 22 07:35:23 ip-10-0-0-110 systemd[1]: mongos.service: Failed with result 'exit-code'.
```
Solution- When you set mongos.service, make sure to set ```User as mongodb``` when using Ubuntu AWS AMI and ```mongod``` when usinf centOS AMI

2) Unrecognized option: keyFile
```
ubuntu@ip-10-0-0-110:/opt/mongodb$ sudo systemctl start mongos
ubuntu@ip-10-0-0-110:/opt/mongodb$ systemctl status mongos
● mongos.service - Mongo Cluster Router
   Loaded: loaded (/lib/systemd/system/mongos.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Thu 2018-11-22 08:01:06 UTC; 5s ago
  Process: 15563 ExecStart=/usr/bin/mongos --config /etc/mongos.conf (code=exited, status=2)
 Main PID: 15563 (code=exited, status=2)

Nov 22 08:01:06 ip-10-0-0-110 systemd[1]: Started Mongo Cluster Router.
Nov 22 08:01:06 ip-10-0-0-110 mongos[15563]: Unrecognized option: keyFile
Nov 22 08:01:06 ip-10-0-0-110 mongos[15563]: try '/usr/bin/mongos --help' for more information
Nov 22 08:01:06 ip-10-0-0-110 systemd[1]: mongos.service: Main process exited, code=exited, status=2/INVALIDARGUMENT
Nov 22 08:01:06 ip-10-0-0-110 systemd[1]: mongos.service: Unit entered failed state.
Nov 22 08:01:06 ip-10-0-0-110 systemd[1]: mongos.service: Failed with result 'exit-code'.
```
Solution- Make sure that you have given proper indentation in mongos.conf file. The file must be indented with 2 spacing.

3) Make sure the host file has all the hostnames otherwise mongos will not be able to recognise the congigDB parameters and gives error


# Week 6 (11/11/2018) to (11/17/2018)
**Riak Week**

## Plan
1) To setup Riak cluster on AWS Ec2 isntances.

2) To do replication on the Riak cluster by following the design.

3) Test for AP properties of Riak setup that we have done earlier.

4) Write test cases.

## Status
**Riak Design**
![Riak Design](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/Riak_AP_architecture.png)

**AWS Riak Setup**
Available at- http://docs.basho.com/riak/kv/2.2.3/setup/installing/amazon-web-services/

**Step 1: Launch 5 Private EC2 Free-Tier Instance**
```
Name: Riak-coordinator
AWS Marketplace: Riak KV 2.2 Series
Instance Type:   t2.micro
Number of Instances: 5
VPC:             cmpe281
Network:         private subnet (us-west-1c)
Auto Public IP:  disable
Security Group:  riak-cluster
SG Open Ports:   8098, 8087, 4369, 9080, 22, 6000-7999
Key Pair:        cmpe281-us-west-1
```

**Step 2: Riak Configuration setup**
```
1) On each Riak EC2 instance do the following
- Set min and max port in riak.conf
	cd /etc/riak
	sudo vi riak.conf

	erlang.distribution.port_range.minimum = 6000
	erlang.distribution.port_range.maximum = 7999

- Start Riak cluster
	sudo riak start
	sudo riak-admin cluster status

The output will be like this-
---- Cluster Status ----
Ring ready: true

+---------------------+------+-------+-----+-------+
|        node         |status| avail |ring |pending|
+---------------------+------+-------+-----+-------+
| (C) riak@10.0.1.195 |valid |  up   |100.0|  --   |
+---------------------+------+-------+-----+-------+

- Joining nodes to the cluster
Perform this on all 4 members other than coordinator. Here we are taking the first node as the coordinator node so we will join all other nodes to the first one

	sudo riak-admin cluster join riak@<IP address of coordinator>
	sudo riak-admin cluster join riak@10.0.1.195    // run this command in all other 4 members

The output will be like this-
Success: staged join request for 'riak@10.0.1.202' to 'riak@10.0.1.195'

- Now add all the nodes to the cluster by executing the below command only in the coordinator AWS EC2 instance
	sudo riak-admin cluster plan
	sudo riak-admin cluster commit
	sudo riak-admin cluster status

The output will be like this-
Output after plan-
================================= Membership ==================================
Status     Ring    Pending    Node
-------------------------------------------------------------------------------
valid     100.0%     20.3%    'riak@10.0.1.195'
valid       0.0%     20.3%    'riak@10.0.1.202'
valid       0.0%     20.3%    'riak@10.0.1.225'
valid       0.0%     20.3%    'riak@10.0.1.39'
valid       0.0%     18.8%    'riak@10.0.1.67'
-------------------------------------------------------------------------------
Valid:5 / Leaving:0 / Exiting:0 / Joining:0 / Down:0

Transfers resulting from cluster changes: 51
  12 transfers from 'riak@10.0.1.195' to 'riak@10.0.1.67'
  13 transfers from 'riak@10.0.1.195' to 'riak@10.0.1.39'
  13 transfers from 'riak@10.0.1.195' to 'riak@10.0.1.202'
  13 transfers from 'riak@10.0.1.195' to 'riak@10.0.1.225'

Output after commit-
Cluster changes committed

Output after status-
Ring ready: true
+---------------------+------+-------+-----+-------+
|        node         |status| avail |ring |pending|
+---------------------+------+-------+-----+-------+
| (C) riak@10.0.1.195 |valid |  up   | 23.4|  20.3 |
|     riak@10.0.1.202 |valid |  up   | 18.8|  20.3 |
|     riak@10.0.1.225 |valid |  up   | 20.3|  20.3 |
|     riak@10.0.1.39  |valid |  up   | 20.3|  20.3 |
|     riak@10.0.1.67  |valid |  up   | 17.2|  18.8 |
+---------------------+------+-------+-----+-------+
```

**Step 3: Insert data into Riak cluster**
```
- Insert data into database

	Create a bucket name restaurant
	curl  http://10.0.1.195:8098/buckets/restaurant/keys?keys=true

	Insert data into restaurant bucket
	curl -XPUT http://10.0.1.195:8098/buckets/restaurant/keys/key1?returnbody=true -d '{"Burger King":"San Jose"}'

	Check whether the data is inserted or not
	curl http://10.0.1.195:8098/buckets/restaurant/keys/key1

	Check all the keys
	curl  http://10.0.1.195:8098/buckets/restaurant/keys?keys=true

	Delete a key
	curl -XDELETE http://10.0.1.195:8098/buckets/restaurant/keys/key1

- Accessing data from database
    curl http://10.0.1.195:8098/buckets/restaurant/keys/key1

Always change the hostname according to the node IP. If running on member1 then set the IP of member1 - like this curl http://10.0.1.202:8098/buckets/restaurant/keys/key1
```
![MongoDB EC2 instances](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/Riak_EC2_Instances.png)

## Test Cases for Riak AP testing
Below are the test cases created for the Consistency of MongoDB during Partition Tolerance

## Test 1 : Replication Test

  **Test Plan-** All Member nodes should be able to replicate the data inserted into one node

  **Expected Outcome-** Should be able to query member nodes and be able to read data

  **Actual Outcome-** Able to read the data from all member node showing that replication is working properly.

  **Test Exceution-**
1. Create bucket by
curl  http://10.0.1.195:8098/buckets/restaurant/keys?keys=true

2. Insert data into coordinator
curl -XPUT http://10.0.1.195:8098/buckets/restaurant/keys/key1?returnbody=true -d '{"Five Guys":"Santa Clara"}'

3. To see all keys
	curl  http://10.0.1.195:8098/buckets/restaurant/keys?keys=true

4. On every member node check whether the data is replicated

	curl http://10.0.1.195:8098/buckets/restaurant/keys/key1

	curl http://10.0.1.202:8098/buckets/restaurant/keys/key1

	curl http://10.0.1.39:8098/buckets/restaurant/keys/key1

	curl http://10.0.1.225:8098/buckets/restaurant/keys/key1

	curl http://10.0.1.67:8098/buckets/restaurant/keys/key1

**Test Result-**
![Riak test1 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/Riak_Test1_Result.png)

## Test 2 Testing stale data read after Network Partition

  **Test Plan-** Make a network partition by disconnecting a member node from all other nodes.

  **Expected Outcome-** When a member node is disconnected then that disconnected node will have stale data and we should be able to read it.

  **Actual Outcome-** Able to read stale data from the member disconnected node. Inserted a new data in primary and other secondary nodes got updated but the one which was disconnected gives the stale data to read.

  **Test Exceution-**

1. Go to any of your member node and execute the below commands to disconnect that node from the other nodes (Here I am disconnecting member2)

	```
	DROP connection

	sudo iptables -A INPUT -s 10.0.1.195  -j DROP
	sudo iptables -A INPUT -s 10.0.1.39  -j DROP
	sudo iptables -A INPUT -s 10.0.1.225  -j DROP
	sudo iptables -A INPUT -s 10.0.1.67  -j DROP

	sudo iptables -L  // to list the rules

	Check the status
		sudo riak-admin cluster status
	```

2. Then update some data that you have already inserted in your RIAK cluster

curl -XPUT http://10.0.1.195:8098/buckets/restaurant/keys/key1?returnbody=true -d '{"Five Guys":"Seattle"}'

//I am updating the place to Seattle which was earlier Santa Clara

3. Now check the updaated key1 in all members

	curl http://10.0.1.202:8098/buckets/restaurant/keys/key1   //getting Santa Clara

	curl http://10.0.1.39:8098/buckets/restaurant/keys/key1		//getting Seattle

	curl http://10.0.1.225:8098/buckets/restaurant/keys/key1	//getting Seattle

	curl http://10.0.1.67:8098/buckets/restaurant/keys/key1		//getting Seattle

	For all other connected nodes you will get the updated key1 but for disconnected node you will get old value.

4. Update the key1 again in the disconnected node and we will see later that on partition recovery the key value with the latest updated data is replicated. This shows that even during the partition tolerance the node is avalaile for read and write.

	In the dicsonned node changing the key value again-
	```
	curl -XPUT http://10.0.1.202:8098/buckets/restaurant/keys/key1?returnbody=true -d '{"Five Guys":"New York"}'
	```
5. You can also delete any key with the below command
	```
	curl -XDELETE http://10.0.1.202:8098/buckets/restaurant/keys/key1
	```
**Test Result-**
![Riak test2 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/Riak_Test2_Result.png)

## Test 3: Testing Network Recovery
   **Test Plan-** Reconnect the disconnected member node again and see netowrk recovery.

  **Expected Outcome-** When member node is connected again then the data should be eventually consistent with the key value of the latest updated data.

  **Actual Outcome-** Member node is connected again and the data is getting eventually consistent with the key value of the latest updated data..

  **Test Exceution-**

1. Recover your node by connecting it to all
	```
	sudo iptables -D INPUT -s 10.0.1.195  -j DROP
	sudo iptables -D INPUT -s 10.0.1.225  -j DROP
	sudo iptables -D INPUT -s 10.0.1.39  -j DROP
	sudo iptables -D INPUT -s 10.0.1.67  -j DROP

	Check the status
		sudo riak-admin cluster status
	```
 2. Check for Key1 updated earlier we see that is it now avaialble with latest time stamp

	curl  http://10.0.1.195:8098/buckets/restaurant/keys/key1

	curl  http://10.0.1.202:8098/buckets/restaurant/keys/key1

	curl  http://10.0.1.39:8098/buckets/restaurant/keys/key1

	//Its availble now and we are getting key1 value now

  **Test Result-**
![Riak test3 Result](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Screenshots/Riak_Test3_Result.png)

# Assignment Questions for AP
1. How does the system function during normal mode (i.e. no partition)
- During normal mode the system works file i.e. it is allowing insertion of data and replicating it to all the nodes. In Riak you can insert/update/delete from any node and the data is replicating accross the cluster.

2. What happens to the nodes during a partition?
- When the partion happens the disconnected node was still avaialble for read and write. However, the data that we were reading from that node is stale and inconsistent as during partition, updations on data might have been done.

3. Can stale data be read from a node during a partition?
- Yes, stale data can be read from the node during partition. The node is avalaible for all read/write operations.

4. What happens to the system during partition recovery?
- During partiton recovery the system was getting eventually consistent with the updated data of the latest data. During partition we changed the key value on disconnected member node as well as on the coordinator node. After partition recovery the key value of the latest data is available on all the nodes.

# Week 7 (11/18/2018) to (11/24/2018)
**Kubernetes Week [WOW Factor]**

## Plan
1. To use Amazon EKS for using Kubernetes in cloud and demonstrating repliation in a NoSQL database.
2. Setting up Amazon EKS and decide the design of the system in terms of number of pods and database to use.

## Status
Database used for testing our kubernetes cluster is - Redis

**AmazonEKS setup**

Amazon EKS is integrated with other services of Amazon such as ELB, IAM and VPC. We will be setting up these to configure EKS.

Setting up EKS on Amazon steps is as follows

**Step1 Creating Amazon EKS service role**

1. Open IAM console- https://console.aws.amazon.com/iam/ and login
2. Choose Roles -> Create role
3. Then EKS -> Allows Amazon EKS to manage your clusters on your behalf -> Next: Permissions -> Next: Tags -> Next: Review
4. Enter Role name -> eksServiceRole_281 -> Create Role

**Step2 Creating Policies for the role we created above**
We create this policy for restricting the calls to our kubernetes cluster and later will attach the role with it
1. Open the IAM console at https://console.aws.amazon.com/iam/ and login
2. Choose Policies -> Create policies
3. In the JSON tab add this below JSON
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
            ],
            "Resource": "*"
        }
    ]
}
```
4. Give name to policy- AmazonEKSAdminPolicy_281 -> Create Policy
5. Go to policy again -> Create Policy -> In JSON tab add our Role ARN to be able to access EKS cluster from your host
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::326071200148:role/eksServiceRole_281"
        }
    ]
}
```
6. Give name to policy- AmazonEKSPassRole_281 -> Create Policy

**Step3 Attach policies to role**
1. In you IAM console -> Choose role (eksServiceRole_281)
2. Attach policies- AmazonEKSAdminPolicy_281, AmazonEKSPassRole_281

**Step4 Creating a group to attach to the user**
1. Go to IAM console -> Choose Group -> Create group -> Group name- AWS_EKSGroup_281
2. Attach below policies
	AmazonEKSClusterPolicy
    	AmazonEKSWorkerNodePolicy
    	AmazonEKSServicePolicy
    	AmazonEKS_CNI_Policy
	AmazonEKSAdminPolicy_281
	AmazonEKSPassRole_281

**Step5 Creating User**
1. Go to IAM console -> Choose User -> Add user -> User name- AWS_EKSUser_281
2. Go add Group created previously -> AWS_EKSGroup_281 -> next Tags -> review -> done

**Step6 Creating VPC cluster**
1. Go to AWS CloudFormation console at https://console.aws.amazon.com/cloudformation.
2. Select region- US West (Oregon) (us-west-2) as it supports EKS
3. Choose Create Stack -> Specify an Amazon S3 template URL -> enter this - https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-11-07/amazon-eks-vpc-sample.yaml
4. Specify Details -> then Create
```
Stack Name- CMPE281-EKS-VPC
Rest default
```
5. Wait for your VPC to get created -> then select the VPC -> Go to Output tag
```
Security Group - sg-061411acb61b3ec60
VpcId - vpc-0e69bfbf33bf1046e
SubnetId - subnet-061a96394001c00c0,subnet-09cfadc3366abd9b3,subnet-088ec3226bfbc1684
```
![EKS VPC cluster](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/VPC_Cluster.png)

**Step7 Installing aws-iam-authenticator for EKS**
1. Download the binary from

MacOS: https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-07-26/bin/darwin/amd64/aws-iam-authenticator

2. Adding it to the path
```
chmod +x ./aws-iam-authenticator
cd ~ -> mkdir bin -> mkdir aws-iam-authenticator
cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator
export PATH=$HOME/bin/aws-iam-authenticator:$PATH
echo 'export PATH=$HOME/bin/aws-iam-authenticator:$PATH' >> ~/.bash_profile
aws-iam-authenticator help
```

**Step8 Installing awscli for configuring user and EKS cluster creation**
1. Install aws cli

	```brew install awscli```

2. Configuring aws
```
aws configure
AWS Access Key ID [None]: AKIAJ3OAN73JKDVHORTQ
AWS Secret Access Key [None]: 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
Default region name [None]: us-west-2
Default output format [None]:

Create CLuster-
aws eks create-cluster --name eksCluster --role-arn arn:aws:iam::326071200148:role/eksServiceRole_281 --resources-vpc-config subnetIds=subnet-061a96394001c00c0,subnet-09cfadc3366abd9b3,subnet-088ec3226bfbc1684,securityGroupIds=sg-061411acb61b3ec60

Output-
{
    "cluster": {
        "name": "eksCluster",
        "arn": "arn:aws:eks:us-west-2:326071200148:cluster/eksCluster",
        "createdAt": 1543700693.297,
        "version": "1.10",
        "roleArn": "arn:aws:iam::326071200148:role/eksServiceRole_281",
        "resourcesVpcConfig": {
            "subnetIds": [
                "subnet-061a96394001c00c0",
                "subnet-09cfadc3366abd9b3",
                "subnet-088ec3226bfbc1684"
            ],
            "securityGroupIds": [
                "sg-061411acb61b3ec60"
            ],
            "vpcId": "vpc-0e69bfbf33bf1046e"
        },
        "status": "CREATING",
        "certificateAuthority": {},
        "platformVersion": "eks.2"
    }
}

check status by-
aws eks describe-cluster --name eksCluster --query cluster.status

Output-
Active
```
![EKS Cluster creation](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/eks_cluster.png)

**Step9 Configure kubectl for Amazon EKS**
1. View your aws cli identity
```aws sts get-caller-identity ```

2. Update your kubeconfig ```aws eks update-kubeconfig --name eksCluster```

3. Test your configuration ```kubectl get svc```

![Kubectl configuration](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/kubectl_config.png)

**Step10 Configure Amazon EKS Worker Nodes**
1. Download the pem file from the region that you have already selected for EKS. Here we are using oregon.

1. Go to https://console.aws.amazon.com/cloudformation

2. Select region- US West (Oregon) (us-west-2) as it supports EKS
3. Choose Create Stack -> Specify an Amazon S3 template URL -> enter this - https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-11-07/amazon-eks-nodegroup.yaml
4. Specify Details -> then Create
```
Stack Name- CMPE281-EKS-WorkerNode
Cluster Name- eksCluster   // enter your eks cluster name created earlier
ClusterControlPlaneSecurityGroup - Your VPC security group
NodeGroupName- Give any name (CMPE281-EKS-WorkerNodeGroup)
NodeAutoScalingGroupMinSize-1
NodeAutoScalingGroupMaxSize-3
NodeInstanceType-t2.small
NodeImageId- ami-0f54a2f7d2e9c88b3 (this is specific for oregon)
KeyName- Your key file of the region
VpcId- your VPC id
Subnet- Your subet of the VPC
```
5. Record the NodeInstanceRole for the node group
NodeInstanceRole - arn:aws:iam::326071200148:role/CMPE281-EKS-WorkerNode-NodeInstanceRole-1HYLG1EQZMF0R
NodeSecurityGroup - sg-0ff60ddb9523851a7

6. Enable worker nodes to join your cluster
```
Download AWS authenticator configuration map
curl -O https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-11-07/aws-auth-cm.yaml

Open this file and replace rolearn with the NodeInstanceRole value you get earlier.
```
7. Apply the configuration changes
```kubectl apply -f aws-auth-cm.yaml```
![EKS Worker Node](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/EKS_Worker_Node.png)

**Step11 create your guest book application**
This is used for testing the cluster we created earlier
1. Creating Redis Master Replication Controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/v1.10.3/examples/guestbook-go/redis-master-controller.json
```
2. Creating Redis Master Service.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/v1.10.3/examples/guestbook-go/redis-master-service.json
```

3. Creating Redis Slave Replication controller.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/v1.10.3/examples/guestbook-go/redis-slave-controller.json
```

4. Create Redis Slave Service.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/v1.10.3/examples/guestbook-go/redis-slave-service.json
```

5. Creating Guestbook Replication Controller.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/v1.10.3/examples/guestbook-go/guestbook-controller.json
```

6. Create the guestbook service.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/kubernetes/v1.10.3/examples/guestbook-go/guestbook-service.json
```

7. Query the services in your cluster and wait until the External IP column for the guestbook service is populated.
```
kubectl get services -o wide
```
This will give you External IP for your Guidebook. With the IP you can access the Guestbook on port 3000
http://a0a6ac68df5c311e8bbb006e936d175a-175315281.us-west-2.elb.amazonaws.com:3000

![GuestBook configuration](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/guestbook_configuration.png)

**Detailed steps with screenshot is in the pdf file on below link-**
https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/AmazonEKS_Steps.pdf


**Step12 Test NoSQL database redis for replication**

## Test redis
1. Get your Kubernetes running pods
```
kubectl get pods
kubectl get services -o wide
```
2. Exec redis master shell to write data - take the pod name and exec into it
```
kubectl exec -it redis-master-5mhgm -- /bin/bash
redis-cli

Query
set FiveGuys 20
set BurgerPlace 10
```

3. Exec redis slave shell to read data
```
Slave 1

kubectl exec -it redis-slave-9t2zh  -- /bin/bash
redis-cli
get FiveGuys
get BurgerPlace

Slave 2
kubectl exec -it redis-slave-tgqvn  -- /bin/bash
redis-cli
get FiveGuys
get BurgerPlace
```
4. Exec redis master shell to delete data
```
kubectl exec -it redis-slave-9t2zh  -- /bin/bash
redis-cli
del FiveGuys
```
## Test Results
Redis database is working as expected on the Amazon EKS (Elastic container service for Kubernetes). When we insert data into the master then the slave is replicating the data. We are not allowed to do update opreations on the slave node which preserves data consistency.

![Test Insertion to master](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/RedisMasterInsertion.png)
![Test Reading from Slave (Repliation)](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/RedisSlaveRead.png)
![Test Deleting from Master](https://github.com/nguyensjsu/cmpe281-ManaliJain06/blob/master/Kubernetes-EKS/RedisDeletion.png)

## Mistakes encountered
1) Check the version of aws cli. It should be 1.16 or above. If it's less then try updating your python version to 2.7.
2) While creating Worker Node you should correclty ender the node ID based on region.
