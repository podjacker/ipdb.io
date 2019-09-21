<!---
Copyright BigchainDB GmbH and BigchainDB contributors
SPDX-License-Identifier: (Apache-2.0 AND CC-BY-4.0)
Code is Apache-2.0 and docs are CC-BY-4.0
--->

# How to Set Up a BigchainDB Network

Note 1: These instructions will also work for a "network" with only one node.

Note 2: You might not need to set up your own network yet. You should start by creating a proof-of-concept app that writes to [the BigchainDB Testnet](https://testnet.bigchaindb.com/), and if that goes well, then you can look into setting up your own network.

Note 3: If you want to set up a node or network so that you can contribute to developing and testing the BigchainDB code, then see [the docs about contributing to BigchainDB](https://docs.bigchaindb.com/projects/contributing/en/latest/index.html).

<hr>

The process to create a network is both *social* and *technical*: *social* because someone (that we will call **Coordinator**) needs to find at least three other **Members** willing to join the network, and coordinate the effort; *technical* because each member of the network needs to set up a machine running BigchainDB. (Note: a Coordinator is a Member as well.)

A **BigchainDB Network** (or just *Network*) is a set of **4 or more BigchainDB Nodes** (or *Nodes*). Every Node is independently managed by a Member, and runs an instance of the [BigchainDB Server software][bdb:software]. At the **Genesis** of a Network, there **MUST** be at least **4** Nodes ready to connect. After the Genesis, a Network can dynamically add new Nodes or remove old Nodes.

A Network will stop working if more than one third of the Nodes are down or faulty _in any way_. The bigger a Network, the more failures it can handle. A Network of size 4 can tolerate only 1 failure, so if 3 out of 4 Nodes are online, everything will work as expected. Eventually, the Node that was offline will automatically sync with the others.

## Before We Start

This tutorial assumes you have basic knowledge on how to manage a GNU/Linux machine.

**Please note: The commands on this page work on Ubuntu 18.04. Similar commands will work on other versions of Ubuntu, and other recent Debian-like Linux distros, but you may have to change the names of the packages, or install more packages.**

We don't make any assumptions about **where** you run the Node.
You can run BigchainDB Server on a Virtual Machine on the cloud, on a machine in your data center, or even on a Raspberry Pi. Just make sure that your Node is reachable by the other Nodes. Here's a **non-exhaustive list of examples**:

- **good**: all Nodes running in the cloud using public IPs.
- **bad**: some Nodes running in the cloud using public IPs, some Nodes in a private network.
- **good**: all Nodes running in a private network.

The rule of thumb is: if Nodes can ping each other, then you are good to go.

The next sections are labelled with **Member** or **Coordinator**, depending on who should follow the instructions. Remember, a Coordinator is also a Member.

## Member: Set Up a Node

Every Member in the Network **must** set up its own Node. The process consists of installing three components, BigchainDB Server, Tendermint Core, and MongoDB, and configuring the firewall.

**Important note on security: it's up to the Member to harden their system.**

### Install the Required Software

Make sure your system is up to date.

```
sudo apt update
sudo apt full-upgrade
```

#### Install BigchainDB Server

BigchainDB Server requires **Python 3.6+**, so make sure your system has it. Install the required packages:

```
# For Ubuntu 18.04:
sudo apt install -y python3-pip libssl-dev
# Ubuntu 16.04, and other Linux distros, may require other packages or more packages
```

Now install the latest version of BigchainDB. You can find the latest version by going to the [BigchainDB project release history page on PyPI][bdb:pypi]. For example, to install version 2.0.0b3, you would do:

```
# Change 2.0.0b3 to the latest version as explained above:
sudo pip3 install bigchaindb==2.0.0b3
```

Check that you installed the correct version of BigchainDB Server using `bigchaindb --version`.

#### Install (and Start) MongoDB

Install a recent version of MongoDB. BigchainDB Server requires version 3.4 or newer.

```
sudo apt install mongodb
```

If you install MongoDB using the above command (which installs the `mongodb` package), it also configures MongoDB, starts MongoDB (in the background), and installs a MongoDB startup script (so that MongoDB will be started automatically when the machine is restarted).

Note: The `mongodb` package is _not_ the official MongoDB package from MongoDB the company. If you want to install the official MongoDB package, please see [the MongoDB documentation](https://docs.mongodb.com/manual/installation/). Note that installing the official package _doesn't_ also start MongoDB.

#### Install Tendermint

The version of BigchainDB Server described in these docs only works well with Tendermint 0.22.8 (not a higher version number). Install that:

```
sudo apt install -y unzip
wget https://github.com/tendermint/tendermint/releases/download/v0.22.8/tendermint_0.22.8_linux_amd64.zip
unzip tendermint_0.22.8_linux_amd64.zip
rm tendermint_0.22.8_linux_amd64.zip
sudo mv tendermint /usr/local/bin
```

### Set Up the Firewall

Make sure to accept inbound connections on ports `9984`, `9985`, and `26656`. You might also want to add port `22` so that you can continue to access the machine via SSH.

```
sudo ufw allow 22/tcp
sudo ufw allow 9984/tcp
sudo ufw allow 9985/tcp
sudo ufw allow 26656/tcp
sudo ufw enable
```

Some cloud providers, like Microsoft Azure, require you to change "security groups" (virtual firewalls) using their portal or other APIs (such as their CLI).

## Member: Configure BigchainDB Server

To configure BigchainDB Server, run:

```
bigchaindb configure
```

The first question is ``API Server bind? (default `localhost:9984`)``. To expose the API to the public, bind the API Server to `0.0.0.0:9984`. Unless you have specific needs, you can keep the default value for all other questions.

## Member: Generate the Private Key and Node id

A Node is identified by the triplet `<hostname, node_id, public_key>`.

As a Member, it's your duty to create and store securely your private key, and share your `hostname`, `node_id`, and `public_key` with the other members of the network.

To generate all of that, run:

```
tendermint init
```

The `public_key` is stored in the file `.tendermint/config/priv_validator.json`, and it should look like:

```json
{
  "address": "E22D4340E5A92E4A9AD7C62DA62888929B3921E9",
  "pub_key": {
    "type": "tendermint/PubKeyEd25519",
    "value": "P+aweH73Hii8RyCmNWbwPsa9o4inq3I+0fSfprVkZa0="
  },
  "last_height": "0",
  "last_round": "0",
  "last_step": 0,
  "priv_key": {
    "type": "tendermint/PrivKeyEd25519",
    "value": "AHBiZXdZhkVZoPUAiMzClxhl0VvUp7Xl3YT6GvCc93A/5rB4fvceKLxHIKY1ZvA+xr2jiKercj7R9J+mtWRlrQ=="
  }
}
```

To extract your `node_id`, run the command:

```
tendermint show_node_id
```

An example `node_id` is `9b989cd5ac65fec52652a457aed6f5fd200edc22`.

An example hostname is `charlie5.cloudservers.company.com`. You can also use a public IP addres, like `46.145.17.32`, instead of a hostname, but make sure that IP address won't change.

Share the `node_id`, `pub_key.value` and hostname of your Node with all other Members.

**Important note on security: each Member should take extra steps to verify the public keys they receive from the other Members have not been tampered with, e.g. a key signing party would be one way.**

## Coordinator: Initialize the Network

At this point the Coordinator should have received the data from all the Members, and should combine them in the `.tendermint/config/genesis.json` file:

```json
{
   "genesis_time":"0001-01-01T00:00:00Z",
   "chain_id":"test-chain-la6HSr",
   "consensus_params":{
      "block_size_params":{
         "max_bytes":"22020096",
         "max_txs":"10000",
         "max_gas":"-1"
      },
      "tx_size_params":{
         "max_bytes":"10240",
         "max_gas":"-1"
      },
      "block_gossip_params":{
         "block_part_size_bytes":"65536"
      },
      "evidence_params":{
         "max_age":"100000"
      }
   },
   "validators":[
      {
         "pub_key":{
            "type":"AC26791624DE60",
            "value":"<Member 1 public key>"
         },
         "power":10,
         "name":"<Member 1 name>"
      },
      {
         "pub_key":{
            "type":"AC26791624DE60",
            "value":"<Member 2 public key>"
         },
         "power":10,
         "name":"<Member 2 name>"
      },
      {
         "...":{

         },

      },
      {
         "pub_key":{
            "type":"AC26791624DE60",
            "value":"<Member N public key>"
         },
         "power":10,
         "name":"<Member N name>"
      }
   ],
   "app_hash":""
}
```

**Note:** `consensus_params` in the `genesis.json` are default values for Tendermint consensus.

The new `genesis.json` file contains the data that describes the Network. The key `name` is the Member's moniker; it can be any valid string, but put something human-readable like `"Alice's Node Shop"`.

At this point, the Coordinator must share the new `genesis.json` file with all Members.

## Member: Connect to the Other Members

At this point the Member should have received the `genesis.json` file.

**Important note on security: each Member should verify that the `genesis.json` file contains the correct public keys.**

The Member must copy the `genesis.json` file in the local `.tendermint/config` directory. Every Member now shares the same `chain_id`, `genesis_time`, used to identify the Network, and the same list of `validators`.

The Member must edit the `.tendermint/config/config.toml` file and make the following changes:

```
moniker = "Name of our node"
create_empty_blocks = false
log_level = "main:info,state:info,*:error"

persistent_peers = "<Member 1 node id>@<Member 1 hostname>:26656,\
<Member 2 node id>@<Member 2 hostname>:26656,\
<Member N node id>@<Member N hostname>:26656,"

send_rate = 102400000
recv_rate = 102400000

recheck = false
```

## Member: Start MongoDB

If you installed MongoDB using `sudo apt install mongodb`, then MongoDB should already be running in the background. You can check using `systemctl status mongodb`.

If MongoDB isn't running, then you can start it using the command `mongod`, but that will run it in the foreground. If you want to run it in the background (so it will continue running after you logout), you can use `mongod --fork --logpath /var/log/mongodb.log`. (You might have to create the `/var/log` directory if it doesn't already exist.)

If you installed MongoDB using `sudo apt install mongodb`, then a MongoDB startup script should already be installed (so MongoDB will start automatically when the machine is restarted). Otherwise, you should install a startup script for MongoDB.

## Member: Start BigchainDB and Tendermint Using Monit

This section describes how to manage the BigchainDB and Tendermint processes using [Monit][monit], a small open-source utility for managing and monitoring Unix processes. BigchainDB and Tendermint are managed together, because if BigchainDB is stopped (or crashes) and is restarted, *Tendermint won't try reconnecting to it*. (That's not a bug. It's just how Tendermint works.)

Install Monit:

```
sudo apt install monit
```

If you installed the `bigchaindb` Python package as above, you should have the `bigchaindb-monit-config` script in your `PATH` now. Run the script to build a configuration file for Monit:

```
bigchaindb-monit-config
```

Run Monit as a daemon, instructing it to wake up every second to check on processes:

```
monit -d 1
```

Monit will run the BigchainDB and Tendermint processes and restart them when they crash. If the root `bigchaindb_` process crashes, Monit will also restart the Tendermint process.

You can check the status by running `monit status` or `monit summary`.

By default, it will collect program logs into the `~/.bigchaindb-monit/logs` folder.

To learn more about Monit, use `monit -h` (help) or read [the Monit documentation][monit-manual].

Check `bigchaindb-monit-config -h` if you want to arrange a different folder for logs or some of the Monit internal artifacts.

If you want to start and manage the BigchainDB and Tendermint processes yourself, then look inside the file [bigchaindb/pkg/scripts/bigchaindb-monit-config](https://github.com/bigchaindb/bigchaindb/blob/master/pkg/scripts/bigchaindb-monit-config) to see how *it* starts BigchainDB and Tendermint.

## How Others Can Access Your Node

If you followed the above instructions, then your node should be publicly-accessible with BigchainDB Root URL `http://hostname:9984` (where hostname is something like `bdb7.canada.vmsareus.net` or `17.122.200.76`). That is, anyone can interact with your node using the [BigchainDB HTTP API](http-client-server-api.html) exposed at that address. The most common way to do that is to use one of the [BigchainDB Drivers](./drivers-clients/index.html).

## Refreshing Your Node

If you want to refresh your node back to a fresh empty state, then your best bet is to terminate it and deploy a new virtual machine, but if that's not an option, then you can:

- drop the `bigchain` database in MongoDB using `bigchaindb drop` (but that only works if MongoDB is running)
- reset Tendermint using `tendermint unsafe_reset_all`
- delete the directory `$HOME/.tendermint`

## Shutting down BigchainDB

If you want to stop/kill BigchainDB, you can do so by sending `SIGINT`, `SIGQUIT` or `SIGTERM` to the running BigchainDB
process(es). Depending on how you started BigchainDB i.e. foreground or background. e.g. you started BigchainDB in the background as mentioned above in the guide:

```bash
$ nohup bigchaindb start 2>&1 > bigchaindb.log &

$ # Check the PID of the main BigchainDB process
$ ps -ef | grep bigchaindb
<user>    *<pid> <ppid>   <C> <STIME> <tty>        <time> bigchaindb
<user>     <pid> <ppid>*  <C> <STIME> <tty>        <time> gunicorn: master [bigchaindb_gunicorn]
<user>     <pid> <ppid>*  <C> <STIME> <tty>        <time> bigchaindb_ws
<user>     <pid> <ppid>*  <C> <STIME> <tty>        <time> bigchaindb_ws_to_tendermint
<user>     <pid> <ppid>*  <C> <STIME> <tty>        <time> bigchaindb_exchange
<user>     <pid> <ppid>   <C> <STIME> <tty>        <time> gunicorn: worker [bigchaindb_gunicorn]
<user>     <pid> <ppid>   <C> <STIME> <tty>        <time> gunicorn: worker [bigchaindb_gunicorn]
<user>     <pid> <ppid>   <C> <STIME> <tty>        <time> gunicorn: worker [bigchaindb_gunicorn]
<user>     <pid> <ppid>   <C> <STIME> <tty>        <time> gunicorn: worker [bigchaindb_gunicorn]
<user>     <pid> <ppid>   <C> <STIME> <tty>        <time> gunicorn: worker [bigchaindb_gunicorn]
...

$ # Send any of the above mentioned signals to the parent/root process(marked with `*` for clarity)
# Sending SIGINT
$ kill -2 <bigchaindb_parent_pid>

$ # OR

# Sending SIGTERM
$ kill -15 <bigchaindb_parent_pid>

$ # OR

# Sending SIGQUIT
$ kill -3 <bigchaindb_parent_pid>

# If you want to kill all the processes by name yourself
$ pgrep bigchaindb | xargs kill -9
```

If you started BigchainDB in the foreground, a `Ctrl + C` or `Ctrl + Z` would shut down BigchainDB.

## Member: Dynamically Add a New Member to the Network

TBD.

## Log rotation

Please go over the [document describing log rotation of a BigchainDB node](../appendices/log-rotation.md).


[bdb:software]: https://github.com/bigchaindb/bigchaindb/
[bdb:pypi]: https://pypi.org/project/BigchainDB/#history
[tendermint:releases]: https://github.com/tendermint/tendermint/releases
[monit]: https://www.mmonit.com/monit
[monit-manual]: https://mmonit.com/monit/documentation/monit.html
