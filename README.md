# Efficient Sparse Merkle Tree Library for Fe

An optimised implementation of a Sparse Merkle Tree written in Fe that omits empty subtrees when computing roots. Empty subtrees are assumed to have a special hash value of zero. See [post on ethresear.ch](https://ethresear.ch/t/optimizing-sparse-merkle-trees/3751) for the original idea proposed by Vitalik.

## How to generate insert & update proofs

Below is an example JavaScript snippet that instantiates an SMT and generates proofs that can be used with this Fe lib.

```sh
npm install --save @kevincharm/sparse-merkle-tree @noble/hashes
```

```ts
import { SparseMerkleTreeKV } from "@kevincharm/sparse-merkle-tree";
import { ZeroHash, concat, hashMessage, Wallet, Contract } from "ethers";

// Initialise client representation of an empty SMT with depth=160
const smt = new SparseMerkleTree(160);

// Insert a new (K,V) entry
const index = BigInt(Wallet.createRandom().address);
const value = hashMessage("Fred Fredburger");

// Get SMT proof of insertion
const {
    newLeaf,
    leaf: oldLeaf
    index,
    enables,
    siblings,
} = smt.insert(index, value);
```

Then once we have these values, we can compute some roots:

```fe
use smt_fe::compute_address_smt_root

contract Main {
    root: u256

    pub fn main(
        mut self,
        newLeaf: u256,
        oldLeaf: u256,
        index: address,
        enables: u256,
        siblings: Array<u256, 160>
    ) {
        let current_root: u256 = compute_address_smt_root(
            leaf: oldLeaf,
            index,
            enables,
            siblings
        )
        assert self.root = current_root

        let new_root: u256 = compute_address_smt_root(
            leaf: newLeaf,
            index,
            enables,
            siblings
        )
        self.root = new_root
    }
}
```
