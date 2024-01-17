BockX Mainnet
```
RPC  : https://rpc-blockx.dnsarz.xyz
API  : https://api-blockx.dnsarz.xyz

genesis  : https://raw.githubusercontent.com/aidilfahmi/MainNet/main/BlockX/genesis.json
addrbook : https://raw.githubusercontent.com/aidilfahmi/MainNet/main/BlockX/addrbook.json

echo $(blockxd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.blockxd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
peer : 5cb265623b9297748be84b7030d5f9f5a3ce2296@194.163.167.138:48656

explorer : https://explorer.dnsarz.xyz/blockx-mainnet
```
