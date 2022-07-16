# Farmer
> A lightweight, general purpose solution for working with server farms. 

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

> ***Farmer*** is designed to allow you to start small and build out the environment to meet your needs.

## A simple installation
The minimum requirement is a Linux based environment running Bash V4.0 or above with ssh access to the other servers in the farm.  The Windows Subsystem for Linux works fine.

1. Download the Farm software to a location on your workstation, for example /opt/Farmer.
2. Add the following to your .bashrc file.
```
export FARM_HOME=/opt/Farmer
export FARM_CONFIG=$FARM_HOME/config/config.yaml
. $FARM_HOME/Shell/Commands
```
3. Update the config.yaml file with the names and group information for your farm.  See the **An example configuration** section below for a simple starting configuration.
4. Execute ```source ~./bashrc``` to load the farm configuration.
5. You can test that it is working with a few commands.  
```
# Print out the configuration
ShowConfig

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

## An flexible configuration
The examples below are based around the following configuration.

```
.-------------.      .--------------------.      .-----------------.
| Workstation | ---> | Application server | ---> | Database server |
'-------------'  |   '--------------------'  |   '-----------------'
                 |                           |
                 |   .--------------------.  |
                 |-> | Application server | -|
                 |   '--------------------'  
                 |                           
                 |   .--------------.  
                 \-> | Other server | 
                     '--------------'
```

The configuration consists of a Workstation and four servers.  Three of the servers are being used to develop a web based application and the other is an unused spare. 

A set of overlapping server groups are defined in this configuration:
* **MyApplication** - contains all servers being used for the web based application development, being the workstation and servers 1 through 3.
* **Sandpit** - The development sandpit on the workstation only.
* **AppSvr** - The web application servers, servers 1 and 2.
* **DB** - The database server, server3.
* **Unused** - The server4 is a spare server.

**config.yaml**
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
        - MyApplication
- server:
      name: server1
      group:
        - AppSvr
        - MyApplication
- server:
      name: server2
      group:
        - AppSvr
        - MyApplication
- server:
      name: server3
      group:
        - DB
        - MyApplication
- server:
      name: server4
      group:
        - Unused
```

After making changes you will need to reload the configuration file using the ```ParseConfig``` command.  The ```ShowConfig``` command will display the current loaded configuration.  

The configuration contains a couple of additional tags that describe the Farm network configuration.
* **network** sets the network interface used to connect to a server.  **network** is used when checking if the remote server is available.
* **transport** sets the transport used to connect to the server.  The default transport is **ssh**.  The **local** transport creates a bash shell on the local server. 

The **network** and **transport** tags can be set globally at the top of the configuration file, or be set individually for a server.  In the example configuration, the default method of connection is via the ssh command over the eth0 interface.  The workstation is configured to use a local bash shell.

## Kicking the tires
Now that you have a working installation you can run a few commands.  We have already used a few of them during the installation process.

There are four basic commands for operating with the server farm

**Farm.OnAll** runs a command on all servers in a server group.  To check *nginx* is running you could use 
> Farm.OnAll -g AppSvr ps -ax | grep nginx

**Farm.OnOne** runs a command on one server in a server group.  To connect to *postgres* on the database server you could use 
> Farm.OnOne -g DB -m SESSION psql

**Farm.ForAll** runs a command on the local server, substituting SERVER in the command with each of the server names in a server group.  This is useful when you want to repeat a command locally for each server in a group.  For example, to copy updated web content to the *nginx* servers you could use 
> Farm.ForAll -g AppSvr rcp -R https SERVER:/var/www/html

**Farm.ForOne** runs a command on the local server, substituting SERVER in the command with one of the server names in a server group.  

The previous examples all execute a single command on the remote server.  To execute more than one command you can pass a script.  

For example, with following **updateConfig** script:
```
# ========================================================
# updateNginxConfig
# ========================================================
# Update the nginx configuration and restart the server

echo -e "${GREEN}=== SERVER ===${NC}"
rcp ~/Build/nginx.conf SERVER:/etc/nginx
ssh SERVER sudo service nginx restart
```
you can load a new configuration with the command 
> Farm.ForAll -g AppSvr -f updateNginxConfig

Similarly, with:
```
# ========================================================
# getserverinfo
# ========================================================
# Report the characteristics for a server
#

echo ========================================================
echo "Session         : "`whoami`@`hostname`
grep Model /proc/cpuinfo | uniq
grep "model name" /proc/cpuinfo | uniq
grep -i bogomips /proc/cpuinfo | uniq
grep MemFree /proc/meminfo
grep MemAvailable /proc/meminfo
grep -i swap /proc/meminfo
```
you can check the configuration of all servers in the farm with the command 
> Farm.OnAll -a -f getServerInfo

## Getting under the hood
### Syntax
All **Farm.xxx** commands have the same syntax

```
[Context] Farm.xxxx [Switches] [Commands]

 Context:
  SERVER_GROUP=<server group name>
  SERVER_LIST=<list of server names>

 Switches:
  -h | -a | -g <group> | -l <list of servers> | -m COMMAND|SESSION

 Commands:
  <list of commands>
  -f <list of files>
```

| Switch | Purpose            | Context            |
| :----: | :----------------- | :----------------- |
| -h | Prints the help message. |  |
| -a | Selects all servers in the farm. | SERVER_GROUP=\<farmname\>, SERVER_LIST=\<all servers\> |
| -g | Selects all servers in a group. | SERVER_LIST=\<groupList\> |
| -l | Selects an arbitrary list of servers. | SERVER_GROUP=\"\", SERVER_LIST=\<list of servers\> |
| -f | Reads commands from \<list of files\> | |
| -m | Execute a COMMAND, or start an interactive SESSION. | |

### Working with the Context
You can select which servers to operate on either by using the Switches or through the Context environment variables.

For example, ```Farm.ForAll -l "server1 server2" uptime``` is the equivalent of ```SERVER_LIST="server1 server2" Farm.ForAll uptime```.

All commands leave the Context environment variables set.  This enables you to chain a sequence of commands together.  For example, you could have refreshed the nginx configuration with:
```
Farm.ForAll -g AppSvr rcp ~/Build/nginx.conf SERVER:/etc/nginx
Farm.OnAll sudo service nginx restart
```

There are a few additional commands you can use to work with the Context.
* ***SetContext [Switches]*** sets the context variables using the Switches above.
* ***ShowContext*** Displays the current context variables
* ***ClearContext*** Unsets the current context variables.

The **Farm.GetActiveServers** and **Farm.SelectActiveServer** commands update the context using only the servers in the list that are currently online.

The full list of context variables are:
| Context | Purpose |
| :------ | :------ |
| COMMAND | The command to be executed |
| SERVER | The selected server. |
| SERVER_GROUP | The name of the group of servers that have been selected. |
| SERVER_LIST | The list of servers selected. |
| TRANSPORT | The default transport |
| NETWORK | The default network |
| TIMEOUT | Timeout used when checking if the server is online |
| FUNCTION | The name of the function that set the context |
| ACTIVE_SERVERS | The list of currently active servers |
| ACTIVE_SERVER_COUNT | How many active servers are there |
| SELECTION_ALGORITHM | Algorithm used to select the server from the ACTIVE_SERVERS list.  Currently only support RANDOM selection. |

### Extending the CLI
***Farmer*** commands can be used to create additional farm aware commands.  A number of these have been addded to the $FARM_HOME/Shell/Commands script.

For example, the **fuptime** command used to check the intial configuration of the farm is defined as:
```
function fuptime ()
{
  FUNCTION=$FUNCNAME
  SetContext "$@" || return $?

  Farm.OnAll uptime
}
```

A set of custom farm commands can be constructed using this approach.  

For example, the previous examples for updating the Nginx configuration used a fixed location to source the file.  A more robust command might be:

```
function fupdateNginxConfig ()
{
  FUNCTION=$FUNCNAME
  SetContext "$@" || return $?

  if [ -z "$COMMAND" ]
  then
    echo -e "${RED}Error: No config file selected${NC}" >&2
    echo -e "${GREEN}Usage: fupdateNginxConfig [switches] configfile ${NC}" >&2
    return 1
  fi
  
  Farm.ForAll rcp $COMMAND SERVER:/etc/nginx
  Farm.OnAll sudo service nginx restart
}
```

This allows you to push an arbitrary configuration file with the command ```fupdateNginxConfig -g AppSvr ./nginx.conf```.

## Command reference
The full list of ***Farmer*** commands is

| Command | Description |
| ------- | :---------- |
| | |

## To Do
* Nice to be able to run this under Windows.  Currently running this from WSL.
* Code does not conform to the Open/Closed principle - how to do this with Bash?
* Commands.InPath.  Check if the required commands are in the PATH, eg ifconfig, ...
* ShowServers() generates a list of all servers in a particular group.
* ShowGroups() generates a list of groups configured on the farm.
* The ability to set the remote username for a server in the configuration, or at the command line.
* Transports for cloud CLI's (eg AWS, Azure, Google).  Should allow you to connect to either a VM or Container as if it is a standalone server.
* No reason to have -s <server>.  You can use -l <server> to achieve the same effect.
* -A select from Active servers in a group.  Load balancing?
    
2. Leverages **standard distributed computing tools** so that you can configure your environment as you need it.  The minimum requirement is ssh access to other members of the server farm, under the local username.  The Networked sharing of resources is used to provide seamless integration for remotely running processes.
