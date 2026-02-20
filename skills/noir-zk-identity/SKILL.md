---
name: noir-zk-identity
description: Build Zero-Knowledge proofs for identity verification using Noir. Focus on selective disclosure of attributes (age, name matching) without revealing underlying passport data. Use when implementing privacy-preserving KYC where users prove claims (over-18, name match) without exposing full identity documents. Generates Noir circuits, client-side proving, and Solana-compatible verification.
---

# Noir ZK Identity Verification

This skill teaches agents to build Zero-Knowledge circuits for privacy-preserving identity verification.

## Scope

Focus on **two core claims** for MinKYC Phase 1:
1. **Age Verification** (`over_18_claim`) — Prove user is ≥18 years old
2. **Identity Binding** (`name_match_claim`) — Prove submitted name matches passport

## Why Noir for MinKYC

- **Rust-like syntax** — Easy for developers familiar with Solana/Anchor
- **Backend agnostic** — Compile to any proving system (Barretenberg, Halo2, etc.)
- **NoirJS** — Prove in browser/mobile (no server needed)
- **Selective disclosure** — Reveal only the claim, not the passport data

---

## Core Concepts

### 1. Age Verification Circuit (`over_18_claim`)

**What it proves:** "I was born before [date 18 years ago]" without revealing birthdate

**Inputs:**
- **Private:** Date of birth (from passport chip)
- **Public:** Current date, threshold date (today - 18 years)
- **Output:** Boolean (true if ≥18)

**Noir Pattern:**
```rust
fn over_18_claim(dob: Field, current_date: Field) -> bool {
    let threshold = current_date - (18 * 365 * 24 * 60 * 60); // 18 years in seconds
    dob <= threshold
}
```

**Real Implementation:**
```rust
use dep::std;

// Passport date format: YYYYMMDD as Field
fn over_18_claim(
    dob: Field,           // Private: Date of birth from passport
    current_date: Field   // Public: Today's date (provided by verifier)
) -> bool {
    // Calculate 18 years ago in YYYYMMDD format
    let cutoff_year = (current_date / 10000) - 18;
    let cutoff_month_day = current_date % 10000;
    let cutoff_date = (cutoff_year * 10000) + cutoff_month_day;
    
    // Prove DOB is before or equal to cutoff
    dob <= cutoff_date
}

#[test]
fn test_over_18() {
    // DOB: 1990-01-01 (19900101)
    // Current: 2026-02-20 (20260220)
    // Cutoff: 2008-02-20 (20080220)
    assert(over_18_claim(19900101, 20260220));
    
    // DOB: 2010-01-01 (20100101) — should fail (only 16)
    assert(!over_18_claim(20100101, 20260220));
}
```

---

### 2. Identity Binding Circuit (`name_match_claim`)

**What it proves:** "The name I submitted hashes to the same value as the name on my passport" without revealing either name

**Inputs:**
- **Private:** Passport name hash (pre-computed), Submitted name hash
- **Public:** None (or optional salt for replay protection)
- **Output:** Boolean (true if hashes match)

**Why hashing:**
- Passport names may have encoding issues (accents, special chars)
- Hashing normalizes comparison
- Revealing hash doesn't reveal name

**Noir Pattern:**
```rust
use dep::std::hash::pedersen;

fn name_match_claim(
    passport_name_hash: Field,    // Private: Hash of name from passport
    submitted_name_hash: Field,   // Private: Hash of name user entered
    salt: Field                   // Public: Prevents rainbow table attacks
) -> bool {
    // Both names hashed with same salt should match
    passport_name_hash == submitted_name_hash
}

// Helper: Hash name off-circuit (in TypeScript/Rust)
// fn hash_name(name: &str, salt: &str) -> Field {
//     pedersen_hash(name.to_lowercase().trim() + salt)
// }
```

**Real Implementation:**
```rust
use dep::std::hash::pedersen;
use dep::std::hash::pedersen_hash;

// Hash computed client-side before circuit
fn name_match_claim(
    passport_name_hash: Field,    // Private
    submitted_name_hash: Field,   // Private
    _salt: Field                  // Public (ensures same normalization)
) -> bool {
    // Simple equality check — hashing done client-side
    // to handle string encoding complexities
    passport_name_hash == submitted_name_hash
}

#[test]
fn test_name_match() {
    // Both "John Smith" hashed with same salt
    let hash1 = 0x1234567890abcdef; // pretend hash
    let hash2 = 0x1234567890abcdef;
    assert(name_match_claim(hash1, hash2, 12345));
    
    // Different names
    let hash3 = 0xfedcba0987654321;
    assert(!name_match_claim(hash1, hash3, 12345));
}
```

---

## Combined Circuit (Both Claims)

For efficiency, combine both into one proof:

```rust
fn minkyc_verification(
    // Private inputs (never revealed)
    dob: Field,
    passport_name_hash: Field,
    submitted_name_hash: Field,
    
    // Public inputs (known to verifier)
    current_date: Field,
    salt: Field
) -> (bool, bool) {
    let age_ok = over_18_claim(dob, current_date);
    let name_ok = name_match_claim(passport_name_hash, submitted_name_hash, salt);
    
    (age_ok, name_ok)
}
```

---

## Client-Side Flow (Mobile App)

```typescript
// 1. Read passport via NFC
const passportData = await readPassportNFC();

// 2. User enters name
const submittedName = await promptUser("Enter your full name");

// 3. Compute hashes client-side
const salt = generateRandomSalt();
const passportNameHash = pedersenHash(passportData.name + salt);
const submittedNameHash = pedersenHash(submittedName + salt);

// 4. Generate ZK proof
const proof = await generateProof({
    dob: passportData.dob,
    passportNameHash,
    submittedNameHash,
    currentDate: getCurrentDate(),
    salt
});

// 5. Submit to Solana
await submitProofToSolana(proof, {
    requirementHash: hash("over_18 + name_match"),
    publicSignals: [currentDate, salt]
});
```

---

## Solana Verification

```rust
// Anchor program verifies proof
pub fn verify_kyc(
    ctx: Context<VerifyKYC>,
    proof: Vec<u8>,
    public_inputs: Vec<Field>,
    requirement_hash: [u8; 32]
) -> Result<bool> {
    // Verify proof using Noir verifier
    let is_valid = noir_verifier::verify(
        &MINKYC_CIRCUIT_VK,  // Verification key (stored on-chain)
        &proof,
        &public_inputs
    )?;
    
    // Prevent replay
    require!(!ctx.accounts.proof_receipt.used, ErrorCode::ProofAlreadyUsed);
    
    // Emit verification event
    emit!(VerificationEvent {
        identity: ctx.accounts.identity.key(),
        requirement_hash,
        timestamp: Clock::get()?.unix_timestamp
    });
    
    Ok(is_valid)
}
```

---

## Security Considerations

### 1. Replay Protection
```rust
// Add nullifier to prevent same proof reused
fn minkyc_verification(
    // ... other inputs
    nullifier: Field  // Derived from passport unique ID + salt
) -> (bool, bool, Field) {
    // ... verification logic
    (age_ok, name_ok, nullifier)
}
```

### 2. Date Manipulation
- Use blockchain timestamp, not client-provided date
- Or require verifier-provided `current_date` as public input

### 3. Name Normalization
- Hash lowercase, trimmed names
- Handle accents/unicode consistently
- Document exact normalization rules

---

## Testing Strategy

```rust
#[test]
fn test_full_verification() {
    // Valid: 25-year-old, name matches
    let result = minkyc_verification(
        20000101,           // DOB: 25 years old
        hash("john smith"), // Passport name hash
        hash("john smith"), // Submitted name hash
        20260220,           // Current date
        12345               // Salt
    );
    assert(result.0 == true);  // Age OK
    assert(result.1 == true);  // Name OK
}

#[test]
fn test_underage_fails() {
    let result = minkyc_verification(
        20100101,           // DOB: 16 years old
        hash("john smith"),
        hash("john smith"),
        20260220,
        12345
    );
    assert(result.0 == false); // Age fails
}

#[test]
fn test_name_mismatch_fails() {
    let result = minkyc_verification(
        20000101,           // Age OK
        hash("john smith"), // Passport: John Smith
        hash("jane doe"),   // Submitted: Jane Doe
        20260220,
        12345
    );
    assert(result.0 == true);  // Age OK
    assert(result.1 == false); // Name fails
}
```

---

## File Structure

```
MinKYC/
├── circuits/
│   ├── Nargo.toml              # Noir project config
│   ├── src/
│   │   ├── main.nr             # Combined circuit
│   │   ├── age_verification.nr # Over-18 claim
│   │   └── name_matching.nr    # Identity binding
│   └── Prover.toml             # Test inputs
├── js/
│   ├── src/
│   │   ├── prove.ts            # Client-side proving
│   │   └── hash.ts             # Name normalization
│   └── package.json
└── solana/
    └── programs/
        └── verifier.rs         # On-chain verification
```

---

## Next Steps (Phase 2)

| Feature | Circuit | Priority |
|---------|---------|----------|
| EU residency | `eu_resident_claim` | Medium |
| Sanctions check | `not_sanctioned_claim` | Medium |
| Passport expiry | `passport_not_expired` | Low |
| Multiple attributes | `composite_claim` | Low |

---

## Resources

- **Noir Docs:** https://noir-lang.org/docs/
- **NoirJS:** Browser proving without server
- **Barretenberg:** Aztec's proving backend (fast)
- **MinKYC Repo:** https://github.com/GetOutOfThatGarden/MinKYC

---

*Version 1.0 — Focus: Age verification + Name matching*
*Created for MinKYC privacy-preserving KYC*
