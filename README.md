# Noble Node Deployment with Cosigner using Horcrux

This repository contains an Ansible-based deployment setup for a Noble Node and Cosigner using Horcrux. The structure ensures efficient configuration and management for full nodes and cosigner nodes in a blockchain setup.

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

Navigate to the `cosigner` directory and run the Ansible playbook:

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

1. **Copy Validator Configuration**:
   Ensure `validator.json` is properly configured with your validator details.

2. **Submit a Create Validator Transaction**:
   Use the blockchain CLI to submit a `create-validator` transaction. Example:

   ```bash
   noble tx staking create-validator \
     --amount=<stake_amount> \
     --pubkey=$(noble tendermint show-validator) \
     --moniker=<validator_name> \
     --chain-id=<chain_id> \
     --from=<key_name>
   ```

3. **Verify Validator Status**:
   Check the status of your validator:

   ```bash
   noble query staking validator <validator_address>
   ```

4. **Monitor Logs**:
   Keep track of validator and node logs to ensure smooth operation:

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


## License

This project is licensed under the [MIT License](LICENSE).
