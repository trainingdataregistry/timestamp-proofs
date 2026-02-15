# Training Data Registry - Timestamp Proofs

This repository contains independently verifiable timestamp proofs for the [Training Data Registry](https://trainingdataregistry.org).

## What This Repository Contains

Daily Merkle roots of all domain verifications and URL registrations. Each file represents one day's batch of verified events.

```
2026/
  02/
    2026-02-14.json
    2026-02-15.json
```

## File Format

Each JSON file contains **only the Merkle root** (not individual records):

```json
{
  "version": "1.0",
  "schema": "merkle-root-only",
  "period_date": "2026-02-14",
  "root_hash": "a1b2c3d4e5f6...",
  "verification_count": 42,
  "generated_at": "2026-02-15T00:05:00.000Z",
  "verification_instructions": "To verify a timestamp, request a Merkle proof from the Training Data Registry API."
}
```

## Privacy by Design

**Only the Merkle root is published here.** Individual URLs, domains, and leaf hashes are stored privately in our database. This ensures:

- No personal data is exposed publicly (URLs may contain PII)
- No pseudonymized data that could be matched or reversed
- Full cryptographic verifiability is maintained via API-provided proofs

## How to Verify a Timestamp

### Step 1: Request Your Proof

Use the Training Data Registry API to get your Merkle proof:

```
GET https://trainingdataregistry.org/api/timestamp-proof?domain_id=YOUR_DOMAIN_ID
GET https://trainingdataregistry.org/api/timestamp-proof?registration_id=YOUR_REGISTRATION_ID
```

### Step 2: Receive Your Proof Data

```json
{
  "timestamped": true,
  "leaf_hash": "e5f6g7h8...",
  "leaf_type": "domain_verification",
  "merkle_root": "a1b2c3d4e5f6...",
  "merkle_proof": [
    { "hash": "sibling1...", "position": "right" },
    { "hash": "sibling2...", "position": "left" }
  ],
  "period_date": "2026-02-14",
  "github": {
    "commit_sha": "abc123...",
    "commit_url": "https://github.com/trainingdataregistry/timestamp-proofs/commit/abc123"
  }
}
```

### Step 3: Verify Independently

```javascript
// 1. You already have your leaf hash from the API response

// 2. Walk the Merkle proof to compute the root
let currentHash = leafHash;
for (const step of merkleProof) {
  const pair = step.position === 'right'
    ? [currentHash, step.hash]
    : [step.hash, currentHash];
  pair.sort(); // Sorted pair hashing
  currentHash = sha256(pair[0] + pair[1]);
}

// 3. Verify it matches the root in this repo
const fileContent = await fetch(`https://raw.githubusercontent.com/trainingdataregistry/timestamp-proofs/main/2026/02/2026-02-14.json`);
const { root_hash } = await fileContent.json();
assert(currentHash === root_hash);

// 4. Check the Git commit timestamp for authoritative proof of when this existed
```

## Why This Matters

- **Independence**: Anyone can verify timestamps without trusting TDR
- **Immutability**: Git commits cannot be altered without changing the hash
- **Transparency**: All timestamp proofs are public and auditable
- **Privacy**: Individual records are never exposed publicly
- **Legal Evidence**: Provides court-admissible proof of when preferences were declared

## Technical Details

- **Hash algorithm**: SHA-256
- **Tree construction**: Binary Merkle tree with sorted pair hashing
- **Processing schedule**: Daily at 00:05 UTC
- **Leaf format**: `domain:{domain}|verified_at:{timestamp}` or `url:{url}|registered_at:{timestamp}|training:{bool}|inference:{bool}|archive:{bool}`

## Related Resources

- [Training Data Registry](https://trainingdataregistry.org)
- [Public Registry Search](https://trainingdataregistry.org/search)
- [API Documentation](https://trainingdataregistry.org/for-ai-companies)

## License

The data in this repository is public domain (CC0). Anyone may use it for verification purposes.

