# Noble Node Deployment with Cosigner using Horcrux

This repository provides an Ansible-based deployment setup for a Noble Node and Cosigner using Horcrux. It facilitates efficient configuration and management for full nodes and cosigner nodes in a blockchain environment.

## Project Structure

```plaintext
.
├── README.md
├── cosigner
│   ├── Dockerfile
│   ├── README.md
│   ├── ansible.cfg
│   ├── docker_install.yml
│   ├── horcrux.yml
│   └── hosts.ini
└── fullnode
    ├── Dockerfile
    ├── ansible.cfg
    ├── docker-compose.yml
    ├── docker_install.yml
    ├── full_node_cp.yml
    ├── hosts.ini
    └── validator.json
```

### `cosigner` Directory

- **`Dockerfile`**: Defines the containerized environment for the cosigner node.
- **`README.md`**: Documentation specific to cosigner setup.
- **`ansible.cfg`**: Configuration file for Ansible.
- **`docker_install.yml`**: Ansible playbook for installing Docker.
- **`horcrux.yml`**: Ansible playbook to set up and configure Horcrux.
- **`hosts.ini`**: Inventory file for defining hosts.

### `fullnode` Directory

- **`Dockerfile`**: Defines the containerized environment for the full node.
- **`ansible.cfg`**: Configuration file for Ansible.
- **`docker-compose.yml`**: Docker Compose file for orchestrating full node services.
- **`docker_install.yml`**: Ansible playbook for installing Docker.
- **`full_node_cp.yml`**: Ansible playbook to copy and configure the full node setup.
- **`hosts.ini`**: Inventory file for defining hosts.
- **`validator.json`**: Configuration for the validator node.

## Prerequisites

- **Ansible**: Ensure Ansible is installed and configured on your control node.
- **Docker**: Required for containerization on target nodes.
- **Horcrux**: Used for multi-party signing and cosigner configuration.

## Deployment Instructions

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-folder>
```

### 2. Configure Inventory

Update the `hosts.ini` files in both `cosigner` and `fullnode` directories to match your infrastructure.

### 3. Deploy Cosigner

Navigate to the `cosigner` directory and run the Ansible playbooks:

```bash
cd cosigner
ansible-playbook -i hosts.ini docker_install.yml
ansible-playbook -i hosts.ini horcrux.yml
```

### 4. Deploy Full Node

Navigate to the `fullnode` directory and run the Ansible playbooks:

```bash
cd fullnode
ansible-playbook -i hosts.ini docker_install.yml
ansible-playbook -i hosts.ini full_node_cp.yml
```

### 5. Validate the Deployment

- Ensure Docker containers are running correctly on both cosigner and full node machines.
- Check logs for any errors.

### 6. Set Up Validator

Once the full node synchronization is complete, follow these steps to configure the validator:

#### Step 1: Ensure Full Node Synchronization

- Use the following command to check if the full node has fully synchronized:
  ```bash
  noble status | jq
  ```
- Confirm that the `catching_up` field is `false` in the output.

#### Step 2: Create Validator

- ```bash
  nobled keys add <wallet_name>
  ```

#### Step 3: Generate and Retrieve Validator Address

- Create a wallet and retrieve your validator address using the following command:
  ```bash
  nobled keys show <wallet_name>
  ```
- Replace `<wallet_name>` with your desired wallet name. The output will display the validator’s address, public key, and additional details:
  ```plaintext
  - address: noble1xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    name: <wallet_name>
    pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AnXXXXXXXXXXXXXXXXXXXXXX"}'
    type: local
  ```
- Note the `address` for future steps.

#### Step 4: Add Funds to Wallet

- Use the Noble Faucet to add funds to your wallet:
  ```bash
  Visit https://faucet.circle.com/ and submit your wallet address.
  ```

#### Step 5: Submit a Create Validator Transaction

- Update the `pubkey` and `moniker` in `fullnode/validator.json` with the appropriate values from your setup.
- Use the blockchain CLI to create your validator:
  ```bash
  nobled tx staking create-validator /noble/validator.json --from <validator_address> --chain-id grand-1
  ```
- Replace `<validator_address>` with your address.

#### Step 6: Configure Validator to Use Cosigner

- Update the `horcrux` configuration to point to your validator.
- Ensure communication between the cosigner and validator is secured using mTLS.
- Verify the cosigner setup by checking logs:
  ```bash
  docker logs -f <cosigner_container_name>
  ```

#### Step 7: Verify Validator Status

- Check the status of your validator:
  ```bash
  noble query staking validator <validator_address>
  ```
- Replace `<validator_address>` with your validator’s address.

#### Step 8: Monitor Logs

- Ensure smooth operation by monitoring validator and node logs:
  ```bash
  docker logs -f <container_name>
  ```

## Monitoring and Alerting

### Sidecar for Monitoring

- Add a sidecar (e.g., Vector) to scrape system logs, node metrics, and Cosmos Tendermint metrics.
- Monitor key metrics such as:
  - Node synchronization status.
  - Disk space availability.

### Protecting the Validator

- Use a **sentry node architecture** to hide the validator from public access.
- Ensure the validator communicates only with the sentry nodes.

### Cosigner Deployment Best Practices

- Deploy cosigners across a mix of cloud providers and bare-metal servers with Hardware Security Modules (HSMs).
- Store cosigner shard secrets in secure secret management solutions like HashiCorp Vault or cloud-specific secret managers.
- Periodically rotate cosigners and implement an automated mechanism for secret rotation without manual intervention.
- Secure communication between cosigners using **mTLS**.
- Validators should communicate with cosigners via TLS authentication for added defense-in-depth.
- Use [Horcrux Proxy](https://github.com/strangelove-ventures/horcrux-proxy) to further reduce the attack surface.

## Security Hardening

### Operating System Hardening

- Ensure OS hardening measures are applied:
  - Disable root SSH login.
  - Ensure the new user is not in the `sudoers` group.
  - Enable SSH key-based authentication and disable password-based authentication.
  - Change the default SSH port.
  - Restrict SSH access to specific users.
  - Configure SSH login grace time.
  - Implement `fail2ban` for intrusion prevention.
  - Configure an SSH banner to warn unauthorized users.

### Firewall Configuration

- Use `ufw` to secure connections:
  - Allow only necessary ports (e.g., SSH, Tendermint).
  - Drop all other incoming traffic by default.

## Additional Information

- Refer to the `cosigner/README.md` for detailed cosigner configuration.
- Refer to `validator.json` for full node validator setup.