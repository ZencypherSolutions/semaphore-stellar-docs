# SemaphoreVerifierKeyPts Soroban Implementation - Complete Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Installation Guide](#installation-guide)
4. [Contract Implementation Details](#contract-implementation-details)
5. [Usage Guide](#usage-guide)
6. [Testing](#testing)
7. [Troubleshooting](#troubleshooting)

## Introduction

### What is SemaphoreVerifierKeyPts?

The SemaphoreVerifierKeyPts contract is a crucial component of the Semaphore zero-knowledge proof system, responsible for managing verification key points. This Soroban implementation ports the original Solidity contract to the Stellar blockchain ecosystem.

### Key Features

- Verification key point storage and management
- Merkle tree depth-based point retrieval
- Data structure validation
- Robust error handling

## Installation Guide & Setup

1. **Rust Toolchain**

   ```bash
   # Install Rust
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

   # Add Wasm target
   rustup target add wasm32-unknown-unknown

   # Verify installation
   rustc --version
   cargo --version
   ```

2. **Soroban CLI**

   ```bash
   # Install Soroban CLI
   cargo install soroban-cli --version 20.5.0

   # Verify installation
   soroban --version
   ```

3. **Development Tools**

   ```bash
   # Ubuntu/Debian
   sudo apt-get update
   sudo apt-get install build-essential git pkg-config libssl-dev

   # MacOS
   brew install openssl pkg-config
   ```

### Dependencies Configuration

Create/update `Cargo.toml`:

```toml
[package]
name = "semaphore_contract"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
soroban-sdk = "20.5.0"

[dev-dependencies]
soroban-sdk = { version = "20.5.0", features = ["testutils"] }

[features]
testutils = ["soroban-sdk/testutils"]

[profile.release]
opt-level = "z"
overflow-checks = true
debug = 0
strip = "symbols"
debug-assertions = false
panic = "abort"
codegen-units = 1
lto = true
```

```rust
// Indicates that this is a no_std crate, meaning it doesn't depend on the Rust standard library
// This is required for Soroban contract development
#![no_std]

// Import required dependencies from soroban_sdk
// - contract: Attribute macro for marking a struct as a Soroban contract
// - contracterror: Attribute macro for defining contract-specific errors
// - contractimpl: Attribute macro for implementing contract functions
// - symbol_short: Macro for creating storage keys
// - BytesN: Fixed-size byte array type
// - Env: Soroban environment type
// - Vec: Soroban vector type
use soroban_sdk::{
    contract, contracterror, contractimpl, symbol_short, BytesN, Env, Vec
};

// Constants used throughout the contract
// POINT_SIZE: Size of each verification key point in bytes
// This is set to 32 bytes to accommodate the full point data
const POINT_SIZE: usize = 32;

// SET_SIZE: Number of points in each set
// This defines how many points are processed together
const SET_SIZE: u32 = 8;

// Contract Error Definition
// #[contracterror] - Marks this enum as a contract error type
// Various derive macros to implement necessary traits:
// - Copy: Allow the type to be copied
// - Clone: Allow the type to be cloned
// - Debug: Enable debug formatting
// - Eq, PartialEq: Enable equality comparisons
// - PartialOrd, Ord: Enable ordering comparisons
// #[repr(u32)] - Specifies that the enum should be represented as u32
#[contracterror]
#[derive(Copy, Clone, Debug, Eq, PartialEq, PartialOrd, Ord)]
#[repr(u32)]
pub enum Error {
    // Error variant for when the verification key points invariant is violated
    // The value 1 is the error code used to identify this error
    VKPtBytesMaxDepthInvariantViolated = 1,
}

// Contract Definition
// #[contract] - Marks this struct as a Soroban contract
#[contract]
pub struct SemaphoreVerifierKeyPts;

// Contract Implementation
// #[contractimpl] - Implements the contract functions
#[contractimpl]
impl SemaphoreVerifierKeyPts {
    /// Initialize the contract with verification key points
    ///
    /// # Parameters
    /// * `env: Env` - Soroban environment object
    ///
    /// # Function Flow
    /// 1. Creates a new vector to store points
    /// 2. Defines sample data points
    /// 3. Stores points in contract storage
    pub fn initialize(env: Env) {
        // Create new vector in the Soroban environment
        let mut points = Vec::new(&env);

        // Define sample verification key points
        // Each point is represented as a 32-bit integer
        // The _i32 suffix explicitly defines the type
        // The u32 as i32 conversion is used where needed
        let sample_data = [
             // First point set - each value represents a verification key point
             0x289691d7_i32,    // Point 1
             0x70593405_i32,    // Point 2
             0x504ae4bd_i32,    // Point 3
             0x7be283f3_i32,    // Point 4
             0x465af66f_i32,    // Point 5
             0x62fc7f1e_i32,    // Point 6
             0x66f03876_i32,    // Point 7
             0xb445efddu32 as i32, // Point 8 (with type conversion)
        ];

        // Iterate through sample data and add each point to the vector
        // iter() - Creates an iterator over references
        // points.push_back - Adds element to end of vector
        for value in sample_data.iter() {
            points.push_back(*value);  // Dereference to get the actual value
        }

        // Store the points in contract storage
        // symbol_short!("POINTS") - Creates a storage key
        // instance().set - Sets value in contract's instance storage
        env.storage().instance().set(&symbol_short!("POINTS"), &points);
    }

    /// Retrieve points for a specific Merkle tree depth
    ///
    /// # Parameters
    /// * `env: Env` - Soroban environment object
    /// * `merkle_tree_depth: i32` - The depth in the Merkle tree to retrieve points for
    ///
    /// # Returns
    /// * `Vec<BytesN<32>>` - Vector of 32-byte points
    pub fn get_pts(env: Env, merkle_tree_depth: i32) -> Vec<BytesN<32>> {
        // Retrieve stored points from contract storage
        // unwrap_or_else creates new vector if none exists
        let stored_points: Vec<i32> = env.storage().instance()
            .get(&symbol_short!("POINTS"))
            .unwrap_or_else(|| Vec::new(&env));

        // Create new vector for results
        let mut result = Vec::new(&env);

        // Calculate starting index based on Merkle tree depth
        // Subtracts 1 from depth and multiplies by SET_SIZE
        let start_idx = ((merkle_tree_depth - 1) * SET_SIZE as i32) as u32;

        // Calculate ending index, ensuring we don't exceed stored points
        let end_idx = (start_idx + SET_SIZE).min(stored_points.len());

        // Process points within calculated range
        for i in start_idx..end_idx {
            if let Some(value) = stored_points.get(i) {
                // Create byte array for point data
                let mut bytes = [0u8; POINT_SIZE];
                // Convert i32 value to bytes and copy to array
                bytes[0..4].copy_from_slice(&value.to_be_bytes());

                // Create BytesN from array and add to result
                result.push_back(BytesN::from_array(&env, &bytes));
            }
        }

        result
    }
```

```rust
    /// Validate the verification key points data structure
    ///
    /// # Parameters
    /// * `env: Env` - Soroban environment object
    /// * `max_depth: i32` - Maximum depth to validate against
    ///
    /// # Returns
    /// * `Result<(), Error>` - Ok(()) if valid, Err with specific error if invalid
    ///
    /// # Error Cases
    /// Returns VKPtBytesMaxDepthInvariantViolated if:
    /// - The number of stored points doesn't match expected length
    pub fn check_invariant(env: Env, max_depth: i32) -> Result<(), Error> {
        // Retrieve stored points from contract storage
        // If no points exist, creates new empty vector
        let stored_points: Vec<i32> = env.storage().instance()
            .get(&symbol_short!("POINTS"))
            .unwrap_or_else(|| Vec::new(&env));

        // Calculate expected length based on max_depth and SET_SIZE
        // Cast to u32 to match storage length type
        let expected_len = (max_depth * SET_SIZE as i32) as u32;

        // Verify that stored points length matches expected length
        // If not, return error indicating invariant violation
        if stored_points.len() != expected_len {
            return Err(Error::VKPtBytesMaxDepthInvariantViolated);
        }

        // Return Ok if validation passes
        Ok(())
    }
}

/// Test module for the SemaphoreVerifierKeyPts contract
/// #[cfg(test)] ensures these tests only compile when running tests
#[cfg(test)]
mod test {
    // Import all items from parent module
    use super::*;

    /// Test initialization of contract
    /// Verifies that:
    /// 1. Contract can be initialized
    /// 2. Points are properly stored
    #[test]
    fn test_initialize() {
        // Create new test environment
        let env = Env::default();

        // Register contract in test environment
        // None indicates no specific ID is required
        let contract_id = env.register_contract(None, SemaphoreVerifierKeyPts);

        // Create client for interacting with contract
        let client = SemaphoreVerifierKeyPtsClient::new(&env, &contract_id);

        // Call initialize function
        client.initialize();

        // Verify initialization by retrieving points
        let points = client.get_pts(&1);

        // Assert points were properly stored
        // Points vector should not be empty after initialization
        assert!(!points.is_empty(), "Points should be initialized");
    }

    /// Test point retrieval functionality
    /// Verifies that:
    /// 1. Points can be retrieved
    /// 2. Correct number of points are returned
    #[test]
    fn test_get_pts() {
        // Create test environment and client
        let env = Env::default();
        let contract_id = env.register_contract(None, SemaphoreVerifierKeyPts);
        let client = SemaphoreVerifierKeyPtsClient::new(&env, &contract_id);

        // Initialize contract with points
        client.initialize();

        // Retrieve points for depth 1
        let points = client.get_pts(&1);

        // Verify retrieved points
        // 1. Check that points were returned
        assert!(!points.is_empty(), "Should return non-empty points array");
        // 2. Verify exact number of points (should match SET_SIZE)
        assert_eq!(points.len(), 8, "Should return exactly 8 points");
    }

    /// Test invariant checking functionality
    /// Verifies that:
    /// 1. Valid configurations pass
    /// 2. Invalid configurations fail
    /// 3. Edge cases are handled properly
    #[test]
    fn test_check_invariant() {
        // Setup test environment and contract
        let env = Env::default();
        let contract_id = env.register_contract(None, SemaphoreVerifierKeyPts);
        let client = SemaphoreVerifierKeyPtsClient::new(&env, &contract_id);

        // Initialize contract
        client.initialize();

        // Test 1: Success case with correct number of points (depth = 1)
        // Should return Ok(Ok(())) for valid configuration
        assert_eq!(client.try_check_invariant(&1), Ok(Ok(())));

        // Test 2: Failure case with incorrect depth (depth = 2)
        // Should return Err with VKPtBytesMaxDepthInvariantViolated
        assert_eq!(
            client.try_check_invariant(&2),
            Err(Ok(Error::VKPtBytesMaxDepthInvariantViolated))
        );

        // Test 3: Edge case with zero depth
        // Should return Err with VKPtBytesMaxDepthInvariantViolated
        assert_eq!(
            client.try_check_invariant(&0),
            Err(Ok(Error::VKPtBytesMaxDepthInvariantViolated))
        );
    }

    /// Test panic behavior for invariant violations
    /// #[should_panic] attribute indicates this test should panic
    /// Verifies that:
    /// 1. Contract properly panics on invalid configurations
    /// 2. Correct error message is provided
    #[test]
    #[should_panic(expected = "Error(Contract, #1)")]
    fn test_check_invariant_panic() {
        // Setup test environment and contract
        let env = Env::default();
        let contract_id = env.register_contract(None, SemaphoreVerifierKeyPts);
        let client = SemaphoreVerifierKeyPtsClient::new(&env, &contract_id);

        // Initialize contract
        client.initialize();

        // Attempt to check invariant with invalid depth
        // This should panic with the specified error message
        client.check_invariant(&2);
    }
}
```

### Running Tests

```bash
# Run all tests
cargo test

# Run specific test
cargo test test_initialize

# Run tests with output
cargo test -- --nocapture
```

## Deployment Guide

### Build Process

```bash
# Debug build
cargo build

# Release build
cargo build --target wasm32-unknown-unknown --release
```

## Troubleshooting

### Common Issues and Solutions

1. **Build Failures**

   ```bash
   # Issue: wasm32-unknown-unknown target missing
   rustup target add wasm32-unknown-unknown

   # Issue: soroban-cli not found
   cargo install soroban-cli
   ```

2. **Runtime Errors**

   - Issue: Storage errors
     - Solution: Check environment initialization
   - Issue: Point conversion errors
     - Solution: Verify data format and byte conversion

3. **Test Failures**
   - Issue: Assertion failures
     - Solution: Check expected vs actual values
   - Issue: Panic messages
     - Solution: Verify error conditions

### References

- [Soroban Documentation](https://soroban.stellar.org/docs/)
- [Original Semaphore Implementation](https://github.com/semaphore-protocol/semaphore)
- [Rust Documentation](https://doc.rust-lang.org/book/)
