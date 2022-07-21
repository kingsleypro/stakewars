# stakewars
Stake Wars is a program that helps the community become familiar with what it means to be a NEAR validator, and gives the community an early 
chance to access the chunk-only producer. Stake Wars offers rewards that support new members who want to join mainnet as a validator starting
from the end of September 2022.


# How to run Validator Node for Shardnet on contabo Platform (Stake Wars: Episode III step-by-step instructions)



## 1. Create Shardnet wallet, setup Google Cloud instance & deploy NEAR CLI (Challenge 001)

Create your Shardnet wallet & deploy the NEAR CLI. This is designed to be your very first challenge: use it to understand how staking on NEAR works.

### 1.1 Create a wallet
Follow the instructions on the website to create a NEAR wallet on the Shardnet testnet.

Link: https://wallet.shardnet.near.org/

Successful wallet creation, you will receive ~2000 NEAR to the wallet (this Testnet tokens have no value on Mainnet)

For the VPS you can buy in contabo i see it cheapest

![image](https://user-images.githubusercontent.com/28772660/179903311-bee521e2-6479-43b4-b819-8d76ad3463d6.png)


### 1.3 Deploy NEAR-CLI

NEAR-CLI is a command-line interface that communicates with the NEAR blockchain via remote procedure calls (RPC):

* Setup and Installation NEAR CLI
* View Validator Stats

> Note: For security reasons, it is recommended that NEAR-CLI be installed on a different computer than your validator node and that no full access keys be kept on your validator node.

First, let's make sure the linux machine is up-to-date.
```
sudo apt update && sudo apt upgrade -y
```

##### Install developer tools, Node.js, and npm
First, we will start with installing `Node.js` and `npm`:
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```

Check `Node.js` and `npm` version:
```
node -v
```
> v18.x.x

```
npm -v
```
> 8.x.x

![2022-07-17_23-14](https://user-images.githubusercontent.com/46512075/179504952-7d462c7c-75f8-41e0-a04c-0418d46afb61.png)


##### Install NEAR-CLI
Here's the Github Repository for NEAR CLI.: https://github.com/near/near-cli. To install NEAR-CLI, unless you are logged in as root, which is not recommended you will need to use `sudo` to install NEAR-CLI so that the near binary is saved to /usr/local/bin

```
sudo npm install -g near-cli
```

![2022-07-17_23-19](https://user-images.githubusercontent.com/46512075/179505616-e8431edf-cea4-4a62-9ccf-31ae89a30e9c.png)

### Validator Stats

Now that NEAR-CLI is installed, let's test out the CLI and use the following commands to interact with the blockchain as well as to view validator stats. There are three reports used to monitor validator status:


###### Environment
The environment will need to be set each time a new shell is launched to select the correct network.

Networks:
- GuildNet
- TestNet
- MainNet
- **Shardnet** (this is the network we will use for Stake Wars)

Command:
```
export NEAR_ENV=shardnet
```

You can also run this command to set the Near testnet Environment persistent:
```
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```
![2022-07-17_23-21](https://user-images.githubusercontent.com/46512075/179505977-1b80d346-d3de-4bd8-8786-ede5cfbbbfe6.png)


#### NEAR CLI Commands Guide:

###### Proposals
A proposal by a validator indicates they would like to enter the validator set, in order for a proposal to be accepted it must meet the minimum seat price.

Command:
```
near proposals
```


###### Validators Current
This shows a list of active validators in the current epoch, the number of blocks produced, number of blocks expected, and online rate. Used to monitor if a validator is having issues.

Command:
```
near validators current
```



###### Validators Next
This shows validators whose proposal was accepted one epoch ago, and that will enter the validator set in the next epoch.

Command:
```
near validators next
```

## 2. Setup a validator and sync it to the actual state of the network (Challenge 002)

This challenge is focused on deploying a node (nearcore), downloading a snapshot, syncing it to the actual state of the network, then activating the node as a validator. 

### 2.1 Setup your node

#### Install required software & set the configuration

##### Prerequisites:

Before you start, you may want to confirm that your machine has the right CPU features. 

```
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
```
> Supported

##### Install developer tools:
```
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
```
#####  Install Python pip:

```
sudo apt install python3-pip
```
##### Set the configuration:

```
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

##### Install Building env
```
sudo apt install clang build-essential make
```

##### Install Rust & Cargo
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

You will see the following:

![image](https://user-images.githubusercontent.com/28772660/179904034-4c876091-5b53-4ff0-a78d-1c6d0a2d07c4.png)



Press 1 and press enter.

##### Source the environment
```
source $HOME/.cargo/env
```

#### Clone `nearcore` project from GitHub
First, clone the [`nearcore` repository](https://github.com/near/nearcore).

```
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

Checkout to the commit needed. Please refer to the commit defined here https://github.com/near/stakewars-iii/blob/main/challenges/commit.md
```
git checkout <commit>
```

#### Compile `nearcore` binary
In the `nearcore` folder run the following commands:

```
cargo build -p neard --release --features shardnet
```
The binary path is `target/release/neard`. If you are seeing issues, it is possible that cargo command is not found. Compiling `nearcore` binary may take a little while.
Download may reached to 683



#### Initialize working directory

In order to work properly, the NEAR node requires a working directory and a couple of configuration files. Generate the initial required working directory by running:

```
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```


This command will create the directory structure and will generate `config.json`, `node_key.json`, and `genesis.json` on the network you have passed. 


#### Replace the `config.json`

From the generated `config.json`, there two parameters to modify:
- `boot_nodes`: If you had not specify the boot nodes to use during init in Step 3, the generated `config.json` shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
- `tracked_shards`: In the generated `config.json`, this field is an empty empty. You will have to replace it to `"tracked_shards": [0]`

```
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```
![image](https://user-images.githubusercontent.com/46512075/179711127-6c700b77-094d-4f2f-a5b9-2bc7d84478ff.png)

### 2.2  Start the validator node

Setup Systemd

```
sudo tee /etc/systemd/system/neard.service > /dev/null <<EOF
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=$USER
#Group=near
WorkingDirectory=$home/root/.near
ExecStart=/root/nearcore/target/release/neard 
run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed
[Install]
WantedBy=multi-user.target
EOF
```

Command:

```
sudo systemctl enable neard
```
Command:

```
sudo systemctl start neard
```
If you need to make a change to service because of an error in the file. It has to be reloaded:

```
sudo systemctl reload neard
```
Check logs:

```
journalctl -n 100 -f -u neard
```



## 3. Deploy a new staking pool for your validator (Challenge 003)

Deploy a new staking pool for your validator. Do operations on your staking pool to delegate and stake NEAR.

In order to become a validator and enter the validator set, a minimum set of success criteria must be met.

* The node must be fully synced
* The `validator_key.json` must be in place
* The contract must be initialized with the public_key in `validator_key.json`
* The account_id must be set to the staking pool contract id
* There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.shardnet.near.org/nodes/validators).
* A proposal must be submitted by pinging the contract
* Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
* Once in the validator set the validator must produce great than 90% of assigned blocks

Check running status of validator node. If “Validator” is showing up, your pool is selected in the current validators list.

### 3.1 Authorize Wallet Locally
A full access key needs to be installed locally to be able to sign transactions via NEAR-CLI.


* You need to run this command:

```
near login
```

> Note: This command launches a web browser allowing for the authorization of a full access key to be copied locally.

1 – Copy the link in your browser

![image](https://user-images.githubusercontent.com/72667420/180126063-f15fb8cc-5cf2-44df-9f07-90c752c82815.png)

2 – Grant Access to Near CLI

![image](https://user-images.githubusercontent.com/72667420/180125542-c5c5f854-3119-4917-aa2e-dbafc913e5d1.png)


3 – After Grant, you will see a page like this, go back to console

![anker3](https://user-images.githubusercontent.com/72667420/180125635-0e61e230-b398-4ff7-b1f7-d64d1ce39e34.PNG)

4 – Enter your wallet and press Enter

![image](https://user-images.githubusercontent.com/72667420/180126009-c2185c0d-04d2-421e-9a24-d150ce0f200c.png)


####  Create the validator_key.json
* Copy the file generated to shardnet folder:
Make sure to replace <pool_id> by your accountId
```
cp ~/.near-credentials/shardnet/YOUR_WALLET.json ~/.near/validator_key.json
```
* Edit “account_id” => xx.factory.shardnet.near, where xx is your PoolName
* Change `private_key` to `secret_key`

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\

File content must be in the following pattern:
for example:
```
{
  "account_id": "psg.factory.shardnet.near",
  "public_key": "ed25519:BPPwVXpWW**********",
  "secret_key": "ed25519:****"
}
```
### 3.2 Mounting a staking pool

NEAR uses a staking pool factory with a whitelisted staking contract to ensure delegators’ funds are safe. In order to run a validator on NEAR, a staking pool must be deployed to a NEAR account and integrated into a NEAR validator node. Delegators must use a UI or the command line to stake to the pool. A staking pool is a smart contract that is deployed to a NEAR account.

#### Deploy a Staking Pool Contract
##### Deploy a Staking Pool
Calls the staking pool factory, creates a new staking pool with the specified name, and deploys it to the indicated accountId.

```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```
From the example above, you need to replace:

pool id: psg

accountId: psg.shardnet.near

public_key: BPPwVXpWW**********

> Be sure to have at least 30 NEAR available, it is the minimum required for storage.
Example : near call stake_wars_validator.factory.shardnet.near --amount 30 --accountId stakewars.shardnet.near --gas=300000000000000


To change the pool parameters, such as changing the amount of commission charged to 1% in the example below, use this command:
```
near call <pool_name> update_reward_fee_fraction '{"reward_fee_fraction": {"numerator": 1, "denominator": 100}}' --accountId <account_id> --gas=300000000000000
```



If there is a “True” at the End. Your pool is created.

**You have now configure your Staking pool.**

Check your pool is now visible on https://explorer.shardnet.near.org/nodes/validators

Something like this:

![image](https://user-images.githubusercontent.com/72667420/180127023-6c002078-b293-48b3-a07a-2d9f312fd8b1.png)



### 3.3 Transactions Guide
##### Deposit and Stake NEAR

Command:
```
near call <staking_pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000


##### Unstake NEAR
Amount in yoctoNEAR.

Run the following command to unstake:
```
near call <staking_pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
To unstake all you can run this one:
```
near call <staking_pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```
##### Withdraw

Unstaking takes 2-3 epochs to complete, after that period you can withdraw in YoctoNEAR from pool.

Command:
```
near call <staking_pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Command to withdraw all:
```
near call <staking_pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```

##### Ping
A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

Command:
```
near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```
Balances
Total Balance
Command:
```
near view <staking_pool_id> get_account_total_balance '{"account_id": "<accountId>"}'
```
##### Staked Balance
Command:
```
near view <staking_pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'
```
##### Unstaked Balance
Command:
```
near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```
##### Available for Withdrawal
You can only withdraw funds from a contract if they are unlocked.

Command:
```
near view <staking_pool_id> is_account_unstaked_balance_available '{"account_id": "<accountId>"}'
```
##### Pause / Resume Staking
###### Pause
Command:
```
near call <staking_pool_id> pause_staking '{}' --accountId <accountId>
```
###### Resume
Command:
```
near call <staking_pool_id> resume_staking '{}' --accountId <accountId>
```

## 4. Setup tools for monitoring node status (Challenge 004)

Setup tools for monitoring node status. Install and use RPC on port 3030 to get useful information for keep your node working.

### Monitor and make alerts 

An email notification can make it more comfortable to maintain a validator up and running. Achieve to be a validator confirming transactions on testnet and get >95% of uptime.

#### Log Files
The log file is stored either in the ~/.nearup/logs directory or in systemd depending on your setup.


Systemd Command:
```
journalctl -n 100 -f -u neard | ccze -A
```


#### RPC
Any node within the network offers RPC services on port 3030 as long as the port is open in the nodes firewall. The NEAR-CLI uses RPC calls behind the scenes. Common uses for RPC are to check on validator stats, node version and to see delegator stake, although it can be used to interact with the blockchain, accounts and contracts overall.

Find many commands and how to use them in more detail here:

https://docs.near.org/docs/api/rpc



Command:
```
sudo apt install curl jq
```
###### Common Commands:
####### Check your node version:
Command:
```
curl -s http://127.0.0.1:3030/status | jq .version
```
####### Check Delegators and Stake
Command:
```
near view <your pool>.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId <accountId>.shardnet.near
```
####### Check Reason Validator Kicked
Command:
```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```
####### Check Blocks Produced / Expected
Command:
```
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id | contains ("POOL_ID"))'
```
# create file cronjob check ping to node


## Steps

Create a new file on /home/<USER_ID>/scripts/ping.sh

```
#!/bin/sh
# Ping call to renew Proposal added to crontab

export NEAR_ENV=shardnet
export LOGS=/home/<USER_ID>/logs
export POOLID=<YOUR_POOL_ID>
export ACCOUNTID=<YOUR_ACCOUNT_ID>

echo "---" >> $LOGS/all.log
date >> $LOGS/all.log
near call $POOLID.factory.shardnet.near ping '{}' --accountId $ACCOUNTID.shardnet.near --gas=300000000000000 >> $LOGS/all.log
near proposals | grep $POOLID >> $LOGS/all.log
near validators current | grep $POOLID >> $LOGS/all.log
near validators next | grep $POOLID >> $LOGS/all.log

```
Create a new crontab, running every 5 minutes:

```
crontab -e
*/5 * * * * sh /home/<USER_ID>/scripts/ping.sh
```

List crontab to see it is running:
```
crontab -l
```

Review your logs 

```
cat home/<USER_ID>/logs/all.log
```

## Acceptance criteria:
* Ping is done periodically to network. (Every 5 minutes)


