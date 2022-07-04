# Farmer
> A lightweight, general purpose solution for working with server farms.

## Introduction
A large number of solutions exist for working with large collections of servers.  These generally address specific needs such as clustered application deployment (application servers), workload management (containerisation, batch processing) or high performance computing (Hadoop, MPI).  

***Farmer*** provides a simplified command line solution for working with groups of servers.  This can be useful in any situation where you need to treat a small group of servers as a single distributed computer solution.  For example:  
* Developing, testing or supporting large applications which do not fit within the footprint of a single workstation.
* Offloading long running tasks such as application builds, or tasks requiring large datasets to a more suitable server.
* Providing shared access to specialised infrastructure.

In my case I am using a cluster of Raspberry Pi servers as an experimentation workbench.

***Farmer*** has been designed to allow you to start small and build out the environment to meet your needs.  ***Farmer***:  

1. Provides a **simple installation** that makes it easy to kick the tires.  
2. Leverages **standard distributed computing tools**.  The minimum requirement is ssh access to other members of the server farm, under the local username.  The Networked sharing of resources is used to provide seamless integration for remotely running processes.
3. Has a **flexible configuration** that supports grouping and classification of servers for different purposes.
4. Has an **extensible command line interface** that allows you to construct your own distributed commands.

## Dependencies
* Requires Bash V4 or above.

## Installation
1. Download the Farm software to a location on your workstation.
2. Add the following to your .bashrc file.
```
export FARM_HOME=<location of the software>
export FARM_CONFIG=$FARM_HOME/config/config.yaml
. $FARM_HOME/Shell/Commands
```
3. Update the config.yaml file with the names and group information for your farm.
4. Configure SSH access to the servers on your farm.
5. Logout and back in.

$FARM_HOME/Shell/Commands contains a few pre-defined commands that show different ways to make use of this software.  Additional commands can be configured here.

## Configuring commands

### Setting Context

### Using Farmer Primitives

### Defining new commands

## To Do
* Nice to be able to run this under Windows.  Currently running this from WSL.
* Code does not conform to the Open/Closed principle - how to do this with Bash?
