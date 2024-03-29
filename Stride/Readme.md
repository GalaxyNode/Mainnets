<a id="anchor"></a>
# Stride Mainnet guide



<p align="right">
  <a href="https://discord.com/invite/stride-988945059783278602"><img src="https://img.shields.io/badge/Discord-7289DA?style=for-the-badge&logo=discord&logoColor=white" /></a> &nbsp;
  <a href="https://twitter.com/stride_zone"><img src="https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" /></a> &nbsp;
  <a href="https://stride.zone/blog"><img src="https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white" /></a> &nbsp;
</p>

|Sections|Description|
|-----------------------:|------------------------------------------:|
| [Install the basic environment](#go) | Install golang. Command to check the version|
| [Install other necessary environments](#necessary) | Clone repository. Compilation project |
| [Run Node](#run) |  Initialize node. Create configuration files. Check logs & sync status. |
| [Create Validator](#validator) |  Create valdator & wallet, check your balance. |
| <a href="https://stride.explorers.guru/validators" target="_explorer">Explorer</a> |  Check whether your validator is created successfully |


 <p align="center"><a href="https://docs.stride.zone/"><img align="right"width="100px"alt="Stride" src="https://i.ibb.co/JcmMGJQ/S9-IIkg-PS-400x400.png"></p</a>

| Minimum configuration                                                                                |
|------------------------------------------------------------------------------------------------------|
- 8 CPU                                                                                                
- 16 GB RAM
- 300GB SSD                                                                                            

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
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout v18.0.0
make install
```
After the installation is complete, you can run `strided version` to check whether the installation is successful.

Display should be v18.0.0
<a id="run"></a>
### -Run node

#### Initialize the node

```
moniker=YOUR_MONIKER_NAME
strided init $moniker --chain-id=stride-1
strided config chain-id stride-1
```

#### Download the Genesis file

```
curl -s hhttps://raw.githubusercontent.com/Stride-Labs/mainnet/main/mainnet/genesis.json > ~/.strided/config/genesis.json
```

#### Set peer and seed

```
SEEDS=""
PEERS="8504bdec910782f31e26d90a439f2670dff40ef6@51.79.101.159:26656,37a6a664eb693cf6fc438e202526401a07d515f1@51.89.7.234:26639,04b797b5a56fb939a97a3c7d9c3230d09b85e8d7@93.189.30.118:26656,a0d2f043f71ed37da4bc2c18fda854542ba70b46@65.108.230.45:2000,9a49054747e57cf5c4084cdf0710a23eeb6d0b03@95.179.201.94:12256,967d8818df04f661057912e804d8dcd1b8aadb4f@168.119.78.111:26656"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.strided/config/config.toml
```
[Up to sections ↑](#anchor)

#### Pruning settings
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.strided/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.strided/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.strided/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.strided/config/app.toml
  
 sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0ustrd"|g' $HOME/.strided/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.strided/config/config.toml
```
#### Start node 
```
sudo tee <<EOF >/dev/null /etc/systemd/system/strided.service
[Unit]
Description=strided daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable strided && \
sudo systemctl start strided 
```
___

#### Show log
```
sudo journalctl -u strided -f
```
#### Check sync status
```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
The display `"catching_up":` shows `false` that it has been synchronized. Synchronization takes a while, maybe half an hour to an hour. If the synchronization has not started, it is usually because there are not enough peers. You can consider adding a Peer or using someone else's addrbook.

[Up to sections ↑](#anchor)
#### Replace addrbook
```
wget -O $HOME/.stride/config/addrbook.json "https://raw.githubusercontent.com/GalaxyNode/Mainnets/main/Stride/addrbook.json"
```
<a id="validator"></a>
### Create a validator
#### Create wallet
```
strided keys add WALLET_NAME
```
----
## `Note please save the mnemonic and priv_validator_key.json file! If you don't save it, you won't be able to restore it later.`
----
$request WALLET_ADDRESS
```
#### Can be used later
```
strided query bank balances WALLET_ADDRESS
```
#### Query the test currency balance.
#### Create a validator
`After enough test coins are obtained and the node is synchronized, a validator can be created. Only validators whose pledge amount is in the top 100 are active validators.`
```
daemon=strided
denom=ustrd
moniker=MONIKER_NAME
chainid=stride-1
$daemon tx staking create-validator \
    --amount=1000000$denom \
    --pubkey=$($daemon tendermint show-validator) \
    --moniker=$moniker \
    --chain-id=$chainid \
    --commission-rate=0.05 \
    --commission-max-rate=0.1 \
    --commission-max-change-rate=0.1 \
    --min-self-delegation=1 \
    --fees 600$denom \
    --from=WALLET_NAME\
    --yes
```

#### After that, you can go to the block [explorer](https://explorer.nodestake.top/stride) to check whether your validator is created successfully.
----

  <h4 align="center"> More information </h4>
  
<table width="400px" align="center">
    <tbody>
        <tr valign="top">
          <td>
            <a href="https://stride.zone/" target="site">Official website</a> </td>
          <td><a href="https://twitter.com/stride_zone" target="twitt">Official twitter</a> </td> 
          <td><a href="https://discord.com/invite/stride-988945059783278602" target="discord">Discord</a></td> 
          <td><a href="https://github.com/Stride-Labs/stride/" target="git">Github</a> </td>
          <td><a href="https://docs.stride.zone/" target="doc">Documentation</a></td>   </tr>
    </tbody>
</table> 


### [Up to sections ↑](#anchor)




