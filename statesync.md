# Umee Mainnet state sync guide

1. Stop Umee background service and reset database:
```
systemctl stop umeed
umeed unsafe-reset-all
```
2. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
PEER="4f17f8d85540a706edbab5385a8fb7d9a00a515d@185.255.131.161:26656"
SNAP="http://185.255.131.161:26657"
LATEST_HEIGHT=$(curl -s $SNAP/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP,$SNAP\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.umee/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEER\"/" $HOME/.umee/config/config.toml
```
3. Run Umee background service:
```
sudo systemctl start umeed
sudo journalctl -u umeed -f -o cat
```
