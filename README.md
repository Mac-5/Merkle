# Merkle Tree Generator

A pure JavaScript implementation for generating cryptographic hashes and building Merkle trees with detailed logging. This implementation uses ethers.js for SHA-256 and Keccak-256 hash functions, ensuring consistency across blockchain applications.

## Features

- 🔐 Generate random cryptographic hashes (SHA-256 or Keccak-256)
- 🌲 Build Merkle trees with automatic algorithm propagation
- 📊 Detailed step-by-step logging of tree construction
- ✅ Algorithm consistency enforcement (prevents mixing hash functions)
- 🔄 Handles odd number of leaves automatically
- 📝 Hash string utility function

## Installation

1. Ensure you have Node.js installed (v16 or higher recommended)

2. Install dependencies:
```bash
npm install ethers
```

3. Configure your `package.json` to use ES modules:
```json
{
  "type": "module"
}
```

## Usage

### Basic Example

```javascript
import { generateRandomHashes, buildMerkleRoot } from './merkle.js';

// Generate 7 random hashes using Keccak-256
const result = generateRandomHashes(7, 'keccak256');

// Build Merkle tree (automatically uses the same algorithm)
const merkleRoot = buildMerkleRoot(result);

console.log('Merkle Root:', merkleRoot);
```

### Switching Hash Algorithms

```javascript
// Use SHA-256 instead
const result = generateRandomHashes(10, 'sha256');
const merkleRoot = buildMerkleRoot(result);
```

### Hashing Strings

```javascript
import { hashString } from './merkle.js';

const result = hashString('Hello, World!', 'keccak256');
console.log('Hash:', result.hash);
console.log('Algorithm:', result.algorithm);
```

## API Reference

### `generateRandomHashes(count, algorithm)`

Generates a specified number of random cryptographic hashes.

**Parameters:**
- `count` (number): Number of hashes to generate
- `algorithm` (string): Hash algorithm to use - `'sha256'` or `'keccak256'` (default: `'keccak256'`)

**Returns:**
- Object with:
  - `hashes` (string[]): Array of hash strings in hexadecimal format (with '0x' prefix)
  - `algorithm` (string): The algorithm used for generation

**Example:**
```javascript
const result = generateRandomHashes(5, 'sha256');
// result = {
//   hashes: ['0xabc...', '0xdef...', ...],
//   algorithm: 'sha256'
// }
```

### `buildMerkleRoot(leavesInput, algorithmOverride?)`

Builds a Merkle tree and returns the root hash with detailed logging.

**Parameters:**
- `leavesInput` (Object | string[]): 
  - Preferred: Result object from `generateRandomHashes()` (automatically uses correct algorithm)
  - Alternative: Plain array of hash strings (requires `algorithmOverride`)
- `algorithmOverride` (string, optional): Hash algorithm if providing plain array

**Returns:**
- `string`: The Merkle root hash

**Example:**
```javascript
const result = generateRandomHashes(7, 'keccak256');
const root = buildMerkleRoot(result); // Algorithm is implicit

// Or with plain array (not recommended)
const hashes = ['0x123...', '0x456...', '0x789...'];
const root = buildMerkleRoot(hashes, 'keccak256');
```

### `hashString(text, algorithm)`

Hashes a string using the specified algorithm.

**Parameters:**
- `text` (string): Text to hash
- `algorithm` (string): Hash algorithm - `'sha256'` or `'keccak256'` (default: `'keccak256'`)

**Returns:**
- Object with:
  - `hash` (string): The resulting hash
  - `algorithm` (string): The algorithm used

**Example:**
```javascript
const result = hashString('Hello, World!', 'sha256');
console.log(result.hash); // '0x...'
```

### `combineHashes(left, right, algorithm)`

Internal function that combines two hashes. Automatically called by `buildMerkleRoot()`.

**Parameters:**
- `left` (string): Left hash
- `right` (string): Right hash
- `algorithm` (string): Hash algorithm to use

**Returns:**
- `string`: Combined parent hash

## How It Works

### Merkle Tree Construction

1. **Leaf Generation**: Random data is generated and hashed using your chosen algorithm
2. **Pairing**: Leaf hashes are paired and combined (sorted alphabetically before hashing)
3. **Level Building**: Parent hashes form the next level of the tree
4. **Odd Nodes**: If there's an odd number of nodes, the last node is promoted to the next level
5. **Root Calculation**: Process repeats until a single root hash remains

### Algorithm Consistency

The implementation ensures that the hash algorithm used for leaf generation is automatically used throughout the entire tree construction:

```javascript
// ✅ Correct - Algorithm flows through automatically
const result = generateRandomHashes(10, 'sha256');
const root = buildMerkleRoot(result); // Uses SHA-256

// ❌ Prevented - Can't accidentally mix algorithms
const leaves = generateRandomHashes(10, 'sha256');
// No way to accidentally use keccak256 when building the tree!
```

This is critical because mixing hash functions would produce an invalid Merkle tree and incorrect proofs.

## Logging Output

The implementation provides detailed logging at each step:

```
========================================
BUILDING MERKLE TREE
========================================
Algorithm: KECCAK256
Total leaves: 7
========================================

LEVEL 0 (Leaves):
  [0] 0x123...
  [1] 0x456...
  ...

LEVEL 1:
  Processing 7 nodes -> 4 parent nodes
  
  Pair 0:
    Left:  0x123...
    Right: 0x456...
    Combining: 0x123... + 0x456...
    Result:    0xabc...
  ...

========================================
MERKLE ROOT (Final):
0xfinal...
========================================
```

## Use Cases

- **Blockchain Development**: Generate Merkle proofs for transaction verification
- **Data Integrity**: Create compact proofs of data membership
- **Cryptographic Testing**: Test Merkle tree implementations
- **Learning**: Understand how Merkle trees work with detailed logging

## Algorithm Choice

- **Keccak-256**: Standard for Ethereum and many blockchain applications
- **SHA-256**: Bitcoin standard and widely used in general cryptography

## Security Considerations

- Uses `crypto.getRandomValues()` via ethers.js for cryptographically secure random generation
- Hashes are sorted before combination to prevent second-preimage attacks
- Both SHA-256 and Keccak-256 are cryptographically secure hash functions

## Dependencies

- [ethers.js](https://docs.ethers.org/) - For cryptographic hash functions (SHA-256, Keccak-256) and random byte generation

## License

MIT

## Contributing

Contributions are welcome! Please ensure:
- Algorithm consistency is maintained
- Logging remains detailed and clear
- Tests pass for both SHA-256 and Keccak-256

## Examples

### Small Tree (3 leaves)
```javascript
const result = generateRandomHashes(3, 'keccak256');
const root = buildMerkleRoot(result);
```

### Large Tree (100 leaves)
```javascript
const result = generateRandomHashes(100, 'sha256');
const root = buildMerkleRoot(result);
```

### Custom Data
```javascript
// Hash your own data
const data = ['tx1', 'tx2', 'tx3', 'tx4'];
const result = {
  hashes: data.map(tx => hashString(tx, 'keccak256').hash),
  algorithm: 'keccak256'
};
const root = buildMerkleRoot(result);
```

## Support

For issues or questions, please open an issue on the repository.