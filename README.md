![2022-12-25_15-55-50](https://user-images.githubusercontent.com/98663407/209468772-5fda2ec1-4446-49b4-a605-0cb55c0897e8.png)

RPC http://65.108.129.29:27657
gRPC http://65.108.129.29:9190/
Live Peers 2ca4e1bed94cfe9fad160e704ccbabf95f438dee@65.108.129.29:27657
API Endpoint http://65.108.129.29::1417


# State Sync okp4


  ```
sudo systemctl stop okp4d

cp $HOME/.okp4d/data/priv_validator_state.json $HOME/.okp4d/priv_validator_state.json.backup
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book

SNAP_RPC="http://65.108.129.29:27657"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peers="2ca4e1bed94cfe9fad160e704ccbabf95f438dee@65.108.129.29:27657"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.okp4d/config/config.toml

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.okp4d/config/config.toml

mv $HOME/.okp4d/priv_validator_state.json.backup $HOME/.okp4d/data/priv_validator_state.json

sudo systemctl restart okp4d
sudo journalctl -u okp4d -f --no-hostname -o cat
  ```
  ![image](https://user-images.githubusercontent.com/98663407/209471103-3cd1dba0-27c0-40f8-8e8e-2154e41cf9e6.png)

# guide state sync okp4  

How to run your own RPC with State Sync
Using the okp4 network as an example.

Install okp4 binary https://docs.okp4.network/nodes/installation
Download the blockchain in a convenient way (Sync)
Configure the RPC settings for State-sync snapshots:
Parameter app.toml
  ```
sed -i.bak -e "s/^pruning *=.*/pruning = \"custom"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \""100"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^pruning-keep-every *=.*/pruning-keep-every = \""1000"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \"10"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \""1000"\"/" $HOME/.okp4d/config/app.toml
sed -i.bak -e "s/^snapshot-keep-recent *=.*/snapshot-keep-recent = \""2"\"/" $HOME/.okp4d/config/app.toml
  ```
The snapshot-interval/snapshot-keep-recent value could be, for example, 500-5000 for space saving.

Parameter config.toml
  ```
sed -i.bak -e "s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0:26657\"%" $HOME/.okp4d/config/config.toml
sed -i.bak -E "s|^(pex[:space:]]+=[[:space:]]+).*$|\1true|" $HOME/.okp4d/config/config.toml
  ```
Restart your node
  ```
sudo systemctl restart okp4d
  ```
Use your browser to check that the example (http://http://65.108.129.29:27657/) can be accessed.
Done! Wait for your server to snapshot and everyone can use it.

Do on the new server which requires sync
  ```
$HOME/.okp4d/config/config.toml

peers="2ca4e1bed94cfe9fad160e704ccbabf95f438dee@65.108.129.29:27657" 
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.okp4d/config/config.toml
  ```

Enter the commands one by one

  ```
SNAP_RPC=65.108.129.29:27657
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); {\
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[:space:]]+=[:space:]]+).*$||\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[:space:]]+).*$||1$BLOCK_HEIGHT| ;\
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$||\1\"\"|" ~/.okp4d/config/config.toml
  ```

Delete all downloaded date and restart okp4d.service
  ```
sudo systemctl stop okp4d && \
icad tendermint unsafe-reset-all --home $HOME/.okp4d && \
sudo systemctl restart okp4d
  ```
