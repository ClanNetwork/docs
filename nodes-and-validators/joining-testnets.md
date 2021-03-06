---
description: General instructions on how to join the clan testnets
---

# Joining Testnets

## Current testnets

Below is the list of Clan testnets and their current status. You will need to know the version tag for installation of the `cland` binary.

For details of upgrades on the current testnet, as well as syncing, you can [check out the testnets repo, which is the definitive source of truth](https://github.com/ClanNetwork/testnets).

If you get stuck, then please ask on Discord.



| chain-id      | Current Github version tag |           Description          | Links                                                                                                                                                                                                                                                                                | Status  |
| ------------- | -------------------------- | :----------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| playstation-2 | v1.0.4-alpha               | airdrop and pre-mainnet launch | <ul><li><a href="https://secretnodes.com/clan/chains/playstation-2">block explorer</a></li></ul><ul><li><a href="https://faucet-testnet.clan.network">faucet interface</a> </li></ul><ul><li> <a href="https://stake-testnet.clan.network/">staking/voting interaface </a></li></ul> | current |

## Minimum Hardware Requirements

The minimum recommended hardware requirements for running a validator for the Clan testnets are:

| Chain-id      | Requirements                                                                         |
| ------------- | ------------------------------------------------------------------------------------ |
| playstation-2 | <ul><li>4GB RAM</li><li>50GB+ of disk space</li><li>2 Cores (modern CPU's)</li></ul> |

{% hint style="warning" %}
These specifications are the minimum recommended. As Clan Network is a smart contract platform, it can at times be very demanding on hardware. Low spec validators WILL get stuck on difficult to process blocks.
{% endhint %}

{% hint style="info" %}
Note that the testnets accumulate data as the blockchain continues. This means that you will need to expand your storage as the blockchain database gets larger with time.
{% endhint %}

## cland Installation

To get up and running with the cland binary, please follow the instructions [here](getting-setup.md)

## Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

### Choose a testnet

Choose the `<chain-id>` testnet you would like to join from [here](joining-the-testnets.md#current-testnets). Set the `CHAIN_ID`:

```bash
CHAIN_ID=<chain-id>

#Example
CHAIN_ID=playstation-2
```

### Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```bash
MONIKER_NAME=<moniker-name>

#Example
MONIKER_NAME="testmoniker"
```

### **Set persistent peers**

Persistent peers will be required to tell your node where to connect to other nodes and join the network. To retrieve the peers for the chosen testnet:

```bash
#Set the base repo URL for the testnet & retrieve peers
CHAIN_REPO="https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN_ID" && \
export PEERS="$(curl -s "$CHAIN_REPO/persistent-peers.txt")"

# check it worked
echo $PEERS
```

{% hint style="info" %}
NB: If you are unsure about this, you can ask in discord for the current peers and explicitly set them in `~/.clan/config/config.toml` instead.
{% endhint %}

### Set 0 gas prices

In `$HOME/.clan/config/app.toml`, set gas prices:

```
# note testnet denom
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uclan\"/" ~/.clan/config/app.toml
```

## Setting up the Node

{% hint style="info" %}
Running a node is different from running a Validator. In order to run a Validator, you must create and sync a node, and then upgrade it to a Validator.
{% endhint %}

These instructions will direct you on how to initialise your node, synchronise to the network and upgrade your node to a validator.

### **Initialize the chain**

```bash
cland init $MONIKER_NAME --chain-id=$CHAIN_ID
```

This will generate the following files in `~/.clan/config/`

* `genesis.json`
* `node_key.json`
* `priv_validator_key.json`

{% hint style="info" %}
Note that this means if you jumped ahead and already downloaded the genesis file, this command will replace it and you will get an error when you attempt to start the chain.
{% endhint %}

### Download the genesis file

```
curl https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN_ID/genesis.json > ~/.clan/config/genesis.json
```

This will replace the genesis file created using `cland init` command with the genesis file for the testnet. \*\*\*\*

### **Set persistent peers**

Using the peers variable we[ set earlier](joining-the-testnets.md#set-persistent-peers), we can set the `persistent_peers` in `~/.clan/config/config.toml`:

```bash
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.clan/config/config.toml
```

### **Create a local key pair**

Create a new key pair for your validator:

```bash
cland keys add <key-name>

# Query the keystore for your public address
cland keys show <key-name> -a
```

Replace `<key-name>` with a key name of your choosing.

If you already have a key from a previous testnet, you can recover it using the mnemonic:

```bash
cland keys add <key-name> --recover
```

{% hint style="danger" %}
After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.
{% endhint %}

### **Get some testnet tokens**

Testnet tokens can be requested from the [Faucet](https://faucet-testnet.clan.network).

## Start your node

Now that everything is setup and ready to go, you can start your node.

```
cland start
```

You will need some way to keep the process always running. If you're on linux, you can do this by creating a service.&#x20;

**Note: First run `which cland` and replace `/usr/local/bin/cland` with the output if you find any differences**&#x20;

```
sudo tee /etc/systemd/system/cland.service > /dev/null <<'EOF'
[Unit]
Description=Clan daemon
After=network-online.target

[Service]
User=<your-username>
ExecStart=/usr/local/bin/cland start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

Then update and start the node

```
sudo -S systemctl daemon-reload
sudo -S systemctl enable cland
sudo -S systemctl start cland
```

You can check the status with:

```
systemctl status cland
```

## Syncing the node

After starting the cland daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup, but could take a very long time. To query the status of your node:

```bash
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network current block and you are safe to proceed to upgrade to a validator node.

## Upgrade to a validator

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```bash
cland tx staking create-validator \
  --amount 1000000000uclan \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(cland tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0uclan \
  --from <key-name>
```

## Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.clan/config/`:

* `priv_validator_key.json`
* `node_key.json`

It is recommended that you encrypt the backup of these files.
