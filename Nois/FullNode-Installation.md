# $${\color{lightgreen}Manual \space Node \space Installation}$$

## ${\color{lightblue}Optional \space Configuration}$
### Custom User

```
sudo adduser nois
sudo adduser nois sudo
su - nois
```
### Custom Port
```diff
- Skip this step if running default port configurations
```
```
PORT=58
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
 
## ${\color{orange}Preparing}$	
### Server Update
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

### Install GO
```
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version


```
## ${\color{orange}Node Installation}$	
### Get Binary
```
cd $HOME
git clone https://github.com/BBlockLabs/BonusBlock-chain
cd BonusBlock-chain
git checkout v1.0.0
make build

```

#### Install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p $HOME/.noisd/cosmovisor/genesis/bin
mv $HOME/noisd/build/noisd $HOME/.noisd/cosmovisor/genesis/bin
ln -s $HOME/.noisd/cosmovisor/genesis $HOME/.noisd/cosmovisor/current
sudo ln -s $HOME/.noisd/cosmovisor/current/bin/noisd /usr/bin/noisd
```

### Init Generation

Replace `moniker_name` with your own moniker name
```
noisd config chain-id nois-1
noisd config keyring-backend test
noisd init moniker_name --chain-id nois-1
```

### Set Peers and Seeds
```

seeds="b3e3bd436ee34c39055a4c9946a02feec232988c@seeds.cros-nest.com:56656,babc3f3f7804933265ec9c40ad94f4da8e9e0017@seed.rhinostake.com:17356,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:17356,72cd4222818d25da5206092c3efc2c0dd0ec34fe@161.97.96.91:36656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:17356"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" ~/.noisd/config/config.toml


```

### Download Genesis and Addrbook
```
curl -Ls https://raw.githubusercontent.com/noislabs/networks/nois1.final.1/nois-1/genesis.json> $HOME/.noisd/config/genesis.json
```

### Set Port app.toml and config.toml
```diff
- This Step If you running custom port, if you running default port, skip this step
```
```
noisd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.noisd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.noisd/config/app.toml
```

### Set Config Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noisd/config/app.toml
```

### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025unois\"/" $HOME/.noisd/config/app.toml
```
### Disable Indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.noisd/config/config.toml
```
### Create Service

```
sudo tee /etc/systemd/system/noisd.service > /dev/null << EOF
[Unit]
Description=BonusBlock Testnet
After=network-online.target
[Service]
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.noisd"
Environment="DAEMON_NAME=noisd"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target

EOF
```

### Register And Start Service
```
sudo systemctl enable noisd
sudo systemctl daemon-reload
sudo systemctl restart noisd && sudo journalctl -fu noisd -o cat
```

## ${\color{orange}State sync}$	

### Stop the service and reset the data
```
Later....
```

### Configure state sync information
```
Later ....
```
### Restart Node and Check the logs
```
sudo systemctl start noisd && sudo journalctl -fu noisd -o cat
```

## Deleting Node
```
sudo systemctl stop noisd && \
sudo systemctl disable noisd && \
rm /etc/systemd/system/noisd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .noisd && \
rm -rf $(which noisd)
```
