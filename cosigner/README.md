## horcrux Dockerfile
# Base image
FROM ubuntu:22.04

# Install necessary tools
RUN apt-get update && apt-get install -y wget nano systemd

# Download and configure horcrux
RUN wget https://github.com/strangelove-ventures/horcrux/releases/download/v3.3.1/horcrux_linux-amd64 && \
    mv horcrux_linux-amd64 horcrux && \
    chmod +x horcrux && \
    mv horcrux /usr/bin/horcrux && \
    mkdir -p /usr/local/sbin && \
    cp /usr/bin/horcrux /usr/local/sbin/horcrux

# Create horcrux directory
RUN mkdir -p /root/.horcrux

# Set the entrypoint
ENTRYPOINT ["/usr/bin/horcrux"]
CMD ["--help"]


## Horcrux local setup
mkdir Horcrux
HORCRUX_PATH=$(pwd)/Horcrux
wget https://github.com/strangelove-ventures/horcrux/releases/download/v3.3.1/horcrux_linux-amd64

mv horcrux_linux-amd64 horcrux
sudo cp horcrux /usr/bin/
sudo cp horcrux /usr/local/sbin/horcrux

sudo nano /etc/systemd/system/horcrux.service

cat > horcrux.servie
[Unit]
Description= horcrux Signer For Namada
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
EOF

sudo systemctl daemon-reload
sudo systemctl enable horcrux.service
sudo systemctl start horcrux.service
sudo systemctl status horcrux.service

#### horcrux cluster 
horcrux config init \
  --node "tcp://10.168.0.1:1234" \
  --cosigner "tcp://10.168.1.1:2222" \
  --cosigner "tcp://10.168.1.2:2222" \
  --cosigner "tcp://10.168.1.3:2222" \
  --cosigner "tcp://10.168.1.4:2222" \
  --cosigner "tcp://10.168.1.5:2222" \
  --threshold 3 \
  --grpc-timeout 1000ms \
  --raft-timeout 1000ms \
  -o

mkdir -p ./config
sudo vim ./config/priv_validator_key.json

test key

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


## Horcrux Generate key for cosigner containers

Generate Required Configuration and Shards
The key files to be prepared are:

- ecies_keys.json [ horcrux create-ecies-shards --out ./shards --shards 5 ]
- {chain-id}_shard.json [horcrux create-ed25519-shards --chain-id "grand-1" --key-file ./config/priv_validator_key.json --out ./shards --threshold 3 --shards ]
- {chain-id}_priv_validator_state.json (modified)

1. Generate ECIES Keys
horcrux create-ed25519-shards --chain-id "grand-1" --key-file ./config/priv_validator_key.json --out ./shards --threshold 3 --shards 5

horcrux create-ecies-shards --out ./shards --shards 5


2. Generate and Split Validator Key
Export the private key from the validator node:
horcrux export-validator-key --output validator_key.json

<!-- Split the private key into shards (e.g., 3 shards, with 2 required to sign):
horcrux split-key --input validator_key.json --output-dir ./shards --threshold 2 -->

Prepare priv_validator_state.json (where I can find this?)

file structure for each cosigner
cosigner_1/
  └── .horcrux/
      ├── ecies_keys.json
      ├── {chain-id}_shard.json
      ├── {chain-id}_priv_validator_state.json


### On cosigner 1 and so on...
<!-- mkdir -p cosigner_1/.horcrux
cp ecies_keys.json cosigner_1/.horcrux/
cp shards/{chain-id}_shard_1.json cosigner_1/.horcrux/{chain-id}_shard.json
cp {chain-id}_priv_validator_state.json cosigner_1/.horcrux/ -->

#### configure each cosigner

this is the config file in each conatiner's /root/.horcrux path
chain_id: "{chain-id}"
cosigner:
  id: 1
  threshold: 2
peers:
  - id: 2
    address: "cosigner_2:12342"
  - id: 3
    address: "cosigner_3:12343"
rpc:
  listen_address: "tcp://0.0.0.0:12341"


sudo docker run --name cosigner_1 -v $(pwd)/cosigner_1:/root/.horcrux -p 12341:12341 horcrux-cosigner:v2 start
