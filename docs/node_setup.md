---
id: node_setup
title: Setting up a node
---

import useBaseUrl from '@docusaurus/useBaseUrl';

## Initial setup

### Requirements

The most common way to run a validator is on a cloud server running Linux. You may choose whatever VPS provider that you prefer and we recommend using Linux as operating system, but you can choose whatever operating system you are comfortable with. For this tutorial we will be using **Ubuntu 20.04**, but the instructions should be similar for other platforms.

#### Recommended hardware
We recommend using at least **Intel Core i7-7700K CPU with 64GB of RAM and an NVMe SSD with capacity 100-200GB** *(this capacity should be ok for next 6-months, but it should be monitored and adjusted as required)*. This are not *minimum requirements*, however you should be aware that you might hit performance issues if you decide to use less performant hadware.

#### Install and configure network time protocol (NTP) client

NTP is a networking protocol designed to synchronize the clocks of computers over a network. NTP allows you to synchronize the clocks of all the systems within the network. Currently it is required that validator's local clocks stay reasonably in sync, so you should be running NTP or a similar service. You can check whether you have the NTP client by running:

*On ubuntu 20.04 NTP Client should be installed by default.*
```bash
timedatectl
```
If NTP is installed and running, you should see `System clock synchronized: yes` (or a similar message). If you do not see it, you can install it by executing:
```bash
sudo apt install ntp
```
ntpd will be started automatically after install. You can query ntpd for status information to verify that everything is working:
```bash
sudo ntpq -p
```

#### Firewal requirements 

* `30333` *allowed* - p2p port have to be exposed publicly to allow other nodes to connect to your node
* `9944`  *optional* - rpc websocket port, expose this port only if want to allow external apps connect to your node (e.g https://polkadot.js.org/apps/) 
* `9933`  *optional* - http rpc port, expose this port only if want to allow external http req to your node

:::caution
We highly recommend **TO NOT** expose RPC ports (`9944, 9933`) publicly if you are running node as validator. This ports can be abused by 3-rd party to manipulate with your node and cause slashing. 
:::



### Getting binaries 

#### Download binaries

You can download released binaries from our [github](https://github.com/galacticcouncil/HydraDX-node/releases).

#### Building binary from source

:::important
Please use `git tags` with lates version *(e.g v2.0.0)* if you are building binary from souce.
:::
```bash
git clone git@github.com:galacticcouncil/HydraDX-node.git
cd HydraDX-node/
git checkout v2.0.0 #this is latest release in the time of writing this doc. Please check github for latest release tag before running this cmd

curl https://getsubstrate.io -sSf | bash -s -- --fast

cargo build --release
```

##### Running binary
```bash
./target/release/hydra-dx --chain lerna --name your-validator-name --validator
```

:::important
`--name` will be dislayed in [Telemetry](https://telemetry.polkadot.io/#list/HydraDX%20Snakenet). Telemetry show all nodes in HydraDX network. It is important to use unique name so you can identify your node.
:::

### Running validator node

#### Systemd
TODO

### Connect [PolkadotJS-APPS](https://polkadot.js.org/apps/) to your node

:::caution
We highly **NOT RECOMMEND** to expose *9944, 9933* publicly if you are runnging node as validator. Steps bellow will describe only how to connect to your local node.
:::

This is optional. You don't need to connect PolkadotJS-APPS with your node for setup validator and start validate.

*Prerequirements: your node is running on your local machine or you have port `9944` forwarded to your local machine.*

Open [PolkadotJS-APPS](https://polkadot.js.org/apps/)  and click to left upper corner(you may be connected to different chain by default):

<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/PolkadotJS-APPS-1.png')} />
</div>

Click *"Development"* in the list and select *"Local Node"*
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/PolkadotJS-APPS-2.png')} />
</div>

Click *"Switch"* button
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/PolkadotJS-APPS-3.png')} />
</div>

#### Links to public HydraDX rpc nodes
This nodes can be used by anybody to e.g. *setting session keys* 

* [rpc node 01](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc-01.snakenet.hydradx.io#/explorer)
* [rpc node 02](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc-02.snakenet.hydradx.io#/explorer)


### Bonding HDX
:::warning
Transfers are disabled on snakenet. This means that you have to setup validator with single(stash) account. We **highly recommend** to you to change to stash/controller accounts as soon as possible. 
:::
:::important
Text bellow will describe current setup with single account, but we HIGHLY RECOMMEND controller and stash account to be separate accounts.
:::

Go to *"Network"* > *"Staking"* > *"Account actions"* and click *"+ Stash"* button.
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/bond-hdx-1.png')} />
</div>

:::caution
Do not bond all your HDX because you will not be able to pay transaction fees. 
:::
Bonded HDX will be at stake and may be slashed. If you are not confident to run validator node by yourself, it may be better for you to nominate your HDX to some trusted validator.

* *"Stash account"* - Account with majority of your HDX. Please make sure this account have more HDX as you are going to bond + transaction fees. 
* *"Controller account"* - This account also need some HDX to start/stop validating and pay transaction fees. (As stated before - for now this is same as stash account, but we are recommending to change to two different accounts as soon as possible).
* *"Value bonded*" - Amount of HDX you are going to bond.
* *"Payment destination"* -  The account where the rewards from validating are sent.

When everything is filled properly click *"Bond"* button and sign transaction.

<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/bond-hdx-2.png')} />
</div>

### Set session keys
:::important
The session keys are consensus critical, so please make sure you set your session keys properly.
:::

#### Generating session keys
You can generate your session keys by using *cli on remote server* or *PolkadotJS-APPS*. This methods are equivalent and you should choose one of them. If you did not exposed PRC ports publicly and you are you are running node on remote server, you will have to use *cli on remote server* option.

##### Option 1: Cli on remote server
If you are on a remote server, you can run this command on remote server(if you didn't change default http rpc port):
```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```
Example output of cmd above.*"result"* contains session keys(copy `0x...` and use it to set session keys)
```json
{"jsonrpc":"2.0","result":"0x9257c7a88f94f858a6f477743b4180f0c9a0630a1cea85c3f47dc6ca78e503767089bebe02b18765232ecd67b35a7fb18fc3027613840f27aca5a5cc300775391cf298af0f0e0342d0d0d873b1ec703009c6816a471c64b5394267c6fc583c31884ac83d9fed55d5379bbe1579601872ccc577ad044dd449848da1f830dd3e45","id":1}
```

##### Option 2: PolkadotJS-APPS
After successful connection to your validator node, you can generate session keys by calling *"Developer"* > *"RPC calls"* > *author_rotateKeys*
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/gen-session-keys-1.png')} />
</div>
Example output: (copy `0x...` and use it to set session keys)
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/gen-session-keys-2.png')} />
</div>

#### Set session keys
You have to submit *"Extrinsics"* > *"session_setKeys"* transaction to tell chain your session keys. This transaction will associate your validator with your Controller account.

* "using the selected account" - controller account
* "keys" - *"result"* from [Generating session keys](node_setup.md/#generating-session-keys) (`0x...`)
* "proof" - `0`

Click *"Submit transaction"* and sign transaction.
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/set-session-keys-1.png')} />
</div>

Now you are ready to strat validate.

### Validate

:::caution
Please make sure your node is fully synced and live before you start validating.
:::
You can verify that you node is live and synchronized in [Telemetry](https://telemetry.polkadot.io/#list/HydraDX%20Snakenet). 

Go to *"Network"* > *"Staking"* > *"Account actions"* and click *"Validate"* button on desired validator.
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/validate-1.png')} />
</div>

Set the percentage of the rewards that will be paid to you(reward commission percentage). Rest of the rewards will be split among your nominators. You can also set 0 reward for yourself 😇. And click *"Validate"* and sign transaction.
<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/validate-2.png')} />
</div>

If you go to the *"Network"* > *"Staking"* > *"Staking overview"* tab, you will see a list of active validators currently running on the network. At the top of the page, it shows the number of validator slots that are available as well as the number of nodes that have signaled their intention to be a validator. You can go to the *"Waiting"* tab to double check to see whether your node is listed there.

<div style={{textAlign: 'center'}}>
  <img src={useBaseUrl('/node_setup/validate-3.png')} />
</div>

Validator set is updated every era. If there will be slot in next era, you will be selected to valdiator set and your node will became validator. Your validator will stay in *Waiting queue* until he will be selected to validator set. You don't have to do anything if you are not selected in a particular era.