# NIM Network Mainnet

## Key Information

### Chain ID
```bash
nim_1122-1
```

### Genesis file
Available in this repository: [genesis.json](./genesis.json)
```sh
curl -s  https://raw.githubusercontent.com/Nim-Network-Foundation/mainnet/main/genesis.json > ~/.rollapp_evm/config/genesis.json

sha256sum ~/.rollapp_evm/config/genesis.json
cbff2650625cb02240757e76b3c00a0f04ff9347e51ec69eaddbc2b40f2c2335
```

### Seed Node

NOT YET PROVIDED

### Persistent Peers

NOT YET PROVIDED

### Binary version
Actually, we run on the binary `v2.1.0`
You can make sure of the version by running:

```sh
$ rollapp-evm version
v2.1.0
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
$ git checkout v2.1.0
$ export BECH32_PREFIX=nim
$ make install
```

### Make sure you have the right version installed

```sh
rollapp-evm version --long
```
```
build_tags: netgo,ledger
commit: b5b1824a3ae081c88a53b41426ff5455b9a1438c
cosmos_sdk_version: v0.46.16
go: go version go1.22.2 darwin/arm64
name: dymension-rdk
server_name: rollapp-evm
version: v2.1.0
```

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
curl -s  https://raw.githubusercontent.com/Nim-Network-Foundation/mainnet/main/genesis.json > ~/.rollapp_evm/config/genesis.json
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
import { fromBech32, fromHex, toBech32, toHex } from ‘cosmjs/packages/encoding’;

export  const  convertToHexAddress  = (bech32Address:  string):  string  => {
	return  '0x'  +  toHex(fromBech32(bech32Address).data);
};

export  const  convertToBech32Address  = (hexAddress:  string, bech32Prefix:  string):  string  => {
	return  toBech32(bech32Prefix, fromHex(hexAddress.replaceAll(/^0x/g, '')));
};
```

## Listening to events

### Listening to coin transfers events

For example, in order to subscribe to all the transfers which has been made to specific account, one can use the tendermint RPC’s subscribe method with the combination of the following filter: 

`tm.event = 'Tx' AND transfer.recipient = 'nim1u4jwp05ejr2s5cwrlheqzfu6cjfuwghgc7zegh'`