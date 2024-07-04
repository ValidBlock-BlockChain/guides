### Details
* Network Chain ID: buenavista-1
* Denom: uward
* Binary: wardend
* Working directory: .warden
* RPC: https://t-warden.rpc.utsa.tech/
* API: https://t-warden.api.utsa.tech/
* Explorers: 1 2
* Docs: https://docs.wardenprotocol.org/learn/spaceward/tutorial-spaceward
* Docs: https://docs.wardenprotocol.org/category/validate

### Faucet: 
`curl -XPOST -d '{"address": "warden1ct40npn9sr2ut0tqsushm8l95v07hf55l8r9sq"}' https://faucet.buenavista.wardenprotocol.org`

### Updates
```
# # # # checking the status of prevotes/precommits
FOLDER=.warden

# find out the RPC port
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/$FOLDER/config/config.toml | egrep -o ":[0-9]+")\033[0m"

PORT=<enter your port>

# Checking prevotes/precommits. Useful for updates
curl -s localhost:$PORT/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array' &&\
curl -s localhost:$PORT/consensus_state | jq '.result.round_state.height_vote_set[0].precommits_bit_array'
```

### Server Preparation
```
# updating
apt update && apt upgrade -your repositories

# install the necessary utilities
apt install curl iptables build-essential wget wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### File2Ban 
```
# install and copy the config, which will have a higher priority
apt install fail2ban -y &&\
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local && \
nano /etc/fail2ban/jail.local
# uncomment and add your IP: ignoreip = 127.0.0.1/8 ::1 <ip>
systemctl restart fail2ban

# checking the status
systemctl status fail2ban
# check which jails are active (by default, only sshd)
fail2ban-client status
# checking the sshd statistics
fail2ban-client status sshd
# looking at the logs
tail /var/log/fail2ban.log
# stop working and delete from startup
#systemctl stop fail2ban && systemctl disable fail2ban
```

### Installing GO
```
ver="1.21.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### New node installation
WARNING — in the commands below, we change everything in <> to its value and remove it ourselves <>

Устанавливаем бинарники:
```
# if the GO directory is not installed, create it
mkdir -p $HOME/go/bin
```

```
git clone --depth 1 --branch v0.3.0 https://github.com/warden-protocol/wardenprotocol
cd wardenprotocol
wget https://github.com/warden-protocol/wardenprotocol/releases/download/v0.3.0/wardend_Linux_x86_64.zip
unzip wardend_Linux_x86_64.zip
rm -rf wardend_Linux_x86_64.zip
chmod +x wardend
mv wardend $HOME/go/bin

wardend version --long | grep -e version -e commit
# version: 0.3.0
# commit: 338a8105486cfc3fa027d3caf32f17a21b230db6
```

Initialize the year to create the necessary configuration files
`wardend init UTSA_guide --chain-id buenavista-1`

Download Genesis
```
#download genesis
wget -O $HOME/.warden/config/genesis.json "https://raw.githubusercontent.com/warden-protocol/networks/main/testnets/buenavista/genesis.json"

# Let's check the genesis
of sha256sum ~/.warden/config/genesis.json
# 084571d20aa6bb8c69e59308a19a407035d5fc93ad538feab0211f3e95e4bfc8
```

We check that the state of the validator is at the initial stage
```
cd && cat .warden/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}

# if not, then run the command
ward end tendermint unsafe-reset-all --home $HOME/.warden
```

Download Addrbook
`wget -O $HOME/.warden/config/addrbook.json "https://share.utsa.tech/warden/addrbook.json"`

### Setting up the node configuration
Manually add the chain-id of buenavista-1 to the client.tool or add the key --chain-id=buenavista-1 to each transaction
```
# we are correcting the config, so that we can no longer use the chain-id flag for each CLI command in client.toml
#wardend config chain-id buenavista-1

# if necessary, configure the keyring-backend in client.toml 
#wardend config keyring-backend os

# setting the minimum gas price in app.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uward\"/;" ~/.warden/config/app.toml

# adding seeds/bpeers/peers to config.toml
external_address=$(wget -qO- eth0.me )
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.warden/config/config.toml

peers="b14f35c07c1b2e58c4a1c1727c89a5933739eeea@warden-testnet-peer.itrocket.net:18656,61446070887838944c455cb713a7770b41f35ac5@37.60.249.101:26656,8288657cb2ba075f600911685670517d18f54f3b@65.108.231.124:18656,0be8cf6de2a01a6dc7adb29a801722fe4d061455@65.109.115.100:27060"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.warden/config/config.toml
seeds="8288657cb2ba075f600911685670517d18f54f3b@warden-testnet-seed.itrocket.net:18656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.warden/config/config.toml

# if necessary, increase the number of incoming and outgoing peers to connect, except for permanent peers in config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 40/g' $HOME/.warden/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 10/g' $HOME/.warden/config/config.toml

# configuring filtering of "bad" peers
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.warden/config/config.toml
```

(OPTIONAL) Set up pruning with one command warr.toml
```
pruning="custom"
pruning_keep_recent="1000"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.warden/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.warden/config/app.toml
```

(OPTIONAL) Turn off indexing in config.toml
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.warden/config/config.toml
```

(OPTIONAL) On/off snapshots warr.toml
```
# By default, snapshots are disabled "snapshot-interval=0"
snapshot_interval=1000
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.warden/config/app.yaml
```

(OPTIONAL) State Sync
```
# if necessary, download wasm
curl -L https://share.utsa.tech/warden/wasm-warden.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.garden --strip-components 2
```
```
# adding a peer
peers="8fa48bbd20d316382f339bd00f31d3a2678682d2@144.76.29.90:26656 "
sed -i.bak -e "s/^постоянные пользователи *=.*/постоянные пользователи = \"$peers\"/" $HOME/.warden/config/config.tool
```
```
# SNAP_RPC=https://rpc.sentry-1.buenavista-1.wardenprotocol.org:443
SNAP_RPC=https://t-warden.rpc.utsa.tech:443

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.warden/config/config.toml
```
`systemctl restart wardend && journalctl -u wardend -f -o cat`

Creating a service file
```
tee /etc/systemd/system/wardend.service > /dev/null <<EOF
[Unit]
Description=wardend
After=network-online.target

[Service]
User=$USER
ExecStart=$(which wardend) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```
systemctl daemon-reload
systemctl enable wardend
systemctl restart wardend && journalctl -u wardend -f -o cat
```

#### Creating or restoring a wallet and saving the withdrawal
```
# create
warden keys wallet add <name_wallet> --keyring-backend os

# restore wallet (after the insert seed command)
wardend keys add <name_wallet> --recover --keyring-backend os

# restore wallet for EVM networks
wardend keys add <name_wallet> --recover --coin-type 118 --algo secp256k1
```

### Creating a validator
1. Get your pubkey
`wardend comet show-validator`
2. Creating validator.json
`nano $HOME/.warden/validator.json`
3. Insert our config
```
{
  "pubkey": {#pubkey},
  "amount": "1000000uward",
  "moniker": "STAVR",
  "identity": "",
  "website": "",
  "security": "",
  "details": "",
  "commission-rate": "0.05",
  "commission-max-rate": "0.5",
  "commission-max-change-rate": "0.5",
  "min-self-delegation": "1"
}
```
4. Отправляем транзакцию
```
wardend tx staking create-validator $HOME/.warden/validator.json \
    --from=<key-name> \
    --chain-id=buenavista-1 \
    --fees=500uward
```


