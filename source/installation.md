## Installation


<!-- tabs:start -->
#### **Custom User & Port**
- Custom user</span>

```bash
sudo adduser mantra
sudo adduser mantra sudo
su - mantra
```

- Custom Port </span>

```bash
PORT=45
echo "export PORT=${PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
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
rm -rf source
git clone https://github.com/Source-Protocol-Cosmos/source.git
cd source
git checkout v3.0.0
make  install
sudo mv ~/go/bin/sourced /usr/local/bin
sourced version
```

- Install CosmWasm Library
```bash
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
```

#### **Init Generation**

```bash
sourced config chain-id source-1
sourced config keyring-backend test
sourced init moniker_name --chain-id source-1
```
```bash
curl -Ls https://snapshots.indonode.net/source/genesis.json > $HOME/.source/config/genesis.json
curl -Ls https://snapshots.indonode.net/source/addrbook.json > $HOME/.source/config/addrbook.json
```

#### **Custom Config**
- Custom Peers and Seeds
```bash
PEERS="96d63849a529a15f037a28c276ea6e3ac2449695@34.222.1.252:26656,0107ac60e43f3b3d395fea706cb54877a3241d21@35.87.85.162:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.source/config/config.toml
```
- Custom Port
```bash
sourced config node tcp://localhost:${PORT}657
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.source/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%" $HOME/.source/config/app.toml
sed -i.bak -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${PORT}091\"%" $HOME/.source/config/app.toml
```
- Custom Pruning
```bash
pruning="custom"
pruning_keep_every="0"
pruning_keep_recent="100"
pruning_interval="19"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.source/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.source/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.source/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.source/config/app.toml
```

- Indexer & Gas Price
```bash
# Setting Indexer Off
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.source/config/config.toml
```
```bash
# Minimum Gas Price
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.25usource\"/" $HOME/.source/config/app.toml
```
#### **Service**
```bash
sudo tee /etc/systemd/system/sourced.service > /dev/null <<EOF
[Unit]
Description=Source Mainnet Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which sourced) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable sourced
sudo systemctl restart sourced && sudo journalctl -fu sourced -o cat
```

<!-- tabs:end -->



## Create Validator

<!-- tabs:start -->

#### **Wallet**
- Create Wallet
```bash
sourced keys add wallet
```

- Recover Wallet
```bash
sourced keys add wallet --recover
```

- Wallet List
```bash
sourced keys list
```

#### **Create Validator**

```bash
sourced tx staking create-validator \
  --amount 1000000usource \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.10" \
  --min-self-delegation "1000000" \
  --pubkey $(sourced tendermint show-validator) \
  --moniker YOUR_MONIKER \
  --chain-id source-1 \
  --identity=  \
  --website="" \
  --details=" " \
  --gas="auto" \
  --gas-prices="0.025usource" \
  --gas-adjustment="1.15" \
  -y
```

<!-- tabs:end -->

