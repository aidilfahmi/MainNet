
<p align="center">
  <img height="200" height="auto" src="https://raw.githubusercontent.com/cosmos/chain-registry/master/lumenx/images/lumen.png">
</p>

## LumenX Mainnet | Chain ID : Lumen
### Hardware requirements

The following requirements are recommended for running LumenX:

```ruby
4 or more physical CPU cores
At least 200GB of SSD disk storage
At least 8GB of memory (RAM)
At least 100mbps network bandwidth
```

### Manual Installation (Ubuntu 22 Version)
You can setup your LumenX fullnode [Here](https://github.com/aidilfahmi/MainNet/blob/main/LumenX/FullNode-Installation.md)


### State Sync
```
sudo systemctl stop lumenxd
cp $HOME/.lumenx/data/priv_validator_state.json $HOME/.lumenx/priv_validator_state.json.backup
lumenxd tendermint unsafe-reset-all --home $HOME/.lumenx

STATE_SYNC_RPC=https://rpc.lumenx-m.nodexcapital.com:443
STATE_SYNC_PEER="bc22063df30a0644df742cdb2764b1004df6e3e3@node1.lumenex.io:26656,9cd5f77ac27254891f64801470b0c3432188c62c@node2.lumenex.io:26656,78669849476c8b728abe178475c6f016edf175cf@node3.lumenex.io:26656,48444a4bacc0cafa049d777152473769ab17c0c3@node4.lumenex.io:26656"
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 2000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i \
  -e "s|^enable *=.*|enable = true|" \
  -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
  -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
  -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
  -e "s|^persistent_peers *=.*|persistent_peers = \"$STATE_SYNC_PEER\"|" \
  $HOME/.lumenx/config/config.toml

mv $HOME/.lumenx/priv_validator_state.json.backup $HOME/.lumenx/data/priv_validator_state.json

sudo systemctl start lumenxd && sudo journalctl -u lumenxd -f --no-hostname -o cat
```
