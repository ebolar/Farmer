# Farmer
> A simple, lightweight, general purpose solution for working with server farms.

## Introduction
A large number of solutions exist for working with large collections of servers.  These generally address specific needs such as clustered application deployment (application servers), workload management (containerisation, batch processing) or high performance computing (Hadoop, MPI).  

***Farmer*** provides a simplified command line solution for working with small groups of servers.  This can be useful when: 
* Developing, testing or supporting large applications which do not fit within the footprint of a single workstation.
* Offloading long running tasks such as application builds, or tasks requiring large datasets to a more suitable server.
* Providing shared access to specialised infrastructure.
* Or any other situation where you need to treat a small group of servers as a single distributed computer solution.  

In my case I am using a cluster of Raspberry Pi servers as an experimentation workbench.

***Farmer*** has been designed with a number of considerations in mind.

1. **Simple Agent-less installation.** ***Farmer*** should be installed on the master node only.  
2. **Leverage standard distributed computing tools.**   ***Farmer*** makes use of ssh for remote access.
3. **Configurable.**  The server farm is configured through a yaml file. 
4. **Extensible command line interface.**  It should be possible to construct your own commands using ***Farmer***.
5. **Good CS practices.** The code should be readable, understandable and maintainable.  The code includes automated tests.  It should be relatively easy to extend the tool, for example to leverage cloud CLI's.

## Dependencies
* Requires Bash V4 or above.
* A working **ssh** connection to other members of the server farm, under the local username.  

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

## To Do
