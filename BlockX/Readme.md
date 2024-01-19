# BockX Mainnet
## End Point
```
RPC  : https://rpc-blockx.dnsarz.xyz
API  : https://api-blockx.dnsarz.xyz
```
## Genesis and Addrbook
```
genesis  : https://raw.githubusercontent.com/aidilfahmi/MainNet/main/BlockX/genesis.json
addrbook : https://raw.githubusercontent.com/aidilfahmi/MainNet/main/BlockX/addrbook.json
```
## Peer
```
5cb265623b9297748be84b7030d5f9f5a3ce2296@194.163.167.138:48656
```
## Explorer 
```
https://explorer.dnsarz.xyz/blockx-mainnet
```

## State Sync

- Reset chain and backup state data :
```
sudo systemctl stop blockxd
cp $HOME/.blockxd/data/priv_validator_state.json $HOME/.blockxd/priv_validator_state.json.backup
blockxd tendermint unsafe-reset-all --keep-addr-book --home $HOME/.blockxd
```
- Configure state sync
```
peers="5cb265623b9297748be84b7030d5f9f5a3ce2296@194.163.167.138:48656"  
SNAP_RPC="https://rpc-blockx.dnsarz.xyz:443"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.blockxd/config/config.toml
STATE_SYNC_RPC=https://rpc-blockx.dnsarz.xyz:443
STATE_SYNC_PEER=5cb265623b9297748be84b7030d5f9f5a3ce2296@194.163.167.138:48656,8ebf5e70dad7268a66a9198dbe9006f9140415b6@217.182.211.81:26656,bbe679ddc774dc91b962985c7339a2e7934b8451@207.180.250.5:26656,adcd9c90cc9fba509fb283e365ecd31bd5c37ff5@49.13.166.213:26656,bc152258668e673a3b63f964fa75afdd478078c7@185.246.85.48:39656,72639ce4ce7e0260d7ae129e6acc07dcb54d6af1@167.235.102.45:20656,49d33c76a5b47e86f04695752f8be45e7b148bf1@65.109.115.56:38656,34d08633547fc406095ff6d730fdfe65d34b96d0@158.69.125.73:11356,e15f4d31281036c69fa17269d9b26ff8733511c6@147.182.238.235:26656,85d0069266e78896f9d9e17915cdfd271ba91dfd@146.190.153.165:26656,9b84b33d44a880a520006ae9f75ef030b259cbaf@137.184.38.212:26656,8cd5f922e33c7b6b4d930592c8c0663bc59c435e@103.97.111.34:26656,dc240d568509fa275cb870b93de4db1869d7187a@5.78.103.187:26656
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $SYNC_BLOCK_HEIGHT $SYNC_BLOCK_HASH

sed -i   -e "s|^enable *=.*|enable = true|"   -e "s|^rpc_servers *=.*|rpc_servers = "$STATE_SYNC_RPC,$STATE_SYNC_RPC"|"   -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|"   -e "s|^trust_hash *=.*|trust_hash = "$SYNC_BLOCK_HASH"|"   -e "s|^persistent_peers *=.*|persistent_peers = "$STATE_SYNC_PEER"|"   $HOME/.blockxd/config/config.toml
```
- Restoring backup state data
```
mv $HOME/.blockxd/priv_validator_state.json.backup $HOME/.blockxd/data/priv_validator_state.json
```
- Restart node
```
sudo systemctl restart blockxd && sudo journalctl -u blockxd -f
```
## Snapshot
```
soon
```
