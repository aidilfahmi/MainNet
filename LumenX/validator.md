### Create or Recover wallet
```
lumenxd keys add <walletname>
  OR
lumenxd keys add <walletname> --recover
```

### Apply SnapShot
```
sudo apt install lz4 -y
```
#### Reset Chain Data
```
sudo systemctl stop lumenxd
cp $HOME/.lumenx/data/priv_validator_state.json $HOME/.lumenx/priv_validator_state.json.backup
rm -rf $HOME/.lumenx/data
```
#### Download Data
```
curl -L https://lumenx.service.indonode.net/lumenx-snapshot.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.lumenx
mv $HOME/.lumenx/priv_validator_state.json.backup $HOME/.lumenx/data/priv_validator_state.json
```
#### Start Node
```
sudo systemctl restart lumenxd
sudo journalctl -fu lumenxd -o cat
```
### Create Validator
Create Validator (Staking) with 100 Lumen
```
lumenxd tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 1 \
--commission-max-change-rate 1 \
--min-self-delegation "1" \
--amount 100000000ulumen \
--pubkey $(lumenxd tendermint show-validator) \
--from <wallet> \
--moniker="moniker_name" \
--chain-id LumenX \
--gas auto  \
--gas-prices="0.0025ulumen" \
--identity="" \
--website="" \
--details=""
```  
### Sync Info
```
lumenxd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```
lumenxd status 2>&1 | jq .NodeInfo
```
### Check node logs
```
sudo journalctl -u andromedad -f -o cat
```
### Check Balance
```
lumenxd query bank balances $(planqd keys show wallet_name -a)
```
