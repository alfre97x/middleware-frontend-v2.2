# FI Messages Testing Guide

## ✅ Unit Tests - PASSED

All unit tests have passed successfully:

```
============================================================
SUMMARY
============================================================
✓ PASS: Imports
✓ PASS: Router exists
✓ PASS: Endpoints defined
✓ PASS: Schemas defined
✓ PASS: ISO generators exist
✓ PASS: Router registered

6/6 tests passed

🎉 All tests passed! Implementation is correct.
```

### Running Unit Tests

Unit tests verify the code implementation without requiring a running server:

```bash
python tests/test_fi_messages_unit.py
```

## Integration Tests

Integration tests require a running server to test the actual HTTP endpoints.

### Prerequisites

1. **Start the server:**
   ```bash
   python -m uvicorn app.main:app --reload --port 8001
   ```

2. **Ensure database is initialized** (if using SQLite):
   ```bash
   # Database will auto-create if AUTO_CREATE_DB=true in settings
   ```

3. **Set environment variables** (if needed):
   ```bash
   # Copy .env.example to .env and configure
   cp .env.example .env
   ```

### Running Integration Tests

Run the full test suite that tests all 4 endpoints:

```bash
python scripts/test_fi_suite.py
```

Expected output:
```
RECORD 200 {'receipt_id': '...', 'status': 'pending'}
ARTS_BEFORE 200 [...]
C056 200 {'message_id': 'camt056-...', 'type': 'camt.056', ...}
C029 200 {'message_id': 'camt029-...', 'type': 'camt.029', ...}
P007 200 {'message_id': 'pacs007-...', 'type': 'pacs.007', ...}
P009 200 {'message_id': 'pacs009-...', 'type': 'pacs.009', ...}
ARTS_AFTER 200 [...]
HAVE_TYPES ['camt.029', 'camt.056', 'pacs.007', 'pacs.009', ...]
MISSING []
```

## Manual Testing with curl

### 1. Record a Tip (Get Receipt ID)

```bash
curl -X POST http://localhost:8001/v1/iso/record-tip \
  -H "Content-Type: application/json" \
  -d '{
    "tip_tx_hash": "0xabc123def456...",
    "chain": "flare",
    "amount": "0.01",
    "currency": "FLR",
    "sender_wallet": "0xSENDER_ADDRESS",
    "receiver_wallet": "0xRECEIVER_ADDRESS",
    "reference": "test:payment:001"
  }'
```

Response:
```json
{
  "receipt_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "pending"
}
```

### 2. Generate camt.056 (Cancellation Request)

```bash
curl -X POST http://localhost:8001/v1/iso/camt056/550e8400-e29b-41d4-a716-446655440000 \
  -H "Content-Type: application/json" \
  -d '{"reason_code": "CUST"}'
```

Response:
```json
{
  "message_id": "camt056-...",
  "type": "camt.056",
  "receipt_id": "550e8400-e29b-41d4-a716-446655440000",
  "url": "/files/550e8400-e29b-41d4-a716-446655440000/camt056.xml"
}
```

### 3. Generate camt.029 (Resolution)

```bash
curl -X POST http://localhost:8001/v1/iso/camt029/550e8400-e29b-41d4-a716-446655440000 \
  -H "Content-Type: application/json" \
  -d '{"resolution_code": "APPR"}'
```

### 4. Generate pacs.007 (Payment Reversal)

```bash
curl -X POST http://localhost:8001/v1/iso/pacs007/550e8400-e29b-41d4-a716-446655440000 \
  -H "Content-Type: application/json" \
  -d '{"reason_code": "TECH"}'
```

### 5. Generate pacs.009 (FI Credit Transfer)

```bash
curl -X POST http://localhost:8001/v1/iso/pacs009/550e8400-e29b-41d4-a716-446655440000
```

### 6. List All Artifacts for a Receipt

```bash
curl http://localhost:8001/v1/iso/messages/550e8400-e29b-41d4-a716-446655440000
```

Response:
```json
[
  {
    "type": "pain.001",
    "url": "/files/550e8400-.../pain001.xml",
    "sha256": "0x...",
    "created_at": "2026-02-03T00:00:00"
  },
  {
    "type": "camt.056",
    "url": "/files/550e8400-.../camt056.xml",
    "sha256": "0x...",
    "created_at": "2026-02-03T00:01:00"
  },
  ...
]
```

### 7. Download Generated XML

```bash
curl http://localhost:8001/files/550e8400-e29b-41d4-a716-446655440000/camt056.xml
```

## Testing with Authentication

If authentication is enabled, add API key header:

```bash
curl -X POST http://localhost:8001/v1/iso/camt056/{receipt_id} \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key-here" \
  -d '{"reason_code": "CUST"}'
```

## API Documentation

Once the server is running, access the interactive API documentation:

- **Swagger UI:** http://localhost:8001/docs
- **ReDoc:** http://localhost:8001/redoc

These provide interactive testing interfaces for all endpoints.

## Verification Checklist

- [ ] Unit tests pass
- [ ] Server starts without errors
- [ ] Can record a tip transaction
- [ ] Can generate camt.056 for a receipt
- [ ] Can generate camt.029 for a receipt
- [ ] Can generate pacs.007 for a receipt
- [ ] Can generate pacs.009 for a receipt
- [ ] XML files are created in artifacts directory
- [ ] Database records are created for each artifact
- [ ] Can list all artifacts for a receipt
- [ ] Can download generated XML files
- [ ] Authentication works (if enabled)
- [ ] Multi-tenant isolation works (if enabled)

## Troubleshooting

### Server Won't Start

1. Check if port 8001 is already in use
2. Verify Python dependencies are installed: `pip install -r requirements.txt`
3. Check database connection settings

### Tests Fail with Connection Error

1. Ensure server is running on port 8001
2. Check firewall settings
3. Verify server is listening on 127.0.0.1

### Import Errors

1. Ensure you're in the project root directory
2. Check PYTHONPATH includes the project directory
3. Verify all dependencies are installed

### 404 Not Found

1. Verify the receipt ID exists
2. Check the URL path is correct
3. Ensure the router is registered in app_factory.py

### 403 Forbidden

1. Check authentication credentials (API key or SIWE)
2. Verify project_id matches for multi-tenant setups
3. Ensure the principal has access to the receipt

## Test Results Location

- Unit test output: Console
- Integration test output: Console
- Generated XML files: `artifacts/{receipt_id}/`
- Database records: Check `iso_artifacts` table

## Performance Testing

For performance testing, use tools like:

```bash
# Apache Bench
ab -n 100 -c 10 http://localhost:8001/v1/iso/pacs009/{receipt_id}

# wrk
wrk -t4 -c100 -d30s http://localhost:8001/v1/iso/pacs009/{receipt_id}
```

## Success Criteria

✅ All 4 endpoints are accessible
✅ Each endpoint generates the correct ISO message type
✅ XML files are properly formatted and valid
✅ Database artifacts are created
✅ Authentication and authorization work correctly
✅ Error handling provides clear messages
✅ Performance is acceptable (< 500ms per request)
