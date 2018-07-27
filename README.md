# Hadoop
*Learning how to tame the Big Data with Hadoop and related technologies*
* Hadoop is an **open source** software platform for **distributed storage** and **distributed processing** of **very large datasets** on **computer clusters** built from commodity hardware
* Why Hadoop?
  * Data is too big
  * Vertical scaling isn't an option
    * Disk seek times
    * Hardware failures
    * Processing times
  * Horizontal scaling is linear
  * You can do much more instead of just batch processing

## Installation
* Download Virtual Box from https://www.virtualbox.org/
* Download image of Hadoop to run on Virtual Box
  * (Horton Works Data Platform) **HDP 2.5 Sandbox** is preffered because it boots up faster than new versions
    * Download from https://hortonworks.com/downloads/#sandbox
* Import the image into Virtual Box
* Once you bootup, you will have CentOS instance that has Hadoop up and running
* We can use CLI, it also has browser interface
  * **Ambari** is available to easily navigate and manage different systems on Hadoop
  * Goto http://localhost:8888
* Launch Dashboard and login to Ambari
  * Username: maria_dev
  * Password: maria_dev
* #### Trouble shooting
  * Enable **virtualization** in your BIOS
  * Disable **Hyper-V acceleration** in Windows
    * this option is in the “Turn Windows Features On and Off” control panel
  * Make sure you have atleast 8+ GB of RAM
  
## Overview of Hadoop Ecosystem
* Core Hadoop Ecosytem could be visualized as:
* <p align="center"><img src="https://i.imgur.com/Dqf8wEz.png"></p>
* For external Datastorage, we have
  * MySQL
  * MongoDB
  * Cassandra
* Query Engines which run on top of Hadoop clusters are:
  * Apache Drill
  * Hue
  * Apache Phoenix
  * Presto
  * Apache Zeppelin
  
## HDFS
* The Hadoop Distributed File System
* It is mostly for handling very large files
  * could be data from sensors, or logs from web servers etc
* It breaks them into many blocks
  * one block is 128 MB by default
* These blocks can be stored across several computers
* HDFS Architecture consists of
  * **Name Node**
    * It maintains logs of different blocks where they reside and their state
  * **Data  Node**
    * It stores each block of data
* For reading a file
  * Client Node talks with Name Node and checks for file location
  * Name Node informs the Client Node of the position of blocks of this file on Data Nodes
  * Then Client Node retrieves data from Data Nodes
* <p align="center"><img src="https://i.imgur.com/q5OBRlF.png"></p>
* For writing a file
  * Client Node talks with Name Node so that it could keep track of this new file
  * Name Node then creates an entry of this file and Client Node writes it to Data Nodes
  * Data Nodes sent acknowledgement to Client Node
  * Client Node then updates Name Node to add entry of this new file
* <p align="center"><img src="https://i.imgur.com/VLQwHXc.png"></p>
* There are multiple Data Nodes so the data can be retrieved even if a single Data Node fails
* But what if a Name Node fails, that would be a single point of failure
  * It creates a backup of Metadata
    * Namenode writes to local disk and NFS
  * There can be a secondary Namenode
    * It contains a merged copy of edit logs to restore from
  * Each Name node manages a specific namespace volume
* HDFS has high availability
  * Hot standby Namenode using shared edit log
  * Zookeper tracks active Namenode
  * Uses extreme measures to ensure only one namenode is used at a time
* We can use HDFS through:
  * UI (Ambari)
  * CLI
  * HTTP/HDFS Proxy
  * Java interface
  * NFS Gateway
* To manipulate files with GUI, we can use **Ambari** through HTTP interface
  * Goto http://localhost:8080
  * Login with maria_dev
  * Click on **HDFS** from different options available
  * Goto grid icon and click on **Files View**
  * It shows you the HDFS thats running on your Hadoop cluster
    * You can now perform operations on the files
      * Upload, rename, make directories, concatenate, download files etc
* To manipulate files using CLI, we need to download Putty Client
  * Download from https://putty.org/
* In the Hostname field, type
  * maria_dev@127.0.0.1
* In the Port field, type
  * 2222 (default for Hortonworks sandbox)
* Select connection type as
  * SSH
* Click on open and type password
  * maria_dev
* You can now write commands to manipulate files on HDFS
* Command syntax is **hadoop fs -[command]**
  * hadoop fs -ls
  * hadoop fs -mkdir abc
  * wget *url*
    * To download files into your Virtual Box
  * hadoop fs -copyFromLocal *source* *Destination*
* To check all the commands, type
  * hadoop fs

## MapReduce
* Distributes the processing of data on your cluster
* Divides your data up into partitions that are **MAPPED** (transformed) and **REDUCED** (aggregated) by mapper and reducer functions you define
* Resilient to failure
  * an **Application Master** monitors your mappers and reducers on each partition
* For example if we are trying to find how many movies did each user rate in a large data set on a cluster
  * If we have *UserID*, *MovieID*, *Rating*, and *Timestamp* data in a file
  * Mapper transforms each line of data into Key Value pairs
  * Then MapReduce sorts and groups the mapped data
    * This step is also called **Shuffle and Sort**
  * Now the Reducer processes each key's values to produce the output we want
    * In this case, we check the length of keys (which is UserID)
      * To get how many movies did this *UserID* rated
* <p align="center"><img src="https://i.imgur.com/V6d2QA4.png"></p>
### Architecture
* A **Client Node** requests for a job
* This request goes to **YARN Resource Manager** (Yet Another Resource Negotiator)
  * It is the core piece of Hadoop that manages what gets run on which machine, what machine is available and their capacity etc
* Client node also copies data into **HDFS** which it needs to perform the job on
  * so that this data is available to different Nodes which will process it later on
* YARN communicates with **Application Master** which comes under **Node Manager**
  * This Application Master is responsible for managing different individual Map and Reduce tasks
  * It works with YARN Resource Manager to distribute these taks to different Nodes across the cluster
* Different Nodes communicate with HDFS to receive the data to perform Map and Reduce tasks
  * And output the processed data to HDFS in the end
* <p align="center"><img src="https://i.imgur.com/uEikvnb.png"></p>
#### How are Mapper and Reducers written?
* Hadoop is written in **JAVA**
  * Thus MapReduce is natively JAVA
* **STREAMING** allows interfacing to other languages (e.g. **Python**)
### Handling Failures
* Since the cluster consists of commodity hardware
  * Any node or computer can go down anytime
* If a working **Node** goes down
  * Application master is monitoring it
  * Restarts the task 
  * Preferably on a different node
* If the **Application Master** goes down
  * YARN can try to restart it on a different node
* What if the **Resource Manager** itself goes down
  * It is difficult to handle
  * We need to setup *high availability* (HA) using **Zookeeper**
    * To have a hot standby
    * Zookeeper can automatically redirect to a second backup resource manager
    * This option is only used if we can't tolerate failure of cluster at any cost
### Example
*How many of each movie rating type exists?*
* We need to find out the count of ratings
  * how many 1,2,3,4 or 5 star ratings have been given
* Download the IMBD movies dataset from https://grouplens.org/
  * Download **MovieLens100k** (contains 100,000 records)
  * We could use bigger dataset but lower will do since we only have 1 virtual machine in a cluster
  * Dataset contains *UserID*, *MovieID*, *Rating* and *Timestamp* columns
* This can be solved with MapReduce approach
  * **MAP** each input line to (rating, 1)
    * this way it forms a key/value pair from each line
      * Key is rating (for example 3 if movie was rated 3 stars)
      * Value is 1 (meaning true - its just to form a key/value pair)
  * **REDUCE** each rating with the sum of all the 1's
    * we need to sum all those different key/values (aggregate them) into a single key/value pair
    * this step is performed after shuffle and sort
* <p align="center"><img src="https://i.imgur.com/uGAqrnB.png"></p>
* Let's write the code in Python
```python
from mrjob.job import MRJob
from mrjob.step import MRStep

class RatingsBreakdown(MRJob):

	def steps(self):
		return [
			MRStep( mapper=self.mapper_get_ratings,
				reducer=self.reducer_count_ratings)
		]
    
	def mapper_get_ratings(self, _, line):
		(userID, movieID, rating, timestamp) = line.split('\t')
		yield rating, 1
    
	def reducer_count_ratings(self, key, values):
		yield key, sum(values)
    
if __name__ == '__main__':
	RatingBreakdown.run()
```
  * **```mrjob```** is a package for writing map-reduce jobs in python very quickly
    * it abstracts away all the complexities to deal with the streaming interfaces
  * each job we will do will be wrapped inside a class
    * just for organizing functions and data together into a single entity
  *  **```steps()```** tells the framework what functions are used for Mapper and Reducers in a job
  * We have a single **```MRStep(...)```** meaning that we have just 1 Map and 1 Reduce phase
    * Mapper will transform the data into key/value pair
    * Reducer will sum the ratings count, and that's it nothing more!
    * here mapper is **```mapper_get_ratings()```**
    * and reducer is **```reducer_count_raints()```**
