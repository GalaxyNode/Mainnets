<a id="anchor"></a>
# Uptick Mainnet guide



<p align="right">
  <a href="https://discord.com/invite/teqX78VZUV"><img src="https://img.shields.io/badge/Discord-7289DA?style=for-the-badge&logo=discord&logoColor=white" /></a> &nbsp;
  <a href="https://twitter.com/Uptickproject"><img src="https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" /></a> &nbsp;
  <a href="https://uptickproject.medium.com/"><img src="https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white" /></a> &nbsp;
</p>

|Sections|Description|
|-----------------------:|------------------------------------------:|
| [Install the basic environment](#go) | Install golang. Command to check the version|
| [Install other necessary environments](#necessary) | Clone repository. Compilation project |
| [Run Node](#run) |  Initialize node. Create configuration files. Check logs & sync status. |
| [Create Validator](#validator) |  Create valdator & wallet, check your balance. |
| <a href="https://uptick.explorers.guru/validators" target="_explorer">Explorer</a> |  Check whether your validator is created successfully |


 <p align="center"><a href="https://docs.uptick.network/"><img align="right"width="100px"alt="uptick" src="https://i.ibb.co/HqT4jtX/Zw-Ciwc-R8-400x400.jpg"></p</a>

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
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.4
make install
```
After the installation is complete, you can run `uptickd version` to check whether the installation is successful.

Display should be v0.2.4
<a id="run"></a>
### -Run node

#### Initialize node

```
moniker=YOUR_MONIKER_NAME
uptickd init $moniker --chain-id=uptick_117-1
uptickd config chain-id uptick_117-1
```

#### Download the Genesis file

```
curl -s https://raw.githubusercontent.com/obajay/nodes-Guides/main/Uptick/genesis.json > ~/.uptickd/config/genesis.json
```

#### Set peer and seed

```
SEEDS=""
PEERS="170397e75ca2b0f4e9f3b1bb5d0d23f9b10f01c7@uptick-sentry-1.p2p.brocha.in:30597,c0b33353fb70d8d71dcb9c8848b3b4207bd56951@uptick-sentry-2.p2p.brocha.in:30598,23e76540bea9b6851b92e280d7e0c123a0d49521@uptick-sentry-3.p2p.brocha.in:30599,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@uptick-rpc.p2p.brocha.in:30601,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick.seed.brocha.in:30600,48e7e8ca23b636f124e70092f4ba93f98606f604@54.37.129.164:55056"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.uptickd/config/config.toml
```
[Up to sections ↑](#anchor)

#### Pruning settings
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.uptickd/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.uptickd/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.uptickd/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.uptickd/config/app.toml
  
 sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001auptick"|g' $HOME/.uptickd/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.uptickd/config/config.toml
```
#### Start node 
```
sudo tee <<EOF >/dev/null /etc/systemd/system/uptickd.service
[Unit]
Description=uptickd daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable uptickd && \
sudo systemctl start uptickd 
```
___

#### Show log
```
sudo journalctl -u uptickd -f
```
#### Check sync status
```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
The display `"catching_up":` shows `false` that it has been synchronized. Synchronization takes a while, maybe half an hour to an hour. If the synchronization has not started, it is usually because there are not enough peers. You can consider adding a Peer or using someone else's addrbook.

[Up to sections ↑](#anchor)
#### Replace addrbook
```
wget -O $HOME/.uptick/config/addrbook.json "https://raw.githubusercontent.com/GalaxyNode/Mainnets/main/Uptick/addrbook.json"
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
#### Go to Uptick discord https://discord.com/invite/teqX78VZUV
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
daemon=uptickd
denom=auptick
moniker=MONIKER_NAME
chainid=uptick_117-1
$daemon tx staking create-validator \
    --amount=1000000000000000000$denom \
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

#### After that, you can go to the block [explorer](https://uptick.explorers.guru/) to check whether your validator is created successfully.
----

  <h4 align="center"> More information </h4>
  
<table width="400px" align="center">
    <tbody>
        <tr valign="top">
          <td>
            <a href="https://www.uptick.network/" target="site">Official website</a> </td>
          <td><a href="https://twitter.com/Uptickproject" target="twitt">Official twitter</a> </td> 
          <td><a href="https://discord.com/invite/teqX78VZUV" target="discord">Discord</a></td> 
          <td><a href="" target="git">Github</a> </td>
          <td><a href="https://docs.uptick.network/" target="doc">Documentation</a></td>   </tr>
    </tbody>
</table> 


### [Up to sections ↑](#anchor)



