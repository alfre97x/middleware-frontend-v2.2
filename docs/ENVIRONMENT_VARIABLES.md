# Environment Variables Reference

Comprehensive reference for all environment variables used in the ISO 20022 Middleware platform.

## Table of Contents

1. [Database Configuration](#database-configuration)
2. [Redis Configuration](#redis-configuration)
3. [Blockchain Configuration](#blockchain-configuration)
4. [API Configuration](#api-configuration)
5. [Worker Configuration](#worker-configuration)
6. [Frontend Configuration](#frontend-configuration)
7. [Agent Configuration](#agent-configuration)
8. [Security & Cryptography](#security--cryptography)
9. [AI/ML Configuration](#aiml-configuration)
10. [X402 Payment Configuration](#x402-payment-configuration)
11. [Development & Testing](#development--testing)

---

## Database Configuration

### DATABASE_URL
- **Type**: String (URL)
- **Required**: Yes
- **Services**: API, Worker, Agent (optional)
- **Format**: `postgresql+psycopg://user:password@host:port/database`
- **Example**: `postgresql+psycopg://iso_user:iso_pass@localhost:5432/iso_mw`
- **Description**: PostgreSQL database connection string
- **Railway**: Auto-provided via `${{Postgres.DATABASE_URL}}`

### AUTO_CREATE_DB
- **Type**: Boolean (0 or 1)
- **Required**: No
- **Default**: `0`
- **Services**: API
- **Description**: Automatically create database if it doesn't exist (development only)
- **Production**: Always set to `0`

### RUN_MIGRATIONS
- **Type**: Boolean (0 or 1)
- **Required**: Yes
- **Services**: API (1), Worker (0)
- **Description**: Whether to run Alembic database migrations on startup
- **Important**: Only enable on API service, disable on Worker

---

## Redis Configuration

### REDIS_URL
- **Type**: String (URL)
- **Required**: Yes
- **Services**: API, Worker
- **Format**: `redis://host:port/db`
- **Example**: `redis://localhost:6379/0`
- **Description**: Redis connection string for job queue and caching
- **Railway**: Auto-provided via `${{Redis.REDIS_URL}}`

### RQ_QUEUES
- **Type**: String (comma-separated)
- **Required**: Yes (Worker)
- **Default**: `default`
- **Services**: Worker
- **Example**: `default,high_priority,low_priority`
- **Description**: Queue names that the worker should process

---

## Blockchain Configuration

### FLARE_RPC_URL
- **Type**: String (URL)
- **Required**: Yes
- **Services**: API, Worker
- **Default**: `https://flare-api.flare.network/ext/C/rpc`
- **Example**: `https://flare-api.flare.network/ext/C/rpc`
- **Description**: Flare blockchain RPC endpoint URL
- **Alternatives**: 
  - Testnet: `https://coston2-api.flare.network/ext/C/rpc`
  - Custom node: Your own RPC endpoint

### ANCHOR_CONTRACT_ADDR
- **Type**: String (Ethereum address)
- **Required**: Yes
- **Services**: API, Worker
- **Format**: `0x` + 40 hex characters
- **Example**: `0x1234567890123456789012345678901234567890`
- **Description**: Deployed evidence anchor smart contract address
- **How to obtain**: Deploy contract using scripts in `/scripts`

### ANCHOR_PRIVATE_KEY
- **Type**: String (Private key)
- **Required**: Yes
- **Services**: API, Worker
- **Format**: `0x` + 64 hex characters
- **Example**: `0xabcdef...`
- **Description**: Private key for blockchain transactions
- **Security**: 
  - Never commit to version control
  - Use separate keys for development/production
  - Ensure wallet has sufficient FLR for gas fees

### ANCHOR_ABI_PATH
- **Type**: String (File path)
- **Required**: Yes
- **Services**: API, Worker
- **Default**: `contracts/EvidenceAnchor.abi.json`
- **Description**: Path to smart contract ABI file (relative to project root)

---

## API Configuration

### ALLOW_ORIGINS
- **Type**: String (comma-separated URLs)
- **Required**: Yes
- **Services**: API
- **Format**: Comma-separated full URLs (with protocol)
- **Example**: `http://localhost:3000,https://app.example.com`
- **Description**: CORS allowed origins for browser requests
- **Important**: 
  - No trailing slashes
  - Include protocol (http/https)
  - No wildcards in production

### ARTIFACTS_DIR
- **Type**: String (Directory path)
- **Required**: No
- **Default**: `artifacts`
- **Services**: API, Worker
- **Description**: Directory for storing generated receipts and artifacts
- **Railway**: Consider using Railway volumes or external storage (S3)

### PORT
- **Type**: Integer
- **Required**: No (auto-set by Railway)
- **Default**: `8000`
- **Services**: API, Frontend
- **Description**: HTTP server port
- **Railway**: Auto-provided via `$PORT` environment variable

---

## Worker Configuration

Worker uses most of the same environment variables as the API, but with these differences:

- `RUN_MIGRATIONS=0` (must be disabled)
- `RQ_QUEUES=default` (specify queues to process)
- Does not need `ALLOW_ORIGINS`

---

## Frontend Configuration

### NEXT_PUBLIC_API_BASE_URL
- **Type**: String (URL)
- **Required**: Yes
- **Services**: Frontend
- **Format**: Full URL including protocol
- **Example**: `https://api.example.com`
- **Description**: Backend API base URL for client-side requests
- **Railway**: Use `${{api.RAILWAY_PUBLIC_DOMAIN}}` or full public URL
- **Important**: 
  - Must be publicly accessible (browser access)
  - Include in API's `ALLOW_ORIGINS`
  - No trailing slash

### NODE_ENV
- **Type**: String
- **Required**: No
- **Default**: `production` (in Railway)
- **Values**: `development`, `production`, `test`
- **Services**: Frontend
- **Description**: Node.js environment mode

---

## Agent Configuration

### API_BASE_URL
- **Type**: String (URL)
- **Required**: Yes
- **Services**: Agent
- **Example**: `https://api.example.com` or `http://api.railway.internal`
- **Description**: Backend API base URL for agent requests
- **Railway**: Can use private domain `${{api.RAILWAY_PRIVATE_DOMAIN}}`

### AGENT_ID
- **Type**: String (UUID or unique identifier)
- **Required**: Yes
- **Services**: Agent
- **Example**: `agent-x402-01`
- **Description**: Unique identifier for this agent instance

### AGENT_API_KEY
- **Type**: String (API key)
- **Required**: Yes
- **Services**: Agent
- **Description**: API key for authenticating agent requests
- **How to obtain**: Generate via API endpoints or admin panel

### LOG_LEVEL
- **Type**: String
- **Required**: No
- **Default**: `info`
- **Values**: `debug`, `info`, `warn`, `error`
- **Services**: Agent, API, Worker
- **Description**: Logging verbosity level

---

## Security & Cryptography

### SERVICE_PRIVATE_KEY
- **Type**: String (Hex-encoded)
- **Required**: Yes
- **Services**: API, Worker
- **Format**: 64 hex characters (32 bytes)
- **Example**: `a1b2c3d4e5f6...` (64 chars)
- **Description**: Ed25519 private key for signing operations
- **Generation**:
  ```bash
  openssl rand -hex 32
  ```
- **Security**: 
  - Must be same across API and Worker
  - Never commit to version control
  - Rotate regularly

### SERVICE_PUBLIC_KEY
- **Type**: String (PEM format)
- **Required**: Yes
- **Services**: API, Worker
- **Format**: Base64-encoded public key in PEM-like format
- **Example**: 
  ```
  -----BEGIN PUBLIC KEY-----
  Zm9vYmFy...
  -----END PUBLIC KEY-----
  ```
- **Description**: Ed25519 public key for verification
- **Must match**: SERVICE_PRIVATE_KEY

### JWT_SECRET (if applicable)
- **Type**: String
- **Required**: Optional
- **Services**: API
- **Description**: Secret for JWT token signing
- **Generation**: Use strong random string

---

## AI/ML Configuration

### OPENAI_API_KEY
- **Type**: String (API key)
- **Required**: No (optional feature)
- **Services**: API
- **Format**: `sk-...`
- **Description**: OpenAI API key for AI-powered features
- **How to obtain**: https://platform.openai.com/api-keys

### OPENAI_MODEL
- **Type**: String
- **Required**: No
- **Default**: `gpt-4`
- **Services**: API
- **Values**: `gpt-4`, `gpt-4-turbo`, `gpt-3.5-turbo`
- **Description**: OpenAI model to use for AI features

### OPENAI_MAX_TOKENS
- **Type**: Integer
- **Required**: No
- **Default**: `2000`
- **Services**: API
- **Description**: Maximum tokens per AI request

---

## X402 Payment Configuration

### X402_ENABLED
- **Type**: Boolean (true/false)
- **Required**: No
- **Default**: `false`
- **Services**: API
- **Description**: Enable X402 HTTP Payment Protocol

### X402_PRICE_PER_QUERY
- **Type**: Float
- **Required**: No
- **Default**: `0.001`
- **Services**: API
- **Description**: Price in FLR per API query (when X402 enabled)

### X402_WALLET_ADDRESS
- **Type**: String (Ethereum address)
- **Required**: If X402_ENABLED=true
- **Services**: API
- **Description**: Wallet address to receive X402 payments

---

## Development & Testing

### DEBUG
- **Type**: Boolean (true/false)
- **Required**: No
- **Default**: `false`
- **Services**: All
- **Description**: Enable debug mode (verbose logging, etc.)
- **Production**: Always set to `false`

### TESTING
- **Type**: Boolean (true/false)
- **Required**: No
- **Default**: `false`
- **Services**: All
- **Description**: Enable test mode
- **Usage**: Automated testing only

### MOCK_BLOCKCHAIN
- **Type**: Boolean (true/false)
- **Required**: No
- **Default**: `false`
- **Services**: API, Worker
- **Description**: Use mock blockchain instead of real Flare network
- **Usage**: Local development and testing

---

## Complete Environment Variable Templates

### API Service (.env)

```bash
# Database
DATABASE_URL=postgresql+psycopg://iso_user:iso_pass@localhost:5432/iso_mw
AUTO_CREATE_DB=0
RUN_MIGRATIONS=1

# Redis
REDIS_URL=redis://localhost:6379/0

# Blockchain
FLARE_RPC_URL=https://flare-api.flare.network/ext/C/rpc
ANCHOR_CONTRACT_ADDR=0x1234567890123456789012345678901234567890
ANCHOR_PRIVATE_KEY=0xabcdef...
ANCHOR_ABI_PATH=contracts/EvidenceAnchor.abi.json

# API Configuration
ALLOW_ORIGINS=http://localhost:3000,https://app.example.com
ARTIFACTS_DIR=artifacts
PORT=8000

# Security
SERVICE_PRIVATE_KEY=a1b2c3d4e5f6... # 64 hex chars
SERVICE_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----\nZm9vYmFy...\n-----END PUBLIC KEY-----

# Queue
RQ_QUEUES=default

# AI (Optional)
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4

# X402 (Optional)
X402_ENABLED=true
X402_PRICE_PER_QUERY=0.001
X402_WALLET_ADDRESS=0x...

# Development
DEBUG=false
LOG_LEVEL=info
```

### Worker Service (.env)

```bash
# Database
DATABASE_URL=postgresql+psycopg://iso_user:iso_pass@localhost:5432/iso_mw
RUN_MIGRATIONS=0

# Redis
REDIS_URL=redis://localhost:6379/0

# Blockchain
FLARE_RPC_URL=https://flare-api.flare.network/ext/C/rpc
ANCHOR_CONTRACT_ADDR=0x1234567890123456789012345678901234567890
ANCHOR_PRIVATE_KEY=0xabcdef...
ANCHOR_ABI_PATH=contracts/EvidenceAnchor.abi.json

# Storage
ARTIFACTS_DIR=artifacts

# Security (must match API)
SERVICE_PRIVATE_KEY=a1b2c3d4e5f6... # Same as API
SERVICE_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----\nZm9vYmFy...\n-----END PUBLIC KEY-----

# Queue
RQ_QUEUES=default

# Development
LOG_LEVEL=info
```

### Frontend Service (.env.local)

```bash
# API Connection
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000

# Or for production
# NEXT_PUBLIC_API_BASE_URL=https://api.example.com
```

### Agent Service (.env)

```bash
# API Connection
API_BASE_URL=http://localhost:8000

# Agent Identity
AGENT_ID=agent-x402-01
AGENT_API_KEY=your-api-key-here

# Database (optional)
DATABASE_URL=postgresql+psycopg://iso_user:iso_pass@localhost:5432/iso_mw

# Logging
LOG_LEVEL=info
```

---

## Railway-Specific Variables

When deploying to Railway, use these variable reference formats:

### Database Reference
```bash
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

### Redis Reference
```bash
REDIS_URL=${{Redis.REDIS_URL}}
```

### Service-to-Service References
```bash
# Public domain (for browser access)
NEXT_PUBLIC_API_BASE_URL=https://${{api.RAILWAY_PUBLIC_DOMAIN}}

# Private domain (for service-to-service)
API_BASE_URL=http://${{api.RAILWAY_PRIVATE_DOMAIN}}
```

---

## Security Best Practices

1. **Never commit sensitive variables** to version control
2. **Use different values** for development, staging, and production
3. **Rotate keys regularly**, especially:
   - ANCHOR_PRIVATE_KEY
   - SERVICE_PRIVATE_KEY
   - API keys
4. **Limit access** to environment variable configuration
5. **Use strong random values** for all secrets
6. **Monitor for unauthorized access** via logging
7. **Document all variables** and their purpose
8. **Validate variables** on application startup

---

## Troubleshooting

### Variable Not Being Read

1. Check variable name spelling (case-sensitive)
2. Restart service after adding variable
3. Check variable is set in correct service
4. Verify Railway variable references syntax

### Connection Errors

1. Verify DATABASE_URL format
2. Check REDIS_URL is accessible
3. Ensure blockchain RPC URL is reachable
4. Verify CORS origins include frontend URL

### Authentication Failures

1. Confirm SERVICE_PRIVATE_KEY matches SERVICE_PUBLIC_KEY
2. Check AGENT_API_KEY is valid
3. Verify ANCHOR_PRIVATE_KEY has sufficient balance

---

## Generating Required Values

### Blockchain Private Key

```bash
# Using OpenSSL
openssl ecparam -genkey -name secp256k1 -out private_key.pem
openssl ec -in private_key.pem -text -noout
```

### Service Keys (Ed25519)

```python
from nacl.signing import SigningKey
from nacl.encoding import RawEncoder
import base64

# Generate key pair
signing_key = SigningKey.generate()
verify_key = signing_key.verify_key

# Private key (hex)
private_hex = signing_key.encode(encoder=RawEncoder).hex()
print(f"SERVICE_PRIVATE_KEY={private_hex}")

# Public key (base64 PEM-like)
public_b64 = base64.b64encode(verify_key.encode()).decode()
print(f"SERVICE_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----\\n{public_b64}\\n-----END PUBLIC KEY-----")
```

### API Keys

```python
import secrets
api_key = secrets.token_urlsafe(32)
print(f"AGENT_API_KEY={api_key}")
```

---

## Additional Resources

- [Railway Deployment Guide](./RAILWAY_DEPLOYMENT_GUIDE.md)
- [Development Setup](../README.md)
- [Security Best Practices](./SECURITY.md)
- [API Documentation](./API.md)
