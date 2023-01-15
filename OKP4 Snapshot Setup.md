

sudo systemctl stop okp4d

cp $HOME/.okp4d/data/priv_validator_state.json $HOME/.okp4d/priv_validator_state.json.backup
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book

wget http://65.109.69.240:8000/okp4data.tar.gz

tar -C $HOME/ -zxvf okp4data.tar.gz --strip-components 1

mv $HOME/.okp4d/priv_validator_state.json.backup $HOME/.okp4d/data/priv_validator_state.json

sudo systemctl restart okp4d
sudo journalctl -u okp4d -f --no-hostname -o cat

cd $HOME
rm okp4data.tar.gz
