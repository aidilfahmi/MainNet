# Installation

### Custom user (OPTIONAL)
```
sudo adduser lumenx
sudo adduser lumenx sudo
su - lumenx
```

## Variable
```
NODENAME=YourNodeName
SOURCE=lumenx
WALLET=wallet
BINARY=lumenxd
FOLDER=.lumenx
CHAIN=LumenX
VERSION=v1.3.3
DENOM=ulumen
COSMOVISOR=cosmovisor
REPO=https://github.com/cryptonetD/lumenx.git
GENESIS=https://raw.githubusercontent.com/cryptonetD/lumenx/master/config/genesis.json
PORT=58


echo "export NODENAME=${NODENAME}" >> $HOME/.bash_profile
echo "export SOURCE=${SOURCE}" >> $HOME/.bash_profile
echo "export WALLET=${WALLET}" >> $HOME/.bash_profile
echo "export BINARY=${BINARY}" >> $HOME/.bash_profile
echo "export CHAIN=${CHAIN}" >> $HOME/.bash_profile
echo "export FOLDER=${FOLDER}" >> $HOME/.bash_profile
echo "export DENOM=${DENOM}" >> $HOME/.bash_profile
echo "export VERSION=${VERSION}" >> $HOME/.bash_profile
echo "export REPO=${REPO}" >> $HOME/.bash_profile
echo "export COSMOVISOR=${COSMOVISOR}" >> $HOME/.bash_profile
echo "export GENESIS=${GENESIS}" >> $HOME/.bash_profile
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Install dependencies Required
```
sudo apt -q update
sudo apt -qy upgrade
sudo apt -qy install curl git jq lz4 build-essential wget make gcc tmux chrony -y
```
### Install GO
```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
rm go1.19.5.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### Get mainnet version of LumenX
```
cd $HOME
rm -rf $SOURCE
git clone $REPO
cd $SOURCE
git checkout $VERSION
make install
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
```

### Prepare binaries for Cosmovisor
```
mkdir -p $HOME/$FOLDER/$COSMOVISOR/genesis/bin
mv $HOME/go/bin/$BINARY $HOME/$FOLDER/$COSMOVISOR/genesis/bin/
```

### Create application symlinks
```
ln -s $HOME/$FOLDER/$COSMOVISOR/genesis $HOME/$FOLDER/$COSMOVISOR/current
sudo ln -s $HOME/$FOLDER/$COSMOVISOR/current/bin/$BINARY /usr/local/bin/$BINARY
```

### Init Config
```
$BINARY config chain-id $CHAIN
$BINARY config keyring-backend file
$BINARY config node tcp://localhost:${PORT}657
$BINARY init $NODENAME --chain-id $CHAIN
```

### Set peers and seeds
```
PEERS="bc22063df30a0644df742cdb2764b1004df6e3e3@node1.lumenex.io:26656,9cd5f77ac27254891f64801470b0c3432188c62c@node2.lumenex.io:26656,78669849476c8b728abe178475c6f016edf175cf@node3.lumenex.io:26656,48444a4bacc0cafa049d777152473769ab17c0c3@node4.lumenex.io:26656,3d99b79129adeebd237d4153cbba6a749e0ce489@node1.olivestory.co.kr:26656,8246854d88bbba7acec7b4d86c9b418c90816f1f@rpc.lumenx.indonode.net:24656,2c341d570e537683d23102e64e7b73f4bbaef829@65.21.201.244:26766,1d94c81f6b25a51be173d22523f6267113bfcbec@45.134.226.70:26656,39d8e366837505e3a31948d761cc08ac8ed4a44b@188.165.232.199:26666,9a49635f0ecb7ba93fc9eba952cbe58767557010@185.215.180.70:26656,e91a86a4bec23993f584f346208e7b47285eb632@65.21.226.230:27656,3b584334f64ab60f92388ea22bc870dcacf4c157@157.90.179.182:56656,43c4eb952a35df720f2cb4b86a73b43f682d6cb1@37.187.149.93:26696,3c7c6c284806053c21b0e0dbfd3ca59797eab1d7@65.108.7.44:51656,e3989262b8dff3596f3b1d5e44372e9326362552@192.99.4.66:26666,b9aee01d4a878d0cf6beff20cabc9d4659cdd441@65.108.44.100:27656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/$FOLDER/config/config.toml
sed -i -e "s|^seeds *=.*|seeds = \"0da8132a62468581db52774e9a513eb032179edd@45.94.58.246:17656\"|" $HOME/$FOLDER/config/config.toml
```

### Download genesis and addrbook
```
curl -Ls $GENESIS > $HOME/$FOLDER/config/genesis.json
```

### Set Custom Port
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.core/coreum-testnet-1/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.core/coreum-testnet-1/config/app.toml
```

### Set Config Pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/$FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/$FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/$FOLDER/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/$FOLDER/config/app.toml
```

### Set Config prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/$FOLDER/config/config.toml
```
### Disable Indexing
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.lumenx/config/config.toml
```
### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025$DENOM\"/" $HOME/$FOLDER/config/app.toml
```

### Create Service
```
sudo tee /etc/systemd/system/$BINARY.service > /dev/null << EOF
[Unit]
Description=$BINARY
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/$FOLDER"
Environment="DAEMON_NAME=$BINARY"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF
```

### Enable and Start Service
```
sudo systemctl daemon-reload
sudo systemctl enable $BINARY
sudo systemctl start $BINARY
```
