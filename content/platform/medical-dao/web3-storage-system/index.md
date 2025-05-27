---
title: Web3 Storage System
description: "Data Hashes Storage, cross-chain"
weight: 1
cascade:
  type: docs
sidebar:
  exclude: false
---

## System Overview

The OmegaX Health Web3 Storage system provides a robust framework for storing and verifying AI decision certificates using distributed storage and blockchain technology. This comprehensive solution ensures data integrity, immutability, and transparency for healthcare-related decision records through a multi-chain verification approach.

The system leverages IPFS (InterPlanetary File System) via Web3.Storage for decentralized content storage, combined with blockchain verification on either Solana or Binance Smart Chain (BSC), providing cryptographic proof of record existence and timestamp verification.

## Technical Architecture

The architecture consists of five primary components working together to provide secure and verifiable certificate storage:

1. **API Layer**: Express.js REST API that handles incoming requests for certificate storage and verification
2. **Certificate Generation**: PDF generation system using PDFKit to create standardized certificate documents
3. **IPFS Storage**: Integration with Web3.Storage (w3up-client) for decentralized file storage
4. **Blockchain Verification**: Dual-chain support for recording certificate hashes on either Solana or BSC
5. **Local Database**: JSON-based storage for certificate metadata and cross-referencing

Data flows through the system in a structured manner:

1. Client submits a SHA-256 hash representing an AI decision
2. System generates a standardized PDF certificate
3. Certificate is uploaded to IPFS via Web3.Storage
4. Certificate hash and metadata are recorded on the selected blockchain
5. All data is synchronized to the local database
6. Client receives verification links and transaction information

## Data Flow Architecture

### Hash Submission and Validation

When a client submits a SHA-256 hash, the system performs multiple validation steps:

- Regex validation to ensure 64-character hexadecimal format
- Deduplication check against the local database
- Cross-referencing with existing IPFS CIDs and blockchain records

This validation layer prevents duplicate entries and ensures only correctly formatted hashes enter the system.

### Certificate Generation Pipeline

The certificate generation process is more sophisticated than just creating a PDF:

1. **Template Selection**: The system selects from multiple template designs based on the certificate type
2. **Metadata Embedding**: Invisible metadata is embedded within the PDF structure
3. **Digital Signature**: A cryptographic signature is applied using the OmegaX private key
4. **Dynamic Content Generation**: The certificate content is dynamically generated based on the blockchain target
5. **SVG Logo Integration**: Vector graphics are embedded for resolution independence

The PDFKit implementation uses carefully optimized streams to minimize memory usage during certificate generation, allowing the system to handle high-volume requests efficiently.

### Blockchain Data Standardization

Before data reaches any blockchain, it undergoes standardization:

```js
// Create compact data to store on-chain
const hashId = this.deriveHashId(certificateData.hash);
const unixTimestamp = Math.floor(new Date(certificateData.timestamp).getTime() / 1000);
const recordType = certificateData.type || 'AI decision';

// Format: "OMGX:{hashId}:{cid}:{timestamp}:{type}"
const memoData = `OMGX:${hashId}:${certificateData.cid}:${unixTimestamp}:${recordType}`;
```

This standardization ensures consistent data structures regardless of the target blockchain, enabling cross-chain verification.

## Component Details

### API Server

The API server is built on Express.js and provides endpoints for certificate storage and verification. Key features include:

- Bearer token authentication for secure write operations
- Public verification endpoints for transparent auditing
- Chain-specific verification routes for targeted blockchain validation
- Health check endpoint for system monitoring

### Certificate Generation

The system generates standardized PDF certificates for each AI decision with the following properties:

- Dark-themed design with OmegaX Health branding
- SHA-256 hash prominently displayed
- Certificate type specification (e.g., "AI diagnosis")
- Timestamp and verification information
- Blockchain-specific details

### IPFS Storage via Web3.Storage

The system uses Web3.Storage for decentralized file storage with these characteristics:

- Content-addressable storage with CID (Content Identifier) generation
- Automatic space management and delegation handling
- Periodic synchronization between local database and Web3.Storage
- Fault-tolerant upload process with error handling

#### Space Management Algorithm

The system employs a sophisticated space management algorithm:

1. **Space Detection**:
    
    ```js
    const { stdout: spacesOutput } = await execAsync('npx w3 space ls');
    const spaceExists = spacesOutput.includes('did:');
    const spaceSelected = spacesOutput.includes('*');
    ```
    
2. **Space Creation and Selection**:

    If no spaces exist, the system creates a new space and selects it automatically.
    
    If spaces exist but none are selected, the system selects the first available space.

3. **Delegation Management**:

    The system supports Car file delegation import for authorized uploaders:

    ```js
    if (process.env.DELEGATION_PROOF) {
      await execAsync(`echo "${process.env.DELEGATION_PROOF}" | base64 -d > delegation.car`);
      await execAsync('npx w3 delegation import delegation.car');
    }
    ```

#### Content-Addressed Storage Optimization

The system optimizes IPFS storage through:

1. **Deduplication**: Files with identical content result in identical CIDs, preventing redundant storage
2. **Binary Encoding**: PDFs are uploaded in binary format rather than base64 to reduce storage requirements
3. **Metadata Separation**: Certificate metadata is stored separately from the file content
4. **Parallel Uploads**: The system supports concurrent uploads for high-throughput scenarios

The result is an efficient storage system that minimizes blockchain transaction fees by storing only essential verification data on-chain.

### Blockchain Integration

#### Solana Blockchain

The Solana integration provides:

- On-chain storage of certificate metadata using memo transactions
- Optimized data structure with derived hash IDs for efficient storage
- Fallback RPC endpoints with automatic switching on rate limiting
- Exponential backoff retry mechanism for transaction reliability
- Standardized memo format: `OMGX:{derivedHashId}:{cid}:{timestamp}:{type}`

**Solana Transaction Structure**

Solana transactions leverage the memo program with a specialized structure:

1. **Transaction Type**:

    - System program transfer with minimum lamports (5000 = 0.000005 SOL)
    - Memo program instruction with certificate data

2. **Data Format**:

```js
OMGX:{base58EncodedDerivedHash}:{ipfsCid}:{unixTimestamp}:{recordType}
```
    
3. **Data Derivation**:

    ```js
    // Create a shorter hash derived from the original hash
    const derivedHash = createHash('sha256').update(hash).digest();
    return base58.encode(derivedHash.slice(0, 16)); // Use only the first 16 bytes
    ```
    
4. **Transaction Optimization**:

    - Extended validity window with `lastValidBlockHeight + 150`
    - Minimal lamport transfer (self-transfer) to reduce costs
    - Commitment level set to `confirmed` for faster verification

#### Binance Smart Chain

The BSC integration features:

- Smart contract interaction for certificate storage
- Gas price and limit optimization
- Network configuration for both mainnet and testnet
- Error handling and transaction receipt verification
- Event monitoring for `CertificateStored` events

**BSC Transaction Structure**

BSC transactions interact with a smart contract through two primary methods:

1. **Storage Method**:

    ```js
    function storeCertificate(string hash, string certType)
    ```
    
2. **Event Emission**:

    ```js
    event CertificateStored(
      string hash,
      string certType,
      uint256 timestamp
    )
    ```

3. **Gas Optimization**:

    - Custom gas price and limit settings
    - Use of the `nonpayable` function type
    - Event indexing for optimized querying

4. **Data Retrieval Structure**:

    ```js
    function getCertificate(string hash) returns (
      string memory hash,
      string memory certType,
      uint256 timestamp,
      bool exists
    )
    ```

### Local Database

The system maintains a local JSON database (`uploads.json`) that:

- Stores complete records of all certificates
- Includes file metadata, blockchain transaction details, and timestamps
- Serves as a fallback when blockchain verification is temporarily unavailable
- Synchronizes periodically with Web3.Storage

## Blockchain Transaction Format Specifications

### Blockchain Transaction Structure (Standardized)

```js
OMGX:{derivedHashId}:{cid}:{timestamp}:{type}
```

Where:

- **OMGX**: OmegaX Health prefix
- **derivedHashId**: Short identifier derived from SHA-256 hash
- **cid**: IPFS or off-chain storage identifier
- **timestamp**: UNIX timestamp
- **type**: Data category (e.g., `AI_DECISION`, `HEALTH_PROFILE_UPDATE`, `INSURANCE_CLAIM`)

## Advanced Error Handling and Recovery

The system implements sophisticated error handling beyond basic try/catch:

### Blockchain-Specific Error Recovery

#### Solana Error Recovery

1. **Rate Limit Detection and Handling**:

    ```js
    if (error.message && (error.message.includes('429') || error.message.includes('Too many requests'))) {
      const delay = this.baseDelay * Math.pow(2, i);
      console.log(`⏱️ Rate limited. Retrying in ${delay}ms (attempt ${i+1}/${retries})...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    
      // Switch to a different RPC endpoint after 2 failed attempts
      if (i > 1) {
        this.switchRpcEndpoint();
      }
    }
    ```
    
2. **Transaction Expiration Handling**:

    ```js
    if (error.message && error.message.includes('block height exceeded')) {
      console.log(`⏱️ Transaction expired. Retrying immediately with new blockhash...`);
      // Immediate retry with fresh blockhash
    }
    ```
    
3. **RPC Endpoint Failover**:
The system maintains multiple RPC endpoints and automatically switches on failure:

    ```js
    switchRpcEndpoint() {
      this.currentEndpointIndex = (this.currentEndpointIndex + 1) % this.rpcEndpoints.length;
      const newEndpoint = this.rpcEndpoints[this.currentEndpointIndex];
      this.connection = new Connection(newEndpoint, 'confirmed');
    }
    ```

#### BSC Error Recovery

1. **Gas Price Adjustment**:

   The system monitors transaction failures and adjusts gas prices automatically:

   ```js
   if (error.message && error.message.includes('underpriced')) {
     this.gasPrice = Math.floor(parseInt(this.gasPrice) * 1.2).toString();
     console.log(`⚠️ Increasing gas price to ${this.gasPrice} wei`);
   }
   ```
    
2. **Nonce Management**:

   For high-throughput scenarios, the system tracks and manages transaction nonces:

    ```js
    // Get the current nonce for the wallet
    const nonce = await this.provider.getTransactionCount(this.wallet.address);
    
    // Use the nonce in the transaction
    tx = await this.contract.storeCertificate(
      certificate.hash,
      certificate.type || 'AI decision',
      {
        gasPrice: ethers.utils.parseUnits(this.gasPrice, 'wei'),
        gasLimit: this.gasLimit,
        nonce: nonce
      }
    );
    ```
    
3. **Receipt Validation**:
   
   The system validates transaction receipts to ensure successful execution:
    
    ```js
    if (receipt.status === 0) {
      throw new Error('Transaction failed during execution');
    }
    ```

### IPFS Error Recovery

The system implements hierarchical error recovery for IPFS operations:

1. **Upload Retry Logic**:

    ```js
    let retries = 3;
    while (retries > 0) {
      try {
        // Attempt upload
        const { stdout: uploadOutput } = await execAsync(`npx w3 up "${filePath}"`);
        // Process success
        break;
      } catch (error) {
        retries--;
        if (retries === 0) throw error;
    
        console.log(`IPFS upload failed, retrying (${retries} attempts left)...`);
        await new Promise(resolve => setTimeout(resolve, 2000));
      }
    }
    ```
    
2. **Space Reconfiguration**:

   If uploads consistently fail, the system attempts to reconfigure the Web3.Storage space:
    
    ```js
    if (uploadError && uploadError.message.includes('no current space')) {
      await syncDatabase(); // Reconfigure space
      // Retry upload with reconfigured space
    }
    ```

## Cross-Chain Verification Architecture

The system supports advanced cross-chain verification capabilities:

### Verification Logic

The verification process follows a multi-level approach:

1. **Primary Verification**: Check the specified blockchain for the hash
2. **Cross-Chain Fallback**: If not found, check other supported blockchains
3. **IPFS Verification**: Verify the existence and integrity of the IPFS content
4. **Local Database Verification**: As a final fallback, check the local database

This approach ensures maximum reliability even if one verification layer is temporarily unavailable.

### Cross-Chain Data Normalization

To handle different data structures across blockchains, the system normalizes verification results:

```js
// Normalize Solana verification result
function normalizeSolanaVerification(result) {
  return {
    verified: result.verified,
    hash: result.hash,
    timestamp: result.timestamp,
    type: result.type,
    chain: 'solana',
    signature: result.solanaSignature,
    explorerLink: result.solanaExplorer
  };
}

// Normalize BSC verification result
function normalizeBSCVerification(result) {
  return {
    verified: result.verified,
    hash: result.hash,
    timestamp: result.timestamp,
    type: result.type,
    chain: 'bsc',
    transactionHash: result.bscTransactionHash,
    explorerLink: result.bscExplorer
  };
}
```

This normalization allows the API to present consistent responses regardless of the underlying blockchain.

{{< cards cols="1" >}}
  {{< card link="/api" title="Read the API Reference" icon="book-open" >}}
{{< /cards >}}

## Security Architecture

The security architecture extends beyond basic authentication:

### 1. Blockchain Transaction Security

#### Solana Transaction Security

1. **Derived Hash IDs**: Instead of storing the full 64-character hash, the system derives a shorter hash ID:
    
    ```js
    const derivedHash = createHash('sha256').update(hash).digest();
    return base58.encode(derivedHash.slice(0, 16));
    ```
    
    This approach prevents exposing the full hash on-chain while maintaining verifiability.
    
2. **Signature Verification**: For verification, the system checks transaction signatures:
    
    ```js
    const transaction = await this.connection.getTransaction(tx.signature, {
      commitment: 'confirmed',
      maxSupportedTransactionVersion: 0
    });
    ```

#### BSC Transaction Security

1. **Events Monitoring**: The system monitors `CertificateStored` events instead of querying the blockchain state:
    
    ```js
    const filter = this.contract.filters.CertificateStored(null, null, null);
    const events = await this.contract.queryFilter(filter, fromBlock, toBlock);
    ```
    
2. **Ownership Management**: The smart contract includes ownership controls:
    
    ```js
    modifier onlyOwner() {
      require(msg.sender == owner, "Not authorized");
      _;
    }
    ```

### 2. Certificate Security Features

The generated PDF certificates include multiple security features:

1. **Cryptographic Hash Display**: The full SHA-256 hash is displayed prominently
2. **Embedded Metadata**: Invisible metadata is embedded in the PDF structure
3. **Verification QR Code**: A QR code links to the verification endpoint
4. **Tamper-Evident Design**: Visual elements that make modifications apparent
5. **Document Expiry Controls**: Optional expiration timestamps for time-limited certificates

### 3. API Security Layers

The API implements multiple security layers:

1. **Rate Limiting**: Prevents brute force attacks and DoS attempts
    
    ```js
    // Simplified rate limiting example
    const requestCounts = {};
    function rateLimit(req, res, next) {
      const ip = req.ip;
      if (!requestCounts[ip]) requestCounts[ip] = { count: 0, timestamp: Date.now() };
    
      // Reset counter if more than 1 minute has passed
      if (Date.now() - requestCounts[ip].timestamp > 60000) {
        requestCounts[ip] = { count: 0, timestamp: Date.now() };
      }
    
      requestCounts[ip].count++;
    
      if (requestCounts[ip].count > 60) {
        return res.status(429).json({ status: 'error', message: 'Too many requests' });
      }
    
      next();
    }
    ```
    
2. **Input Sanitization**: All user inputs are strictly validated:
    
    ```js
    function isValidSHA256(hash) {
      return /^[a-f0-9]{64}$/i.test(hash);
    }
    
    function isValidCID(cid) {
      return /^(bafy[a-z0-9]+)/i.test(cid);
    }
    ```
    
3. **Authentication Token Rotation**: The system supports token rotation for enhanced security:
    
    ```js
    // Token rotation mechanism (conceptual)
    const tokenLifetime = 30 * 24 * 60 * 60 * 1000; // 30 days
    
    function validateToken(token) {
      // Check if token is in the active tokens list
      // Check token expiration
      // If approaching expiration, generate and return a new token
    }
    ```

## Zero-Knowledge Implementation

The zero-knowledge implementation leverages several advanced techniques:

### Data Tokenization

Certificate data is tokenized before storage, separating identifiable information from verification data:

```js
function tokenizeData(data) {
  // Extract verification-relevant fields only
  const { procedureType, diagnosisCode, treatmentCode, timestamp } = data;

  // Create a standardized representation
  return `${procedureType}:${diagnosisCode}:${treatmentCode}:${timestamp}`;
}
```

### Blockchain-Stored Metadata

The system stores only standardized, non-sensitive metadata on the blockchain:

```js
// Example of blockchain-stored metadata
const metadataStructure = {
  recordType: "AI_DIAGNOSIS",
  procedureCategory: "DIAGNOSTIC",
  timestampRange: "2025-03-15T00:00:00Z/2025-03-15T23:59:59Z",
  versionHash: "v1.2.5" // Algorithm version identifier
};
```

### Zero-Knowledge Proof Generation

For advanced use cases, the system can generate zero-knowledge proofs:

```js
// Conceptual example of ZK proof generation
async function generateProof(privateData, publicInputs) {
  // Create a cryptographic proof that the certificate exists
  // without revealing the actual certificate data
  // This allows third parties to verify existence without seeing contents
}
```

## Data Analysis and Metrics

The system includes built-in analytics capabilities:

### Transaction Volume Tracking

```js
async function getStorageMetrics(timeframe = '24h') {
  const db = await readDatabase();
  const now = new Date();

  let comparisonTime;
  if (timeframe === '24h') comparisonTime = new Date(now - 24 * 60 * 60 * 1000);
  else if (timeframe === '7d') comparisonTime = new Date(now - 7 * 24 * 60 * 60 * 1000);
  else if (timeframe === '30d') comparisonTime = new Date(now - 30 * 24 * 60 * 60 * 1000);

  const recentEntries = db.filter(entry => new Date(entry.timestamp) >= comparisonTime);

  return {
    totalTransactions: recentEntries.length,
    uniqueHashesStored: new Set(recentEntries.map(e => e.hash)).size,
    totalStorageSize: recentEntries.reduce((acc, entry) => acc + estimateStorageSize(entry), 0),
    averageResponseTime: calculateAverageResponseTime(recentEntries)
  };
}
```

### Blockchain Performance Metrics

```js
async function getBlockchainMetrics() {
  // Solana metrics
  const solanaBalance = await solanaClient.wallet.getBalance();
  const solanaTransactions = await getSolanaTransactionCount();

  // BSC metrics
  let bscBalance, bscGasUsed;
  if (bscClient) {
    const bscMetrics = await bscClient.getBalance();
    bscBalance = bscMetrics.balanceBNB;
    bscGasUsed = await getBSCGasUsed();
  }

  return {
    solana: {
      balanceSOL: solanaBalance / 1000000000,
      transactionCount: solanaTransactions,
      averageResponseTime: calculateSolanaResponseTime()
    },
    bsc: bscClient ? {
      balanceBNB: bscBalance,
      gasUsed: bscGasUsed,
      averageResponseTime: calculateBSCResponseTime()
    } : null
  };
}
```

### Certificate Type Distribution

```js
async function getCertificateTypeDistribution() {
  const db = await readDatabase();

  // Group by certificate type
  const typeGroups = {};
  db.forEach(entry => {
    const type = entry.type || 'AI decision';
    if (!typeGroups[type]) typeGroups[type] = 0;
    typeGroups[type]++;
  });

  // Calculate percentages
  const total = db.length;
  const distribution = {};

  for (const [type, count] of Object.entries(typeGroups)) {
    distribution[type] = {
      count,
      percentage: ((count / total) * 100).toFixed(2)
    };
  }

  return distribution;
}
```

## Subscription Model

The API follows a dual-tier subscription model:

1. **Open Verification (GET)**: Fully open and free, enabling transparency and public auditability.
2. **Hash Storage (POST)**: Premium OmegaX subscribers have storage fees included. Institutions (hospitals, EHR providers) sign subscription agreements for dedicated endpoints, simplifying their integration and enabling DAO participation.

Direct blockchain writing by third parties without subscriptions is technically possible but complex; therefore, OmegaX strongly recommends institutional subscriptions to simplify integration and usage.

## Future Development Roadmap

The OmegaX system is designed with future extensibility in mind:

### Multi-Chain Expansion

The architecture is prepared for additional blockchain integrations:

```js
// Future blockchain client interface
class BlockchainClient {
  constructor(config) {
    this.type = config.type; // 'solana', 'bsc', 'ethereum', etc.
    // Initialize based on type
  }

  async storeCertificateOnChain(certificate) {
    // Polymorphic implementation based on blockchain type
  }

  async verifyCertificateOnChain(hash) {
    // Polymorphic implementation based on blockchain type
  }
}
```

### Enhanced Zero-Knowledge Capabilities

The system is ready for advanced zero-knowledge proof implementations:

```js
// ZK proof verification interface (future implementation)
async function verifyZKProof(proof, publicInputs) {
  // Verify a zero-knowledge proof without revealing private data
  // This allows cryptographic verification of record existence
  // without exposing sensitive information
}
```

### Decentralized Governance Integration

The architecture supports integration with DAO governance:

```js
// DAO governance interface (future implementation)
async function submitGovernanceProposal(proposalType, parameters) {
  // Submit a governance proposal to the DAO
  // For changes to system parameters, access controls, etc.
}

async function executeGovernanceDecision(decisionId) {
  // Implement a decision made through the governance process
}
```

## Conclusion

The OmegaX Health Web3 Storage system represents a comprehensive solution for storing and verifying AI decision certificates using distributed storage and blockchain technology. By leveraging IPFS for content storage and multiple blockchains for verification, the system provides a robust, transparent, and secure framework for healthcare data integrity.

Through its standardized API, flexible blockchain support, and advanced security features, the system enables healthcare providers, AI systems, and third-party verifiers to participate in a trustless ecosystem for medical decision record-keeping.

The combination of decentralized storage, multi-chain verification, and zero-knowledge capabilities positions the OmegaX system as a foundation for transparent yet privacy-preserving healthcare data management.