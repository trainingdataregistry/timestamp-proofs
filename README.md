# Training Data Registry - Timestamp Proofs

This repository contains independently verifiable timestamp proofs for the [Training Data Registry](https://trainingdataregistry.org).

## What This Repository Contains

Daily Merkle roots of all domain verifications and URL registrations. Each file represents one day's batch of verified events.

```
2026/
  02/
    2026-02-10.json
    2026-02-11.json
    2026-02-12.json
```

## File Format

Each JSON file contains:

```json
{
  "version": "1.0",
  "period_date": "2026-02-10",
  "root_hash": "a1b2c3d4...",
  "verification_count": 42,
  "generated_at": "2026-02-11T00:05:00.000Z",
  "leaves": [
    {
      "hash": "e5f6g7h8...",
      "type": "domain_verification",
      "domain": "example.com",
      "timestamp": "2026-02-10T14:32:00.000Z"
    },
    {
      "hash": "i9j0k1l2...",
      "type": "url_registration",
      "url": "https://example.com/article",
      "timestamp": "2026-02-10T15:45:00.000Z"
    }
  ]
}
```

## How to Verify a Timestamp

1. **Find the relevant date file** based on when the verification occurred
2. **Check the Git commit** - The commit timestamp proves when this data existed
3. **Verify the Merkle path** - Use the leaf hashes to reconstruct the root

### Verification Steps

```javascript
// 1. Compute the leaf hash
const leafData = `domain:example.com|verified_at:2026-02-10T14:32:00.000Z`;
const leafHash = sha256(leafData);

// 2. Verify the leaf appears in the file
const found = leaves.find(l => l.hash === leafHash);

// 3. Reconstruct the Merkle root from all leaves
const computedRoot = computeMerkleRoot(leaves.map(l => l.hash));

// 4. Verify it matches the stored root
assert(computedRoot === root_hash);

// 5. Check the Git commit timestamp
// The commit date is the authoritative timestamp
```

## Why This Matters

- **Independence**: Anyone can verify timestamps without trusting TDR
- **Immutability**: Git commits cannot be altered without changing the hash
- **Transparency**: All timestamp proofs are public and auditable
- **Legal Evidence**: Provides court-admissible proof of when preferences were declared

## Related Resources

- [Training Data Registry](https://trainingdataregistry.org)
- [Public Registry Search](https://trainingdataregistry.org/search)
- [API Documentation](https://trainingdataregistry.org/for-ai-companies)

## License

The data in this repository is public domain (CC0). Anyone may use it for verification purposes.
