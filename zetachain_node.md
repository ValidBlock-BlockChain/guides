# ZetaChain Validator Node Setup Guide


## Hardware Requirements

- **Memory:** 8 GB RAM
- **CPU:** 4 cores
- **Disk:** 500 GB SSD Storage
- **Bandwidth:** 1 Gbps for Download/Upload

---

## Prerequisites

### Operating System
- Ubuntu 22.04 LTS

### Dependencies

1. **Update system packages:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. **Install necessary tools:**
    ```bash
    sudo apt install build-essential jq git -y
    ```

3. **Install Go (version 1.22):**
    ```bash
    wget https://golang.org/dl/go1.22.linux-amd64.tar.gz
    sudo tar -C /usr/local -xzf go1.22.linux-amd64.tar.gz
    export PATH=$PATH:/usr/local/go/bin
    ```

---

## Install ZetaChain Node

1. **Download the ZetaChain node binary and make it executable:**
    ```bash
    wget https://github.com/zeta-chain/node/releases/download/VERSION/zetacored-linux-amd64 -O /usr/local/bin/zetacored
    chmod a+x /usr/local/bin/zetacored
    ```
   Replace `VERSION` with the latest version number from the [ZetaChain GitHub releases](https://github.com/zeta-chain/node/releases).

2. **Verify the installation:**
    ```bash
    zetacored version
    ```

---

## Initialize the Node

1. **Set your node's moniker (replace `<your_moniker>` with your chosen name):**
    ```bash
    zetacored init <your_moniker> --chain-id zetachain_7000-1
    ```

---

## Configure the Node

1. **Download the latest configuration files:**
    ```bash
    wget https://raw.githubusercontent.com/zeta-chain/network-config/main/mainnet/genesis.json -O ~/.zetacored/config/genesis.json
    wget https://raw.githubusercontent.com/zeta-chain/network-config/main/mainnet/client.toml -O ~/.zetacored/config/client.toml
    wget https://raw.githubusercontent.com/zeta-chain/network-config/main/mainnet/config.toml -O ~/.zetacored/config/config.toml
    ```

2. **Set minimum gas prices in `app.toml`:**
    ```toml
    minimum-gas-prices = "0.025uzetacoin"
    ```

3. **Configure seeds and persistent peers in `config.toml` as per the official documentation.**

---

## Install Cosmovisor

Cosmovisor is a process manager for Cosmos SDK-based applications that facilitates automated upgrades.

1. **Install Cosmovisor:**
    ```bash
    go install cosmossdk.io/tools/cosmovisor/cmd/[email protected]
    mv $(go env GOPATH)/bin/cosmovisor /usr/local/bin/cosmovisor
    ```

2. **Set up Cosmovisor directories:**
    ```bash
    mkdir -p ~/.zetacored/cosmovisor/genesis/bin
    mkdir -p ~/.zetacored/cosmovisor/upgrades
    cp /usr/local/bin/zetacored ~/.zetacored/cosmovisor/genesis/bin
    ```

3. **Set environment variables:**
    ```bash
    export DAEMON_NAME=zetacored
    export DAEMON_HOME=$HOME/.zetacored
    export DAEMON_ALLOW_DOWNLOAD_BINARIES=true
    ```

---

## Create a Systemd Service

1. **Create a systemd service file for `zetacored`:**
    ```bash
    sudo tee /etc/systemd/system/zetacored.service > /dev/null <<EOF
    [Unit]
    Description=ZetaChain Node
    After=network-online.target

    [Service]
    User=$USER
    ExecStart=/usr/local/bin/cosmovisor start
    Restart=always
    RestartSec=3
    LimitNOFILE=4096
    Environment="DAEMON_NAME=zetacored"
    Environment="DAEMON_HOME=$HOME/.zetacored"
    Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"

    [Install]
    WantedBy=multi-user.target
    EOF
    ```

2. **Enable and start the service:**
    ```bash
    sudo systemctl enable zetacored
    sudo systemctl start zetacored
    ```

---

## Create a Validator

1. **Ensure your node is fully synchronized:**
    ```bash
    zetacored status 2>&1 | jq .SyncInfo.catching_up
    ```
    This should return `false`, indicating synchronization is complete.

2. **Create a new wallet or import an existing one:**
    ```bash
    zetacored keys add <wallet_name>
    ```

3. **Obtain the public key:**
    ```bash
    zetacored tendermint show-validator
    ```

4. **Submit the validator creation transaction:**
    ```bash
    zetacored tx staking create-validator \
      --amount=1000000uzetacoin \
      --pubkey=$(zetacored tendermint show-validator) \
      --moniker="<your_moniker>" \
      --chain-id=zetachain_7000-1 \
      --commission-rate="0.10" \
      --commission-max-rate="0.20" \
      --commission-max-change-rate="0.01" \
      --min-self-delegation="1" \
      --from=<wallet_name> \
      --fees=5000uzetacoin
    ```

---

## Backup Important Files

Securely back up the following files located in `$HOME/.zetacored/config/`:

- `priv_validator_key.json`
- `node_key.json`

It's recommended to encrypt these backups for enhanced security.

---

## Monitoring and Maintenance

- Set up monitoring tools like Prometheus to keep track of node performance.
- Regularly update your node to the latest software versions to ensure optimal performance and security.

For additional details, visit the [ZetaChain Documentation](https://zetachain.com/docs).


