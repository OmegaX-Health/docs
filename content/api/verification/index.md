---
title: Verification
#description: ""
slug: verification
weight: 3
cascade:
  type: docs
sidebar:
  exclude: false
---

## GET /verify

Enables verification of blockchain-stored hashes and cross-checking with off-chain stored data.

**Authentication**: Not required (Public)

Query Parameters:

- `hash`: SHA-256 hash (64 characters)
- `cid`: IPFS Content Identifier

Response (Success):

```json
{
  "success": true,
  "hash": "6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae",
  "cid": "bafybeibfyuyi5taf4wldkdvsxz4qmym5vpilkhpjn2lvyar3247o6srq54",
  "type": "AI diagnosis",
  "timestamp": "2025-02-26T10:35:08.889Z",
  "ipfsLink": "https://bafybeibfyuyi5taf4wldkdvsxz4qmym5vpilkhpjn2lvyar3247o6srq54.ipfs.w3s.link/6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae.pdf",
  "blockchainType": "0",
  "solanaVerified": true,
  "solanaSignature": "46WCjdiyMPGUpdv1mQNUjQ4x6rxi8S4Ao9YHn6teHnJGhWuKNMxYU2uAZGA9JshTZ6nFcHaob15CGhUEaMzGCk6Q",
  "solanaLink": "https://solscan.io/tx/46WCjdiyMPGUpdv1mQNUjQ4x6rxi8S4Ao9YHn6teHnJGhWuKNMxYU2uAZGA9JshTZ6nFcHaob15CGhUEaMzGCk6Q",
  "solanaOnChainVerified": true
}
```

## POST /solana/verify

Chain-specific verification for Solana blockchain.

**Authentication**: Not required (Public)

Request Body:

```json
{
  "hash": "6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae"
}
```

Response (Success):

```json
{
  "status": "success",
  "verifiedOnSolana": true,
  "hash": "6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae",
  "cid": "bafybeibfyuyi5taf4wldkdvsxz4qmym5vpilkhpjn2lvyar3247o6srq54",
  "type": "AI diagnosis",
  "timestamp": "2025-02-26T10:35:08.889Z",
  "solanaSignature": "46WCjdiyMPGUpdv1mQNUjQ4x6rxi8S4Ao9YHn6teHnJGhWuKNMxYU2uAZGA9JshTZ6nFcHaob15CGhUEaMzGCk6Q",
  "solanaExplorer": "https://solscan.io/tx/46WCjdiyMPGUpdv1mQNUjQ4x6rxi8S4Ao9YHn6teHnJGhWuKNMxYU2uAZGA9JshTZ6nFcHaob15CGhUEaMzGCk6Q"
}
```

## POST /bsc/verify

Chain-specific verification for Binance Smart Chain.

**Authentication**: Not required (Public)

Request Body:

```json
{
  "hash": "f54a61103bdf5f6bc5f502769c82b9ea11a321beda95fb5d4fbbb5e6b0a34a4c"
}
```

Response (Success):

```json
{
  "status": "success",
  "verifiedOnBSC": true,
  "hash": "f54a61103bdf5f6bc5f502769c82b9ea11a321beda95fb5d4fbbb5e6b0a34a4c",
  "type": "AI diagnosis",
  "timestamp": "2025-03-26T17:04:43.688Z",
  "blockchainTimestamp": "1711472683"
}
```