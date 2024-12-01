# Adapting the Semaphore Protocol to the BLS12-381 Curve

## Introduction

With the introduction of BLS12-381 support in Stellar's Protocol Version 22, there is an opportunity to utilize zero-knowledge proof (ZKP) operations on the Stellar network. However, the Semaphore protocol, which facilitates anonymous signaling and voting, is inherently tied to the BN254 (alt_bn_128) elliptic curve used in Ethereum Virtual Machine (EVM) networks. This report outlines the challenges and necessary steps to adapt Semaphore to the BLS12-381 curve supported by Stellar.

## Challenges in Curve Compatibility

### Incompatibility Issues

- **Different Elliptic Curves**: Semaphore operates on the BN254 curve, whereas Stellar supports BLS12-381. These curves have different mathematical properties, making them incompatible without significant adaptation.
- **Interoperability Concerns**: Adapting Semaphore to BLS12-381 would result in proofs that are not interoperable with other networks using BN254, potentially diminishing cross-chain utility.

## Steps for Migrating the Circom Circuit and Adapting Semaphore

### 1. Modify the Circom Circuit

#### a. Migrate Companion Curve

- **Transition from Baby Jubjub to Jubjub or Bandersnatch**:
  - Semaphore uses the Baby Jubjub curve as a companion to BN254.
  - For BLS12-381, a compatible curve like [Jubjub](https://github.com/zkcrypto/jubjub) or [Bandersnatch](https://ethresear.ch/t/introducing-bandersnatch-a-fast-elliptic-curve-built-over-the-bls12-381-scalar-field/9957) is needed.
- **Challenges**:
  - **Lack of Existing Circom Circuits**: There are no readily available Circom circuits for Jubjub or Bandersnatch.
  - **Expertise Requirement**: Developing new circuits requires deep knowledge in elliptic curve cryptography and circuit design.

#### b. Develop or Adapt Circuits

- **Research Existing Implementations**: Look for any available Circom circuits for Jubjub or Bandersnatch that can be adapted.
- **Create New Circuits**: If none are available, develop new Circom circuits for the chosen companion curve.

### 2. Generate New Verification Keys

- **Update Trusted Setup**:
  - Generate new verification keys specific to BLS12-381.
  - This involves running a new trusted setup ceremony to produce the necessary parameters securely.
- **Process**:
  - **Compile Updated Circuits**: Use the modified Circom circuits to compile new proving and verification keys.
  - **Ensure Security**: Follow best practices to prevent vulnerabilities during the trusted setup.

### 3. Adapt the Poseidon Hash Function

- **Use a BLS12-381 Compatible Version**:
  - The current Poseidon hash function in Semaphore is designed for BN254.
  - **Implementation**: Use an existing Poseidon implementation for BLS12-381, such as [Poseidon for BLS12-381 Circom implementation](https://github.com/jmagan/poseidon-bls12381-circom).
- **Integration**:
  - Replace the BN254-specific Poseidon hash function in the Circom circuits with the BLS12-381 version.
  - Test thoroughly to ensure correctness and performance.

### 4. Plan for Security Audits and Testing

- **Security Audits**:
  - Engage with third-party security experts to audit the adapted protocol.
- **Testing**:
  - Perform extensive unit and integration tests to ensure functionality and security.

## Conclusion

Adapting Semaphore to the BLS12-381 curve involves significant modifications to the cryptographic foundations of the protocol. The key steps include modifying Circom circuits, generating new verification keys, and adapting the Poseidon hash function.

While this adaptation is technically feasible, it requires considerable effort and expertise. Additionally, the loss of interoperability with other networks using BN254 is a significant consideration. Careful evaluation of the benefits versus the resources required is essential before proceeding.

## Recommendations

- **Assess the Trade-offs**:
  - Consider whether the benefits of adaptation outweigh the loss of interoperability.
- **Seek Collaboration**:
  - Engage with the broader cryptographic and developer community for support and potential collaboration.
- **Incremental Implementation**:
  - Start with adapting individual components before fully transitioning the entire protocol.
- **Stay Informed on Industry Trends**:
  - Monitor whether other networks adopt BLS12-381, which may influence the decision.

## References

- [Poseidon for BLS12-381 Circom Implementation](https://github.com/jmagan/poseidon-bls12381-circom)
- [Jubjub Elliptic Curve](https://github.com/zkcrypto/jubjub)
- [Bandersnatch Curve Introduction](https://ethresear.ch/t/introducing-bandersnatch-a-fast-elliptic-curve-built-over-the-bls12-381-scalar-field/9957)
- [Arkworks Documentation on BLS12-381](https://docs.rs/ark-ed-on-bls12-381/latest/ark_ed_on_bls12_381/)

---

By following these steps and considerations, you can adapt the Semaphore protocol to work with the BLS12-381 curve on the Stellar network, enabling advanced privacy features while navigating the challenges posed by cryptographic incompatibilities.