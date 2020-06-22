# Dynamic Validators Procedure 

## Alastria related projects

### Ethereum install

The generic ethereum tools are needed to, for example, generate the nodekey.

```bash
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get install ethereum
```

### Istanbul-tools

`istanbul-tools` contains tools for configuring Istanbul BFT (IBFT) network, integration tests for both IBFT Geth and Quorum, and load testing utilities for IBFT Geth.

**Tool to generate the extra data**

We are using this tool to generate the genesis's *extradata* field, needed to 

// Not working: Github Route: [alastria/istanbul-tools](https://github.com/alastria/istanbul-tools)

#### Install and setup

##### Go setup (Must be 1.10)

```bash
cd ~/projects
mkdir GO
sudo apt-get update
sudo apt-get -y upgrade
wget https://dl.google.com/go/go1.10.linux-amd64.tar.gz
sudo tar -xvf go1.10.linux-amd64.tar.gz
sudo mv go /usr/local
sudo rm go1.10.linux-amd64.tar.gz
export GOROOT=/usr/local/go
export GOPATH=$HOME/projects/GO
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

**Optional**: If you want to keep these variables, add the 3 exports at the end of your `~/.profile`.

##### Istanbul tools install

```bash
mkdir -p GO/src/github.com/getamis
cd GO/src/github.com/getamis
# We need another version (local copy)
git clone https://github.com/alastria/istanbul-tools.git 
cd istambul-tools
make istanbul
```

The binary file is located inside ***build/bin*** folder.

If you choose another path to place the istanbul-tools folder, it might not work properly.

### Quorum

Alastria's T network node's main source code is located here. We have a base modification that basically adds another list of validators, the validator's pool. So we end having two different array of validators.

- Github route: [alastria/quorum](https://github.com/alastria/quorum)

- Dynamic Validators (branch): [alastria/quorum/tree/feature/dynamic_validators](https://github.com/alastria/quorum/tree/feature/dynamic_validators)

#### Install and setup

```bash
cd ~/projects
git https://github.com/alastria/quorum.git
git branch origin/feature/dynamic_validators
```

Or you can only clone one branch

```bash
git clone --single-branch --branch feature/dynamic_validators \ https://github.com/alastria/quorum.git
cd quorum
make
export GETHDIR=`pwd`; export PATH=$GETHDIR/build/bin:$PATH
```

- Note: if you get this error: "`github.com/ethereum/go-ethereum/crypto/bn256/cloudflare.gfpMul: relocation target runtime.support_bmi2 not defined`" **check GO installed version**, should be the **1.10**.

## Netstats (**Optional** but recommended)

To check the status of the local network with a graphical interface listening in `http://localhost:3000`:

```bash
# Alastria netstats cloning
git clone https://github.com/alastria/eth-netstats.git
cd eth-netstats
# Installing
npm install
npm install -g grunt-cli
grunt
# Start with secret "secret"
export WS_SECRET=secret
npm start
```

## Project file structure setup

Here we define a common project file structure to follow.

- Local test network path: `export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/`. Here will be all network related files.

- Istanbul tools path: `export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin`

- The Quorum folder that was previously created from "dynamic validator's" branch is located in `~/projects/quorum`

- (**IMPORTANT**) Every time you open a terminal window, it uses the default geth client from ethereum, so you have to export the variables for each terminal window or the Quorum related stuff won't work:

  ```bash
  cd quorum
  export GETHDIR=`pwd`; export PATH=$GETHDIR/build/bin:$PATH
  # OR
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
  # Replace with your paths
  export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
  export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
  ```

### Genesis

1. Copy Alastria **genesis.json** file from [https://github.com/alastria/alastria-node/tree/testnet2/data](https://github.com/alastria/alastria-node/tree/testnet2/data) to the projects path `$LOCAL_NET_PATH/genesis.json`
2. Change the **"chainId" to 30** in the genesis.json
3. Leve the **ExtraData** field **empty**

## Local Test Network - First Validator (Option 1: using **istanbul setup**)

The first validator node has to be configured in a different way than the others. That is because we need to validate that It is a correct/valid validator node, but we can't use another validator to vote It in, so we need to use the genesis file. For that, we have to use the istanbul tools to generate the *extradata* field of the genesis file.

### Generate a setup file with one node

```bash
# Change to Istanbul tools folder
cd $ISTANBUL_PATH
# Generate al the validators needed info
./istanbul setup --num 1 --nodes --verbose
# Sample output:
> validators
{
        "Address": "0x488cb1906e19ac1d603a05156690ea044608d108",
        "Nodekey": "76225f22f3f1746669632a7ed0cbf58c7896d3615c51c7a20d424eeccddf2213",
        "NodeInfo": "enode://5f7e13af03343b1fbb149cb50e9082cdf0b9b28924c5a2ed6e078597d9622436324b2324386e485ecca22be7b54de4d320bb296839972aed3d2c7acaf127f059@0.0.0.0:30303?discport=0"
}

static-nodes.json
[
        "enode://5f7e13af03343b1fbb149cb50e9082cdf0b9b28924c5a2ed6e078597d9622436324b2324386e485ecca22be7b54de4d320bb296839972aed3d2c7acaf127f059@0.0.0.0:30303?discport=0"
]

genesis.json
{
    [ ............... ]
    "timestamp": "0x5edf3593",
    "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000f870d594488cb1906e19ac1d603a05156690ea044608d108d594488cb1906e19ac1d603a05156690ea044608d108b8410000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c0",
    "gasLimit": "0x47b760",
    [ ............... ]
}
```

Now we have all the information needed to initialize and start the first validator node. Important generated data to be aware of:

- "Nodekey": is the most important data from witch all the other are generated from. It should be stored in the validators folder as "nodekey".
- "NodeInfo": this is the enode generated from the nodekey and use to connect and identify nodes within the p2p network.
- "Address": address that identifies the validator node within the consensus/validation mechanism.

### Change ExtraData field in Genesis file

We need to copy the generated ***extraData*** field to the previously generated genesis.json without extraData.

### Initialize the validator

```bash
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Export the validator number
export VAL_NUM=00
# Create validator directory
mkdir validator$VAL_NUM
# Create the nodekey (use the generated from the istanbul tools)
nano validator$VAL_NUM/nodekey
	> 76225f22f3f1746669632a7ed0cbf58c7896d3615c51c7a20d424eeccddf2213
# Initialize
geth --datadir validator$VAL_NUM init genesis.json
```

### Run First Validator Node

Now that we have all set and ready, we can start the validator and the network:

```bash
geth --datadir validator$VAL_NUM --networkid 30 --identity VAL_Local_Telsius_2_4_$VAL_NUM --permissioned --rpc --rpcaddr 127.0.0.1 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul --rpcport 220$VAL_NUM --port 210$VAL_NUM --istanbul.requesttimeout 10000 --ethstats VAL_Local_Telsius_2_4_$VAL_NUM:secret@localhost:3000 --verbosity 3 --vmdebug --emitcheckpoints --targetgaslimit 18446744073709551615 --syncmode full --gcmode full --vmodule consensus/istanbul/core/core.go=5 --nodiscover --maxpeers 100 --mine --minerthreads $(grep -c "processor" /proc/cpuinfo)
```

All should work smoothly and It should start validation blocks.

## Local Test Network - First Validator (Option 2: using **istanbul  --validators**)

The option 1 is the recommended, straightforward method, but if you have problems using it, you can use this alternative one. Using this method we first create the nodekey and then run geth to see what address we have. Then, we stop it and use the address to generate the extradata for de genesis file. We modify the genesis file, initialize the node again and then run geth correctly.

### Initialize the Validator

1. Generate nodekey and initialize validator

```bash
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Export the validator number
export VAL_NUM=00
# Create validator directory
mkdir validator$VAL_NUM
# Create the nodekey
cd validator$VAL_NUM
bootnode -genkey nodekey
echo $(bootnode -nodekey nodekey -writeaddress)
>046ec139f01a8acbacebcc582a44b0c220695c5bb4af31d86604e65e8846e0c1c823db6ca7219c8d61fcbc85d303e9e3e8390956ef5ac910d9547be5fe8cbc50
cd ..
# Initialize
geth --datadir validator$VAL_NUM init genesis.json
```

2. Get the address of the validator

```bash
geth --datadir validator$VAL_NUM --networkid 30 --identity VAL_Local_Telsius_2_4_$VAL_NUM --permissioned --rpc --rpcaddr 127.0.0.1 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul --rpcport 220$VAL_NUM --port 210$VAL_NUM --istanbul.requesttimeout 10000 --ethstats VAL_Local_Telsius_2_4_$VAL_NUM:secret@localhost:3000 --verbosity 3 --vmdebug --emitcheckpoints --targetgaslimit 18446744073709551615 --syncmode full --gcmode full --vmodule consensus/istanbul/core/core.go=5 --nodiscover --maxpeers 100 --mine --minerthreads $(grep -c "processor" /proc/cpuinfo)
```

​		Get the address from the terminals output.

​		For example: "0x488cb1906e19ac1d603a05156690ea044608d108"

3. Generate Extra Data and copy it into the genesis file

```bash
# Change directory to istanbul's binary directory
cd $ISTANBUL_PATH
./istanbul extra encode --validators 0x488cb1906e19ac1d603a05156690ea044608d108 --pool 0x488cb1906e19ac1d603a05156690ea044608d108

>0x0000000000000000000000000000000000000000000000000000000000000000f85ad594aed2e9f27bd9d74d29588239155fcbd0bc36896eb8410000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c0
# Change directory back to local test network
cd $LOCAL_NET_PATH
# Edit the genesis file with new extradata
nano genesis.json
# Re-initialize validator00
sudo rm -r validator$VAL_NUM/geth/
geth --datadir validator$VAL_NUM init genesis.json
```

### Run First Validator Node

Now that we have all set and ready, we can start the validator and the network:

```bash
geth --datadir validator$VAL_NUM --networkid 30 --identity VAL_Local_Telsius_2_4_$VAL_NUM --permissioned --rpc --rpcaddr 127.0.0.1 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul --rpcport 220$VAL_NUM --port 210$VAL_NUM --istanbul.requesttimeout 10000 --ethstats VAL_Local_Telsius_2_4_$VAL_NUM:secret@localhost:3000 --verbosity 3 --vmdebug --emitcheckpoints --targetgaslimit 18446744073709551615 --syncmode full --gcmode full --vmodule consensus/istanbul/core/core.go=5 --nodiscover --maxpeers 100 --mine --minerthreads $(grep -c "processor" /proc/cpuinfo)
```

All should work smoothly and It should start validation blocks.

## Local Test Network - Five Validators

The minimum recommended number of validators to see this working is five due to the 2/3 consensus rule and the fact that we have two arrays of validator nodes.

### Generate a setup file with five nodes

```bash
# Remember
    export GETHDIR=~/projects/Alastria/dynamicValidators/quorum
    export PATH=$GETHDIR/build/bin:$PATH
    # Replace with your paths
    export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
    export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Stop the validator00 if still running
# Get his enode
echo $(bootnode -nodekey validator00/nodekey -writeaddress)
>a6ab97b66ac780648ca6237ebd397b2bdb50222972161a8109aae4037c67bd8b6aac765a071bc7fd6a76313f75b9961bd229b20414978257979947642623418b
# Change to Istanbul tools folder
cd $ISTANBUL_PATH
# Generate al the validators needed info
./istanbul setup --num 4 --nodes --verbose
# Sample output:
> validators
{
        "Address": "0x2422a54f8fa4fbf878e5ff1abfcf1eff9b043963",
        "Nodekey": "97609d6f3e81319942097ee5c6f8433abd525814d4410dd9197011b15f195a52",
        "NodeInfo": "enode://598060177aecbaea62456ca5a40f4a04d76034e6bd583d901616921e72d3b099469b4432ebe37c7d352b68d0a0dd0390c8c706be6da6c3d6239223f4b6514197@0.0.0.0:30303?discport=0"
}
{
        "Address": "0x1861fdec885db1094b20e00e2c11d717b93f9539",
        "Nodekey": "7f6420d135893a4e95f18728c5f934e8fd2917466ac09408ecdf58196bf5060b",
        "NodeInfo": "enode://c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165@0.0.0.0:30303?discport=0"
}
{
        "Address": "0xbc3f229f4af3e9e17feeec5745fff11289338ff1",
        "Nodekey": "4cc90e288bc9f4b9450b62784dcac90e6bc94eb695b7b8b29e606175ae4e0f40",
        "NodeInfo": "enode://7dc2b6278e436d65730037d242ca5b43c04d3491e5ec70c277b281118f5cf86f57b083a2c4c4b6863b7199ab800a7c6ab70a840caaba2ed0c7f2c457682ca260@0.0.0.0:30303?discport=0"
}
{
        "Address": "0x823684e58eeb14836e846a0868bf142016930633",
        "Nodekey": "11982aa6175e94d88d342ec820fcc356e104cf2e8276e1ce769bb57acfde5a03",
        "NodeInfo": "enode://43239d2e47cce45db0c593635e321c3d38d13080731bba007d78acbd01844f2c9774489fead0c241db4a6b84132a456ef929c677cb8cb34d07c7dfad66d16a4b@0.0.0.0:30303?discport=0"
}

static-nodes.json
[
"enode://598060177aecbaea62456ca5a40f4a04d76034e6bd583d901616921e72d3b099469b4432ebe37c7d352b68d0a0dd0390c8c706be6da6c3d6239223f4b6514197@0.0.0.0:30303?discport=0",

"enode://c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165@0.0.0.0:30303?discport=0",
       "enode://7dc2b6278e436d65730037d242ca5b43c04d3491e5ec70c277b281118f5cf86f57b083a2c4c4b6863b7199ab800a7c6ab70a840caaba2ed0c7f2c457682ca260@0.0.0.0:30303?discport=0",
       "enode://43239d2e47cce45db0c593635e321c3d38d13080731bba007d78acbd01844f2c9774489fead0c241db4a6b84132a456ef929c677cb8cb34d07c7dfad66d16a4b@0.0.0.0:30303?discport=0"
]

```

Now we have all the information needed to initialize and start the validators.

### static-nodes and permissioned-nodes files

When we start a network with more than one validator, we need to use two files: static-nodes.json and permissioned-nodes.json within each validator's folder. For that we can copy paste the istanbul generated static-nodes, add the validator00's enode and change the IP:port for each one:

- Static/permissioned nodes entry format: "enode://ENODE@IP:PORT"

```bash
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Create directories or 4 validators more
mkdir validator01 validator02 validator03 validator04
# Validator number point to validator00
export VAL_NUM=00
# Static Nodes - "static-nodes.json"
nano validator$VAL_NUM/static-nodes.json
	[
	"enode://a6ab97b66ac780648ca6237ebd397b2bdb50222972161a8109aae4037c67bd8b6aac765a071bc7fd6a76313f75b9961bd229b20414978257979947642623418b@127.0.0.1:21000",
	"enode://598060177aecbaea62456ca5a40f4a04d76034e6bd583d901616921e72d3b099469b4432ebe37c7d352b68d0a0dd0390c8c706be6da6c3d6239223f4b6514197@127.0.0.1:21001",
	"enode://c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165@127.0.0.1:21002",
	"enode://7dc2b6278e436d65730037d242ca5b43c04d3491e5ec70c277b281118f5cf86f57b083a2c4c4b6863b7199ab800a7c6ab70a840caaba2ed0c7f2c457682ca260@127.0.0.1:21003",
	"enode://43239d2e47cce45db0c593635e321c3d38d13080731bba007d78acbd01844f2c9774489fead0c241db4a6b84132a456ef929c677cb8cb34d07c7dfad66d16a4b@127.0.0.1:21004"
	
	]
# Permissioned Nodes - "permissioned-nodes.json"
cp validator$VAL_NUM/static-nodes.json validator$VAL_NUM/permissioned-nodes.json
# Tree of the actual project's structure
.
├── eth-netstats
├── genesis.json
├── validator00
│   ├── geth
│   ├── geth.ipc
│   ├── keystore
│   ├── nodekey
│   ├── permissioned-nodes.json
│   └── static-nodes.json
├── validator01
├── validator02
├── validator03
└── validator04
```

At this point the validator00 is ready.

### Configure the rest of the Validators

Here we are basically replicating the validator00's setup to the rest of them. Repeat the process with the four validators.

```bash
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Validator number point to validatorXX
export VAL_NUM=01
# Generate nodekey using generated ones
nano validator$VAL_NUM/nodekey
	97609d6f3e81319942097ee5c6f8433abd525814d4410dd9197011b15f195a52
# Copy static and permissioned nodes files
cp validator00/static-nodes.json validator00/permissioned-nodes.json validator$VAL_NUM/
# Initialize validator's chain
geth --datadir validator$VAL_NUM init genesis.json
```

### Run Validators test network

```bash
# Open a new terminal
# Valiables
export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
## modified geth
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Validator number point to validatorXX
export VAL_NUM=04
# Run geth
geth --datadir validator$VAL_NUM --networkid 30 --identity VAL_Local_Telsius_2_4_$VAL_NUM --permissioned --rpc --rpcaddr 127.0.0.1 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul --rpcport 220$VAL_NUM --port 210$VAL_NUM --istanbul.requesttimeout 10000 --ethstats VAL_Local_Telsius_2_4_$VAL_NUM:secret@localhost:3000 --verbosity 3 --vmdebug --emitcheckpoints --targetgaslimit 18446744073709551615 --syncmode full --gcmode full --vmodule consensus/istanbul/core/core.go=5 --nodiscover --maxpeers 100 --mine --minerthreads $(grep -c "processor" /proc/cpuinfo)
```

### Add Validator to the Propose Pool and (optional) to the actual consensus

In the actual state of the network, only de validator00 is part of the consensus. To add new validators to the consensus, they must be first in the propose pool. To add new validators to the propose pool, follow the steps below. For any validator to be included in each list it is needed the proposal of at least 2/3 of the actual consensus nodes.

```bash
# Open a new terminal
# Valiables
export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
## modified geth
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
# Export modified geth
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
# Substitute XX with the validator number
geth attach http://localhost:220XX
	# Propose a validator for the validators pool
	> istanbul.proposePool("coinbase_address", true)
	# Propose a validator for the consensus
	> istanbul.propose("coinbase_address", true)
# EXAMPLE: validator00 includes in the pool the validator01
geth attach http://localhost:22000
	# Propose a validator for the validators pool
	> istanbul.proposePool("0x2422a54f8fa4fbf878e5ff1abfcf1eff9b043963", true)
	# Propose a validator for the consensus
	> istanbul.propose("0x2422a54f8fa4fbf878e5ff1abfcf1eff9b043963", true)
```

## Add new validator to the network (full process)

If we got a network with five nodes for example, and we want to add a new one we can always follow this  steps:

```bash
# Valiables
export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
## modified geth
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Validator number point to validatorXX
export VAL_NUM=06
# New validator's directory
mkdir validator$VAL_NUM
# Generate new nodekey
bootnode -genkey validator$VAL_NUM/nodekey
# Get the enode
echo $(bootnode -nodekey validator$VAL_NUM/nodekey -writeaddress)
>c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165
# Add new enode to static and permissioned nodes files
## static-nodes.json
nano validator00/static-nodes.json
	[
	...	enode://c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165@127.0.0.1:21002
	...
	]
## permissioned-nodes.json
cp validator00/static-nodes.json validator00/permissioned-nodes.json
## Copy static and permissioned nodes files to all nodes
cp validator00/static-nodes.json validator00/permissioned-nodes.json validator01/
cp validator00/static-nodes.json validator00/permissioned-nodes.json validator02/
								[...]
cp validator00/static-nodes.json validator00/permissioned-nodes.json validator$VAL_NUM/
# Initialize validator's chain
geth --datadir validator$VAL_NUM init genesis.json
# Run geth
geth --datadir validator$VAL_NUM --networkid 30 --identity VAL_Local_Telsius_2_4_$VAL_NUM --permissioned --rpc --rpcaddr 127.0.0.1 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul --rpcport 220$VAL_NUM --port 210$VAL_NUM --istanbul.requesttimeout 10000 --ethstats VAL_Local_Telsius_2_4_$VAL_NUM:secret@localhost:3000 --verbosity 3 --vmdebug --emitcheckpoints --targetgaslimit 18446744073709551615 --syncmode full --gcmode full --vmodule consensus/istanbul/core/core.go=5 --nodiscover --maxpeers 100 --mine --minerthreads $(grep -c "processor" /proc/cpuinfo)
# Add Validator to the Propose Pool and (optional) to the actual consensus
```

## Add a bootnode to the network (full process)

If we want to add a bootnode we can always follow this  steps:

```bash
# Valiables
export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
## modified geth
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Bootnode number point to bootXX
export BOOT_NUM=00
# New bootnode's directory
mkdir boot$BOOT_NUM
# Generate new nodekey
bootnode -genkey boot$BOOT_NUM/nodekey
# Get the enode
echo $(bootnode -nodekey boot$BOOT_NUM/nodekey -writeaddress)
>c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165
# Add new enode to static and permissioned nodes files
## static-nodes.json
nano validator00/static-nodes.json
	[
	...	enode://c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165@127.0.0.1:21002
	...
	]
## permissioned-nodes.json
cp validator00/static-nodes.json validator00/permissioned-nodes.json
## Copy static and permissioned nodes files to all nodes
cp validator00/static-nodes.json validator00/permissioned-nodes.json validator01/
cp validator00/static-nodes.json validator00/permissioned-nodes.json regular00/
								[...]
cp validator00/static-nodes.json validator00/permissioned-nodes.json boot$BOOT_NUM/
# Initialize bootnode's chain
geth --datadir boot$BOOT_NUM init genesis.json
# Run geth
geth --datadir boot$BOOT_NUM --networkid 30 --identity BOT_Local_Telsius_2_4_$BOOT_NUM --permissioned --port 211$BOOT_NUM --ethstats BOT_Local_Telsius_2_4_$BOOT_NUM:secret@localhost:3000 --targetgaslimit 18446744073709551615 --syncmode fast --nodiscover --maxpeers 200
```

## Add regular node to the network (full process)

If we want to add a regular node we can always follow this  steps:

```bash
# Valiables
export LOCAL_NET_PATH=~/projects/Alastria/dynamicValidators/local-test-net/
export ISTANBUL_PATH=~/projects/GO/src/github.com/getamis/istanbul-tools/build/bin
## modified geth
export GETHDIR=~/projects/Alastria/dynamicValidators/quorum; export PATH=$GETHDIR/build/bin:$PATH
# Change directory to local test network directory
cd $LOCAL_NET_PATH
# Regular number point to regularXX
export REG_NUM=00
# New bootnode's directory
mkdir boot$REG_NUM
# Generate new nodekey
bootnode -genkey regular$REG_NUM/nodekey
# Get the enode
echo $(bootnode -nodekey regular$REG_NUM/nodekey -writeaddress)
>c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165
# Add new enode to static and permissioned nodes files
## static-nodes.json
nano validator00/static-nodes.json
	[
	...	enode://c4d358f00007bcc1d0a016ca42255c64eb58e2906b19f10e79e83d8b5d5212ebf52dfa39ad723cc2c3ee2469154afcc1ddf52612397a8b38274301a0f3fea165@127.0.0.1:21002
	...
	]
## permissioned-nodes.json
cp validator00/static-nodes.json validator00/permissioned-nodes.json
## Copy static and permissioned nodes files to all nodes
cp validator00/static-nodes.json validator00/permissioned-nodes.json validator01/
cp validator00/static-nodes.json validator00/permissioned-nodes.json boot00/
								[...]
cp validator00/static-nodes.json validator00/permissioned-nodes.json regular$REG_NUM/
# Initialize regular's chain
geth --datadir regular$REG_NUM init genesis.json
# Run geth
geth --datadir regular$REG_NUM --networkid 30 --identity REG_Local_Telsius_2_4_$REG_NUM --permissioned --rpc --rpcaddr 0.0.0.0 --rpcapi admin,db,eth,debug,miner,net,shh,txpool,personal,web3,quorum,istanbul --rpcport 222$REG_NUM --port 212$REG_NUM --istanbul.requesttimeout 10000  --ethstats REG_Local_Telsius_2_4_$REG_NUM:secret@localhost:3000 --verbosity 3 --vmdebug --emitcheckpoints --targetgaslimit 8000000 --syncmode full --gcmode full --vmodule consensus/istanbul/core/core.go=5 --nodiscover
```

