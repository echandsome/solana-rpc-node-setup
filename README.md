# Solana-rpc-node-setup

A complete guide to setting up your own Solana RPC node with snapshot restore and Geyser plugin support.

## Requirements

- Ubuntu 22.04 or later
- 500GB+ NVMe SSD (for ledger)
- 16-core CPU / 64GB RAM (recommended)
- Public IP address
- Docker (optional, for Geyser plugins)

## Installation

1. **Install Dependencies**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install curl wget git build-essential pkg-config libssl-dev -y
   ```

2. **Install Solana**
   ```bash
   sh -c "$(curl -sSfL https://release.solana.com/v1.17.17/install)"
   export PATH="$HOME/.local/share/solana/install/active_release/bin:$PATH"
   solana --version
   ```

3. **Create Directories**
   ```bash
   mkdir -p ~/solana/ledger
   mkdir -p ~/solana/accounts
   ```

4. **Download Latest Snapshot**
   ```bash
   solana-validator \
     --ledger ~/solana/ledger \
     --accounts ~/solana/accounts \
     --rpc-port 8899 \
     --entrypoint entrypoint.mainnet-beta.solana.com:8001 \
     --expected-genesis-hash <GENESIS_HASH> \
     --expected-shred-version <SHRED_VERSION> \
     --snapshot-interval-slots 500 \
     --limit-ledger-size \
     --dynamic-port-range 8000-8020
   ```

   > Replace `<GENESIS_HASH>` and `<SHRED_VERSION>` by running:
   > ```bash
   > solana genesis-hash
   > solana shred-version
   > ```

5. **Set Up Geyser Plugin (Optional)**
   - Install Docker:
     ```bash
     sudo apt install docker.io docker-compose -y
     ```
   - Clone the Geyser Plugin repository and follow the plugin setup.

6. **Run Validator Node**
   ```bash
   solana-validator \
     --identity ~/solana/validator-keypair.json \
     --ledger ~/solana/ledger \
     --accounts ~/solana/accounts \
     --rpc-port 8899 \
     --private-rpc \
     --entrypoint entrypoint.mainnet-beta.solana.com:8001 \
     --expected-genesis-hash <GENESIS_HASH> \
     --expected-shred-version <SHRED_VERSION> \
     --wal-recovery-mode skip_any_corrupted_record \
     --no-untrusted-rpc \
     --enable-rpc-transaction-history \
     --enable-cpi-and-log-storage \
     --geyser-plugin-config geyser-config.json
   ```

## Useful Commands

- **Check Node Health**
  ```bash
  solana catchup <VALIDATOR_IDENTITY> http://127.0.0.1:8899
  ```

- **Check Logs**
  ```bash
  journalctl -u solana-validator -f
  ```

- **Monitor**
  ```bash
  solana gossip
  ```

## Notes

- You must open ports `8000-8020`, `8899` (RPC), and `8900` (gossip) in your firewall.
- Always keep your node updated with the latest Solana releases.
- For production environments, it’s recommended to use systemd services for automatic restart and monitoring.

---