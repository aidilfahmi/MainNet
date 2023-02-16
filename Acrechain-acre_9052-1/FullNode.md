# Setup Node

## Install Ubuntu 20.04 on a new server and login as root

## Install `ufw` firewall and configure the firewall

```
apt-get update
apt-get install ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow 22
ufw allow 26656
ufw enable
```

## Create a new User

```
# add user
adduser node

# add user to sudoers
adduser node sudo

# login as user
su - node
```

## Install Prerequisites

```
sudo apt update -y && sudo apt upgrade -y
sudo apt install pkg-config build-essential libssl-dev curl jq git libleveldb-dev -y
sudo apt-get install manpages-dev -y
```
## Install go
```
curl https://dl.google.com/go/go1.18.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
```

# Update environment variables to include go
```
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
```
```
source $HOME/.profile
```
# check go version
```
go version
```

## Install ACREChain Node

```
git clone https://github.com/ArableProtocol/acrechain.git
cd acrechain
git checkout v1.1.1
make install
cd
```

## Create moniker

```
#Choose a name for your validator and use it in place of “<moniker-name>” in the following command:
acred init <moniker-name> --chain-id acre_9052-1

#Example
acred init My_Nodes --chain-id acre_9052-1
```

## Download genesis.json file.

```
wget https://raw.githubusercontent.com/ArableProtocol/acrechain/main/networks/mainnet/acre_9052-1/genesis.json -O $HOME/.acred/config/genesis.json

```

## Update peers list

```
cd

PEERS="ef28f065e24d60df275b06ae9f7fed8ba0823448@46.4.81.204:34656,e29de0ba5c6eb3cc813211887af4e92a71c54204@65.108.1.225:46656,276be584b4a8a3fd9c3ee1e09b7a447a60b201a4@116.203.29.162:26656,e2d029c95a3476a23bad36f98b316b6d04b26001@49.12.33.189:36656,1264ee73a2f40a16c2cbd80c1a824aad7cb082e4@149.102.146.252:26656,dbe9c383a709881f6431242de2d805d6f0f60c9e@65.109.52.156:7656,d01fb8d008cb5f194bc27c054e0246c4357256b3@31.7.196.72:26656,91c0b06f0539348a412e637ebb8208a1acdb71a9@178.162.165.193:21095,bac90a590452337700e0033315e96430d19a3ffa@23.106.238.167:26656"


sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.acred/config/config.toml
```

## Creating Service

```
sudo tee /etc/systemd/system/acred.service > /dev/null <<EOF
[Unit]
Description=Acred Daemon
#After=network.target
StartLimitInterval=350
StartLimitBurst=10

[Service]
Type=simple
User=$USER
ExecStart=/home/node/go/bin/acred start
Restart=on-abort
RestartSec=30

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=1048576
EOF
```

## Reload the daemon, enable and start the service

```
sudo systemctl daemon-reload
sudo systemctl enable acred

# Start the service
sudo systemctl start acred

# Stop the service
sudo systemctl stop acred

# Restart the service
sudo systemctl restart acred


# For Entire log
journalctl -t acred -o cat

# For Entire log reversed
journalctl -t acred -r -o cat

# Latest and continuous
journalctl -fu acred -o cat
```

