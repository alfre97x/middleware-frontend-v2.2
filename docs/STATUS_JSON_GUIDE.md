# Status.json Guide

## Problem

When using `/v1/iso/record-tip`, the system creates an `evidence.zip` bundle containing a `receipt.json` file. However, this file shows `status: "pending"` even after the receipt has been successfully anchored, because the evidence bundle is created early in the process and remains immutable.

## Solution

We've added a `status.json` file that gets created/updated AFTER anchoring completes, providing users with the current, up-to-date status.

## File Structure

After a receipt is processed, you'll find these files:

```
artifacts/{receipt-id}/
├── evidence.zip          ← Immutable (receipt.json shows status="pending")
├── pain001.xml           ← ISO payment message
├── status.json           ← NEW: Current status (updated after anchoring)
├── pain002.xml           ← Status report (if generated)
├── camt054.xml           ← Notification (if anchored)
└── vc.json               ← Verifiable Credential (if configured)
```

## status.json Format

```json
{
  "receipt_id": "77d31961-ec05-4e2d-8e18-23275e39338b",
  "status": "anchored",
  "bundle_hash": "0xb1bcc3fbeedfc5af88d4012fb58b8729dec4ecc2eb7811b9cfbde43f88b1beda",
  "flare_txid": "efcacfc2babf2ea0c0bd579a53c4f523bb96f7faa7e2592f081d6076a0a4e831",
  "anchored_at": "2026-02-08T19:00:11.915480",
  "created_at": "2026-02-08T18:59:53.793163",
  "last_updated": "2026-02-08T19:00:12.123456"
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `receipt_id` | string | Receipt UUID |
| `status` | string | Current status: "pending", "awaiting_anchor", "anchored", or "failed" |
| `bundle_hash` | string | 0x-prefixed SHA256 hash of evidence.zip |
| `flare_txid` | string | Blockchain transaction ID (null if not yet anchored) |
| `anchored_at` | string | ISO 8601 timestamp when anchored (null if not yet anchored) |
| `created_at` | string | ISO 8601 timestamp when receipt was created |
| `last_updated` | string | ISO 8601 timestamp when this file was last updated |

## Accessing status.json

### Via HTTP

```bash
curl http://localhost:8001/files/{receipt-id}/status.json
```

Example:
```bash
curl http://localhost:8001/files/77d31961-ec05-4e2d-8e18-23275e39338b/status.json
```

### Via File System

```bash
cat artifacts/{receipt-id}/status.json
```

## When is status.json Created/Updated?

**Platform Mode:**
- Created/Updated: After anchoring succeeds or fails
- Location: `artifacts/{receipt-id}/status.json`

**Tenant Mode:**
- Created/Updated: When `/v1/iso/confirm-anchor` is called
- Location: `artifacts/{receipt-id}/status.json`

## Use Cases

### 1. Check Current Status Without API Call

Instead of calling `/v1/iso/receipts/{id}`, simply fetch the status.json file:

```bash
curl http://localhost:8001/files/{receipt-id}/status.json | jq '.status'
```

### 2. Monitor Anchoring Progress

Poll status.json to watch for status changes:

```bash
watch -n 2 "curl -s http://localhost:8001/files/{receipt-id}/status.json | jq"
```

### 3. Verify Immutability vs Current State

Compare evidence.zip contents with status.json:

```bash
# Extract and view receipt.json from evidence.zip
unzip -p artifacts/{receipt-id}/evidence.zip receipt.json | jq '.status'
# Output: "pending" (immutable snapshot)

# View current status
cat artifacts/{receipt-id}/status.json | jq '.status'
# Output: "anchored" (current state)
```

## Implementation Details

### Code Locations

**Background Job (Platform Mode):**
- File: `app/jobs.py`
- Function: `process_receipt_job()`
- Trigger: After status update to "anchored" or "failed"

**Confirm Anchor Endpoint (Tenant Mode):**
- File: `app/api/routes/confirm_anchor.py`
- Endpoint: `POST /v1/iso/confirm-anchor`
- Trigger: After status validation and update

### Helper Function

Both locations use the same helper:

```python
def _write_status_json(rec: models.Receipt) -> None:
    """Write current receipt status to status.json file."""
    # Creates/updates artifacts/{receipt-id}/status.json
```

## Benefits

✅ **Non-Breaking:** Existing code continues to work
✅ **Immutability Preserved:** evidence.zip remains cryptographically verifiable
✅ **User-Friendly:** Clear, current status without API calls
✅ **Real-Time:** Updated immediately after status changes
✅ **Simple Integration:** Just read a JSON file

## Migration

No migration required! This feature:
- Adds a new file (status.json)
- Doesn't modify existing files
- Works with all existing receipts
- Backward compatible

For existing receipts without status.json:
- File will be created on next status update
- Or users can still query via `/v1/iso/receipts/{id}` API

## FAQ

**Q: Why does receipt.json inside evidence.zip show "pending"?**
A: The evidence bundle is created before anchoring and must remain immutable for cryptographic verification.

**Q: Which file should I trust for current status?**
A: Use `status.json` for current status, `evidence.zip/receipt.json` for the immutable snapshot.

**Q: Will old receipts get status.json?**
A: Not automatically, but they can still be queried via the API. New receipts will always have status.json after anchoring.

**Q: Can status.json be tampered with?**
A: Yes, but the evidence.zip bundle hash is anchored on-chain, so tampering is detectable. For verification, always check the on-chain bundle hash.

**Q: Is status.json included in evidence.zip?**
A: No, status.json lives alongside evidence.zip in the artifacts directory, not inside it.
