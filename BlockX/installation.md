# Installation


<!-- tabs:start -->
#### **Custom User**
```bash
sudo adduser blockx
sudo adduser blockx sudo
su - blockx
```

#### **Installing Binary**
- Install Update and Dependencies
```bash
sudo apt -q update
sudo apt -qy upgrade
sudo apt -qy install curl git jq wget unzip lz4 build-essential
```

- Installing GO
```bash
sudo rm -rf /usr/local/go && \
curl -Ls https://go.dev/dl/go1.20.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

- Installing Binary
```bash
cd $HOME
rm -rf BlockX-Genesis-Mainnet1
git clone https://github.com/BlockXLabs/BlockX-Genesis-Mainnet1
cd BlockX-Genesis-Mainnet1
make  install
blockxd version
```

- Install CosmWasm Library
```bash

```

#### **Init Generation**

```bash
blockxd config chain-id blockx_100-1
blockxd config keyring-backend test
blockxd init moniker_name --chain-id blockx_100-1
```

#### **Genesis & Addrbook**
```bash
curl -Ls https://raw.githubusercontent.com/aidilfahmi/MainNet/main/BlockX/genesis.json > $HOME/.blockxd/config/genesis.json
curl -Ls https://raw.githubusercontent.com/aidilfahmi/MainNet/main/BlockX/addrbook.json > $HOME/.blockxd/config/addrbook.json
```

#### **Custom Config**
- Custom Peers and Seeds
```bash
PEERS=""
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.blockxd/config/config.toml
```
- Custom Port </span>
```bash
PORT=45
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```bash
blockxd config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.blockxd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.blockxd/config/app.toml
sed -i.bak -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${PORT}091\"%" $HOME/.blockxd/config/app.toml
```
- Custom Pruning
```bash
pruning="custom"
pruning_keep_every="0"
pruning_keep_recent="100"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.blockxd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.blockxd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.blockxd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.blockxd/config/app.toml
```

- Indexer & Gas Price
```bash
# Setting Indexer Off
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.blockxd/config/config.toml
```
```bash
# Minimum Gas Price

```
#### **Service**
```bash
sudo tee /etc/systemd/system/blockxd.service > /dev/null <<EOF
[Unit]
Description=Source Mainnet Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which blockxd) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable blockxd
sudo systemctl restart blockxd && sudo journalctl -fu blockxd -o cat
```

<!-- tabs:end -->
