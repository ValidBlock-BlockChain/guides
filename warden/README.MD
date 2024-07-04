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
