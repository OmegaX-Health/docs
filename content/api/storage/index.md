---
title: Storage
#description: ""
slug: storage
weight: 2
cascade:
  type: docs
sidebar:
  exclude: false
---

## POST /storage

Creates a blockchain record for data hashes referencing off-chain stored data.

```
Authentication: Required (Bearer Token)
```

Request Body:

```json
{
  "hash": "6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae",
  "type": "AI diagnosis",
  "blockchain": "0"  // "0" for Solana, "1" for BSC
}
```

Response (Success):

```json
{
  "status": "success",
  "hash": "6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae",
  "cid": "bafybeibfyuyi5taf4wldkdvsxz4qmym5vpilkhpjn2lvyar3247o6srq54",
  "type": "AI diagnosis",
  "ipfsLink": "https://bafybeibfyuyi5taf4wldkdvsxz4qmym5vpilkhpjn2lvyar3247o6srq54.ipfs.w3s.link/6149588f74d1307b1e241f95b75e8fd71e35fae01b6bfb9a6d1a2a8af40370ae.pdf",
  "blockchainType": "0",
  "solanaVerified": true,
  "solanaSignature": "46WCjdiyMPGUpdv1mQNUjQ4x6rxi8S4Ao9YHn6teHnJGhWuKNMxYU2uAZGA9JshTZ6nFcHaob15CGhUEaMzGCk6Q",
  "solanaLink": "https://solscan.io/tx/46WCjdiyMPGUpdv1mQNUjQ4x6rxi8S4Ao9YHn6teHnJGhWuKNMxYU2uAZGA9JshTZ6nFcHaob15CGhUEaMzGCk6Q"
}
```