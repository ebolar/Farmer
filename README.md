# Farmer
> 
> A lightweight, general purpose solution for working with server farms.
> 

## Introduction
Many solutions exist for working with large collections of servers.  These solutions typically address specific needs such as clustered application deployment (application servers), workload management (containerisation, batch processing) or high performance computing (Hadoop, MPI).  

At the other end of the spectrum there are many situations where you need to operate a small collection of servers as a single distributed computer solution.  These are sometimes referred to as server farms.  Server farms can be encountered in a number of situations:  
* Developing, testing or supporting distributed applications which do not fit within the footprint of a single workstation.  For example full stack development of web or mobile applications, or development and testing of HPC applications.
* Offloading long running tasks such as application builds, or tasks requiring large datasets to more suitable servers.  Offloading tasks to specialised infrastructure is another example.
* Building thin client solutions where applications are run remotely from the workstation.

Software that makes it easy to work with groups of servers is under represented.  ***Farmer*** fills that gap by providing a simplified command line solution for working with server farms.  It has:
* a **simple installation** that makes it easy to **kick the tires**.
* an **extensible command line interface** that allows you to directly use Farm primitives, or to construct your own distributed commands.
* a **flexible configuration** that allows you to group servers by function.

> 
> ***Farmer*** is designed to allow you to start small and build out the environment to meet your needs.
> 

## A simple installation
The minimum requirement is a Linux based environment running Bash V4.0 or above with ssh access to the other servers in the farm.  The Windows Subsystem for Linux works fine.

1. Download the Farm software to a location on your workstation, for example /opt/Farmer.
2. Add the following to your .bashrc file.
```
export FARM_HOME=/opt/Farmer
export FARM_CONFIG=$FARM_HOME/config/config.yaml
. $FARM_HOME/Shell/Commands
```
3. Update the config.yaml file with the names and group information for your farm.  See the **kick the tires** section below if you need some guidance on how to do this.
4. Execute ```source ~./bashrc``` to load the farm configuration.
5. You can test that it is working with a few commands.  
```
# Parse and then print out the configuration
TestConfiguration

# List the names of all servers in the farm
Farm.ForAll -a echo SERVER

# Check that all servers are network accessible
Farm.ForAll -a -f $FARM_HOME/Shell/showServer
```
6. Finally configure SSH access to the servers on your farm.  ***Farmer*** currently assumes you will login under the same username on all servers.  Create the accounts on all servers and then set up SSH as per your security policies.  For example, to configure login using ECC based public keys:
```
# Generate a key pair
ssh-keygen -t ecdsa -b 521

# Copy it to all servers in the farm.  
Farm.ForAll -a ssh-copy-id -i .ssh/id_ecdsa SERVER
```
6. Test out access to the farm.  The following command should run uptime on all servers on the farm.
```
fuptime -a
```

## Kicking the tires
Now that you have a working installation you can run a few commands.  We have already used a few of them during the installation process.

The examples below assume the following configuration, typical for a web based application.

```
.-------------.      .--------------------.      .-----------------.
| Workstation | ---> | Application server | ---> | Database server |
'-------------'  |   '--------------------'  |   '-----------------'
                 |                           |
                 |   .--------------------.  |
                 \-> | Application server | -/
                     '--------------------'
```

```
farm:
  name: Example
  description: A simple example farm
  network: eth0
  transport: ssh
  server-list:
  - server:
      name: localhost
      network: lo
      transport: local
      group:
        - Sandpit
- server:
      name: server1
      group:
        - MyApplication
- server:
      name: server2
      group:
        - MyApplication
- server:
      name: server3
      group:
        - DB
```

There are four basic commands for operating with the server farm

**Farm.OnAll** runs a command on all servers in a server group.  
To check *nginx* is running you could use ```Farm.OnAll -g MyApplication ps -ax | grep nginx```.

**Farm.OnOne** runs a command on one server in a server group.  
To connect to *postgres* on the database server you could use ```Farm.OnOne -g DB -m SESSION psql```.

**Farm.ForAll** runs a command on the local server, substituting SERVER in the command with each of the server names in a server group.
This is useful when you want to repeat a command locally for each server in a group.  For ee to copy updated static web content 

**Farm.ForOne** runs a command on the local server, substituting SERVER in the command with one of the server names in a server group.









Usage: [Context] Farm.ForOne [Switches] [Commands]

Context:
  FUNCTION=<function name>
  SERVER=<server name>
  SERVER_GROUP=<server group name>
  SERVER_LIST=<list of server names>

Switches:
  -h | -a | -s <server name> | -g <group> | -l <list of servers>

Commands:
  <list of commands>
  -f <list of files>

Notes:
  -h prints this help message
  -a sets SERVER_GROUP=<farmname> and SERVER_LIST=<all servers>
  -g sets SERVER_LIST=<groupList>
  -l sets SERVER_GROUP="" and SERVER_LIST=<list of servers>
  -s sets SERVER_GROUP="" and SERVER_LIST=<server name>
  -f read commands from <list of files>
  NETWORK and TRANSPORT are derived from the configuration

$FARM_HOME/Shell/Commands contains a few pre-defined commands that show different ways to make use of this software.  Additional commands can be configured here.

## Customising your environment

### Setting Context
### Using Farmer Primitives
### Defining new commands

## To Do
* Nice to be able to run this under Windows.  Currently running this from WSL.
* Code does not conform to the Open/Closed principle - how to do this with Bash?
* Commands.InPath.  Check if the required commands are in the PATH, eg ifconfig, ...
* ShowServers() generates a list of all servers in a particular group.
* ShowGroups() generates a list of groups configured on the farm.
* The ability to set the remote username for a server in the configuration, or at the command line.
* Transports for cloud CLI's (eg AWS, Azure, Google).  Should allow you to connect to either a VM or Container as if it is a standalone server.

2. Leverages **standard distributed computing tools** so that you can configure your environment as you need it.  The minimum requirement is ssh access to other members of the server farm, under the local username.  The Networked sharing of resources is used to provide seamless integration for remotely running processes.
