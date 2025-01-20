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

# Copy the cosigner configuration, keys, and shard files into the container

# Set the entrypoint to horcrux
ENTRYPOINT ["/usr/bin/horcrux"]
