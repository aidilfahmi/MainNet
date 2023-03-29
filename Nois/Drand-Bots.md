## ${\color{lightblue}Running \space Drand-Bots}$


### ${\color{orange}Preparing}$	
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y git htop joe jq \
  && wget -O nodejs.deb https://deb.nodesource.com/node_16.x/pool/main/n/nodejs/nodejs_16.17.1-deb-1nodesource1_amd64.deb \
  && sudo dpkg -i nodejs.deb \
  && sudo npm install pm2 -g
```
### ${\color{orange}Installing \space Nois \space Bot}$	 
```
git clone https://github.com/noislabs/nois-bot.git \
  && cd nois-bot \
  && npm install
```
### ${\color{orange}Configure}$	
```
cp .env.example .env
```

Edit Configuration with your own
```
nano .env
```
```
MNEMONIC="your_bot_wallet_mnemonic"
ENDPOINT="https://nois-testnet.rpc.kjnodes.com/"
# Optional 2nd endpoint used for broadcasting
ENDPOINT2=""
# Optional 3rd endpoint used for broadcasting
ENDPOINT3=""
NOIS_CONTRACT="nois19w26q6n44xqepduudfz2xls4pc5lltpn6rxu34g0jshxu3rdujzsj7dgu8"
DENOM="unois"
GAS_PRICE="0.05unois"
PREFIX="nois"
# A public nickname for this bot. When set, the bot will be registered on start.
MONIKER="your_bot_name"
```

### ${\color{orange}Start}$	
```
pm2 start index.js
```

### ${\color{orange}Status}$	
```
pm2 ls
pm2 logs --lines 100
```
