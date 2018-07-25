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
