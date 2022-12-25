# Open Knowledge Protocol (OKP4)
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
![2022-12-25_17-07-19](https://user-images.githubusercontent.com/98663407/209471147-bccb6c9c-b060-44aa-b5ec-c608fd907510.png)


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
![4](https://user-images.githubusercontent.com/98663407/209472332-e716836e-33f4-409b-b352-52aeab6c9eae.png)

 # PANIC monitoring and alerting for okp4 blockchain

PANIC is an open source monitoring and alerting solution for Cosmos-SDK, Substrate and Chainlink based nodes by Simply VC. The tool was built with user friendliness in mind, and comes with numerous features such as phone calls for critical alerts, a UI Dashboard, a Web-UI installation process and Telegram/Slack commands for increased control over your alerter.

 # Installation Guide
We will now guide you through the steps required to get PANIC up and running. We recommend that PANIC is installed on a Linux system and that everything needed in the Requirements section is done before the installation procedure is started.

Installation
TIP: If your terminal is telling you that you do not have permissions to run a command try adding sudo to your command e.g, sudo docker --version this will run your command as root.

Installing Git
*Skip if Git is already installed.

  ```
  # install git
  sudo apt install git
  
  check
  git --version
  ```
#### Installing Docker and Docker Compose

Skip if Docker and Docker Compose is already installed

```
# install docker and docker-compose
curl -sSL https://get.docker.com/ | sh
sudo apt install docker-compose -y

check
docker --version
docker-compose --version
```
These should give you the current versions of the software that have been installed. At the time of writing the current working docker version is 20.10.10 while the docker-compose version is 1.25.0. If you have a different version that doesn't allow you to run the docker-compose.yml file then either upgrade your versions of docker and docker-compose or change the version inside of the docker-compose.yml file which is currently at 3.7.

Install configuration

```
# clone the repo and go to the
git clone https://github.com/SimplyVC/panic
cd panic
```

In the `panic` directory, edit the parameters in the `.env` file

- INSTALLER_USERNAME: user

- INSTALLER_PASSWORD: password

Once inside change UI_ACCESS_IP, INSTALLER_USERNAME and INSTALLER_PASSWORD accordingly. Here is an example:

```
UI_ACCESS_IP=1.1.1.1
INSTALLER_USERNAME=panic_user
INSTALLER_PASSWORD=panic_password
```
Then to exit hit the following keys:

To exit your .env file: CTRL + X
To select yes to save your modified file: Y
To confirm the file name and exit: ENTER


Launch Panic

After configuring all the parameters run Panic with this command:

``bash
docker-compose up -d --build
```
If the build fails, we remove the docker images with the following command, fix the cause of the failure and start the build again:

```bash
docker-compose kill
docker system prune -a --volumes
docker-compose up -d --build
```

To make sure that the system is working correctly, you can use the commands `docker-compose logs -f alerter` and `docker-compose logs -f health-checker`.

The log files are located in the `panic/alerter/logs` directory

Setting up Telegram notifications

To create **Telegram bot**, add [@BotFather](https://telegram.me/BotFather) to Telegram, click Start, and perform the following steps:

1. send a command `/newbot` and give the requested data including bot name and user name

2. Write down API token, which looks like:`11111111:AAA-AAA11111111-aaaaa11111`.

3. Click on the link `t.me/<username>` to access the created bot and click `Start`.

4. Follow the link `api.telegram.org/bot<token>/getUpdates`, replacing the `<token>` received bot token. Opens up a list of bot activity, including the messages sent to him

5. There should be at least one message in the list, triggered by clicking Start. If not, send the bot the `/start` command. Fix the value of `"id"` in the `"chat"` section.

6. This completes the creation of the bot, to create another bot, repeat the above steps again

   **As a result we should get

   - Telegram account
   - Telegram bot
   - Telegram bot API token
   - Chat ID (`chat ID`) 

Configuring monitoring and alerts

The configuration of node monitoring and notification channels is done at `https://{UI_ACCESS_IP}:8000`. For authorization we use `INSTALLER_USERNAME` and `INSTALLER_PASSWORD` which we specified in file `.env`. 

After authorization, the monitoring wizard starts. All necessary actions are intuitive and commented in detail.
![3](https://user-images.githubusercontent.com/98663407/209472462-7229951b-8bbb-4a44-954e-efafa09f1b79.png)
