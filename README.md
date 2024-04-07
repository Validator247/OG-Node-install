# OG-Node-install

Installation guide


Install required packages

    sudo apt update && \
    sudo apt install curl git jq build-essential gcc unzip wget lz4 -y

Install Go

    cd $HOME && \
    ver="1.21.3" && \
    wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
    sudo rm -rf /usr/local/go && \
    sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
    rm "go$ver.linux-amd64.tar.gz" && \
    echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
    source $HOME/.bash_profile && \
    go version

 Build evmosd binary

     git clone https://github.com/0glabs/0g-evmos.git
     cd 0g-evmos
     git checkout v1.0.0-testnet
     make install
     evmosd version

 Set up variables

    echo 'export MONIKER="My_Node"' >> ~/.bash_profile
    echo 'export CHAIN_ID="zgtendermint_9000-1"' >> ~/.bash_profile
    echo 'export WALLET_NAME="wallet"' >> ~/.bash_profile
    echo 'export RPC_PORT="26657"' >> ~/.bash_profile
    source $HOME/.bash_profile

Intitialize the node

    cd $HOME
    evmosd init $MONIKER --chain-id $CHAIN_ID
    evmosd config chain-id $CHAIN_ID
    evmosd config node tcp://localhost:$RPC_PORT
    evmosd config keyring-backend os

Download genesis.json

    wget https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json -O $HOME/.evmosd/config/genesis.json

Add seeds and peers to the config.toml

PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
    SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml

Change ports (Optional)

    EXTERNAL_IP=$(wget -qO- eth0.me) \
    PROXY_APP_PORT=26658 \
    P2P_PORT=26656 \
    PPROF_PORT=6060 \
    API_PORT=1317 \
    GRPC_PORT=9090 \
    GRPC_WEB_PORT=9091

        
    sed -i \
    -e "s/\(proxy_app = \"tcp:\/\/\)\([^:]*\):\([0-9]*\).*/\1\2:$PROXY_APP_PORT\"/" \
    -e "s/\(laddr = \"tcp:\/\/\)\([^:]*\):\([0-9]*\).*/\1\2:$RPC_PORT\"/" \
    -e "s/\(pprof_laddr = \"\)\([^:]*\):\([0-9]*\).*/\1localhost:$PPROF_PORT\"/" \
    -e "/\[p2p\]/,/^\[/{s/\(laddr = \"tcp:\/\/\)\([^:]*\):\([0-9]*\).*/\1\2:$P2P_PORT\"/}" \
    -e "/\[p2p\]/,/^\[/{s/\(external_address = \"\)\([^:]*\):\([0-9]*\).*/\1${EXTERNAL_IP}:$P2P_PORT\"/; t; s/\(external_address = \"\).*/\1${EXTERNAL_IP}:$P2P_PORT\"/}" \
    $HOME/.evmosd/config/config.toml    

Configure prunning to save storage (Optional)

    sed -i.bak -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.evmosd/config/app.toml
    sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.evmosd/config/app.toml
    sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.evmosd/config/app.toml

Set min gas price

    sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00252aevmos\"/" $HOME/.evmosd/config/app.toml

 Enable indexer (Optional)

     sed -i "s/^indexer *=.*/indexer = \"kv\"/" $HOME/.evmosd/config/config.toml

 Create a service file

     sudo tee /etc/systemd/system/ogd.service > /dev/null <<EOF
     [Unit]
     Description=OG Node
     After=network.target
     
     [Service]
     User=$USER
     Type=simple
     ExecStart=$(which evmosd) start --home $HOME/.evmosd
     Restart=on-failure
     LimitNOFILE=65535
     
     [Install]
     WantedBy=multi-user.target
     EOF

 Start the node

     sudo systemctl daemon-reload && \
     sudo systemctl enable ogd && \
     sudo systemctl restart ogd && \
     sudo journalctl -u ogd -f -o cat

         
       
