# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```javascript
sudo adduser decentr
sudo adduser decentr sudo
su - decentr
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=53
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
 
## ${\color{orange}Preparing}$	
### Server Update
```javascript
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

### Install GO
```javascript
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version


```
## ${\color{orange}Node Installation}$	
### Get Binary
```javascript
cd $HOME
rm -rf decentr/
git clone https://github.com/Decentr-net/decentr
cd decentr/
git checkout v1.6.2
make build
```

#### Install Cosmovisor
```javascript
mkdir -p $HOME/.decentr/cosmovisor/genesis/bin
mv $HOME/decentr/build/linux/decentrd $HOME/.decentr/cosmovisor/genesis/bin/
ln -s $HOME/.decentr/cosmovisor/genesis $HOME/.decentr/cosmovisor/current
sudo ln -s $HOME/.decentr/cosmovisor/current/bin/decentrd /usr/bin/decentrd
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
```

### Init Generation

Replace `moniker_name` with your own moniker name
```javascript
decentrd config chain-id mainnet-3
decentrd config keyring-backend file
decentrd init moniker_name --chain-id mainnet-3
```

### Set Peers and Seeds
```javascript
sed -E -i 's/seeds = \".*\"/seeds = \"7708addcfb9d4ff394b18fbc6c016b4aaa90a10a@ares.mainnet.decentr.xyz:26656,8a3485f940c3b2b9f0dd979a16ea28de154f14dd@calliope.mainnet.decentr.xyz:26656,87490fd832f3226ac5d090f6a438d402670881d0@euterpe.mainnet.decentr.xyz:26656,3261bff0b7c16dcf6b5b8e62dd54faafbfd75415@hera.mainnet.decentr.xyz:26656,5f3cfa2e3d5ed2c2ef699c8593a3d93c902406a9@hermes.mainnet.decentr.xyz:26656,a529801b5390f56d5c280eaff4ae95b7163e385f@melpomene.mainnet.decentr.xyz:26656,385129dbe71bceff982204afa11ed7fa0ee39430@poseidon.mainnet.decentr.xyz:26656,35a934228c32ad8329ac917613a25474cc79bc08@terpsichore.mainnet.decentr.xyz:26656,0fd62bcd1de6f2e3cfc15852cdde9f3f8a7987e4@thalia.mainnet.decentr.xyz:26656,bd99693d0dbc855b0367f781fb48bf1ca6a6a58b@zeus.mainnet.decentr.xyz:26656\"/' $HOME/.decentr/config/config.toml

or

SEEDS="3fb96f1619340507e7f28fd7c4b81f4cd3d9a7e7@seeds-decentr.sxlzptprjkt.xyz:31656,89f32d5e096eadddb1b3e6e839963503ef4d2d70@rpc.decentr.nodexcapital.com:10856"
sed -i -e "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.decentr/config/config.toml

```


### Download Genesis and Addrbook
```javascript
curl -Ls https://snapshots.indonode.net/decentr/genesis.json > $HOME/.decentr/config/genesis.json
curl -Ls https://snap.nodexcapital.com/decentr/addrbook.json > $HOME/.decentr/config/addrbook.json
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```javascript
decentrd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.decentr/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.decentr/config/app.toml
```

### Set Config Pruning
```javascript
pruning="custom"
pruning_keep_every="0"
pruning_keep_recent="100"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.decentr/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.decentr/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.decentr/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.decentr/config/app.toml
```

### Set minimum gas price
```javascript
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001udec"|g' $HOME/.decentr/config/app.toml
```

### Create Service

```javascript
sudo tee /etc/systemd/system/decentrd.service > /dev/null << EOF
[Unit]
Description=Decentrd Mainnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.decentr"
Environment="DAEMON_NAME=decentrd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
```

### Register And Start Service
```javascript
sudo systemctl enable decentrd
sudo systemctl daemon-reload
sudo systemctl start decentrd && sudo journalctl -fu decentrd -o cat
