<a id="anchor"></a>
# Quicksilver Mainnet guide



<p align="right">
  <a href="https://discord.com/invite/xrSmYMDVrQ"><img src="https://img.shields.io/badge/Discord-7289DA?style=for-the-badge&logo=discord&logoColor=white" /></a> &nbsp;
  <a href="https://twitter.com/quicksilverzone"><img src="https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white" /></a> &nbsp;
  <a href="https://medium.com/quicksilverzone"><img src="https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white" /></a> &nbsp;
</p>

|Sections|Description|
|-----------------------:|------------------------------------------:|
| [Install the basic environment](#go) | Install golang. Command to check the version|
| [Install other necessary environments](#necessary) | Clone repository. Compilation project |
| [Run Node](#run) |  Initialize node. Create configuration files. Check logs & sync status. |
| [Create Validator](#validator) |  Create valdator & wallet, check your balance. |
| <a href="https://quicksilver.explorers.guru/validators" target="_explorer">Explorer</a> |  Check whether your validator is created successfully |


 <p align="center"><a href="https://docs.quicksilver.zone/"><img align="right"width="100px"alt="defund" src="https://i.ibb.co/7jrTDFG/V2g-Pw-Ve-O-400x400.jpg"></p</a>

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
wget https://github.com/ingenuity-build/quicksilver/releases/download/v1.2.10/quicksilverd-v1.2.10-amd64
chmod +x quicksilverd-v1.2.10-amd64
mv quicksilverd-v1.2.10-amd64 $HOME/go/bin/quicksilverd
```
After the installation is complete, you can run `quicksilverd version` to check whether the installation is successful.

Display should be v1.2.10
<a id="run"></a>
### -Run node

#### Initialize the node

```
moniker=YOUR_MONIKER_NAME
quicksilverd init $moniker --chain-id=quicksilver-2
quicksilverd config chain-id quicksilver-2
```

#### Download the Genesis file

```
curl -s https://raw.githubusercontent.com/ingenuity-build/mainnet/main/genesis.json > ~/.quicksilverd/config/genesis.json
```

#### Set peer and seed

```
SEEDS="58fe3a7b075e7302f8b46b8171a0aa19ff4a427a@65.108.195.29:31126"
PEERS="ee14b4bbeb436056952c8e4e7c84826dfb92143b@65.109.105.17:26656,d6246909abf0c5e82f48ce6f623cba587b899e15@217.160.246.138:26656,c207da8baf9ff916285c7fec684fb1bc3ff2ba65@93.115.25.106:47656,05241d21ff9e7c699bbdb4faa73da1860b6d8cd7@128.199.85.168:26656,0a226e70ceb7a4123e66216d1ed83ef22ed8a187@185.119.118.118:2000,ff2055b198685f619897058a26776b9d1b73dc3c@178.63.184.129:26656,03b3e3093b6cd33fba9f00cea6c2a560f89c61d6@195.14.6.2:26656,ffd3a67122d557dbc426972196ded625757b71b6@85.239.242.5:11656,61d96fee29a9615c208c4db72526d23b45094cb4@65.108.195.30:36656,51070ba609ede6d7eb334b8cf0ed585f2b1ab66b@135.181.76.99:26656,83435bc3cbb0204188c666259ccebcd73ac33ec8@65.109.139.182:11656,46a0c8717148c4a4aa86eaaa9727e7bc6bb8e70c@49.12.7.7:26656,765aa57477e21bf94d4c41dda643f297132a1178@51.195.234.250:26656,79b214369c8f52c2d33cf79fc1897677b24cf8cb@94.130.240.229:2000,82c212c73d15ed2c7e6ad7cc5dd68cdd559c0056@65.109.52.178:26656,218078f9caa4253dc5228995f86e8d7ff65d0e04@54.39.107.110:26656,e1a24aaba30a8ff21e52fed92b96b36156b52e80@51.161.208.88:26656,972f5e4b3c977bb6fb73138f9d4d5be5b5aca6c7@65.108.159.225:26656,29c3b582c71d007cc21629b596a721d0e834f77d@65.109.21.75:26656,6da58393fe484687bc5f3067a891717f0e7d0760@167.235.15.79:26656,96bd0e87a5e5b88e8ce637aa3c7aa4f4803b1d03@51.195.234.240:26656"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.quicksilverd/config/config.toml
```
[Up to sections ↑](#anchor)

#### Pruning settings
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.quicksilverd/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.quicksilverd/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.quicksilverd/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.quicksilverd/config/app.toml
  
 sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uqck"|g' $HOME/.quicksilverd/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.quicksilverd/config/config.toml
```
#### Start node 
```
sudo tee <<EOF >/dev/null /etc/systemd/system/quicksilverd.service
[Unit]
Description=quicksilverd daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which quicksilverd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload && \
sudo systemctl enable quicksilverd && \
sudo systemctl start quicksilverd 
```
___

#### Show log
```
sudo journalctl -u quicksilverd -f
```
#### Check sync status
```
curl -s localhost:26657/status | jq .result | jq .sync_info
```
The display `"catching_up":` shows `false` that it has been synchronized. Synchronization takes a while, maybe half an hour to an hour. If the synchronization has not started, it is usually because there are not enough peers. You can consider adding a Peer or using someone else's addrbook.

[Up to sections ↑](#anchor)
#### Replace addrbook
```
wget -O $HOME/.defund/config/addrbook.json "https://snapshots.kjnodes.com/quicksilver/addrbook.json"
```
<a id="validator"></a>
### Create a validator
#### Create wallet
```
quicksilverd keys add WALLET_NAME
```
----
## `Note please save the mnemonic and priv_validator_key.json file! If you don't save it, you won't be able to restore it later.`
----
$request WALLET_ADDRESS
```
#### Can be used later
```
quicksilverd query bank balances WALLET_ADDRESS
```
#### Query the test currency balance.
#### Create a validator
`After enough test coins are obtained and the node is synchronized, a validator can be created. Only validators whose pledge amount is in the top 100 are active validators.`
```
daemon=quicksilverd
denom=uqck
moniker=MONIKER_NAME
chainid=quicksilver-2
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

#### After that, you can go to the block [explorer](https://quicksilver.explorers.guru/validators/) to check whether your validator is created successfully.
----

  <h4 align="center"> More information </h4>
  
<table width="400px" align="center">
    <tbody>
        <tr valign="top">
          <td>
            <a href="https://quicksilver.zone/" target="site">Official website</a> </td>
          <td><a href="https://twitter.com/quicksilverzone" target="twitt">Official twitter</a> </td> 
          <td><a href="https://discord.com/invite/xrSmYMDVrQ" target="discord">Discord</a></td> 
          <td><a href="https://github.com/ingenuity-build" target="git">Github</a> </td>
          <td><a href="https://docs.quicksilver.zone/" target="doc">Documentation</a></td>   </tr>
    </tbody>
</table> 


### [Up to sections ↑](#anchor)




