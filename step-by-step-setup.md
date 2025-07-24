## Juspay HyperSwitch (Card Vault) Setup Guide on AlmaLinux 9.6

This document provides a step-by-step walkthrough to install and run the Juspay HyperSwitch Card Vault (hyperswitch-card-vault) in a **development** environment on **AlmaLinux 9.6** using **PostgreSQL**.

---

### Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [System Preparation](#system-preparation)
4. [Install Rust Toolchain](#install-rust-toolchain)
5. [PostgreSQL Setup](#postgresql-setup)
6. [Clone the Repository](#clone-the-repository)
7. [Configuration](#configuration)
8. [Database Migrations](#database-migrations)
9. [Building the Application](#building-the-application)
10. [Running the Vault](#running-the-vault)
11. [Testing the APIs](#testing-the-apis)
12. [Troubleshooting & Tips](#troubleshooting--tips)

---

## 1. Introduction

The Juspay HyperSwitch Card Vault provides a secure API to manage card data and encryption. This guide will help developers set up and run the vault locally on AlmaLinux 9.6 for development and testing.

## 2. Prerequisites

Ensure you have the following:

* A machine or VM running **AlmaLinux 9.6** with root or sudo privileges.
* **Internet connectivity** to fetch packages and Git repositories.
* **Development environment** only (no SSL, systemd, or SELinux configuration covered).
* **PostgreSQL** (v12+) installed locally.
* **Rust toolchain** (stable) will be installed via rustup.
* Basic familiarity with shell commands, Git, and SQL.

## 3. System Preparation

1. **Update system packages:**

   ```bash
   sudo dnf update -y
   sudo dnf install -y epel-release
   ```

2. **Install essential tools:**

   ```bash
   sudo dnf install -y git curl openssl-devel clang cmake pkgconfig
   ```

## 4. Install Rust Toolchain

We'll use `rustup` to install the stable Rust toolchain.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
rustup default stable
```

Verify:

```bash
rustc --version
cargo --version
```

## 5. PostgreSQL Setup

1. **Install PostgreSQL server and client:**

   ```bash
   sudo dnf install -y postgresql-server postgresql-contrib
   ```

2. **Initialize database cluster:**

   ```bash
   sudo postgresql-setup --initdb
   ```

3. **Start and enable PostgreSQL service:**

   ```bash
   sudo systemctl enable --now postgresql
   ```

4. **Create development database & user:**

   ```bash
   sudo -u postgres psql <<SQL
   CREATE USER vault_dev WITH PASSWORD 'vault_pass';
   CREATE DATABASE vault_dev OWNER vault_dev;
   \q
   SQL
   ```

## 6. Clone the Repository

```bash
cd ~/workspace
git clone https://github.com/juspay/hyperswitch-card-vault.git
cd hyperswitch-card-vault
git checkout main
```
## Generate identity.pem

generate identity:
```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost"

cert.pem key.pem > identity.pem
```
**Generate sample keys (development):**

   ```bash
      openssl genrsa -out locker-private-key.pem 2048
      openssl genrsa -out tenant-private-key.pem 2048
      openssl rsa -in locker-private-key.pem -pubout -out locker-public-key.pem
      openssl rsa -in tenant-private-key.pem -pubout -out tenant-public-key.pem
   ```

Adjust paths in `config.toml` accordingly

**Generate master key

The vault includes a utility to create a master key (and optionally only the master key):

# Only master key without custodians
```bash
cargo run --bin utils -- master-key -w
```

# Full key bundle (master + custodians)
```bash
cargo run --bin utils -- master-key
```
## 7. Configuration

1. **Create a config.tom file inside the config foler:**

   ```bash
   vi config/config.tom
   ```

   ```bash
      [log.console]
      enabled = true
      level = "DEBUG"
      log_format = "default"
      
      [server]
      host = "0.0.0.0"
      port = 8080
      
      [limit]
      request_count = 1
      duration = 60
      
      [database]
      username = "vault_dev"
      password = "vault_pass"
      host = "localhost"
      port = 5432
      dbname = "vault_dev"
      
      [cache]
      tti = 7200 # i.e. 2 hours
      max_capacity = 5000
      
      [secrets]
      locker_private_key = "/root/git_hyper/hyperswitch-card-vault/locker-private-key.pem"
      
      [tenant_secrets]
      public = { master_key = "<your master key>", schema = "public" }
      [external_key_manager]
      url = ""
      cert= ""
      
      [api_client]
      identity = "/root/git_hyper/hyperswitch-card-vault/identity.pem"
      pool_max_idle_per_host = 10
   ```

2. **Edit `config/config.toml`:**

   * Under `[server]`, set `bind_addr = "127.0.0.1:8080"`.
   * Under `[database]`, configure:

     ```toml
     database_url = "postgres://vault_dev:vault_pass@127.0.0.1:5432/vault_dev"
     ```
   * Under `[encryption]`, provide paths or base64 strings for JWE/JWS keys.

Also edit src/bin/locker.rs for rust to accept custom config from config/config.toml
```bash
vi src/bin/locker.rs
```
```bash
src/bin/locker.rs 
use tartarus::{logger, tenant::GlobalAppState};
use std::{env, path::PathBuf};

#[allow(clippy::expect_used)]
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    if cfg!(feature = "dev") {
        eprintln!("This is a dev build, not for production use");
    }

    // 🛠️ Get config path from CLI or default
    let args: Vec<String> = env::args().collect();
    let config_path = args.get(1).map(|s| PathBuf::from(s)).unwrap_or_else(|| PathBuf::from("config/config.toml"));

    println!("Loading config from: {}", config_path.display());

    // ✅ Use new_with_config_path
    let mut global_config =
        tartarus::config::GlobalConfig::new_with_config_path(Some(config_path))
        .expect("Failed while parsing config");

    let _guard = logger::setup(
        &global_config.log,
        tartarus::service_name!(),
        [tartarus::service_name!(), "tower_http"],
    );

    global_config
        .validate()
        .expect("Failed to validate application configuration");

    global_config
        .fetch_raw_secrets()
        .await
        .expect("Failed to fetch raw application secrets");

    let global_app_state = GlobalAppState::new(global_config).await;

    tartarus::app::server_builder(global_app_state)
        .await
        .expect("Failed while building the server");

    Ok(())
}
```

## 8. Database Migrations

Apply Diesel migrations to create necessary tables:

```bash
# Install Diesel CLI if not present
cargo install diesel_cli --no-default-features --features postgres

diesel migration run --database-url "postgres://vault_dev:vault_pass@127.0.0.1:5432/vault_dev"
```

## 9. Building the Application

Compile the project:

```bash
cargo build
```

For a faster development build, you can pass:

```bash
cargo build --features dev
```

## 10. Running the Vault

Run in development mode:

```bash
cargo run -- --config config/config.toml
```

You should see output indicating the server has started on port 8080.

### 11.1 Payload Generation & Testing

Create a helper script `generate_payload.py` to encrypt and sign card data before sending to the vault:

```python
# generate_payload.py
from jose import jwe, jws
import json

# Read keys
with open('config/jwe_public.pem', 'r') as f:
    locker_public_key = f.read()
with open('config/jws_private.pem', 'r') as f:
    tenant_private_key = f.read()

# Step 1: Encrypt the actual sensitive data
sensitive_data = {
    "card_number": "4111111111111111",
    "name_on_card": "John Doe",
    "card_exp_month": "12",
    "card_exp_year": "2026"
}

jwe_token = jwe.encrypt(
    plaintext=json.dumps(sensitive_data),
    key=locker_public_key,
    algorithm='RSA-OAEP-256',
    encryption='A256GCM'
)

# Step 2: Sign the JWE payload
jws_token = jws.sign(jwe_token, tenant_private_key, algorithm='RS256')

# Step 3: Final payload
payload = {
    "merchant_id": "test_merchant",
    "merchant_customer_id": "cust_123",
    "enc_card_data": jws_token,
    "ttl": 3600
}

print(json.dumps(payload, indent=2))
```

Run it to produce the JSON payload, then send it:

```bash
python3 generate_payload.py > payload.json
curl -X POST http://127.0.0.1:8080/v1/card \
     -H "Content-Type: application/json" \
     -d @payload.json
```

### 11.2 Payload Decryption & Verification

Use `decode_payload.py` to verify and decrypt responses (e.g., when retrieving card data):

```python
# decode_payload.py
from jose import jwe, jws
import json

# Read keys
with open('config/jwe_private.pem', 'r') as f:
    locker_private_key = f.read()
with open('config/jws_public.pem', 'r') as f:
    tenant_public_key = f.read()

# Your encrypted response from the API
encrypted_response = "<API_RESPONSE_JWS_STRING>"

# Step 1: Verify and extract the JWE token from JWS
jwe_token = jws.verify(encrypted_response, tenant_public_key, algorithms=['RS256'])

# Step 2: Decrypt the JWE token
decrypted_data = jwe.decrypt(jwe_token, locker_private_key)

# Step 3: Parse the JSON
result = json.loads(decrypted_data)
print(json.dumps(result, indent=2))
```

Replace `<API_RESPONSE_JWS_STRING>` with the actual response and run:

```bash
python3 decode_payload.py
```

## 11. Testing the APIs

Use `curl` to verify basic endpoints:

```bash
# Health check
curl http://127.0.0.1:8080/health

# Create a vault entry (replace with proper JSON)
curl -X POST http://127.0.0.1:8080/v1/card -H "Content-Type: application/json" -d '{"number":"4111111111111111","expiry_month":12,"expiry_year":2025}'
```


## 12. Troubleshooting & Tips

* Check logs for errors in terminal output.
* Ensure database URL is correct and database is running.
* If migrations fail, inspect `migrations/` folder for issues.
* Use `RUST_LOG=debug` to enable verbose logging:

  ```bash
  RUST_LOG=debug cargo run -- --config config/config.toml
  ```

---

*Guide authored by Bijay Shrestha.*
