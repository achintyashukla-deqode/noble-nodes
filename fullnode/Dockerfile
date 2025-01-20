FROM golang:1.23.4-bullseye

# Install required packages
RUN apt-get update && apt-get install -y \
    wget \
    lz4 \
    jq \
    git \
    make \
    && rm -rf /var/lib/apt/lists/*

# Download and install Noble
RUN git clone https://github.com/strangelove-ventures/noble noble && \
    cd noble && \
    git checkout v8.0.0-rc.4 && \
    make install

# Set working directory
WORKDIR /noble

# Initialize Noble node with a moniker
ENV MONIKER=deq-noble-test
RUN nobled init ${MONIKER} --chain-id grand-1

# Configure node
RUN sed -i 's/bootstrap-peers = ""/bootstrap-peers = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:21556"/' ~/.noble/config/config.toml && \
    sed -i 's/pruning = "default"/pruning = "custom"/g' /root/.noble/config/app.toml && \
    sed -i 's/pruning-keep-recent = "0"/pruning-keep-recent = "100"/g' /root/.noble/config/app.toml && \
    sed -i 's/pruning-interval = "0"/pruning-interval = "10"/g' /root/.noble/config/app.toml && \
    sed -i 's/pruning-keep-every = "0"/pruning-keep-every = "0"/g' /root/.noble/config/app.toml && \
    sed -i 's/indexer = "kv"/indexer = "null"/g' /root/.noble/config/config.toml && \
    PEERS=6e43f91fbc6ab984cb3241feeab37b33d222bc82@65.109.67.126:26656,f2f0dc25f7d041b41989de3de8f5606418affe8c@188.34.180.243:26656,63e95eee5e07ba055cdaa00d8ab4f0c8f9339f10@3.127.145.33:26686,3ff836a6b2664083931689ce9aa955cd7b8c0baf@202.61.206.101:27358 && \
    sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.noble/config/config.toml


# State-Sync Server Setup
RUN SNAP_RPC="https://noble-testnet-rpc.polkachu.com:443" && \
    LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height) && \
    BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)) && \
    TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
    sed -i.bak 's/^enable =.*/enable = true/' $HOME/.noble/config/config.toml && \
    sed -i.bak "s|^rpc_servers =.*|rpc_servers = \"$SNAP_RPC,$SNAP_RPC\"|" $HOME/.noble/config/config.toml && \
    sed -i.bak "s|^trust_height =.*|trust_height = $BLOCK_HEIGHT|" $HOME/.noble/config/config.toml && \
    sed -i.bak "s|^trust_hash =.*|trust_hash = \"$TRUST_HASH\"|" $HOME/.noble/config/config.toml
 
# Download the snapshot
RUN wget -O noble_21869078.tar.lz4 https://snapshots.polkachu.com/testnet-snapshots/noble/noble_21869078.tar.lz4 --inet4-only && \ 
    # Back up priv_validator_state.json if needed
    cp ~/.noble/data/priv_validator_state.json  ~/.noble/priv_validator_state.json && \
    # Reset node state
    nobled tendermint unsafe-reset-all --home $HOME/.noble && \
    lz4 -c -d noble_21869078.tar.lz4  | tar -x -C $HOME/.noble && \
    cp ~/.noble/priv_validator_state.json  ~/.noble/data/priv_validator_state.json && \
    rm -v noble_21869078.tar.lz4 

# Download genesis file
RUN wget -O genesis.json https://raw.githubusercontent.com/strangelove-ventures/noble-networks/main/testnet/grand-1/genesis.json --inet4-only && \
    mv genesis.json ~/.noble/config/ 
    
COPY ./validator.json /noble/validator.json

# Expose required ports
# 26656: P2P
# 26657: RPC
# 1317: REST API    
# 9090: gRPC
EXPOSE 26656 26657 1317 9090

# Start the node
CMD ["nobled", "start"]