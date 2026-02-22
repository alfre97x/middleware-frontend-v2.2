# Railway Deployment Guide

Complete guide for deploying the ISO 20022 Middleware platform on Railway.

## Architecture Overview

The platform consists of 5 main services:

1. **Frontend (Next.js)** - User interface (`railway.json`)
2. **API (FastAPI)** - Backend REST API (`api.railway.json`)
3. **Worker (RQ)** - Background job processor (`worker.railway.json`)
4. **Agent (Node.js)** - X402 payment agent (`agents/iso-x402-agent/railway.json`)
5. **PostgreSQL** - Database (Railway managed service)
6. **Redis** - Queue & caching (Railway managed service)

## Prerequisites

- Railway account ([railway.app](https://railway.app))
- GitHub repository with your code
- Flare blockchain wallet with private key
- OpenAI API key (for AI features)

## Deployment Steps

### 1. Create a New Railway Project

1. Log into Railway
2. Click "New Project"
3. Select "Deploy from GitHub repo"
4. Choose your repository
5. Railway will create an initial project

### 2. Add PostgreSQL Database

1. In your project, click "New"
2. Select "Database" → "Add PostgreSQL"
3. Railway will provision a PostgreSQL instance
4. Note: The `DATABASE_URL` variable will be automatically available to services

### 3. Add Redis

1. In your project, click "New"
2. Select "Database" → "Add Redis"
3. Railway will provision a Redis instance
4. Note: The `REDIS_URL` variable will be automatically available to services

### 4. Deploy the API Service

1. Click "New" → "GitHub Repo" → Select your repo
2. Rename the service to "api"
3. In Settings → General:
   - Root Directory: Leave empty (root of repo)
   - Railway Config File Path: `api.railway.json`
4. Add environment variables (see API Environment Variables section below)
5. Deploy the service

### 5. Deploy the Worker Service

1. Click "New" → "GitHub Repo" → Select your repo again
2. Rename the service to "worker"
3. In Settings → General:
   - Root Directory: Leave empty (root of repo)
   - Railway Config File Path: `worker.railway.json`
4. Add environment variables (see Worker Environment Variables section below)
5. Deploy the service

### 6. Deploy the Frontend Service

1. Click "New" → "GitHub Repo" → Select your repo again
2. Rename the service to "frontend"
3. In Settings → General:
   - Root Directory: Leave empty (root of repo)
   - Railway Config File Path: `railway.json`
4. Add environment variables (see Frontend Environment Variables section below)
5. In Settings → Networking:
   - Generate a public domain
   - Note this URL for use in API CORS configuration
6. Deploy the service

### 7. Deploy the X402 Agent Service (Optional)

1. Click "New" → "GitHub Repo" → Select your repo again
2. Rename the service to "x402-agent"
3. In Settings → General:
   - Root Directory: `agents/iso-x402-agent`
   - Railway Config File Path: `railway.json`
4. Add environment variables (see Agent Environment Variables section below)
5. Deploy the service

## Environment Variables

### API Service Environment Variables

```bash
# Database (auto-provided by Railway PostgreSQL)
DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis (auto-provided by Railway Redis)
REDIS_URL=${{Redis.REDIS_URL}}

# Blockchain Configuration
FLARE_RPC_URL=https://flare-api.flare.network/ext/C/rpc
ANCHOR_CONTRACT_ADDR=0xYourContractAddress
ANCHOR_PRIVATE_KEY=0xYourPrivateKey
ANCHOR_ABI_PATH=contracts/EvidenceAnchor.abi.json

# CORS - Add your frontend domain
ALLOW_ORIGINS=https://your-frontend.railway.app

# Storage
ARTIFACTS_DIR=artifacts

# Cryptographic Keys
SERVICE_PRIVATE_KEY=<generate-hex-key>
SERVICE_PUBLIC_KEY=<generate-pem-key>

# Migration Settings
RUN_MIGRATIONS=1
AUTO_CREATE_DB=0

# Queue Configuration
RQ_QUEUES=default

# AI Configuration (Optional)
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4

# X402 Configuration (Optional)
X402_ENABLED=true
X402_PRICE_PER_QUERY=0.001
```

### Worker Service Environment Variables

```bash
# Database (auto-provided by Railway PostgreSQL)
DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis (auto-provided by Railway Redis)
REDIS_URL=${{Redis.REDIS_URL}}

# Blockchain Configuration
FLARE_RPC_URL=https://flare-api.flare.network/ext/C/rpc
ANCHOR_CONTRACT_ADDR=0xYourContractAddress
ANCHOR_PRIVATE_KEY=0xYourPrivateKey
ANCHOR_ABI_PATH=contracts/EvidenceAnchor.abi.json

# Storage
ARTIFACTS_DIR=artifacts

# Cryptographic Keys (must match API)
SERVICE_PRIVATE_KEY=<same-as-api>
SERVICE_PUBLIC_KEY=<same-as-api>

# Migration Settings (worker should not run migrations)
RUN_MIGRATIONS=0
AUTO_CREATE_DB=0

# Queue Configuration
RQ_QUEUES=default
```

### Frontend Service Environment Variables

```bash
# API Connection - Use the Railway internal URL for the API service
NEXT_PUBLIC_API_BASE_URL=${{api.RAILWAY_PUBLIC_DOMAIN}}

# Or use the full URL if public domain is generated
# NEXT_PUBLIC_API_BASE_URL=https://your-api.railway.app
```

### X402 Agent Service Environment Variables

```bash
# API Connection
API_BASE_URL=${{api.RAILWAY_PRIVATE_DOMAIN}}

# Agent Configuration
AGENT_ID=your-agent-unique-id
AGENT_API_KEY=<generate-api-key>

# Database (if agent needs direct DB access)
DATABASE_URL=${{Postgres.DATABASE_URL}}

# Logging
LOG_LEVEL=info
```

## Generating Cryptographic Keys

### Service Private Key (Hex format)
```bash
# Generate a random 32-byte hex key
openssl rand -hex 32
```

### Service Public Key (PEM format)
```python
# Generate using Python
from nacl.signing import SigningKey
from nacl.encoding import RawEncoder

# Generate key pair
signing_key = SigningKey.generate()
verify_key = signing_key.verify_key

# Save private key (hex)
private_key_hex = signing_key.encode(encoder=RawEncoder).hex()
print(f"SERVICE_PRIVATE_KEY={private_key_hex}")

# Save public key (PEM-like base64)
import base64
public_key_b64 = base64.b64encode(verify_key.encode()).decode()
print(f"SERVICE_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----\n{public_key_b64}\n-----END PUBLIC KEY-----")
```

## Service Dependencies

Services should be deployed in this order:

1. **PostgreSQL** (managed service) - First
2. **Redis** (managed service) - Second
3. **API** - Third (runs migrations)
4. **Worker** - Fourth (depends on Redis and PostgreSQL)
5. **Frontend** - Fifth (depends on API)
6. **X402 Agent** - Last (optional, depends on API)

## Service Interconnection

### Internal vs Public URLs

Railway provides two types of URLs for each service:

- **Private Domain** (`RAILWAY_PRIVATE_DOMAIN`): `servicename.railway.internal` - Use for service-to-service communication
- **Public Domain** (`RAILWAY_PUBLIC_DOMAIN`): `servicename.up.railway.app` - Use for external access

### Recommended Configuration

1. **Frontend → API**: Use public domain (browsers can't access private Railway network)
2. **Worker → API**: Can use private domain for better performance
3. **Agent → API**: Can use private domain for better performance
4. **API → PostgreSQL/Redis**: Use Railway-provided connection URLs

## CORS Configuration

Update the API's `ALLOW_ORIGINS` environment variable to include:

```bash
ALLOW_ORIGINS=https://your-frontend.railway.app,https://your-custom-domain.com
```

Multiple origins are comma-separated.

## Post-Deployment Verification

### 1. Check API Health

Visit: `https://your-api.railway.app/health`

Expected response:
```json
{
  "status": "healthy",
  "database": "connected",
  "redis": "connected"
}
```

### 2. Check Database Migrations

In Railway API service logs, verify:
```
INFO: Running database migrations...
INFO: Migrations completed successfully
```

### 3. Check Worker Status

In Railway worker service logs, verify:
```
INFO: Worker started
INFO: Listening on queue: default
```

### 4. Check Frontend

Visit your frontend URL and verify:
- Homepage loads correctly
- No console errors
- API connection works

### 5. Test Basic Flow

1. Create a new project via the UI
2. Submit a payment message
3. Verify the message appears in operations
4. Check that background jobs are processing (worker logs)

## Troubleshooting

### Database Connection Issues

**Problem**: `could not connect to server: Connection refused`

**Solution**: 
1. Verify PostgreSQL service is running
2. Check `DATABASE_URL` variable is correctly referenced
3. Ensure API service has PostgreSQL in dependencies

### Redis Connection Issues

**Problem**: `Error connecting to Redis`

**Solution**:
1. Verify Redis service is running
2. Check `REDIS_URL` variable is correctly referenced
3. Ensure Worker service has Redis in dependencies

### Migration Errors

**Problem**: `relation "xyz" does not exist`

**Solution**:
1. Check API logs for migration errors
2. Ensure `RUN_MIGRATIONS=1` only on API service
3. Verify `RUN_MIGRATIONS=0` on worker service
4. Manually run migrations if needed:
   ```bash
   railway run alembic upgrade head
   ```

### CORS Errors in Frontend

**Problem**: `CORS policy: No 'Access-Control-Allow-Origin' header`

**Solution**:
1. Add frontend URL to API's `ALLOW_ORIGINS`
2. Ensure format is correct (https://domain, no trailing slash)
3. Restart API service after updating

### Worker Not Processing Jobs

**Problem**: Jobs queued but not processing

**Solution**:
1. Check worker logs for startup errors
2. Verify `REDIS_URL` is correct
3. Ensure `RQ_QUEUES=default` matches queue name in code
4. Check worker service is running

### Build Failures

**Problem**: Build fails during deployment

**Solution**:
1. Check build logs for specific errors
2. Verify Dockerfile syntax
3. Ensure all dependencies are in requirements.txt or package.json
4. Check Railway config file path is correct

## Scaling Considerations

### Horizontal Scaling

Railway supports replicating services:

1. **API**: Can run multiple replicas behind load balancer
2. **Worker**: Can run multiple replicas to process jobs in parallel
3. **Frontend**: Can run multiple replicas
4. **Agent**: Usually single instance, or multiple with unique IDs

### Vertical Scaling

Adjust resources in Railway service settings:
- CPU: Increase for compute-intensive operations
- Memory: Increase for large message processing
- Storage: Monitor artifacts directory size

### Database Scaling

For high load:
1. Consider PostgreSQL connection pooling
2. Monitor query performance
3. Add read replicas if needed
4. Optimize indexes

## Monitoring and Logs

### Accessing Logs

1. Click on a service in Railway dashboard
2. Go to "Deployments" tab
3. Click "View Logs" on the latest deployment

### Key Metrics to Monitor

1. **API**: Request latency, error rate, uptime
2. **Worker**: Queue length, job processing time
3. **Database**: Connection pool usage, query performance
4. **Redis**: Memory usage, connection count

### Setting Up Alerts

Use Railway's webhook integrations:
1. Deploy notifications (webhooks)
2. Custom monitoring via API endpoints
3. Third-party monitoring services (Datadog, Sentry)

## Security Best Practices

### 1. Environment Variables

- Never commit sensitive values to git
- Use Railway's environment variable UI
- Rotate keys regularly
- Use different keys for staging/production

### 2. API Keys

- Generate strong random API keys
- Use Railway's secret management
- Implement rate limiting
- Monitor for unauthorized access

### 3. Database

- Use Railway's managed PostgreSQL (automatic backups)
- Enable connection SSL if available
- Limit database user permissions
- Regular security updates

### 4. CORS

- Only allow specific origins
- Don't use wildcards (*) in production
- Regularly audit allowed origins

## Backup and Recovery

### Database Backups

Railway PostgreSQL includes automatic backups:
1. Daily backups retained for 7 days
2. Manual backups via Railway dashboard
3. Export to external storage if needed

### Application Data

1. Artifacts directory: Configure Railway volumes or external storage (S3)
2. Configuration: Keep environment variables documented
3. Deployment history: Railway maintains deployment history

### Disaster Recovery

1. Document all environment variables
2. Keep infrastructure-as-code (Railway config files)
3. Regular backup verification
4. Test restore procedures
5. Maintain staging environment

## Cost Optimization

### Development Environment

Use Railway's free tier:
- Single replica per service
- Shared resources
- Limited uptime

### Production Environment

1. Right-size service resources
2. Use caching effectively (Redis)
3. Optimize database queries
4. Implement request batching
5. Monitor usage and adjust

## Advanced Configuration

### Custom Domains

1. Go to service settings
2. Add custom domain
3. Configure DNS records
4. SSL certificates auto-provisioned

### Environment-Specific Configs

Create separate Railway projects:
- Development
- Staging
- Production

Each with appropriate resource allocation and configuration.

### CI/CD Integration

Railway supports:
- Automatic deployments on git push
- PR deployments for testing
- Manual deployment controls
- Rollback capabilities

## Support and Resources

- Railway Documentation: https://docs.railway.app
- Railway Discord: https://discord.gg/railway
- Project Issues: [GitHub Repository Issues]
- ISO 20022 Middleware Docs: See `/docs` folder

## Deployment Checklist

Before deploying to production:

- [ ] PostgreSQL service deployed and healthy
- [ ] Redis service deployed and healthy
- [ ] API service deployed with correct environment variables
- [ ] Worker service deployed with correct environment variables
- [ ] Frontend deployed with API URL configured
- [ ] Database migrations ran successfully
- [ ] CORS configured correctly
- [ ] API health check returns 200 OK
- [ ] Frontend can communicate with API
- [ ] Worker processes test job successfully
- [ ] All cryptographic keys generated and set
- [ ] Blockchain contract deployed and address configured
- [ ] Custom domain configured (if applicable)
- [ ] Monitoring and alerts set up
- [ ] Backup strategy implemented
- [ ] Documentation updated with deployment URLs
- [ ] Security audit completed
- [ ] Load testing performed

## Quick Reference: Railway Commands

```bash
# Login to Railway
railway login

# Link to existing project
railway link

# Run commands in Railway environment
railway run <command>

# View logs
railway logs

# Open service in browser
railway open

# Deploy manually
railway up

# Set environment variable
railway variables set KEY=value

# Get environment variables
railway variables
