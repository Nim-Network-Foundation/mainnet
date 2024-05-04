# NIM Network Mainnet

## Key Information

### Chain ID
```bash
nim_1122-1
```

### Genesis file
Available in this repository: [genesis.json.zip](./genesis.json.zip)
```sh
wget  https://raw.githubusercontent.com/Nim-Network-Foundation/mainnet/main/genesis.json.zip

unzip genesis.json.zip

mv genesis.json ~/.rollapp_evm/config/genesis.json

sha256sum ~/.rollapp_evm/config/genesis.json
cbff2650625cb02240757e76b3c00a0f04ff9347e51ec69eaddbc2b40f2c2335
```

### Seed Node

NOT YET PROVIDED

### Persistent Peers

NOT YET PROVIDED

### Binary version
Actually, we run on the binary `v2.1.3-rc01`
You can make sure of the version by running:

```sh
$ rollapp-evm version
v2.1.3-rc01
```

### Code SDK

NIM Network is totally compatible with [CosmJS](https://github.com/cosmos/cosmjs)
It has no dedicated SDK yet.

### Public endpoints
* CometBFT RPC: https://nim-mainnet-tendermint.public.blastapi.io
* CometBFT REST: https://nim-mainnet-rest.public.blastapi.io
* EVM JSON-RPC: https://nim-mainnet.public.blastapi.io

### Tools
* Dymension Portal: https://portal.dymension.xyz/rollapp/nim_1122-1
* Block explorer: https://dym.fyi/r/nim

### Wallets
* Keplr
* Leap

## Running a full node

This tutorial assume you already have Golang installed on your machine. Due to various configuration possibilities, we won't enter the details of installing Golang

### Clone the repository and install
```sh
$ git clone https://github.com/dymensionxyz/rollapp-evm.git
$ cd rollapp-evm
$ git checkout v2.1.3-rc01
$ export BECH32_PREFIX=nim
$ make install
```

### Make sure you have the right version installed

```sh
rollapp-evm version --long
```
```
build_tags: netgo,ledger
commit: 37c7b0f907ea97149856f3d344c0a2255ff81c79
cosmos_sdk_version: v0.46.16
go: go version go1.22.2 darwin/arm64
name: dymension-rdk
server_name: rollapp-evm
version: v2.1.3-rc01
```

### Setup a Celestia node

In order to configure your dymint.toml you need a Celestia node available. This readme do not cover that part. You might need your own node.

### Setup a Dymension node to have RPC

In order to configure your dymint.toml you need a Dymension RPC available. This readme do not cover that part, but you can use public ones.

### Setup your node

#### Initialize
Run the following command and replace the `<MONIKER>` by the name you want to give to your node. It can be anything.

You can customize the home path where it creates all configuration by appending `--home` flag along with the destination home. But if you do, you will need to path that home flag on every next command.

```sh
rollapp-evm init <MONIKER> --chain-id nim_1122-1
```

This will create an empty genesis file, along other configuration files.

#### Download Genesis file

This step will override the default genesis file created by the init command.

```sh
wget  https://raw.githubusercontent.com/Nim-Network-Foundation/mainnet/main/genesis.json.zip

unzip genesis.json.zip

mv genesis.json ~/.rollapp_evm/config/genesis.json
```

Make sure you have downloaded the proper genesis file

```sh
$ sha256sum ~/.rollapp_evm/config/genesis.json
cbff2650625cb02240757e76b3c00a0f04ff9347e51ec69eaddbc2b40f2c2335
```

#### Configure your seed node and persistent peers
```sh
nano ~/.rollapp_evm/config/config.toml
```

#### Configure your minimum gas price
```sh
nano ~/.rollapp_evm/config/app.toml
```

### Configure your dymint.toml

* Settlement layer is `dymension`
* Rollapp ID is `nim_1122-1`
* Data availability layer is `celestia`
* Namespace ID is `bcfaef0d36e7428befbd`

#### Create a systemd service file
```sh
nano /etc/systemd/system/rollapp-evm.service
```

Paste the following content inside (that's a default config file. Feel free to edit and change rollapp-evm arguments)
This assume you use default golang and rollapp-evm setup. Your setup may vary.

```
[Unit]
Description=NIM Network daemon
After=network-online.target

[Service]
User=root
ExecStart=/<YOUR_HOME_PATH>/go/bin/rollapp-evm start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

#### Enable and start the service
```sh
systemctl enable rollapp-evm
systemctl start rollapp-evm
```

#### Check status:

```sh
rollapp-evm status
```

#### Check logs:

```sh
journalctl -u rollapp-evm -f
```

## Signing and broadcasting

Using CosmJS and Javascript, here is a very simple example of signing and broadcasting. 
It's similar to other Cosmos-based blockchains.

```js
import { DirectSecp256k1HdWallet } from '@cosmjs/proto-signing'; 
import { GasPrice, SigningStargateClient } from '@cosmjs/stargate'; 
import { Decimal } from '@cosmjs/math'; 

// Create our signer (wallet instance)
const signer = await DirectSecp256k1HdWallet.fromMnemonic('<mnemonic>', { prefix: 'nim' }); 

// Configure our default gas price
const gasPrice = new GasPrice(Decimal.fromUserInput('20000000000', 18), 'anim',); 

// Configure our broadcasting & signing client
const client = await SigningStargateClient.connectWithSigner('<rpc>', signer, { gasPrice });
const fromAddress = (await signer.getAccounts())[0].address; 

// Construct our sample payload. Here we are doing a token send message
const coins = [{denom: '<denom-ibc-hash>', amount: '<amount>'}] 
const response = await client.sendTokens(fromAddress, '<to-address>', coins, 'auto');
```

## Converting bech32 to hex and back

Using CosmJS, you can re-encode a bech32 address to hex and back. Sample code:

```js
import {fromBech32, fromHex, toBech32, toHex} from 'cosmjs/packages/encoding';

export const convertToHexAddress = (bech32Address: string): string => {
    return '0x' + toHex(fromBech32(bech32Address).data);
};

export const convertToBech32Address = (hexAddress: string, bech32Prefix: string): string => {
    return toBech32(bech32Prefix, fromHex(hexAddress.replaceAll(/^0x/g, '')));
};
```

## Listening to events

### Listening to coin transfers events

For example, in order to subscribe to all the transfers which has been made to specific account, one can use the tendermint RPC’s subscribe method with the combination of the following filter: 

`tm.event = 'Tx' AND transfer.recipient = 'nim1u4jwp05ejr2s5cwrlheqzfu6cjfuwghgc7zegh'`

## Running your own governor

According to Dymension Documentation:  
> Governors are similar to validators in Cosmos app-chains or board members in a company. Governors do not operate nodes yet they receive token delegations, vote on onchain governance proposals and disburse the revenue (fees and new mints) generated to their delegators. It is permissionless to become a Governors for a RollApp.

A governor is a validator without a node. You can create one quite easily, once you have the `rollapp-evm` CLI installed. It does not require any fullnode running, you can use a public RPC to create it.

#### Add a key to your local keypair if you don't have one yet

After creating the keypair, make sure to send coins to that address so it actually exists on the network.

Make sure that you backup your keypair. If you loose it, you loose access to your governor, and to any governance / voting rights, as well as coins and inflation rewards.

```sh
$ rollapp-evm keys add my-super-governor
- address: nim17nst0h0f9s2u8juukt3l7ms68vzww6ve88mw7k
  name: my-super-governor
  pubkey: '{"@type":"/ethermint.crypto.v1.ethsecp256k1.PubKey","key":"A5uqp0uHl7ZkT12zyVJP4KdrST85IM9HltQzk0pdop4M"}'
  type: local
```

#### Broadcast the transaction to create your governor
```sh
rollapp-evm tx staking create-validator --moniker "My Super Governor" --commission-max-change-rate=0.01 --commission-max-rate=0.3 --commission-rate=0.05 --from my-super-governor --min-self-delegation 1 --amount 10000000000000000000anim --node "https://nim-mainnet-tendermint.public.blastapi.io:443" --pubkey $(rollapp-evm dymint show-sequencer) --fees 20000000000000anim
```

The important parts of that command are the following:

* Commission max rate and commission max change rate cannot be edited after
* You shouldn't have too large commission change rate.
* You must not impersonate the NIM Network team, name or foundation
* You can customize more things about your governor by using specific parameters (like `--description` or `--website`)
* You can change the initial delegation you're self-staking by changing the `--amount` parameter

Once broadcasted, the CLI will give you a tx hash. You can check that everything went fine by running a query

```sh
rollapp-evm query tx <MY_SUPER_HASH> --node "https://nim-mainnet-tendermint.public.blastapi.io:443"
```

If there was any error, any out of gas or lacking fees, you should see it. Otherwise, after a few minutes, you will be able to see your governor on the Dymension portal.

Congrats !