# Juspay HyperSwitch Card Vault Setup Guide on AlmaLinux 9.6

This document provides a comprehensive step-by-step walkthrough to install and run the Juspay HyperSwitch Card Vault (hyperswitch-card-vault) in a **development** environment on **AlmaLinux 9.6** using **PostgreSQL**.

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [System Preparation](#system-preparation)
4. [Install Rust Toolchain](#install-rust-toolchain)
5. [PostgreSQL Setup](#postgresql-setup)
6. [Clone the Repository](#clone-the-repository)
7. [Key Generation](#key-generation)
8. [Configuration](#configuration)
9. [Source Code Modifications](#source-code-modifications)
10. [Database Migrations](#database-migrations)
11. [Building the Application](#building-the-application)
12. [Running the Vault](#running-the-vault)
13. [Testing the APIs](#testing-the-apis)
14. [Troubleshooting & Tips](#troubleshooting--tips)

## 1. Introduction

The Juspay HyperSwitch Card Vault provides a secure API to manage card data and encryption. This guide enables developers to set up and run the vault locally on AlmaLinux 9.6 for development and testing purposes.

**Important:** This guide is designed for development environments only. Production deployments require additional security configurations including SSL certificates, proper SELinux policies, and systemd service configuration.

## 2. Prerequisites

Before beginning the installation, ensure you have the following:

- A machine or virtual machine running **AlmaLinux 9.6** with root or sudo privileges
- **Internet connectivity** to download packages and Git repositories
- Minimum **4GB RAM** and **10GB free disk space**
- **PostgreSQL** version 12 or higher (will be installed in this guide)
- **Rust toolchain** (stable version will be installed via rustup)
- Basic familiarity with Linux shell commands, Git version control, and SQL databases
- **Python 3** with pip (for testing utilities)

## 3. System Preparation

Begin by updating your system and installing essential development tools.

Update system packages:
```bash
sudo dnf update -y
sudo dnf install -y epel-release
```

Install essential development tools and libraries:
```bash
sudo dnf install -y git curl openssl-devel clang cmake pkgconfig gcc gcc-c++ python3 python3-pip
```

These packages provide the necessary compilation tools and cryptographic libraries required for building the Rust application.

## 4. Install Rust Toolchain

Install the Rust programming language using rustup, which is the recommended installation method:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env
rustup default stable
```

Verify the installation was successful:
```bash
rustc --version
cargo --version
```

You should see version information for both the Rust compiler and Cargo package manager.

## 5. PostgreSQL Setup

Install and configure PostgreSQL as the database backend for the card vault.

Install PostgreSQL server and client tools:
```bash
sudo dnf install -y postgresql-server postgresql-contrib
```

Initialize the database cluster (this creates the initial database files):
```bash
sudo postgresql-setup --initdb
```

Start PostgreSQL service and enable it to start automatically on boot:
```bash
sudo systemctl enable --now postgresql
```

Create a dedicated database and user for the vault application:
```bash
sudo -u postgres psql <<SQL
CREATE USER vault_dev WITH PASSWORD 'vault_pass';
CREATE DATABASE vault_dev OWNER vault_dev;
GRANT ALL PRIVILEGES ON DATABASE vault_dev TO vault_dev;
\q
SQL
```

**Security Note:** The password 'vault_pass' is used for development only. Use a strong, unique password for production environments.

## 6. Clone the Repository

Create a workspace directory and clone the HyperSwitch Card Vault repository:

```bash
mkdir -p ~/workspace
cd ~/workspace
git clone https://github.com/juspay/hyperswitch-card-vault.git
cd hyperswitch-card-vault
git checkout main
```

Verify you are on the correct branch:
```bash
git branch --show-current
```

## 7. Key Generation

The card vault requires several cryptographic keys for secure operations. This section generates all necessary keys for development use.

### 7.1 Generate TLS Certificate

Create a self-signed certificate for HTTPS communication:
```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost"
cat cert.pem key.pem > identity.pem
```

### 7.2 Generate Application Keys

Generate RSA key pairs for encryption and signing operations:
```bash
openssl genrsa -out locker-private-key.pem 2048
openssl genrsa -out tenant-private-key.pem 2048
openssl rsa -in locker-private-key.pem -pubout -out locker-public-key.pem
openssl rsa -in tenant-private-key.pem -pubout -out tenant-public-key.pem
```

### 7.3 Generate Master Key

The vault includes a utility to create a master key for data encryption. Generate the master key using the built-in utility:

For development (master key only):
```bash
cargo run --bin utils -- master-key -w
```

**Important:** Save the generated master key output, as you will need it for the configuration file. The master key will be displayed in the terminal output.

## 8. Configuration

Create the main configuration file that defines how the vault operates.

Create the configuration directory and file:
```bash
mkdir -p config
vi config/config.toml
```

Add the following configuration content, replacing placeholder paths with your actual file locations:

```toml
[log.console]
enabled = true
level = "DEBUG"
log_format = "default"

[server]
host = "0.0.0.0"
port = 8080

[limit]
request_count = 100
duration = 60

[database]
username = "vault_dev"
password = "vault_pass"
host = "localhost"
port = 5432
dbname = "vault_dev"

[cache]
tti = 7200  # Time to idle: 2 hours
max_capacity = 5000

[secrets]
locker_private_key = "/home/$(whoami)/workspace/hyperswitch-card-vault/locker-private-key.pem"

[tenant_secrets]
public = { master_key = "YOUR_GENERATED_MASTER_KEY_HERE", schema = "public" }

[external_key_manager]
url = ""
cert = ""

[api_client]
identity = "/home/$(whoami)/workspace/hyperswitch-card-vault/identity.pem"
pool_max_idle_per_host = 10
```

**Configuration Notes:**
- Replace `YOUR_GENERATED_MASTER_KEY_HERE` with the master key generated in step 7.3
- Update file paths to match your actual installation directory
- The `request_count` limit has been increased from 1 to 100 for practical development use

## 9. Source Code Modifications

The default application expects configuration parameters that need to be modified for our setup.

Edit the main application entry point to accept custom configuration:
```bash
vi src/bin/locker.rs
```

Replace the entire file content with:
```rust
use tartarus::{logger, tenant::GlobalAppState};
use std::{env, path::PathBuf};

#[allow(clippy::expect_used)]
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    if cfg!(feature = "dev") {
        eprintln!("This is a dev build, not for production use");
    }

    // Get config path from command line arguments or use default
    let args: Vec<String> = env::args().collect();
    let config_path = args.get(1)
        .map(|s| PathBuf::from(s))
        .unwrap_or_else(|| PathBuf::from("config/config.toml"));

    println!("Loading configuration from: {}", config_path.display());

    // Initialize global configuration with custom path
    let mut global_config = tartarus::config::GlobalConfig::new_with_config_path(Some(config_path))
        .expect("Failed while parsing configuration file");

    // Set up logging based on configuration
    let _guard = logger::setup(
        &global_config.log,
        tartarus::service_name!(),
        [tartarus::service_name!(), "tower_http"],
    );

    // Validate configuration before proceeding
    global_config
        .validate()
        .expect("Failed to validate application configuration");

    // Fetch secrets defined in configuration
    global_config
        .fetch_raw_secrets()
        .await
        .expect("Failed to fetch raw application secrets");

    // Initialize application state
    let global_app_state = GlobalAppState::new(global_config).await;

    // Start the server
    tartarus::app::server_builder(global_app_state)
        .await
        .expect("Failed while building the server");

    Ok(())
}
```

This modification allows the application to read configuration from our custom `config/config.toml` file.

## 10. Database Migrations

Apply database schema migrations to create the necessary tables and indexes.

Install Diesel CLI tool for database migrations:
```bash
cargo install diesel_cli --no-default-features --features postgres
```

Run the database migrations:
```bash
diesel migration run --database-url "postgres://vault_dev:vault_pass@127.0.0.1:5432/vault_dev"
```

If migrations complete successfully, you will see output indicating which migrations were applied.

## 11. Building the Application

Compile the Rust application with all dependencies.

Build the application in development mode:
```bash
cargo build --features dev
```

The `--features dev` flag enables development-specific features and optimizations. For a production build, use:
```bash
cargo build --release
```

The compilation process may take several minutes as it downloads and compiles all dependencies.

## 12. Running the Vault

Start the card vault service with your custom configuration.

Run the application:
```bash
cargo run --features dev -- config/config.toml
```

You should see log output indicating:
- Configuration file loaded successfully
- Database connection established
- Server started and listening on port 8080

The application is now ready to accept API requests.

## 13. Testing the APIs

Verify that the vault is functioning correctly by testing its endpoints.

### 13.1 Health Check

Test the basic health endpoint:
```bash
curl http://127.0.0.1:8080/health
```

Expected response: `{"status": "ok"}` or similar health status information.

### 13.2 Payload Generation for Testing

For secure card data operations, you need to encrypt and sign payloads. Create a Python utility for testing:

Install required Python libraries:
```bash
pip3 install python-jose[cryptography]
```

Create a payload generation script:
```bash
cat > generate_payload.py << 'EOF'
#!/usr/bin/env python3
from jose import jwe, jws
import json
import sys

def generate_test_payload():
    # Read keys (update paths as needed)
    try:
        with open('locker-public-key.pem', 'r') as f:
            locker_public_key = f.read()
        with open('tenant-private-key.pem', 'r') as f:
            tenant_private_key = f.read()
    except FileNotFoundError as e:
        print(f"Error reading key files: {e}")
        sys.exit(1)

    # Step 1: Prepare sensitive card data
    sensitive_data = {
        "card_number": "4111111111111111",
        "name_on_card": "John Doe",
        "card_exp_month": "12",
        "card_exp_year": "2026"
    }

    # Step 2: Encrypt the sensitive data (JWE)
    jwe_token = jwe.encrypt(
        plaintext=json.dumps(sensitive_data),
        key=locker_public_key,
        algorithm='RSA-OAEP-256',
        encryption='A256GCM'
    )

    # Step 3: Sign the encrypted payload (JWS)
    jws_token = jws.sign(jwe_token, tenant_private_key, algorithm='RS256')

    # Step 4: Create final API payload
    payload = {
        "merchant_id": "test_merchant",
        "merchant_customer_id": "cust_123",
        "enc_card_data": jws_token,
        "ttl": 3600
    }

    return json.dumps(payload, indent=2)

if __name__ == "__main__":
    print(generate_test_payload())
EOF

chmod +x generate_payload.py
```

### 13.3 Creating Card Data

Generate and send a test payload to store card data:
```bash
python3 generate_payload.py > test_payload.json
curl -X POST http://127.0.0.1:8080/v1/card \
     -H "Content-Type: application/json" \
     -d @test_payload.json
```

A successful response will return a card reference token that can be used to retrieve the stored card data later.

### 13.4 Retrieving Card Data

To retrieve previously stored card data, you will need the card reference received from the card creation response. Use the following API call:

```bash
curl -X POST http://127.0.0.1:8080/data/retrieve \
     -H "Content-Type: application/json" \
     -H "x-tenant-id: public" \
     -d '{
       "merchant_id": "test_merchant",
       "merchant_customer_id": "cust_123",
       "card_reference": "<reference_id_received_after_card_add>"
     }'
```

Replace `<reference_id_received_after_card_add>` with the actual card reference token returned from the card creation API.

### 13.5 Decoding Encrypted Responses

The retrieve API returns encrypted card data that must be decrypted using the appropriate keys. Create a decoding utility to process these responses:

```bash
cat > decode_payload.py << 'EOF'
#!/usr/bin/env python3
from jose import jwe, jws
import json
import sys

def decode_encrypted_response(encrypted_response):
    # Read keys (update paths to match your installation)
    try:
        with open('locker-private-key.pem', 'r') as f:
            locker_private_key = f.read()
        with open('tenant-public-key.pem', 'r') as f:
            tenant_public_key = f.read()
    except FileNotFoundError as e:
        print(f"Error reading key files: {e}")
        sys.exit(1)

    # Step 1: Verify JWS signature and extract JWE token
    try:
        jwe_token = jws.verify(encrypted_response, tenant_public_key, algorithms=['RS256'])
    except Exception as e:
        print(f"JWS verification failed: {e}")
        sys.exit(1)

    # Step 2: Decrypt the JWE token
    try:
        decrypted_data = jwe.decrypt(jwe_token, locker_private_key)
    except Exception as e:
        print(f"JWE decryption failed: {e}")
        sys.exit(1)

    # Step 3: Parse and return the JSON result
    try:
        result = json.loads(decrypted_data)
        return json.dumps(result, indent=2)
    except json.JSONDecodeError as e:
        print(f"JSON parsing failed: {e}")
        sys.exit(1)

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 decode_payload.py '<encrypted_response>'")
        print("Replace <encrypted_response> with the actual encrypted response from the API")
        sys.exit(1)
    
    encrypted_response = sys.argv[1]
    decoded_result = decode_encrypted_response(encrypted_response)
    print("Decoded card data:")
    print(decoded_result)
EOF

chmod +x decode_payload.py
```

To use the decoding utility with an API response:
```bash
# Replace <encrypted_response_string> with the actual encrypted response from the retrieve API
python3 decode_payload.py '<encrypted_response_string>'
```

This will output the decrypted card data in a readable JSON format, allowing you to verify that the encryption and decryption process is working correctly.

## 14. Troubleshooting & Tips

Common issues and their solutions:

**Database Connection Issues:**
- Verify PostgreSQL is running: `sudo systemctl status postgresql`
- Check database credentials in `config/config.toml`
- Test database connection: `psql -h localhost -U vault_dev vault_dev`

**Build Failures:**
- Ensure all system dependencies are installed (step 3)
- Update Rust to the latest stable version: `rustup update stable`
- Clear build cache: `cargo clean` then rebuild

**Runtime Errors:**
- Check file permissions on key files: `chmod 600 *.pem`
- Verify all file paths in configuration are absolute and correct
- Enable detailed logging: `RUST_LOG=debug cargo run --features dev -- config/config.toml`

**Port Already in Use:**
- Check if another service is using port 8080: `sudo netstat -tlnp | grep 8080`
- Change the port in `config/config.toml` if needed

**Key Generation Issues:**
- Ensure OpenSSL is properly installed: `openssl version`
- Verify key file permissions and ownership

For additional debugging, enable verbose logging by setting the `RUST_LOG` environment variable to `debug` or `trace` level.

---
