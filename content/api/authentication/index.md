---
title: Authentication
#description: ""
weight: 1
cascade:
  type: docs
sidebar:
  exclude: false
---

Protected endpoints require bearer token authentication:

```
Authorization: Bearer <API_KEY>
```

The API key should be securely stored in the server's environment variables.