<a id="anchor"></a>
# Nois Mainnet guide



<p align="right">
  <a href="https://chat.nois.network/"><img src="https://img.shields.io/badge/Discord-7289DA?style=for-the-badge&logo=discord&logoColor=white" /></a> &nbsp;
  <a href="https://twitter.com/NoisRNG"><img src="https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" /></a> &nbsp;
  <a href="https://docs.nois.network/"><img src="https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white" /></a> &nbsp;
</p>

|Sections|Description|
|-----------------------:|------------------------------------------:|
| [Install the basic environment](#go) | Install golang. Command to check the version|
| [Install other necessary environments](#necessary) | Clone repository. Compilation project |
| [Run Node](#run) |  Initialize node. Create configuration files. Check logs & sync status. |
| [Create Validator](#validator) |  Create valdator & wallet, check your balance. |
| <a href="https://explorer.mcbnode.online/nois-mainnet" target="_explorer">Explorer</a> |  Check whether your validator is created successfully |


 <p align="center"><a href="https://docs.nois.network/"><img align="right"width="100px"alt="defund" src="https://i.ibb.co/pzmbJ6y/nm-I7-Yi-Mb-400x400.jpg"></p</a>

| Minimum configuration                                                                                |
|------------------------------------------------------------------------------------------------------|
- 8 CPU                                                                                                
- 16 GB RAM
- 250GB SSD                                                                                            

--- 
### -Install the basic environment
#### The system used in this tutorial is Ubuntu20.04, please adjust some commands of other systems by yourself. It is recommended to use a foreign VPS.
<a id="go"></a>
#### Install golang
```
sudo rm -rf /usr/local/go;
curl https://dl.google.com/go/go1.20.1.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf - ;
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```
#### After the installation is complete, run the following command to check the version

```
go version
```
<a id="necessary"></a>
[Up to sections ↑](#anchor)
### -Install other necessary environments

#### Update apt
```
sudo apt update && sudo apt full-upgrade -y
sudo apt list --upgradable
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

```
cd $HOME
git clone https://github.com/noislabs/noisd
cd noisd
git checkout v1.0.5
make installll
```
After the installation is complete, you can run `uptickd version` to check whether the installation is successful.

Display should be v1.0.5
<a id="run"></a>
### -Run node

#### Initialize node

```
moniker=YOUR_MONIKER_NAME
uptickd init $moniker --chain-id=nois-1
uptickd config chain-id nois-1
```

#### Download the Genesis file

```
curl -s https://raw.githubusercontent.com/noislabs/networks/nois1.final.1/nois-1/genesis.json > ~/.noisd/config/genesis.json
```

#### Set peer and seed

```
SEEDS="72cd4222818d25da5206092c3efc2c0dd0ec34fe@161.97.96.91:36656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:17356,babc3f3f7804933265ec9c40ad94f4da8e9e0017@testnet-seed.rhinostake.com:17356, b3e3bd436ee34c39055a4c9946a02feec232988c@seeds.cros-nest.com:56656, 20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:17356, c8db99691545545402a1c45fa897f3cb1a05aea6@nois-mainnet-seed.itrocket.net:36656"
PEERS="b3e3bd436ee34c39055a4c9946a02feec232988c@seeds.cros-nest.com:56656,babc3f3f7804933265ec9c40ad94f4da8e9e0017@seed.rhinostake.com:17356,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:17356,72cd4222818d25da5206092c3efc2c0dd0ec34fe@161.97.96.91:36656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:17356"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.noisd/config/config.toml
```
[Up to sections ↑](#anchor)

#### Pruning settings
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.noisd/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.noisd/config/app.toml
  
 sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001unois"|g' $HOME/.noisd/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.noisd/config/config.toml
```
#### Start node 
```
sudo tee <<EOF >/dev/null /etc/systemd/system/noisd.service
[Unit]
Description=noisd daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which noisd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable noisd && \
sudo systemctl start noisd 
```
___

#### Show log
```
sudo journalctl -u noisd -f
```
#### Check sync status
```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
The display `"catching_up":` shows `false` that it has been synchronized. Synchronization takes a while, maybe half an hour to an hour. If the synchronization has not started, it is usually because there are not enough peers. You can consider adding a Peer or using someone else's addrbook.

[Up to sections ↑](#anchor)
#### Replace addrbook
```
wget -O $HOME/.noisd/config/addrbook.json "https://raw.githubusercontent.com/GalaxyNode/Mainnets/main/Nois/addrbook.json"
```
<a id="validator"></a>
### Create a validator
#### Create wallet
```
uptickd keys add WALLET_NAME
```
----
## `Note please save the mnemonic and priv_validator_key.json file! If you don't save it, you won't be able to restore it later.`
----
### Receive test coins
#### Go to Nois discord https://chat.nois.network/
[Up to sections ↑](#anchor)
#### Sent in #faucet channel
```
$request WALLET_ADDRESS
```
#### Can be used later
```
uptickd query bank balances WALLET_ADDRESS
```
#### Query the test currency balance.
#### Create a validator
`After enough test coins are obtained and the node is synchronized, a validator can be created. Only validators whose pledge amount is in the top 100 are active validators.`
```
daemon=noisd
denom=unois
moniker=MONIKER_NAME
chainid=nois-1
$daemon tx staking create-validator \
    --amount=1000000000000000000$denom \
    --pubkey=$($daemon tendermint show-validator) \
    --moniker=$moniker \
    --chain-id=$chainid \
    --commission-rate=0.05 \
    --commission-max-rate=0.1 \
    --commission-max-change-rate=0.1 \
    --min-self-delegation=1000000 \
    --fees 600$denom \
    --from=WALLET_NAME\
    --yes
```

#### After that, you can go to the block [explorer](https://explorer.mcbnode.online/nois-mainnet) to check whether your validator is created successfully.
----

  <h4 align="center"> More information </h4>
  
<table width="400px" align="center">
    <tbody>
        <tr valign="top">
          <td>
            <a href="https://nois.network/" target="site">Official website</a> </td>
          <td><a href="https://twitter.com/NoisRNG" target="twitt">Official twitter</a> </td> 
          <td><a href="https://chat.nois.network/" target="discord">Discord</a></td> 
          <td><a href="https://github.com/noislabs" target="git">Github</a> </td>
          <td><a href="https://docs.nois.network/" target="doc">Documentation</a></td>   </tr>
    </tbody>
</table> 


### [Up to sections ↑](#anchor)



