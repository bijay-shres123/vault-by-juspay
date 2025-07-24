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

3. **Generate sample keys (development):**

   ```bash
   # JWE RSA key pair
   openssl genpkey -algorithm RSA -out config/jwe_private.pem -pkeyopt rsa_keygen_bits:2048
   openssl rsa -in config/jwe_private.pem -pubout -out config/jwe_public.pem

   # JWS RSA key pair
   openssl genpkey -algorithm RSA -out config/jws_private.pem -pkeyopt rsa_keygen_bits:2048
   openssl rsa -in config/jws_private.pem -pubout -out config/jws_public.pem
   ```

Adjust paths in `config.toml` accordingly:

```toml
[jwe]
private_key_path = "path-to-private-key-from-root/jwe_private.pem"
public_key_path  = "path-to-public-key-from-root/jwe_public.pem"

[jws]
private_key_path = "path-to-private-key-from-root/jws_private.pem"
public_key_path  = "path-to-public-key-from-root/jws_public.pem"
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
