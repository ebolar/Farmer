# Farmer
> A lightweight, general purpose solution for working with server farms.

## Introduction
Many solutions exist for working with large collections of servers.  These address specific needs such as clustered application deployment (application servers), workload management (containerisation, batch processing) or high performance computing (Hadoop, MPI).  

A server farm is a small group of servers that operate as a single distributed computer solution.  Server farms can be useful in a variety of ways such as:  
* Developing, testing or supporting large applications which do not fit within the footprint of a single workstation.
* Offloading long running tasks such as application builds, or tasks requiring large datasets to more suitable servers.
* Providing shared access to specialised infrastructure.

***Farmer*** provides a simplified command line solution for working with server farms.  It is designed to allow you to start small and build out the environment to meet your needs.

```
 -------------
| Workstation | 
 -------------    ----------------
       |-------> | Server group 1 |
       |          ----------------
       .
       |          ----------------
       |-------> | Server group n |
                  ----------------
```

## Kicking the tires
***Farmer*** provides a **simple installation** that makes it easy to kick the tires.  

The minimum requirement is a Linux based environment running Bash V4.0 or above with ssh access to the other servers in the farm.  The Windows Subsystem for Linux works fine.

### Installation
1. Download the Farm software to a location on your workstation, for example /opt/Farmer.
2. Add the following to your .bashrc file.
```
export FARM_HOME=<location of the software>
export FARM_CONFIG=$FARM_HOME/config/config.yaml
. $FARM_HOME/Shell/Commands
```
3. Update the config.yaml file with the names and group information for your farm.
4. Logout and back in.

You can test that it is working with a few commands.  
```
# Parse and then print out the configuration
$ TestConfiguration

# List the names of all servers in the farm
$ Farm.ForAll -a echo SERVER

# Check that all servers are network accessible
$ Farm.ForAll -a -f $FARM_HOME/Shell/showServer
```



5. Configure SSH access to the servers on your farm.
```
$ ssh-keygen
$ 
```

$FARM_HOME/Shell/Commands contains a few pre-defined commands that show different ways to make use of this software.  Additional commands can be configured here.

## Configuring commands

### Setting Context

### Using Farmer Primitives

### Defining new commands

## To Do
* Nice to be able to run this under Windows.  Currently running this from WSL.
* Code does not conform to the Open/Closed principle - how to do this with Bash?

***Farmer*** has been designed to allow you to start small and build out the environment to meet your needs.  ***Farmer***:  

1. Provides a **simple installation** that makes it easy to kick the tires.  
2. Leverages **standard distributed computing tools**.  The minimum requirement is ssh access to other members of the server farm, under the local username.  The Networked sharing of resources is used to provide seamless integration for remotely running processes.
3. Has a **flexible configuration** that supports grouping and classification of servers for different purposes.
4. Has an **extensible command line interface** that allows you to construct your own distributed commands.

