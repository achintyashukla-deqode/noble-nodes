Horcrux Setup and Configuration Guide

1. Cosigner Container Dockerfile

# Base image
FROM ubuntu:22.04

# Install necessary tools
RUN apt-get update && apt-get install -y wget nano systemd curl

# Download and configure horcrux
RUN wget https://github.com/strangelove-ventures/horcrux/releases/download/v3.3.1/horcrux_linux-amd64 && \
    mv horcrux_linux-amd64 horcrux && \
    chmod +x horcrux && \
    mv horcrux /usr/bin/horcrux && \
    mkdir -p /usr/local/sbin && \
    cp /usr/bin/horcrux /usr/local/sbin/horcrux

# Create horcrux directory
RUN mkdir -p /root/.horcrux

# Set the entrypoint to horcrux
ENTRYPOINT ["/usr/bin/horcrux"]

2. Horcrux Setup in VM

# Create a directory for Horcrux
mkdir Horcrux
HORCRUX_PATH=$(pwd)/Horcrux

# Download and install Horcrux
wget https://github.com/strangelove-ventures/horcrux/releases/download/v3.3.1/horcrux_linux-amd64
mv horcrux_linux-amd64 horcrux
sudo cp horcrux /usr/bin/
sudo cp horcrux /usr/local/sbin/horcrux

# Configure Horcrux as a systemd service
sudo nano /etc/systemd/system/horcrux.service

Example horcrux.service file:

[Unit]
Description=Horcrux Signer For Namada
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/Desktop/Horcrux
ExecStart=/usr/bin/horcrux start --home /root/Desktop/Horcrux
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target

# Reload systemd and enable the service
sudo systemctl daemon-reload
sudo systemctl enable horcrux.service
sudo systemctl start horcrux.service
sudo systemctl status horcrux.service

3. Horcrux Configuration Initialization

Run the following command to initialize the Horcrux configuration:

horcrux config init \
  --node "tcp://10.168.0.1:1234" \
  --cosigner "tcp://10.168.1.1:2222" \
  --cosigner "tcp://10.168.1.2:2222" \
  --cosigner "tcp://10.168.1.3:2222" \
  --cosigner "tcp://10.168.1.4:2222" \
  --cosigner "tcp://10.168.1.5:2222" \
  --threshold 3 \
  --grpc-timeout 1000ms \
  --raft-timeout 1000ms

4. Copy Private Validator Key from Fullnode to Signer VM

mkdir -p ./config
sudo vim ./config/priv_validator_key.json

Example private validator key:

{
  "address": "8B9F3F9F19C69CE79F888DB29D0E7CBBBF8498C1",
  "pub_key": {
    "type": "tendermint/PubKeyEd25519",
    "value": "BEUjC5tNACjtDZu06kOCVbOtMrdqfhHzX6eMc6WzF1A="
  },
  "priv_key": {
    "type": "tendermint/PrivKeyEd25519",
    "value": "v9g6lXU7pE3FhFHhZZZZZMdFhkd5vQhvX54x7dXwvjw="
  }
}

5. Horcrux Cosigner Containers Configuration

File Structure for Each Cosigner

cosigner_x/
  └── .horcrux/
      ├── ecies_keys.json
      ├── config.yaml
      ├── {chain-id}_shard.json
      ├── {chain-id}_priv_validator_state.json

Example config.yaml File for Cosigners

signMode: threshold
thresholdMode:
  threshold: 3
  cosigners:
  - shardID: 1
    p2pAddr: tcp://10.168.1.1:2222
  - shardID: 2
    p2pAddr: tcp://10.168.1.2:2222
  - shardID: 3
    p2pAddr: tcp://10.168.1.3:2222
  - shardID: 4
    p2pAddr: tcp://10.168.1.4:2222
  - shardID: 5
    p2pAddr: tcp://10.168.1.5:2222
  grpcTimeout: 1000ms
  raftTimeout: 1000ms
rpc:
  listen_address: "tcp://0.0.0.0:12341"
chainNodes:
- privValAddr: tcp://10.168.0.1:1234
debugAddr: ""
grpcAddr: ""
maxReadSize: 1048576

Key Files Required

ecies_keys.json

Generated using:

horcrux create-ecies-shards --out ./shards --shards 5

{chain-id}_shard.json

Generated using:

horcrux create-ed25519-shards \
  --chain-id "grand-1" \
  --key-file ./config/priv_validator_key.json \
  --out ./shards \
  --threshold 3 \
  --shards 5

{chain-id}_priv_validator_state.json

Automatically created by Horcrux.

6. Running Cosigner Containers

Run the following command for each cosigner container:

sudo docker run --name cosigner_X \
  -v $(pwd)/cosigner_X:/root/.horcrux \
  -p 1234X:12341 horcrux-cosigner:v2 start

Replace X with the respective cosigner ID (e.g., 1, 2, 3, etc.).