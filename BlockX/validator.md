## Create Validator

<!-- tabs:start -->

#### **Wallet**
- Create Wallet
```bash
blockxd keys add wallet
```

- Recover Wallet
```bash
blockxd keys add wallet --recover
```

- Wallet List
```bash
blockxd keys list
```

#### **Create Validator**

```bash
blockxd tx staking create-validator \
  --amount 1000000abcx \
  --from wallet_name \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation "1000000" \
  --pubkey $(blockxd tendermint show-validator) \
  --moniker YOUR_MONIKER \
  --chain-id blockx_100-1 \
  --identity=  \
  --website="" \
  --details=" " \
  --gas="auto" \
  --gas-prices="0.025abcx" \
  --gas-adjustment="1.15" \
  -y
```

<!-- tabs:end -->
