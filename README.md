The Hyperswitch Card Vault (Tartarus) is an open-source, highly performant, and secure system designed for the storage of sensitive payment information, such as payment card details and bank account information. It is a critical component within the broader Hyperswitch ecosystem, built using Rust, and emphasizes security by design, including PCI DSS compliance, multi-layered encryption (JWE & JWS), and a robust key management hierarchy involving public/private keys, custodian keys, and a master key.

# Hyperswitch Card Vault (Tartarus) Documentation

## 1. Overview
### 1.1 Introduction to Hyperswitch Card Vault
The **Hyperswitch Card Vault**, also referred to by its internal codename **"Tartarus,"** is an **open-source, highly performant, and secure system** designed for the storage of sensitive payment information, such as payment card details and bank account information , . It is a critical component within the broader Hyperswitch ecosystem, which also includes an application server, a Software Development Kit (SDK), and a control center. The primary objective of the Card Vault is to provide a **secure repository that adheres to Payment Card Industry Data Security Standard (PCI DSS) requirements**, ensuring that sensitive data is handled and stored in a compliant manner. Built using the **Rust programming language**, the vault emphasizes both security and performance. It employs a **multi-layered security approach**, incorporating various cryptographic techniques, including public/private key encryption, custodian keys, and a master key, to protect data both at rest and in transit. This documentation aims to provide a comprehensive overview of the vault's architecture, its constituent components, the configuration process, and the procedures for setting it up and integrating it into a payment infrastructure. The design philosophy behind Tartarus is to offer a **polymorphic solution** capable of handling and storing diverse types of sensitive information, thereby ensuring scalability and broad coverage for various payment methods and processors , .

The Hyperswitch Card Vault is designed to be a **highly performant and secure locker**, capable of handling various types of sensitive information beyond just payment card details, such as bank account information , . This versatility makes it a valuable tool for merchants and payment service providers who need a reliable and compliant way to store and manage sensitive customer data. The system's architecture is built with a strong emphasis on security, incorporating **GDPR-compliant Personal Identifiable Information (PII) storage mechanisms** and employing secure encryption algorithms to meet the stringent requirements of PCI DSS , . The open-source nature of the project allows for transparency and community involvement, fostering trust and enabling organizations to customize and extend its functionality to meet their specific needs. The documentation serves as a guide for understanding how to deploy, configure, and utilize the Card Vault effectively within the Hyperswitch payment orchestration platform or as a standalone secure storage solution.

### 1.2 Key Features and Benefits
The Hyperswitch Card Vault offers a range of features and benefits centered around security, compliance, and operational efficiency. A core feature is its **enhanced security**, achieved by tokenizing and securely storing sensitive card details, which significantly reduces the risk of data breaches and simplifies compliance efforts . This tokenization process replaces actual card data with a unique identifier (token), which is then used for transaction processing, ensuring that sensitive information does not need to be handled extensively by merchant systems . The system is designed in a **polymorphic manner**, allowing it to handle and store various types of sensitive information, making it highly scalable and adaptable to diverse payment methods and processors , . This polymorphic design enables applications to store customer payment method details in a processor-agnostic way and can be extended to securely store other types of customer information .

Another significant benefit is the **improved user experience**. By allowing customers to save their card details securely, they can reuse these saved cards for future transactions, significantly reducing friction during the checkout process and leading to higher conversion rates . This is particularly beneficial for e-commerce businesses and subscription-based services. Furthermore, the vault facilitates **seamless recurring payments**. Automatic updates to tokenized card details, such as when a card expires or is reissued, ensure uninterrupted subscription payments, which helps in minimizing customer churn . The Hyperswitch Card Vault also boasts **global compatibility**, supporting various payment processors and adhering to international standards like PCI DSS and PCI SSS (Security Scanning Standard) . This makes it a suitable solution for businesses operating in multiple regions or dealing with a diverse customer base. The architecture also supports **multi-tenancy**, allowing different merchants or entities (tenants) to utilize the vault with their own set of keys and configurations, ensuring data isolation and customized security policies .

### 1.3 PCI Compliance and Security
The Hyperswitch Card Vault (Tartarus) is engineered with a strong emphasis on security and compliance, particularly with the **Payment Card Industry Data Security Standard (PCI DSS)**. The system is designed to meet industry-leading security standards, including **PCI DSS v4.0, ISO 27001:2022, and GDPR**, ensuring robust protection for stored sensitive data , . To achieve this, Tartarus implements **bank-grade 256-bit AES encryption** for data at rest and utilizes secure encryption algorithms for data in transit , . The architecture incorporates a **multi-layered security model** that involves several types of keys, including public/private key pairs for encrypting and signing communications, custodian keys for managing access to the master key, and the master key itself, which is crucial for the locker's operation , . Furthermore, the vault is designed with **GDPR-compliant personal identifiable information (PII) storage mechanisms** , . Communications between the tenant (e.g., a merchant's application) and the locker (the core data storage component) are secured using **JSON Web Encryption (JWE) for confidentiality and JSON Web Signature (JWS) for integrity and authenticity**, ensuring end-to-end protection of sensitive information throughout its lifecycle within the Hyperswitch ecosystem , . The system's design inherently aims to **reduce the PCI DSS compliance scope for merchants** by ensuring that raw card data remains within the secure confines of the vault , .

The security measures extend to how data is stored within the vault. When card data is received, it is first signed by the Hyperswitch App Server using its private key to ensure integrity. This signed data is then encrypted using the Card Vault's public key for confidentiality during transmission . Upon arrival at the Card Vault, the data is validated using the App Server's public key, decrypted, and then **re-encrypted internally using AES encryption** before being stored in the database. The database itself is often encrypted at rest, providing an additional layer of protection . The system also incorporates features like **internal hashing checks** to prevent data duplication and ensure data consistency , . For businesses handling large volumes of transactions, the Card Vault's architecture supports **PCI DSS Level 1 compliance**, the most stringent level of certification . The documentation and setup guides emphasize secure practices, such as generating keys in a secure environment and managing access to sensitive key material carefully , . Techniques like `zeroize` are also employed to prevent unencrypted card details from lingering in memory, mitigating risks from memory scraping attacks .

## 2. Architecture
### 2.1 Core Design Principles
The Hyperswitch Card Vault (Tartarus) is architected around several core design principles aimed at delivering a secure, performant, and scalable solution for sensitive data storage. A primary principle is **security by design**, which is evident in its multi-layered encryption strategy, adherence to PCI DSS and other security standards, and the use of robust cryptographic protocols like JWE and JWS for all communications , . The system is built using **Rust**, a language known for its memory safety and performance characteristics, contributing to the overall reliability and efficiency of the vault , . Another key principle is **modularity**, allowing the Card Vault to operate as a standalone service or integrate seamlessly with other components of the Hyperswitch ecosystem, such as the app server, SDK, and control center, as well as with existing payment stacks , . This modularity facilitates flexibility in deployment and allows businesses to adopt the vault according to their specific needs. The architecture also emphasizes **statelessness** for the vault itself, relying on the application server for data persistence, which can simplify scalability and resilience , . Furthermore, the design supports **polymorphism**, enabling the vault to handle and store various types of sensitive information beyond just card data, making it adaptable for a wide range of payment methods and processors , .

The design also prioritizes **performance and scalability**. Being built in Rust, the vault benefits from the language's performance characteristics and memory safety guarantees. The polymorphic design allows it to handle and store various types of sensitive information beyond just card details, making it adaptable for a wide range of payment methods and future requirements , . The use of a **middleware for communication** allows for a separation of concerns, where the application logic and encryption logic are decoupled. This also facilitates the extension of APIs without needing to modify the core encryption code . The architecture supports **multi-tenancy**, enabling a single vault instance to securely serve multiple merchants, each with their own isolated data and key configurations. This is crucial for payment service providers or platforms that cater to numerous clients. The overall design aims to provide a robust, secure, and flexible solution for sensitive data storage that minimizes the compliance burden on users.

### 2.2 Modularity and Integration
A cornerstone of the Hyperswitch Card Vault's (Tartarus) architecture is its inherent **modularity**, which facilitates flexible integration with a variety of systems and deployment scenarios , . The vault is designed to function as an **independent, secure component** within the broader Hyperswitch ecosystem, which includes an application server, SDKs, and a control center. This independence allows it to be deployed and managed separately, enhancing security by isolating sensitive data storage from other application logic. The modular design also means that the Card Vault can be integrated into existing payment stacks without requiring a full overhaul of the current infrastructure. It provides **well-defined APIs**, typically secured with JWE and JWS, for interactions, allowing other systems, whether part of Hyperswitch or third-party applications, to communicate with it in a standardized and secure manner , . This approach enables businesses to adopt the Card Vault's capabilities, such as PCI-compliant data storage and tokenization, incrementally. For instance, merchants can use the Hyperswitch Vault SDK to collect and store card data securely, ensuring sensitive information never touches their systems, or they can opt for direct server-to-server integration if they have existing PCI-compliant workflows , . The ability to **tokenize cards across multiple payment processors** through a single unified API further underscores its integration capabilities , .

Integration with the Hyperswitch application typically occurs via a **middleware component** , . This middleware handles the communication between the application server and the vault, ensuring that all requests and responses are properly encrypted and signed using JWE and JWS algorithms , . The vault exposes well-defined APIs (e.g., `/data` and `/cards` endpoints for CRD operations) that facilitate this integration , . The use of standardized cryptographic protocols and API endpoints simplifies the integration process for developers. Furthermore, the **multi-tenant capability** allows a single vault instance to serve multiple merchants, with each tenant having its own set of keys and configurations, ensuring data isolation and security . This makes the Hyperswitch Card Vault a versatile solution suitable for a variety of deployment scenarios, from small businesses to large enterprises with complex payment infrastructures.

### 2.3 Stateless Operation and Data Persistence
The Hyperswitch Card Vault (Tartarus) is designed as a **stateless service**, a key architectural decision that influences its operational characteristics and interaction with other components , . Being stateless means that the **vault itself does not store the persistent state of the sensitive data it manages**. Instead, it relies on an external application server or a dedicated database for data persistence. This separation of concerns enhances security by isolating the encryption and decryption logic within the vault from the actual storage of the encrypted data blobs. When a request to store data (like card details) is received, the vault encrypts the information and then typically passes the encrypted payload back to the calling application server, which is then responsible for storing it in a database. Conversely, when data needs to be retrieved, the application server fetches the encrypted blob from its persistent store and sends it to the vault for decryption. This stateless design **simplifies the scaling of the vault service**, as instances can be added or removed without concerns about data locality or synchronization of in-memory state. It also improves resilience, as the failure of a single vault instance does not lead to data loss, provided the persistent storage managed by the application server remains intact. The `config.toml` file includes database connection details, indicating that while the vault interacts with a database, its primary role is secure data transformation rather than long-term state management , .

The responsibility for managing the lifecycle of the data, including its storage, retrieval, and association with specific merchants and customers, lies with the application server or the middleware . The vault's role is primarily to **securely store and retrieve the encrypted sensitive data** based on the requests it receives. For example, when card data needs to be stored, the application server prepares the data, encrypts it, and sends it to the vault. The vault then stores this encrypted payload and returns a unique identifier (e.g., a token or a reference ID) that the application server can use for future retrieval or processing , . This separation of concerns, where the vault handles the secure storage and cryptographic operations while the application server manages the business logic and data persistence, contributes to a more robust and manageable system architecture. The internal hashing checks within the vault help prevent data duplication, ensuring data integrity even though the vault itself is stateless regarding the broader application context , .

### 2.4 Communication Security (JWE & JWS)
Communication security is paramount in the Hyperswitch Card Vault (Tartarus) architecture, ensuring that all interactions involving sensitive data are protected against unauthorized access and tampering , . The vault employs a dual mechanism of **JSON Web Encryption (JWE) and JSON Web Signature (JWS)** to secure payloads in transit and at rest. **JWE is utilized to encrypt the payload**, providing confidentiality by ensuring that the sensitive data, such as card details, is unreadable to anyone without the appropriate decryption key , . This means that even if the communication channel were compromised, the encrypted data would remain secure. Complementing JWE, **JWS is used to sign the payload**, which guarantees its integrity and authenticity , . The signature allows the recipient to verify that the data has not been altered during transmission and that it indeed originated from a trusted source (the legitimate tenant or the locker). This combination of JWE for encryption and JWS for signing is applied to all communications between the tenant (e.g., a merchant's application) and the locker (the core data storage component of the vault). The public and private keys mentioned in the key hierarchy are instrumental in these processes; for instance, the tenant might encrypt data using the locker's public key (for JWE) and sign it with its own private key (for JWS), while the locker would decrypt with its private key and verify with the tenant's public key , . This robust cryptographic protection ensures **end-to-end security** for sensitive information throughout its lifecycle within the Hyperswitch ecosystem , .

The Hyperswitch application server, acting as the tenant, typically signs the data with its private key before encrypting it with the locker's public key. The locker then verifies the signature using the tenant's public key and decrypts the payload using its private key . This dual mechanism of JWE for encryption and JWS for signing creates a robust security layer for all communications. The official documentation and setup guides detail the generation and configuration of the public/private key pairs required for these JWE and JWS operations, emphasizing the use of strong cryptographic algorithms like **RSA-OAEP-256 for encryption and RS256 for signing** , . This ensures that sensitive data remains protected throughout its lifecycle, from the point of collection to storage and subsequent retrieval. The Python scripts `generate_payload.py` and `decode_payload.py` provided by the user illustrate this workflow, showing how to correctly encrypt and sign payloads using JWE and JWS before sending them to the vault, and how to verify and decrypt responses.

## 3. Key Components
### 3.1 Locker: Secure Data Storage
The **Locker** is the central and most critical component of the Hyperswitch Card Vault (Tartarus), responsible for the **secure management and storage of sensitive data**, primarily payment card information , . It functions as the core engine that handles the cryptographic operations and data management tasks. The Locker supports standard **CRUD (Create, Read, Update, Delete) operations**, which are exposed via API endpoints, such as `/data` for general-purpose data storage and `/cards` for specific card-related operations , . When card data is stored, it is associated with a **unique combination of merchant and customer identifiers**. This ensures that data is segregated by tenant and customer, supporting multi-tenancy and individual customer data management. A crucial feature of the Locker is its **internal hashing mechanism**, which is designed to prevent data duplication , . This means that if the same card details are attempted to be stored multiple times for the same merchant-customer pair, the Locker can detect and manage this, potentially returning a reference to the already stored data rather than creating redundant entries. The Locker's operations are dependent on the successful decryption of the Master Key, which in turn requires the Custodian Keys, adding a significant layer of security before the Locker can become fully operational and process sensitive data , . All interactions with the Locker are secured using JWE for encryption and JWS for signing, ensuring data confidentiality, integrity, and authenticity , .

The locker's design emphasizes security at its core. It operates based on a **master key**, which is itself protected by **custodian keys**. This hierarchical key management system ensures that access to the locker's core functions and the data it holds is tightly controlled. Before the locker can become operational and serve requests, it must be **unlocked through a specific process** that involves providing the custodian keys. This multi-step unlocking procedure adds an extra layer of security, often referred to as a two-person rule, where multiple parties are required to activate the system. The locker's APIs are designed to handle encrypted and signed payloads, ensuring that data is always protected in transit. The internal workings of the locker are built to be **polymorphic**, meaning it can handle various types of sensitive information, not just card details, making it a versatile storage solution , . The focus is on providing a highly secure, performant, and scalable storage backend for sensitive payment information.

### 3.2 Tenant: Merchant Representation
In the Hyperswitch Card Vault (Tartarus) architecture, a **"Tenant" represents a merchant or any other distinct entity** that utilizes the vault to store its customers' sensitive payment information , . This concept is fundamental to enabling **multi-tenancy** within the Card Vault, allowing a single instance of the vault to securely serve multiple, isolated clients. Each tenant is typically associated with its own unique set of cryptographic keys and specific configurations. This segregation ensures that data belonging to one tenant is not accessible to another, maintaining strict data separation and security boundaries. For example, the `config.toml` file provided by the user includes a `[tenant_secrets]` section, which can hold tenant-specific public keys and other configurations, such as a `master_key` and `schema` , . The ability to define separate configurations per tenant allows for customized security policies and operational parameters tailored to individual merchant needs. When a tenant interacts with the Locker to store or retrieve data, their identity is typically verified, and their specific keys are used for cryptographic operations like JWE decryption and JWS verification, ensuring that only authorized tenants can access their respective data. This tenant model is crucial for service providers offering vaulting services to multiple merchants, as it provides a scalable and secure way to manage data for diverse clientele from a unified Card Vault infrastructure.

The multi-tenant capability is essential for payment service providers, platforms, or large enterprises that need to manage payment information for various sub-merchants, departments, or client applications. By providing each tenant with their own key space and configurations, the Card Vault ensures that each entity maintains control over its data and its security policies. The `config.toml` file, as shown in the user's example, includes a `[tenant_secrets]` section where public keys and potentially other tenant-specific master keys or schemas can be defined. This allows for fine-grained control over each tenant's environment. The architecture ensures that operations performed by one tenant do not impact the data or operations of another, providing a secure and isolated environment for each merchant or entity using the vault. This design promotes scalability and operational efficiency for providers managing multiple clients' sensitive data.

### 3.3 Key Management Hierarchy
The Hyperswitch Card Vault (Tartarus) employs a sophisticated key management hierarchy to ensure robust security for stored sensitive data. This hierarchy involves several types of keys, each serving a distinct purpose in the encryption and operational workflow , .

1.  **Public and Private Keys (RSA):**
    *   **Purpose:** These keys are primarily used for securing communications between the tenant (e.g., merchant application) and the Locker. They facilitate **JSON Web Encryption (JWE) for data confidentiality and JSON Web Signature (JWS) for data integrity and authenticity** , .
    *   **Generation:** Typically, RSA key pairs (e.g., 2048-bit) are generated using tools like OpenSSL. Separate key pairs are needed for the Locker and for each Tenant.
        *   `openssl genrsa -out locker-private-key.pem 2048`
        *   `openssl rsa -in locker-private-key.pem -pubout -out locker-public-key.pem`
        *   Similar commands are used for `tenant-private-key.pem` and `tenant-public-key.pem` , .
    *   **Usage:** The Locker's public key is used by the Tenant to encrypt data (JWE), and the Locker uses its private key to decrypt it. The Tenant's private key is used to sign requests (JWS), and the Locker uses the Tenant's public key to verify the signature.

2.  **Custodian Keys (AES):**
    *   **Purpose:** These are AES-generated keys designed to encrypt and decrypt the Master Key. A critical security feature is that **two distinct Custodian Keys are required to unlock the Locker**, implementing a form of dual control or M-of-N key escrow , . This enhances security by distributing key responsibilities, ensuring that no single individual can compromise the Master Key alone.
    *   **Generation:** The provided utility `cargo run --bin utils -- master-key` generates the Master Key along with two Custodian Keys , . The `-w` flag can be used to skip this key custodian feature if desired, though it's a recommended security practice.

3.  **Master Key (AES):**
    *   **Purpose:** This is an AES-generated key that is fundamental to the operation of the Locker and its configurations. It is used to encrypt sensitive data at rest or other operational keys within the vault. The Master Key itself is stored in an encrypted state, protected by the Custodian Keys , .
    *   **Generation:** Also generated by the `cargo run --bin utils -- master-key` command , .
    *   **Activation:** The Master Key must be decrypted (using the two Custodian Keys) before the Locker can become fully operational. This decryption process is part of the "unlock the locker" setup procedure.

This layered key management approach, from high-level communication security (RSA keys) down to the core operational key (Master Key) protected by Custodian Keys, provides a strong defense-in-depth strategy for protecting sensitive information within the Hyperswitch Card Vault.

### 3.4 Configuration (config.toml)
The `config.toml` file serves as the **primary configuration mechanism** for the Hyperswitch Card Vault (Tartarus), defining various operational parameters, connection details, and paths to security assets , . This file uses the TOML (Tom's Obvious, Minimal Language) format, which is designed to be human-readable and easily parsable. The user's provided `config.toml` snippet illustrates several key configuration sections:

*   **`[log.console]`**: Controls console logging.
    *   `enabled = true`: Enables console logging.
    *   `level = "DEBUG"`: Sets the logging level to DEBUG, which provides detailed diagnostic information. Other common levels include INFO, WARN, ERROR.
    *   `log_format = "default"`: Specifies the format of the log messages.

*   **`[server]`**: Defines network settings for the vault service.
    *   `host = "0.0.0.0"`: Binds the server to all available network interfaces.
    *   `port = 8080`: Listens on TCP port 8080 for incoming connections.

*   **`[limit]`**: Configures rate limiting for API requests (though the values `request_count = 1` and `duration = 60` seem unusually restrictive for a production environment, potentially allowing only 1 request per minute).
    *   `request_count`: The maximum number of requests allowed within the `duration`.
    *   `duration`: The time window (in seconds) for the rate limit.

*   **`[database]`**: Contains credentials and connection details for the PostgreSQL database used by the vault.
    *   `username = "db_user"`
    *   `password = "db_pass"`
    *   `host = "localhost"`
    *   `port = 5432`
    *   `dbname = "locker"`

*   **`[cache]`**: Configures caching behavior.
    *   `tti = 7200`: Time-to-idle in seconds (2 hours), after which an unused cache entry may be evicted.
    *   `max_capacity = 5000`: The maximum number of entries the cache can hold.

*   **`[secrets]`**: Specifies paths to critical encryption keys.
    *   `locker_private_key = "/root/git_hyper/hyperswitch-card-vault/locker-private-key.pem"`: Path to the Locker's RSA private key.

*   **`[tenant_secrets]`**: Defines secrets specific to tenants. The user's config shows a tenant named `public`.
    *   `public = { master_key = "481d178b8a4bb53064f60715b8a827754a19601b7e75b683e76cce6b9a0c5417", public_key = "/root/git_hyper/hyperswitch-card-vault/locker-public-key.pem", schema = "public" }`
        *   `master_key`: An AES key, likely used for encrypting data specific to this tenant or for deriving other keys. The provided value appears to be a hex-encoded key.
        *   `public_key`: Path to the Locker's RSA public key. (Note: The name `locker-public-key.pem` suggests it's the Locker's public key, which would typically be used by tenants to encrypt data sent *to* the locker. If this is a tenant-specific secret, it might be more appropriate to have the *tenant's* public key here if the locker needs to verify signatures from this tenant, or the *tenant's* private key if the locker needs to decrypt data encrypted by the tenant's public key. The setup guide indicates `tenant_public_key` is for the locker to verify JWS from the tenant, and `locker_private_key` is for the locker to decrypt JWE.)
        *   `schema = "public"`: Likely refers to the database schema where this tenant's data is stored, supporting multi-tenancy at the database level.

*   **`[external_key_manager]`**: Placeholder for integrating with an external Key Management System (KMS).
    *   `url = ""`
    *   `cert = ""`

*   **`[api_client]`**: Configuration for the vault's HTTP client when it acts as a client (e.g., for calling external services).
    *   `identity = "/root/git_hyper/hyperswitch-card-vault/identity.pem"`: Path to a client certificate for mTLS, if required.
    *   `pool_max_idle_per_host = 10`: Maximum number of idle HTTP connections to keep in the pool per destination host.

The documentation also mentions that environment variables prefixed with `LOCKER__` can be used to override `config.toml` settings, particularly when running the vault in a Docker container. For example, `LOCKER__SERVER__HOST=0.0.0.0` would set the server host , . This provides flexibility for different deployment environments.

## 4. Security Mechanisms
### 4.1 Encryption and Decryption Workflow
The Hyperswitch Card Vault (Tartarus) employs a multi-stage encryption and decryption workflow to protect sensitive data, primarily utilizing **JSON Web Encryption (JWE) and JSON Web Signature (JWS)** , . The process involves both the tenant (e.g., merchant application) and the locker (the vault's core).

**Encryption Workflow (Tenant to Locker):**
1.  **Data Preparation:** The tenant application collects sensitive data, such as card details (card number, expiration date, cardholder name).
2.  **JWE Encryption (Confidentiality):** The tenant encrypts this sensitive data using JWE. The encryption key used is typically the **Locker's RSA public key**. This ensures that only the Locker, possessing the corresponding private key, can decrypt the data. The user's `generate_payload.py` script demonstrates this:
    ```python
    # Step 1: Encrypt the actual sensitive data
    sensitive_data = { ... } # card details
    jwe_token = jwe.encrypt(
        plaintext=json.dumps(sensitive_data),
        key=locker_public_key, # Locker's public key
        algorithm='RSA-OAEP-256',
        encryption='A256GCM'
    )
    ```
    This step transforms the plaintext card data into an encrypted JWE token.
3.  **JWS Signing (Integrity & Authenticity):** The tenant then signs the JWE token using JWS. The signing key is the **tenant's own RSA private key**.
    ```python
    # Step 2: Sign the JWE payload
    jws_token = jws.sign(jwe_token, tenant_private_key, algorithm='RS256')
    ```
    This creates a JWS token that contains the JWE token and a signature. This signature allows the Locker to verify that the payload has not been tampered with and that it originates from a legitimate tenant.
4.  **Payload Transmission:** The final payload, which includes the `merchant_id`, `merchant_customer_id`, the `jws_token` (containing the encrypted and signed card data), and a `ttl` (time-to-live), is sent to the Locker's API endpoint (e.g., `/data` or `/cards`) , .

**Decryption Workflow (Locker processing):**
1.  **JWS Verification:** Upon receiving the payload, the Locker first verifies the JWS token using the **tenant's RSA public key** (retrieved based on `merchant_id` or similar).
    ```python
    # Step 1: Verify and extract the JWE token from JWS (from decode_payload.py, adapted for Locker's perspective)
    # tenant_public_key_for_this_merchant = get_tenant_public_key(payload["merchant_id"])
    # jwe_token = jws.verify(payload["enc_card_data"], tenant_public_key_for_this_merchant, algorithms=['RS256'])
    ```
    If verification fails, the request is rejected.
2.  **JWE Decryption:** After successful JWS verification, the Locker extracts the JWE token and decrypts it using its **own RSA private key**.
    ```python
    # Step 2: Decrypt the JWE token (from decode_payload.py, adapted for Locker's perspective)
    # locker_private_key = get_locker_private_key()
    # decrypted_data = jwe.decrypt(jwe_token, locker_private_key)
    ```
    This step recovers the original plaintext sensitive data.
3.  **Data Processing:** The Locker can now process the decrypted data (e.g., store it after potentially re-encrypting it with a different key for storage, or perform operations based on the API call).

This dual JWE/JWS mechanism ensures end-to-end security: JWE provides confidentiality, and JWS provides integrity and authenticity for all communications involving sensitive data between the tenant and the Locker , .

### 4.2 Role of Public/Private Keys
Public and private keys, typically RSA key pairs, play a fundamental role in the security mechanisms of the Hyperswitch Card Vault (Tartarus), primarily facilitating secure communication through **JSON Web Encryption (JWE) and JSON Web Signature (JWS)** , . Both the Locker (the vault itself) and the Tenant (e.g., a merchant application) possess their own unique public/private key pairs.

**Locker's Key Pair:**
*   **Locker Private Key:** This key is kept secret and secure within the Locker's environment. Its primary purpose is to **decrypt data that has been encrypted using the Locker's public key**. For instance, when a tenant sends sensitive card data to the Locker, they encrypt it using the Locker's public key, and only the Locker can decrypt it with its private key. The path to this key is specified in the `config.toml` file under `[secrets] locker_private_key` , .
*   **Locker Public Key:** This key is distributed to tenants or systems that need to send encrypted data to the Locker. Tenants use this key to perform **JWE encryption of sensitive payloads** before transmitting them to the Locker. The user's `generate_payload.py` script shows this:
    ```python
    with open('/root/git_hyper/hyperswitch-card-vault/locker-public-key.pem', 'r') as f:
        locker_public_key = f.read()
    jwe_token = jwe.encrypt(..., key=locker_public_key, ...)
    ```
    The user's `config.toml` also references a `locker-public-key.pem` under `[tenant_secrets] public`, which might be a slight misconfiguration or a specific use case where the tenant configuration also holds a copy of the locker's public key for its own reference or for specific operations , .

**Tenant's Key Pair:**
*   **Tenant Private Key:** This key is kept secret and secure by the tenant. Its primary purpose is to **sign requests (JWS) sent to the Locker**. By signing a request with its private key, the tenant provides proof of the request's origin and integrity. The Locker can then verify this signature using the tenant's public key. The user's `generate_payload.py` script uses this key:
    ```python
    with open('/root/git_hyper/hyperswitch-card-vault/tenant-private-key.pem', 'r') as f:
        tenant_private_key = f.read()
    jws_token = jws.sign(jwe_token, tenant_private_key, algorithm='RS256')
    ```
*   **Tenant Public Key:** This key is made available to the Locker, typically configured per tenant (e.g., in the `config.toml` under a `tenant_public_key` directive, though the user's specific `config.toml` shows `locker_public_key` under `tenant_secrets` which might be an inconsistency or a specific setup). The Locker uses this key to **verify the JWS signature on incoming requests** from that specific tenant. This ensures that the request is authentic and has not been altered in transit. The user's `decode_payload.py` script (simulating Locker's action) shows this:
    ```python
    with open('/root/git_hyper/hyperswitch-card-vault/tenant-public-key.pem', 'r') as f:
        tenant_public_key = f.read()
    jwe_token = jws.verify(encrypted_response, tenant_public_key, algorithms=['RS256'])
    ```

This asymmetric cryptography model, where data encrypted with a public key can only be decrypted by the corresponding private key, and data signed with a private key can be verified by anyone with the corresponding public key, forms the bedrock of secure communication and data protection in the Hyperswitch Card Vault , .

### 4.3 Custodian Keys and Master Key
Beyond the public/private key pairs used for securing communications, the Hyperswitch Card Vault (Tartarus) implements an additional layer of key management involving **Custodian Keys and a Master Key**, primarily for securing the Locker's operational environment and potentially the data it manages at rest , . This system introduces a "key custodian" feature, enhancing security through distributed control.

**Master Key:**
*   **Purpose:** The Master Key is an **AES-generated key that is critical for the Locker's operation**. It is used to encrypt sensitive configurations or potentially to encrypt data keys that, in turn, protect the actual cardholder data stored by the Locker. Without the decrypted Master Key, the Locker cannot function correctly or access the sensitive information it is designed to protect , .
*   **Generation:** The Master Key is generated using a provided utility: `cargo run --bin utils -- master-key`. This command outputs the Master Key itself , . The user's `config.toml` shows a `master_key` (hex-encoded) within the `[tenant_secrets.public]` section, suggesting that either each tenant has its own master key for its data, or this is a global master key for the vault instance configured for a specific tenant context , . The official setup guide also mentions KMS encrypting the `LOCKER__SECRETS__MASTER_KEY` .

**Custodian Keys:**
*   **Purpose:** Custodian Keys are **AES-generated keys specifically used to encrypt and decrypt the Master Key**. A crucial security feature is that ***two* distinct Custodian Keys are required to unlock the Locker**, implementing a form of dual control or M-of-N key escrow , . This enhances security by distributing the responsibility and knowledge required to unlock the Master Key. It prevents a single point of compromise and often involves multiple individuals (custodians) each holding one key.
*   **Generation:** The same utility command `cargo run --bin utils -- master-key` that generates the Master Key also generates these two Custodian Keys , . The `-w` flag can be used with this command to skip the key custodian feature, meaning the Master Key would not be encrypted by Custodian Keys, which is less secure , .
*   **Usage in Unlocking:** During the Locker startup or activation process, the two Custodian Keys must be provided to the Locker via specific API endpoints (`/custodian/key1` and `/custodian/key2`). Once both keys are received, a subsequent call to `/custodian/decrypt` triggers the decryption of the Master Key using these two Custodian Keys. If successful, the Locker becomes operational , . The official setup guide provides cURL examples for this process .

This hierarchical key structure, where Custodian Keys protect the Master Key, which in turn is essential for the Locker's core functions, adds a significant security measure, ensuring that even if the Locker's binary or configuration is compromised, the sensitive data remains protected unless the Master Key is decrypted, which requires collaboration from multiple key custodians , .

### 4.4 Data Integrity and Authenticity (JWS & JWE)
The Hyperswitch Card Vault (Tartarus) places a strong emphasis on ensuring **data integrity and authenticity** for all sensitive information it handles, primarily through the use of **JSON Web Signature (JWS)** in conjunction with JSON Web Encryption (JWE) , . While JWE provides confidentiality by encrypting the data, JWS is specifically designed to guarantee that the data has not been altered in transit (integrity) and that it originates from a trusted source (authenticity).

**Mechanism of JWS:**
When a tenant (e.g., a merchant application) sends data to the Locker, it first encrypts the sensitive payload using JWE with the Locker's public key. This encrypted payload (the JWE token) is then **signed using JWS with the tenant's private key** , . The signing process involves creating a cryptographic hash of the JWE token and then encrypting this hash with the tenant's private key. This encrypted hash, along with information about the signing algorithm, forms the JWS signature, which is then appended to or transmitted alongside the JWE token.

The user's `generate_payload.py` script illustrates this two-step process:
```python
# Step 1: Encrypt the actual sensitive data
jwe_token = jwe.encrypt(plaintext=json.dumps(sensitive_data), key=locker_public_key, algorithm='RSA-OAEP-256', encryption='A256GCM')

# Step 2: Sign the JWE payload
jws_token = jws.sign(jwe_token, tenant_private_key, algorithm='RS256')
```
The final payload sent to the Locker includes this `jws_token`.

**Verification by the Locker:**
Upon receiving the payload, the Locker performs the reverse operations. It first **verifies the JWS signature using the public key corresponding to the tenant** that purportedly sent the request (identified by `merchant_id` or similar). The Locker decrypts the JWS signature using the tenant's public key to retrieve the original hash, and it also independently calculates the hash of the received JWE token. If the two hashes match, it confirms that:
1.  **Integrity:** The JWE token (and thus the encrypted sensitive data) has not been tampered with during transmission.
2.  **Authenticity:** The request was indeed signed by the tenant possessing the corresponding private key, confirming its origin.

The user's `decode_payload.py` script, which simulates the Locker's decryption process, shows this verification step:
```python
# Step 1: Verify and extract the JWE token from JWS
jwe_token = jws.verify(encrypted_response, tenant_public_key, algorithms=['RS256'])
```
If the JWS verification fails, the Locker rejects the request, preventing unauthorized or tampered data from being processed. This dual mechanism of JWE for confidentiality and JWS for integrity/authenticity ensures a robust security posture for data in transit between the tenant and the Locker , .

## 5. Setup and Configuration
### 5.1 Prerequisites
Before setting up the Hyperswitch Card Vault (Tartarus), several prerequisites must be met to ensure a smooth and successful installation and operation. These typically involve having the necessary software tools, access permissions, and a basic understanding of the system's architecture.

1.  **Rust Toolchain:** Since the Hyperswitch Card Vault is built using Rust, the **Rust toolchain, including `cargo`** (Rust's package manager and build system), must be installed on the system where the vault will be built or run from source. This is essential for compiling the vault and for using utilities like `cargo run --bin utils -- master-key` for key generation , .

2.  **OpenSSL or Equivalent:** The generation of public/private key pairs (RSA keys) for JWE and JWS relies on **OpenSSL commands** (e.g., `openssl genrsa`, `openssl rsa -pubout`) , . Therefore, OpenSSL libraries and command-line tools should be available on the system.

3.  **Database:** The Card Vault requires a **PostgreSQL database** for persistence (as indicated by the `[database]` section in `config.toml`). This database should be set up and accessible from the vault instance. The user will need the database host, port, name, username, and password for configuration , .

4.  **Docker (Optional but Recommended):** While the vault can be run directly using `cargo run`, using **Docker is a common and often more manageable approach** for deployment, especially in production-like environments. If Docker is used, Docker Engine and Docker Compose (if applicable) should be installed , . The official documentation provides Docker images like `juspaydotin/hyperswitch-card-vault:latest` .

5.  **Network Access:** The server hosting the Card Vault needs appropriate network configuration. If the vault is to be accessed remotely (e.g., by an application server), firewall rules and security groups (in cloud environments like AWS) must allow incoming connections to the vault's listening port (e.g., 8080) . However, for security, direct internet access to the locker application is often discouraged; it should ideally be in a private network, accessible via a jump server or a reverse proxy , .

6.  **Key Management Strategy:** Before generating keys, a strategy for managing and storing these keys securely is crucial. This includes decisions on where to store private keys, how to distribute custodian keys, and how to back up the master key. For enhanced security, integrating with an external Key Management System (KMS) like AWS KMS is an option, as hinted in the `config.toml` (`[external_key_manager]`) and detailed in the official cloud setup guide .

7.  **Understanding of Security Practices:** A basic understanding of cryptographic principles, key management, and PCI DSS requirements is beneficial for configuring and operating the vault securely.

The official Hyperswitch CDK deployment guide for AWS also lists prerequisites like Git, Node.js, npm, and an AWS account with administrator access if deploying via their CDK scripts , . For on-premise setups, considerations include having a cloud service provider account (or own cloud), contractual relationships with payment processors, and Kubernetes expertise if applicable .

### 5.2 Key Generation
The setup of the Hyperswitch Card Vault (Tartarus) involves generating several types of cryptographic keys, each serving a specific security purpose within the system. The process is a critical first step before the vault can be configured and run , .

1.  **Master Key and Custodian Keys:**
    *   **Purpose:** The Master Key is an AES-generated key essential for the Locker's operation, potentially encrypting sensitive configurations or data keys. Custodian Keys (two of them) are AES-generated keys used to encrypt and decrypt the Master Key, providing a distributed control mechanism , .
    *   **Generation Command:** These keys are generated using a Rust utility provided with the Hyperswitch Card Vault source code:
        ```bash
        cargo run --bin utils -- master-key
        ```
        Executing this command will output the Master Key and two Custodian Keys. It is crucial to securely store these keys immediately upon generation , .
    *   **Optional Flag:** The `-w` flag can be used with the command (`cargo run --bin utils -- master-key -w`) to skip the key custodian feature. If this flag is used, the Master Key will not be encrypted by Custodian Keys, which simplifies setup but reduces security , .

2.  **Public and Private Keys (RSA Key Pairs):**
    *   **Purpose:** These keys are used for securing communications via JWE (encryption) and JWS (signing) between the tenant and the Locker. Separate key pairs are needed for the Locker and for each Tenant , .
    *   **Generation Commands (using OpenSSL):**
        *   **Locker's Private Key (RSA):**
            ```bash
            openssl genrsa -out locker-private-key.pem 2048
            ```
            This command generates a 2048-bit RSA private key for the Locker and saves it to `locker-private-key.pem`.
        *   **Locker's Public Key (RSA):**
            ```bash
            openssl rsa -in locker-private-key.pem -pubout -out locker-public-key.pem
            ```
            This command extracts the public key from the Locker's private key and saves it to `locker-public-key.pem`.
        *   **Tenant's Private Key (RSA):**
            ```bash
            openssl genrsa -out tenant-private-key.pem 2048
            ```
            This command generates a 2048-bit RSA private key for the Tenant and saves it to `tenant-private-key.pem`.
        *   **Tenant's Public Key (RSA):**
            ```bash
            openssl rsa -in tenant-private-key.pem -pubout -out tenant-public-key.pem
            ```
            This command extracts the public key from the Tenant's private key and saves it to `tenant-public-key.pem` , .
    *   **Security Note:** Private keys (both Locker's and Tenant's) must be kept confidential and secure. Public keys are distributed as needed (e.g., Locker's public key to Tenants, Tenant's public key to the Locker).

The official cloud setup guide also mentions the option to **KMS encrypt these generated keys** (database password, master key, locker private key, tenant public key) using AWS KMS for enhanced security before storing them in the configuration or environment variables . This involves using the `aws kms encrypt` command. For example:
```bash
echo -n "<database password>" | xargs -I {} -0 aws kms encrypt --key-id <your-kms-key-id> --region <your-kms-region> --plaintext "{}"
```
This step is recommended for production deployments to avoid storing plaintext keys .

### 5.3 Configuring the Vault (config.toml)
The primary method for configuring the Hyperswitch Card Vault (Tartarus) is through a `config.toml` file. This file contains various settings that dictate the vault's behavior, network configuration, logging, database connectivity, and paths to cryptographic keys , . The user has provided an example `config.toml`, which serves as a good reference for the structure and typical parameters.

**Key Sections and Parameters in `config.toml`:**

*   **`[log.console]`**: Controls the console logging output.
    *   `enabled`: Set to `true` to enable logging to the console.
    *   `level`: Defines the verbosity of logs (e.g., "DEBUG", "INFO", "WARN", "ERROR"). "DEBUG" provides detailed information useful for development and troubleshooting.
    *   `log_format`: Specifies the format of the log messages (e.g., "default").

*   **`[server]`**: Configures the network interface and port the vault listens on.
    *   `host`: The IP address to bind to. "0.0.0.0" typically means listening on all available network interfaces.
    *   `port`: The TCP port number for incoming HTTP requests (e.g., 8080).

*   **`[limit]`**: Defines rate limiting for API requests to prevent abuse.
    *   `request_count`: The maximum number of requests allowed within the `duration`.
    *   `duration`: The time window (in seconds) for the rate limit. The user's example (`request_count = 1`, `duration = 60`) is very restrictive.

*   **`[database]`**: Contains connection details for the PostgreSQL database.
    *   `username`: Database user name.
    *   `password`: Database password for the specified user.
    *   `host`: Database server hostname or IP address.
    *   `port`: Database server port (default is 5432 for PostgreSQL).
    *   `dbname`: Name of the database to connect to (e.g., "locker").

*   **`[cache]`**: Configures caching parameters.
    *   `tti`: Time-to-idle in seconds (e.g., 7200 seconds or 2 hours). Entries not accessed within this time may be evicted.
    *   `max_capacity`: Maximum number of items the cache can hold (e.g., 5000).

*   **`[secrets]`**: Specifies paths to the Locker's private key.
    *   `locker_private_key`: Absolute or relative path to the PEM file containing the Locker's RSA private key (e.g., `/root/git_hyper/hyperswitch-card-vault/locker-private-key.pem`). This key is used by the Locker to decrypt JWE payloads.

*   **`[tenant_secrets]`**: This section allows for tenant-specific configurations, particularly for multi-tenant setups. The user's example shows a tenant named `public`.
    *   `public = { master_key = "...", public_key = "...", schema = "public" }`
        *   `master_key`: An AES key, potentially used for encrypting data specific to this tenant or for deriving other keys. The value appears to be a hex-encoded string.
        *   `public_key`: Path to an RSA public key. In the user's example, it points to `locker-public-key.pem`. If this is intended for the Locker to verify JWS signatures from this tenant, it should ideally be the *tenant's* public key. If it's for the tenant to encrypt data, then `locker-public-key.pem` is correct, but its placement under `tenant_secrets` might be for the tenant's reference.
        *   `schema`: Likely specifies the database schema for this tenant's data, aiding in data isolation.

*   **`[external_key_manager]`**: Placeholder for external Key Management System (KMS) integration.
    *   `url`: URL of the KMS service.
    *   `cert`: Path to a certificate for authenticating with the KMS, if required.

*   **`[api_client]`**: Settings for the vault's HTTP client when it makes outbound requests.
    *   `identity`: Path to a client certificate (e.g., `identity.pem`) for mTLS if the vault needs to authenticate itself to external services.
    *   `pool_max_idle_per_host`: Maximum number of idle HTTP connections to keep in the connection pool per target host.

**Configuration via Environment Variables:**
As an alternative or override to `config.toml`, especially when using Docker, configuration values can be supplied via environment variables. These variables are typically prefixed with `LOCKER__` and use double underscores to denote nested sections from the TOML structure , . For example:
*   `LOCKER__SERVER__HOST=0.0.0.0`
*   `LOCKER__SERVER__PORT=8080`
*   `LOCKER__SECRETS__LOCKER_PRIVATE_KEY=<path_to_key>`
*   `LOCKER__DATABASE__PASSWORD=<kms_encrypted_password>` (The official cloud guide shows KMS encrypted values being passed here) .

This method is particularly useful for injecting secrets at runtime without modifying the base `config.toml` file, enhancing security in containerized environments. The official cloud setup guide provides an extensive list of such environment variables in an `env-file` example .

### 5.4 Running the Vault
Once the Hyperswitch Card Vault (Tartarus) is configured, primarily through the `config.toml` file and/or environment variables, and the necessary cryptographic keys have been generated, the next step is to run the vault service. There are two primary methods for running the vault: directly using Cargo (Rust's build system and package manager) or using Docker , .

**1. Running Directly with Cargo:**
This method is typically used during development or if Rust is available in the production environment.
*   **Prerequisite:** Ensure the Rust toolchain, including `cargo`, is installed and properly configured on the system.
*   **Command:**
    ```bash
    cargo run
    ```
    This command will compile the Hyperswitch Card Vault project (if not already compiled or if changes were made) and then execute the resulting binary. The application will read its configuration from `config.toml` (usually located in a `config` directory relative to where `cargo run` is executed, or specified via a path) and any relevant environment variables , .
*   **Considerations:** Running directly with `cargo run` might expose more system-level details and dependencies. It's generally recommended for development and testing. For production, a more isolated and managed environment like Docker is often preferred.

**2. Running with Docker:**
Using Docker provides a more standardized and isolated runtime environment, simplifying deployment and management, especially in production.
*   **Prerequisite:** Docker Engine must be installed and running on the host system.
*   **Docker Image:** An official Docker image for Hyperswitch Card Vault is available, for example, `juspaydotin/hyperswitch-card-vault:latest` or version-specific tags like `v0.3.0` or `0.1.3-nokms` , . You can pull this image using:
    ```bash
    docker pull juspaydotin/hyperswitch-card-vault:latest
    ```
*   **Configuration for Docker:** When running with Docker, configuration is typically passed via environment variables rather than a mounted `config.toml` file, although mounting a custom `config.toml` is also possible. An `env-file` containing all necessary `LOCKER__*` environment variables is a common approach .
*   **Docker Run Command (example using environment variables):**
    ```bash
    docker run --env-file path/to/your/envfile -d --net=host juspaydotin/hyperswitch-card-vault:latest
    ```
    *   `--env-file path/to/your/envfile`: Specifies the path to a file containing environment variables for configuration.
    *   `-d`: Runs the container in detached mode (in the background).
    *   `--net=host`: Uses the host's network stack for the container. Other network modes like bridge can also be used depending on the deployment setup.
    *   `juspaydotin/hyperswitch-card-vault:latest`: The name and tag of the Docker image to run .
*   **Docker Compose:** If the setup involves multiple services (e.g., the vault and its database), a `docker-compose.yaml` file can be used to define and manage them together. The user's documentation mentions using Docker with a configured `docker-compose.yaml` , . The official Hyperswitch documentation for the main application often includes Docker Compose examples, and similar principles apply to the Card Vault if it's part of a larger stack or has dependencies like a database.

Regardless of the method used, once the vault is running, it will start listening on the configured host and port (e.g., `0.0.0.0:8080`). However, before it can fully operate and process sensitive data, the Locker typically needs to be "unlocked" by providing the Custodian Keys to decrypt the Master Key , .

### 5.5 Unlocking the Locker
After the Hyperswitch Card Vault (Tartarus) service is up and running, a crucial security step before it can become fully operational and process sensitive data is the **"unlocking" of the Locker**. This process involves decrypting the Master Key, which is essential for the Locker's core functions, and it requires the two Custodian Keys that were generated earlier , . This mechanism ensures that even if the vault software is started, the sensitive data remains protected until explicitly unlocked by authorized personnel (key custodians).

**Unlocking Procedure:**
The unlocking is performed by making a series of HTTP POST requests to specific API endpoints exposed by the running Locker service. The official setup guide provides cURL examples for this process .

1.  **Provide Custodian Key 1:**
    Send a POST request to the `/custodian/key1` endpoint with the first Custodian Key in the request body.
    ```bash
    curl -X 'POST' \
      'http://localhost:8080/custodian/key1' \ # Replace localhost:8080 with the actual vault URL if different
      -H 'accept: text/plain' \
      -H 'Content-Type: application/json' \
      -d '{
      "key": "<Your Custodian Key 1 Here>"
    }'
    ```
    A successful request will typically store this key temporarily within the Locker, awaiting the second key.

2.  **Provide Custodian Key 2:**
    Send a POST request to the `/custodian/key2` endpoint with the second Custodian Key in the request body.
    ```bash
    curl -X 'POST' \
      'http://localhost:8080/custodian/key2' \ # Replace localhost:8080 with the actual vault URL if different
      -H 'accept: text/plain' \
      -H 'Content-Type: application/json' \
      -d '{
      "key": "<Your Custodian Key 2 Here>"
    }'
    ```
    After this step, the Locker should have both parts necessary to attempt decryption of the Master Key.

3.  **Initiate Master Key Decryption:**
    Once both Custodian Keys have been provided, send a POST request to the `/custodian/decrypt` endpoint to trigger the decryption of the Master Key using the two supplied Custodian Keys.
    ```bash
    curl -X 'POST' 'http://localhost:8080/custodian/decrypt' # Replace localhost:8080 with the actual vault URL if different
    ```
    *   **Successful Response:** If the decryption is successful, this endpoint should respond with a message such as "Decrypted Successfully" , . This indicates that the Master Key is now available to the Locker in its decrypted form, and the Locker is fully operational and ready to process sensitive data.
    *   **Failure Response:** If the keys are incorrect or the decryption fails for any reason, an error message will be returned, and the Locker will remain in a locked state.

**Security Considerations for Unlocking:**
*   The Custodian Keys are highly sensitive and should be transmitted securely (e.g., over HTTPS, within a trusted network).
*   The individuals holding the Custodian Keys (key custodians) should be trusted and follow secure procedures for handling and entering these keys.
*   The unlocking process is typically performed once after the Locker service starts or after a restart.
*   If the `-w` flag was used during Master Key generation (`cargo run --bin utils -- master-key -w`), the key custodian feature is disabled, and the Master Key is used directly without this two-key decryption step. This is less secure and generally not recommended for production environments , .

The official cloud setup guide also mentions that after deploying the Card Vault on AWS using their CDK script, you need to SSH into the vault instance (often via a jump server for security) to run these unlock commands . The cURL commands are also displayed at the end of the deployment script for convenience .

## 6. Usage and Operations
### 6.4 Python Scripts for Payload Generation and Decryption
The user has provided two Python scripts, `generate_payload.py` and `decode_payload.py`, which demonstrate how to prepare data for submission to the Hyperswitch Card Vault (Tartarus) and how to process a response from it, respectively. These scripts utilize the `jose` Python library for JWE and JWS operations.

**`generate_payload.py` - Creating an Encrypted and Signed Payload:**

This script simulates the actions a tenant (e.g., a merchant's backend) would perform to securely send sensitive card data to the Locker.

1.  **Key Loading:**
    ```python
    with open('/root/git_hyper/hyperswitch-card-vault/locker-public-key.pem', 'r') as f:
        locker_public_key = f.read()
    with open('/root/git_hyper/hyperswitch-card-vault/tenant-private-key.pem', 'r') as f:
        tenant_private_key = f.read()
    ```
    *   `locker_public_key`: The Locker's RSA public key, used for JWE encryption. This ensures only the Locker can decrypt the data.
    *   `tenant_private_key`: The Tenant's RSA private key, used for JWS signing. This authenticates the request and ensures integrity.

2.  **Sensitive Data Preparation:**
    ```python
    sensitive_data = {
        "card_number": "4111111111111111",
        "name_on_card": "John Doe",
        "card_exp_month": "12",
        "card_exp_year": "2026"
    }
    ```
    This dictionary contains the actual card details to be protected.

3.  **JWE Encryption (Confidentiality):**
    ```python
    jwe_token = jwe.encrypt(
        plaintext=json.dumps(sensitive_data),
        key=locker_public_key,
        algorithm='RSA-OAEP-256',
        encryption='A256GCM'
    )
    ```
    *   `plaintext`: The sensitive data, serialized as a JSON string.
    *   `key`: The Locker's public key.
    *   `algorithm='RSA-OAEP-256'`: The key encryption algorithm (RSA with Optimal Asymmetric Encryption Padding using SHA-256).
    *   `encryption='A256GCM'`: The content encryption algorithm (AES-GCM with a 256-bit key). This encrypts the `sensitive_data`.

4.  **JWS Signing (Integrity & Authenticity):**
    ```python
    jws_token = jws.sign(jwe_token, tenant_private_key, algorithm='RS256')
    ```
    *   `jwe_token`: The encrypted payload from the previous step.
    *   `key`: The Tenant's private key.
    *   `algorithm='RS256'`: The signing algorithm (RSA with SHA-256). This signs the JWE token.

5.  **Final Payload Construction:**
    ```python
    payload = {
        "merchant_id": "test_merchant",
        "merchant_customer_id": "cust_123",
        "enc_card_data": jws_token,
        "ttl": 3600
    }
    print(json.dumps(payload, indent=2))
    ```
    *   `merchant_id`: Identifies the tenant.
    *   `merchant_customer_id`: Identifies the customer within the tenant's system.
    *   `enc_card_data`: The JWS token (which contains the JWE encrypted card data).
    *   `ttl`: Time-to-live for the data, possibly used by the Locker for cache control or data retention policies.
    This `payload` is what would be sent to a Locker API endpoint like `/cards` or `/data`.

**`decode_payload.py` - Decrypting and Verifying a Response:**

This script simulates the actions the Locker would perform upon receiving the payload, or how a tenant would process an encrypted response from the Locker.

1.  **Key Loading (Locker's Perspective):**
    ```python
    with open('/root/git_hyper/hyperswitch-card-vault/locker-private-key.pem', 'r') as f:
        locker_private_key = f.read()
    with open('/root/git_hyper/hyperswitch-card-vault/tenant-public-key.pem', 'r') as f:
        tenant_public_key = f.read()
    ```
    *   `locker_private_key`: The Locker's RSA private key, used for JWE decryption.
    *   `tenant_public_key`: The Tenant's RSA public key, used for JWS verification.

2.  **JWS Verification (Integrity & Authenticity):**
    ```python
    # encrypted_response is the 'enc_card_data' value from the received payload
    jwe_token = jws.verify(encrypted_response, tenant_public_key, algorithms=['RS256'])
    ```
    *   `encrypted_response`: This would be the `jws_token` received from the tenant (i.e., the `enc_card_data` field from the generated payload).
    *   `tenant_public_key`: Used to verify the signature.
    *   `algorithms=['RS256']`: Specifies the allowed signing algorithm.
    If verification fails, an exception is raised. If successful, `jwe_token` contains the original JWE token.

3.  **JWE Decryption (Confidentiality):**
    ```python
    decrypted_data = jwe.decrypt(jwe_token, locker_private_key)
    ```
    *   `jwe_token`: The verified JWE token.
    *   `locker_private_key`: Used to decrypt the JWE token.
    `decrypted_data` now contains the plaintext JSON string of the sensitive data.

4.  **Parsing Decrypted Data:**
    ```python
    result = json.loads(decrypted_data)
    print(json.dumps(result, indent=2))
    ```
    This parses the JSON string back into a Python dictionary, revealing the original sensitive card details.

These scripts clearly illustrate the end-to-end cryptographic protection: JWE for encrypting the sensitive data using the Locker's public key, and JWS for signing the entire encrypted payload using the Tenant's private key. The Locker then verifies the signature with the Tenant's public key and decrypts the data with its own private key , .

## 7. Advanced Topics
### 7.1 Multi-Tenancy Support
The Hyperswitch Card Vault (Tartarus) is designed with **robust multi-tenancy support**, allowing a single instance of the vault to securely serve multiple, isolated clients, typically merchants or distinct business units , . This capability is crucial for payment service providers or platforms that offer vaulting services to numerous merchants, as it enables efficient resource utilization and centralized management while maintaining strict data segregation.

**Key Aspects of Multi-Tenancy in Tartarus:**

1.  **Tenant Identification:** Each request to the Locker typically includes a `merchant_id` or a similar identifier (as seen in the `generate_payload.py` script) that uniquely identifies the tenant making the request. This ID is fundamental for routing requests, applying tenant-specific configurations, and ensuring data isolation.

2.  **Data Segregation:** Sensitive data stored by the Locker is associated with the tenant's identifier. This ensures that data belonging to one tenant is not accessible to another. The user's `config.toml` file includes a `[tenant_secrets]` section, which can hold configurations specific to a tenant named `public` , . This section includes a `schema = "public"` field, suggesting that data for different tenants might be segregated at the database schema level or through other internal mechanisms within the Locker. This logical separation is critical for maintaining data privacy and security boundaries between tenants.

3.  **Tenant-Specific Keys and Configurations:** The architecture allows for each tenant to potentially have its own set of cryptographic keys and configurations. For example:
    *   **JWE/JWS Keys:** While the Locker has its own key pair, each tenant would also have its own key pair. The Locker uses the specific tenant's public key to verify JWS signatures on incoming requests from that tenant. Tenants use the Locker's public key to encrypt data, but their own private key to sign requests.
    *   **Master Key:** The `[tenant_secrets.public]` section in the user's `config.toml` includes a `master_key`. This could imply that each tenant has a unique master key for encrypting their data within the Locker, or it could be a reference to a global master key as perceived by that tenant's context. If each tenant has a distinct master key, it further enhances isolation.
    *   **Other Configurations:** Tenants might have different settings for data retention policies, allowed operations, or integration with external KMS systems.

4.  **API Endpoints:** While core Locker functionalities like `/data` or `/cards` are shared, the operations performed through these endpoints are scoped to the authenticated tenant. For example, when a tenant requests to store or retrieve card data, the Locker ensures that the data is linked to that specific tenant and cannot be accessed by others.

5.  **Security Implications:** Proper implementation of multi-tenancy is crucial for security. The Locker must rigorously enforce access controls, ensuring that one tenant cannot access, modify, or delete another tenant's data. The use of separate cryptographic keys per tenant, as suggested by the configuration, is a strong security measure in a multi-tenant environment.

The ability to define `tenant_secrets` with potentially unique `master_key`, `public_key`, and `schema` for each tenant in the `config.toml` directly supports this multi-tenant model, allowing for fine-grained control and security per client , .

### 7.2 Database Configuration
The Hyperswitch Card Vault (Tartarus) relies on a **PostgreSQL database** for persistent storage of encrypted sensitive data and potentially other operational metadata , . The configuration for this database connection is a critical part of setting up the vault and is specified within the `config.toml` file under the `[database]` section.

**Database Configuration Parameters in `config.toml`:**

The user's provided `config.toml` includes the following parameters for database connectivity:

*   **`username`**: Specifies the database user account that the Card Vault will use to connect to the PostgreSQL server. In the example, it is `db_user`. This user must have the necessary privileges to perform operations on the `locker` database and its schemas/tables.
*   **`password`**: The password for the specified database user. In the example, it is `db_pass`. It is crucial that this password is kept secure. For enhanced security in production, it's recommended to use environment variables to inject this password or to use KMS-encrypted values, as shown in the official cloud setup guide .
*   **`host`**: The hostname or IP address of the machine where the PostgreSQL server is running. In the example, it is `localhost`, indicating that the database is expected to be on the same machine as the Card Vault or accessible via a local socket. For distributed setups, this would be the network address of the database server.
*   **`port`**: The TCP port on which the PostgreSQL server is listening for connections. The default port for PostgreSQL is `5432`, as specified in the example.
*   **`dbname`**: The name of the specific database within the PostgreSQL server that the Card Vault will connect to. In the example, it is `locker`.

**Example Configuration Snippet from `config.toml`:**
```toml
[database]
username = "db_user"
password = "db_pass"
host = "localhost"
port = 5432
dbname = "locker"
```

**Security and Operational Considerations:**

*   **Network Security:** Access to the database server should be restricted. If the database is on a separate host from the Card Vault, firewall rules or security groups should be configured to allow connections *only* from the Card Vault instance(s) to the PostgreSQL port.
*   **Database User Privileges:** The database user (`db_user` in the example) should be granted only the minimum necessary privileges required for the Card Vault to operate. Typically, this includes permissions to SELECT, INSERT, UPDATE, and DELETE on specific tables or schemas used by the vault, and potentially to execute certain functions or manage connections.
*   **Password Management:** Storing database passwords directly in `config.toml` is acceptable for development but not recommended for production. As mentioned, using environment variables (e.g., `LOCKER__DATABASE__PASSWORD`) or integrating with a KMS to provide encrypted passwords at runtime is a more secure practice .
*   **Database Maintenance:** Regular database maintenance tasks, such as backups, vacuuming, and monitoring, are essential for the reliability and performance of the Card Vault.
*   **Multi-Tenancy and Database Schema:** The `schema = "public"` parameter under `[tenant_secrets.public]` in the user's `config.toml` suggests that the database schema can be used as a mechanism for data isolation in multi-tenant environments , . Different tenants' data could be stored in different schemas within the same `locker` database, or different databases could be used, though the configuration points to a single `dbname`.

The official cloud setup guide for AWS also includes steps for setting up an RDS instance (PostgreSQL) and provides commands to KMS encrypt the database password before storing it in the `env-file` used by Docker . This highlights the importance of securing database credentials.

### 7.3 Caching Mechanisms
The Hyperswitch Card Vault (Tartarus) incorporates **caching mechanisms** to improve performance by reducing the need to repeatedly fetch or compute frequently accessed data. The configuration for caching is managed through the `config.toml` file, specifically within the `[cache]` section , . Caching can significantly enhance the responsiveness of the vault, especially under load, by storing copies of data in faster-access memory.

**Cache Configuration Parameters in `config.toml`:**

The user's provided `config.toml` includes the following parameters for cache configuration:

*   **`tti` (Time-To-Idle):** This parameter defines the maximum amount of time (in seconds) that a cache entry can remain in the cache without being accessed before it is considered eligible for eviction. In the example, `tti = 7200` seconds, which is equivalent to 2 hours. This means if a piece of cached data is not requested for 2 hours, it might be removed from the cache to free up space for newer or more frequently accessed data.
*   **`max_capacity`:** This parameter specifies the maximum number of entries or the overall size limit (depending on the cache implementation) that the cache can hold. In the example, `max_capacity = 5000`. Once this limit is reached, the cache will need to evict existing entries (often based on a policy like Least Recently Used - LRU, or based on TTI) to make room for new ones.

**Example Configuration Snippet from `config.toml`:**
```toml
[cache]
tti = 7200 # i.e. 2 hours
max_capacity = 5000
```

**Purpose and Benefits of Caching in the Card Vault:**

1.  **Reduced Database Load:** Frequently accessed data, such as decrypted card information (if cached after decryption, though this needs careful security consideration), frequently accessed configuration parameters, or results of expensive computations, can be stored in the cache. Subsequent requests for this data can be served from the cache, reducing the number of queries to the underlying PostgreSQL database.
2.  **Improved Response Times:** Accessing data from an in-memory cache is significantly faster than retrieving it from a disk-based database. This can lead to lower latency for API requests handled by the Card Vault.
3.  **Scalability:** By reducing the load on the database and improving the speed of individual requests, caching can help the Card Vault handle a larger number of concurrent requests, contributing to better scalability.

**Considerations for Caching Sensitive Data:**

*   **Security:** If the cache stores decrypted sensitive information, even temporarily, it must be secured with the same rigor as other components handling plaintext data. This includes ensuring the cache is protected from unauthorized access, both from external threats and from other processes on the same host.
*   **Cache Eviction Policies:** The effectiveness of the cache depends on its eviction policy. Common policies include LRU (Least Recently Used), LFU (Least Frequently Used), or time-based expiration (like TTI). The choice of policy should align with the access patterns of the data.
*   **Cache Invalidation:** When data is updated or deleted, the corresponding entries in the cache must be invalidated or updated to prevent stale data from being served. This requires careful coordination between data modification operations and the cache.
*   **Distributed Caching (Potential):** For a highly available or distributed deployment of the Card Vault, a distributed cache (e.g., Redis or Memcached) might be considered instead of or in addition to an in-memory cache within each vault instance. This would allow cache sharing across instances. The provided `config.toml` configures a local cache, but the architecture might support or evolve to support external caches.

The specific implementation details of the caching mechanism within Hyperswitch Card Vault (e.g., what data is cached, the exact eviction algorithm, whether it's a write-through or write-behind cache) would require further inspection of the source code or more detailed documentation. However, the presence of `tti` and `max_capacity` suggests a configurable, size-bounded, and time-aware caching strategy , .

### 7.4 Environment Variables for Docker
When deploying the Hyperswitch Card Vault (Tartarus) using Docker, **environment variables offer a flexible and secure way to configure the application**, often preferred over mounting a static `config.toml` file, especially for sensitive information like passwords and keys , . The Card Vault is designed to recognize environment variables that override or supplement settings defined in `config.toml`. These variables typically follow a naming convention using a `LOCKER__` prefix and double underscores (`__`) to represent nested configuration sections.

**Structure of Environment Variables:**
The general structure for environment variables is `LOCKER__<SECTION>__<SUBSECTION>__<KEY>`, where `<SECTION>`, `<SUBSECTION>`, and `<KEY>` correspond to the TOML structure.

**Examples from User Documentation and Official Guides:**

The user's documentation mentions the use of environment variables prefixed with `LOCKER__` if running via Docker, providing an example:
*   `LOCKER__SERVER__HOST=0.0.0.0`
*   `LOCKER__SECRETS__LOCKER_PRIVATE_KEY=<path_to_key>`

The official cloud setup guide  provides a more comprehensive `env-file` example, which can be passed to the `docker run` command using the `--env-file` flag. Key environment variables include:

*   **Server Configuration:**
    *   `LOCKER__SERVER__HOST=0.0.0.0`
    *   `LOCKER__SERVER__PORT=8080`

*   **Logging Configuration:**
    *   `LOCKER__LOG__CONSOLE__ENABLED=true`
    *   `LOCKER__LOG__CONSOLE__LEVEL=DEBUG`
    *   `LOCKER__LOG__CONSOLE__LOG_FORMAT=default`

*   **Database Configuration:**
    *   `LOCKER__DATABASE__USERNAME=db_user`
    *   `LOCKER__DATABASE__PASSWORD=<kms_encrypted_password>` (Note: KMS encrypted value)
    *   `LOCKER__DATABASE__HOST=localhost`
    *   `LOCKER__DATABASE__PORT=5432`
    *   `LOCKER__DATABASE__DBNAME=locker`

*   **Secrets Configuration:**
    *   `LOCKER__SECRETS__LOCKER_PRIVATE_KEY=<kms_encrypted_locker_private_key>`
    *   `LOCKER__SECRETS__MASTER_KEY=<kms_encrypted_master_key>`
    *   `LOCKER__TENANT_SECRETS__PUBLIC__PUBLIC_KEY=<kms_encrypted_tenant_public_key>`
    *   `LOCKER__TENANT_SECRETS__PUBLIC__SCHEMA=public`

*   **Cache Configuration:**
    *   `LOCKER__CACHE__TTI=7200`
    *   `LOCKER__CACHE__MAX_CAPACITY=5000`

*   **External Key Manager (KMS) Configuration:**
    *   `LOCKER__EXTERNAL_KEY_MANAGER__URL=`
    *   `LOCKER__EXTERNAL_KEY_MANAGER__CERT=`

*   **API Client Configuration:**
    *   `LOCKER__API_CLIENT__IDENTITY=/path/to/identity.pem`
    *   `LOCKER__API_CLIENT__POOL_MAX_IDLE_PER_HOST=10`

This method of using environment variables, often stored in an `env-file`, is particularly useful for **managing configurations in containerized environments like Docker**, as it allows for easy injection of settings without modifying the base `config.toml` file. It also enhances security by enabling the use of secrets management systems or KMS-encrypted values for sensitive data like passwords and keys . The double underscore (`__`) notation is crucial for correctly mapping to the nested TOML structure.

## 8. Conclusion
### 8.1 Summary of Features
The Hyperswitch Card Vault (Tartarus) stands out as a **secure, open-source solution** meticulously designed for the storage and management of sensitive payment information, with a strong emphasis on **PCI DSS compliance**. Built using the Rust programming language, it offers a blend of **high performance and robust security**. Key architectural features include its **modular design**, allowing for flexible integration as a standalone service or within the broader Hyperswitch ecosystem, and its **stateless operation**, which simplifies scalability and resilience by delegating data persistence to an application server. The vault's security is multi-layered, employing **JSON Web Encryption (JWE) for data confidentiality and JSON Web Signature (JWS) for integrity and authenticity** in all communications. A sophisticated **key management hierarchy**, involving public/private key pairs, custodian keys, and a master key, ensures that sensitive data is protected both in transit and at rest. The system also supports **multi-tenancy**, enabling secure data isolation for different merchants or entities using a single vault instance. Configuration is primarily managed through a `config.toml` file, with support for environment variables, especially in Dockerized deployments. The setup process involves generating cryptographic keys and an "unlock" procedure for the locker, enhancing operational security. Overall, the Hyperswitch Card Vault provides a comprehensive and reliable platform for organizations seeking to securely manage payment data while minimizing their PCI compliance scope.

### 8.2 Further Resources and Official Documentation
For more detailed information, advanced configurations, and the latest updates on the Hyperswitch Card Vault (Tartarus), users are encouraged to refer to the official documentation and resources provided by Juspay. The primary source of information is the **official GitHub repository** for the Hyperswitch Card Vault. This repository typically contains the source code, detailed setup guides, configuration references, and potentially troubleshooting tips.

Key resources include:
*   **Official GitHub Repository:** `https://github.com/juspay/hyperswitch-card-vault`
    *   Look for directories like `/docs` or `/guides` within the repository for comprehensive documentation.
*   **Setup Guides:** The repository often includes specific guides for different deployment scenarios, such as local setup, Docker-based deployment, and cloud deployments (e.g., on AWS). These guides provide step-by-step instructions for installation, configuration, key generation, and running the vault.
    *   Example path to a setup guide: `https://github.com/juspay/hyperswitch-card-vault/blob/main/docs/guides/setup.md` 
*   **Testing and Integration Guides:** Documentation on how to test the vault's functionality and integrate it with applications, including examples of API usage and payload structures.
    *   Example path to a testing guide: `https://github.com/juspay/hyperswitch-card-vault/blob/main/docs/guides/testing.md` 
*   **Community and Support:** The GitHub repository's issue tracker can be a place to report bugs, ask questions, and engage with the development community.

It is highly recommended to always consult the official documentation corresponding to the specific version of the Hyperswitch Card Vault being used, as features and configurations may evolve. The official guides will provide the most accurate and up-to-date information for deploying and operating the Card Vault securely and effectively.
