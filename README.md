# SHA-256 Implementation and Analysis

## Overview

This project is a complete implementation of the SHA-256 (Secure Hash Algorithm 256-bit) cryptographic hash function from scratch in Python, following the specifications outlined in **FIPS PUB 180-4** (Federal Information Processing Standards Publication 180-4). The implementation demonstrates a deep understanding of cryptographic primitives, bitwise operations, and hash function construction.

### Key Components

The project is structured around five core problems that build upon each other to create a full SHA-256 implementation:

1. **Binary Words and Bitwise Operations** - Implementation of fundamental bitwise functions (`Parity`, `Ch`, `Maj`, `Sigma`, and `sigma` functions) that form the building blocks of SHA-256
2. **Round Constants Generation** - Derivation of the 64 round constants from the fractional parts of cube roots of the first 64 prime numbers
3. **Message Padding** - Implementation of the SHA-256 padding scheme to prepare messages for processing in 512-bit blocks
4. **Hash Compression Function** - The core compression function that processes message blocks and updates the hash state through 64 rounds of operations
5. **Password Cracking Analysis** - Demonstration of dictionary attacks on unsalted SHA-256 password hashes and recommendations for secure password storage

### Technologies Used

- **Python 3.x** - Primary programming language
- **NumPy** - For 32-bit unsigned integer arithmetic (`np.uint32`) to ensure proper modular arithmetic and overflow behavior
- **Jupyter Notebook** - Interactive development and documentation environment

### Educational Purpose

This project was developed as part of a Computational Theory module to understand:
- How cryptographic hash functions operate at a low level
- The mathematical foundations behind SHA-256
- The importance of proper password hashing techniques
- The vulnerabilities of using fast cryptographic hashes for password storage
- The Merkle–Damgård construction used in many hash functions

---

## Problem Solutions

### Problem 1: Binary Words and Operations

**Objective**: Implement the fundamental bitwise logical functions used throughout SHA-256.

**Functions Implemented**:

- **`parity(x, y, z)`** - XOR-based parity function that returns 1 for odd numbers of set bits
- **`ch(x, y, z)`** - Choose function that selects bits from y or z based on x
- **`Maj(x, y, z)`** - Majority function that returns the majority bit value across three inputs
- **`ROTR(x, n)`** - Right rotation of bits with wraparound
- **`SHR(x, n)`** - Right shift of bits with zero-fill
- **`Sigma0(x)`** - Uppercase sigma function using three rotations (2, 13, 22)
- **`Sigma1(x)`** - Uppercase sigma function using three rotations (6, 11, 25)
- **`sigma0(x)`** - Lowercase sigma function using rotations (7, 18) and shift (3)
- **`sigma1(x)`** - Lowercase sigma function using rotations (17, 19) and shift (10)

**Key Insights**: These functions introduce non-linearity and bit diffusion into the hash algorithm, making it cryptographically secure. The combination of AND, OR, XOR, and rotation operations ensures that small changes in input produce unpredictable changes in output (avalanche effect).

---

### Problem 2: Fractional Parts of Cube Roots

**Objective**: Generate the 64 round constants (K values) used in SHA-256 by extracting fractional parts of cube roots of prime numbers.

**Implementation**:

1. **`primes(n)`** - Generates the first n prime numbers using trial division
2. **`fractional_cube_roots(primes_list)`** - Computes cube roots and extracts the first 32 bits of their fractional parts

**Mathematical Foundation**: The round constants are derived from `floor(frac(∛p) × 2³²)` where p is a prime number. This ensures the constants are "nothing up my sleeve" numbers - they come from a transparent mathematical process rather than being arbitrarily chosen, which proves they weren't selected to create backdoors.

**Verification**: All 64 generated constants were verified to match the official SHA-256 constants from FIPS PUB 180-4, Section 4.2.2.

---

### Problem 3: Padding

**Objective**: Implement the SHA-256 message padding scheme to prepare arbitrary-length messages for processing in 512-bit blocks.

**Padding Rules** (FIPS 180-4, Section 5.1.1):
1. Append a single `1` bit (represented as byte `0x80`)
2. Append zero bits until message length ≡ 448 (mod 512)
3. Append original message length as 64-bit big-endian integer

**Implementation**: `block_parse(msg)` - A generator function that yields 64-byte blocks with proper padding applied to the final block(s).

**Edge Cases Handled**:
- Empty messages
- Messages that are exact multiples of 64 bytes
- Messages requiring one vs. two padding blocks
- Messages of any arbitrary length

**Design Choice**: Using a generator function (`yield`) provides memory efficiency for large messages, as blocks are produced on-demand rather than storing all blocks in memory simultaneously.

---

### Problem 4: Hashes

**Objective**: Implement the SHA-256 compression function that processes message blocks and produces hash digests.

**Implementation**: `hashing(current, block)` - Processes a 64-byte block through four stages:

1. **Message Schedule Preparation** - Expands 16 input words into 64 words using sigma functions
2. **Working Variable Initialization** - Loads current hash state into temporary variables (a-h)
3. **Compression Loop** - Runs 64 rounds of mixing operations using Ch, Maj, Sigma functions, and round constants
4. **Feed-Forward Addition** - Adds compressed values back to original hash state (modulo 2³²)

**Initial Hash Values**: Derived from fractional parts of square roots of first 8 primes (2, 3, 5, 7, 11, 13, 17, 19).

**Modular Arithmetic**: All additions in SHA-256 use modulo 2³² arithmetic, causing intentional integer overflow. Python's NumPy handles this correctly with `np.uint32`, though it generates overflow warnings - these warnings are expected and indicate correct operation per FIPS 180-4, Section 3.2.

**Verification**: Implementation tested against known SHA-256 test vectors:
- `SHA256("abc") = ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad` ✓

---

### Problem 5: Passwords

**Objective**: Demonstrate the vulnerabilities of using unsalted SHA-256 for password storage through dictionary attack implementation and propose secure alternatives.

**Attack Implementation**: `dictionary_attack(target_hashes)` - Tests common passwords and variations against target hashes.

**Dictionary Strategy**:
- Top passwords from breach databases (password, 123456, qwerty, etc.)
- Common variations (password1, password123, Password, PASSWORD)
- Food, animal, and color words
- Keyboard patterns (qwertyuiop, asdfghjkl)
- Number sequences and year suffixes
- Leetspeak variations (p@ssw0rd, pa$$word)

**Results**: Successfully cracked all three target password hashes within 1,789 attempts, demonstrating the weakness of unsalted SHA-256.

**Security Recommendations**:

1. **Never use SHA-256 directly for passwords** - It's designed for speed (data integrity), not password security
2. **Use proper password hashing algorithms**:
   - **Argon2id** (OWASP recommendation, memory-hard, configurable)
   - **bcrypt** (time-tested, widely supported)
   - **scrypt** (memory-hard, ASIC-resistant)
3. **Always use unique salts** - Prevents rainbow table attacks and ensures identical passwords have different hashes
4. **Apply key stretching** - Hash iteratively (100,000+ rounds) to slow down brute-force attacks
5. **Consider adding pepper** - Secret key stored separately from database for defense in depth
6. **Implement rate limiting** - Prevent online brute-force attempts through account lockouts and delays

**Why Argon2id**: Memory-hard design makes GPU/ASIC attacks expensive, configurable cost factors allow adjustment as hardware improves, and it's resistant to side-channel timing attacks.

---

## References

### Problem 1: Binary Words and Operations

**FIPS PUB 180-4: Secure Hash Standard**
* National Institute of Standards and Technology (NIST). (2015). *Secure Hash Standard (SHS)*. FIPS PUB 180-4.
* URL: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
* Relevance: Primary specification document for SHA-256
* Why needed: Defines all bitwise functions (Ch, Maj, Parity) and operations (ROTR, SHR) with precise mathematical notation and requirements
* Specific sections:
  * Section 3.2: Operations on Words (defines ROTR, SHR, addition modulo 2³²)
  * Section 4.1.2: SHA-224 and SHA-256 Functions (Equations 4.1-4.8 for Ch, Maj, Σ₀, Σ₁, σ₀, σ₁)

**NumPy Documentation**
* NumPy Developers. (2024). *NumPy Documentation*.
* URL: https://numpy.org/doc/stable/
* Relevance: Essential for understanding 32-bit unsigned integer operations (`numpy.uint32`)
* Why needed: Python's native integers have unlimited precision; NumPy provides fixed 32-bit arithmetic matching FIPS 180-4 requirements with automatic modulo 2³² wrapping
* Specific references:
  * uint32 scalars: https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.uint32
  * Data types: https://numpy.org/doc/stable/reference/arrays.dtypes.html

**Python Language Documentation**
* Python Software Foundation. (2024). *Python Language Reference*.
* Binary bitwise operations: https://docs.python.org/3/reference/expressions.html#binary-bitwise-operations
* Relevance: Official specification of XOR (^), AND (&), OR (|), NOT (~), left shift (<<), and right shift (>>) operators
* Why needed: Understanding Python's operator precedence and behavior for implementing SHA-256 functions

**Menezes, A. J., van Oorschot, P. C., & Vanstone, S. A. (1996)**
* *Handbook of Applied Cryptography*. CRC Press.
* URL: http://cacr.uwaterloo.ca/hac/
* Relevance: Comprehensive reference for cryptographic primitives and design principles
* Why needed: Chapter 9 covers hash functions and the role of non-linear functions in cryptographic security
* Context: Explains why combinations of AND, OR, and XOR operations create cryptographically strong mixing

**Wikipedia: Cryptographic Concepts**
* Confusion and diffusion: https://en.wikipedia.org/wiki/Confusion_and_diffusion
  * Relevance: Core principles behind why SHA-256 uses specific bitwise operations
  * Context: Explains how Parity provides diffusion (bit changes propagate) and Ch provides confusion (complex input-output relationships)
* Avalanche effect: https://en.wikipedia.org/wiki/Avalanche_effect
  * Relevance: Security property where small input changes cause large output changes
  * Context: Why rotation and shift operations (ROTR, SHR) are combined in Σ and σ functions
* Multiplexer (Digital Logic): https://en.wikipedia.org/wiki/Multiplexer
  * Relevance: Hardware analogy for Ch(x,y,z) function behavior
  * Context: Demonstrates how Ch acts as a bit-level selector controlled by x
* SHA-2 Design and Security: https://en.wikipedia.org/wiki/SHA-2#Design_and_security
  * Relevance: Comprehensive overview of SHA-256 architecture and security analysis
  * Context: Explains design rationale for rotation amounts and function choices

---

### Problem 2: Fractional Parts of Cube Roots

**FIPS PUB 180-4: Secure Hash Standard**
* National Institute of Standards and Technology (NIST). (2015). *Secure Hash Standard (SHS)*. FIPS PUB 180-4.
* URL: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
* Relevance: Official specification for deriving SHA-256 constants
* Specific sections:
  * Section 4.2.2: SHA-224 and SHA-256 Constants (defines the 64 K constants)
  * Section 5.3.3: Initial Hash Values (defines H⁽⁰⁾ from square roots)
* Why needed: Specifies that constants are derived from fractional parts of cube roots of first 64 primes

**Wikipedia: Trial Division**
* URL: https://en.wikipedia.org/wiki/Trial_division
* Relevance: Algorithm used to generate prime numbers
* Why needed: Trial division is the method implemented in `primes(n)` function
* Context: Explains why checking divisibility only up to √n is sufficient for primality testing

**Wikipedia: Nothing-up-my-sleeve Number**
* URL: https://en.wikipedia.org/wiki/Nothing-up-my-sleeve_number
* Relevance: Security concept explaining why constants are derived from mathematical operations
* Why needed: Demonstrates that SHA-256 constants weren't arbitrarily chosen to create backdoors
* Context: Cube roots of primes provide transparent, reproducible constant generation

**Python Math Module Documentation**
* Python Software Foundation. (2024). *math — Mathematical functions*.
* URL: https://docs.python.org/3/library/math.html
* Relevance: Documentation for `math.floor()` and floating-point operations
* Why needed: Used to extract fractional parts: `frac = root - math.floor(root)`
* Specific functions:
  * `math.floor()`: https://docs.python.org/3/library/math.html#math.floor
  * Power operations: Understanding `p ** (1/3)` for cube roots

**Schneier, B. (1996)**
* *Applied Cryptography: Protocols, Algorithms, and Source Code in C* (2nd ed.). John Wiley & Sons.
* Relevance: Discussion of constant generation in cryptographic algorithms
* Why needed: Chapter 18 covers hash functions and explains the importance of using mathematically derived constants
* Context: Provides historical perspective on "magic constants" vs. transparent generation methods

**Prime Number Theorem**
* Wikipedia: https://en.wikipedia.org/wiki/Prime_number_theorem
* Relevance: Mathematical foundation for prime number distribution
* Why needed: Understanding why trial division is efficient for finding the first 64 primes
* Context: Explains prime density and computational complexity of prime generation

---

### Problem 3: Padding

**FIPS PUB 180-4: Secure Hash Standard**
* National Institute of Standards and Technology (NIST). (2015). *Secure Hash Standard (SHS)*. FIPS PUB 180-4.
* URL: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
* Relevance: Official specification for SHA-256 padding scheme
* Specific sections:
  * Section 5.1.1: SHA-256 Padding (defines padding requirements)
  * Section 5.2.1: SHA-256 Parsing (defines block structure)
* Why needed: Specifies the three-step padding process (append 1 bit, append zeros, append length)

**Merkle-Damgård Construction**
* Wikipedia: https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction
* Relevance: Theoretical foundation for SHA-256's iterative structure
* Why needed: Explains why padding must include message length to prevent length extension attacks
* Context: Padding ensures collision resistance properties of compression function extend to full hash

**Merkle, R. C. (1989)**
* "One Way Hash Functions and DES." *Advances in Cryptology — CRYPTO' 89 Proceedings*, 428-446.
* DOI: 10.1007/0-387-34805-0_40
* Relevance: Original paper introducing the Merkle-Damgård construction
* Why needed: Explains security requirements for padding schemes in iterated hash functions

**Damgård, I. (1989)**
* "A Design Principle for Hash Functions." *Advances in Cryptology — CRYPTO' 89 Proceedings*, 416-427.
* DOI: 10.1007/0-387-34805-0_39
* Relevance: Independent discovery of the Merkle-Damgård construction
* Why needed: Provides formal security proofs for padding and iteration requirements

**Python Bytes Documentation**
* Python Software Foundation. (2024). *Built-in Types - bytes*.
* URL: https://docs.python.org/3/library/stdtypes.html#bytes
* Relevance: Documentation for byte string operations
* Why needed: Understanding `bytes.to_bytes()`, byte concatenation, and byte slicing
* Specific methods:
  * `int.to_bytes()`: https://docs.python.org/3/library/stdtypes.html#int.to_bytes
  * Bytes operations: concatenation with `+`, indexing, slicing

**Python Generator Documentation**
* Python Software Foundation. (2024). *Yield expressions*.
* URL: https://docs.python.org/3/reference/expressions.html#yield-expressions
* Relevance: Documentation for generator functions using `yield`
* Why needed: `block_parse()` uses generators for memory-efficient block processing
* Context: Generators produce values on-demand rather than storing all blocks in memory

**Kelsey, J., & Schneier, B. (2005)**
* "Second Preimages on n-Bit Hash Functions for Much Less than 2ⁿ Work." *Advances in Cryptology – EUROCRYPT 2005*, 474-490.
* DOI: 10.1007/11426639_28
* Relevance: Security analysis of padding schemes in Merkle-Damgård hashes
* Why needed: Demonstrates importance of proper padding to prevent cryptanalytic attacks
* Context: Shows why appending message length is critical for security

---

### Problem 4: Hashes

**FIPS PUB 180-4: Secure Hash Standard**
* National Institute of Standards and Technology (NIST). (2015). *Secure Hash Standard (SHS)*. FIPS PUB 180-4.
* URL: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf
* Relevance: Complete specification of SHA-256 compression function
* Specific sections:
  * Section 5.3.3: SHA-256 Initial Hash Value (H⁽⁰⁾ values from square roots)
  * Section 6.2.2: SHA-256 Hash Computation (pseudocode for compression function)
  * Section 3.2: Operations on Words (modulo 2³² addition specification)
* Why needed: Primary reference for all four stages of the compression function

**Message Schedule Generation**
* Wikipedia SHA-2: https://en.wikipedia.org/wiki/SHA-2#Pseudocode
* Relevance: Visual explanation of message schedule expansion
* Why needed: Clarifies how 16 input words expand to 64 words using σ₀ and σ₁ functions
* Context: W[t] = σ₁(W[t-2]) + W[t-7] + σ₀(W[t-15]) + W[t-16] for t ∈ [16,63]

**NumPy Overflow Behavior**
* NumPy Documentation - Overflow: https://numpy.org/doc/stable/reference/generated/numpy.seterr.html
* Relevance: Understanding and controlling overflow warnings
* Why needed: Explains why `RuntimeWarning: overflow encountered` is expected behavior
* Context: Modulo 2³² arithmetic requires overflow; NumPy's `uint32` handles this correctly

**Preneel, B. (2010)**
* "The First 30 Years of Cryptographic Hash Functions and the NIST SHA-3 Competition." *Topics in Cryptology – CT-RSA 2010*, 1-14.
* DOI: 10.1007/978-3-642-11925-5_1
* Relevance: Historical perspective on hash function design evolution
* Why needed: Explains design choices in SHA-256 compression function
* Context: Discusses why 64 rounds and specific rotation constants provide security

**Katz, J., & Lindell, Y. (2014)**
* *Introduction to Modern Cryptography* (2nd ed.). CRC Press.
* Relevance: Theoretical foundations of hash functions
* Why needed: Chapter 5 covers hash functions, collision resistance, and security proofs
* Context: Explains why feed-forward addition (adding working variables back to hash state) prevents certain attacks

**Python int.from_bytes() Documentation**
* Python Software Foundation. (2024). *int.from_bytes()*.
* URL: https://docs.python.org/3/library/stdtypes.html#int.from_bytes
* Relevance: Converting byte sequences to integers
* Why needed: Used to parse 4-byte chunks from message blocks into 32-bit words
* Context: `int.from_bytes(four_bytes, byteorder='big')` for big-endian conversion

**SHA-256 Test Vectors**
* NIST: https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/secure-hashing
* Relevance: Official test vectors for validating SHA-256 implementation
* Why needed: Verification that implementation produces correct output for known inputs
* Context: "abc" → ba7816bf... is a standard test case from FIPS 180-4 examples

**Wang, X., Yin, Y. L., & Yu, H. (2005)**
* "Finding Collisions in the Full SHA-1." *Advances in Cryptology – CRYPTO 2005*, 17-36.
* DOI: 10.1007/11535218_2
* Relevance: Demonstrates attacks on SHA-1 and why SHA-256's design improvements matter
* Why needed: Context for understanding why SHA-256 uses more rounds and different functions than SHA-1
* Context: SHA-256's 64 rounds (vs SHA-1's 80) with stronger mixing functions resist similar attacks

---

### Problem 5: Passwords

**Dictionary Attack Fundamentals**
* Wikipedia - Dictionary Attack: https://en.wikipedia.org/wiki/Dictionary_attack
* Relevance: Core concept behind password cracking methodology
* Why needed: Explains why testing common passwords against hashes is effective
* Context: Dictionary attacks exploit predictable human password choices

**OWASP Password Storage Cheat Sheet**
* OWASP Foundation. (2024). *Password Storage Cheat Sheet*.
* URL: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
* Relevance: Industry best practices for password storage
* Why needed: Primary reference for secure password hashing recommendations (Argon2id, bcrypt, scrypt)
* Context: Explains why SHA-256 alone is insufficient for passwords

**Argon2 Algorithm**
* Biryukov, A., Dinu, D., & Khovratovich, D. (2016). "Argon2: New Generation of Memory-Hard Functions for Password Hashing and Other Applications." *2016 IEEE European Symposium on Security and Privacy (EuroS&P)*, 292-302.
* DOI: 10.1109/EuroSP.2016.31
* URL: https://www.password-hashing.net/argon2-specs.pdf
* Relevance: Winner of Password Hashing Competition, OWASP recommendation
* Why needed: Technical specification for memory-hard password hashing
* Context: Explains time cost, memory cost, and parallelism parameters

**bcrypt Algorithm**
* Provos, N., & Mazières, D. (1999). "A Future-Adaptable Password Scheme." *Proceedings of the 1999 USENIX Annual Technical Conference*, 81-91.
* URL: https://www.usenix.org/legacy/events/usenix99/provos/provos.pdf
* Relevance: Widely-used adaptive password hashing algorithm
* Why needed: Understanding key stretching and cost factors
* Context: Demonstrates intentionally slow hashing through multiple iterations

**scrypt Algorithm**
* Percival, C. (2009). "Stronger Key Derivation via Sequential Memory-Hard Functions." *BSDCan 2009*.
* URL: http://www.tarsnap.com/scrypt/scrypt.pdf
* Relevance: Memory-hard key derivation function
* Why needed: Alternative to bcrypt with different resource trade-offs
* Context: Designed to be expensive on specialized hardware (GPUs, ASICs)

**Password Database Research**
* Bonneau, J. (2012). "The Science of Guessing: Analyzing an Anonymized Corpus of 70 Million Passwords." *2012 IEEE Symposium on Security and Privacy*, 538-552.
* DOI: 10.1109/SP.2012.49
* Relevance: Statistical analysis of real-world password choices
* Why needed: Justifies dictionary contents (common words, patterns, variations)
* Context: Shows that most passwords follow predictable patterns

**Florêncio, D., & Herley, C. (2007)**
* "A Large-Scale Study of Web Password Habits." *Proceedings of the 16th International Conference on World Wide Web*, 657-666.
* DOI: 10.1145/1242572.1242661
* Relevance: User behavior research on password creation
* Why needed: Explains why users choose weak passwords and reuse them
* Context: Informs dictionary design with real-world password patterns

**Rainbow Tables**
* Wikipedia - Rainbow Table: https://en.wikipedia.org/wiki/Rainbow_table
* Relevance: Precomputed hash lookup technique
* Why needed: Explains why salting is necessary to defeat precomputation attacks
* Context: Unsalted hashes can be reversed instantly using rainbow tables

**Salt (Cryptography)**
* Wikipedia - Salt: https://en.wikipedia.org/wiki/Salt_(cryptography)
* Relevance: Random data added to passwords before hashing
* Why needed: Explains why unique salts prevent rainbow table attacks
* Context: Even identical passwords have different hashes with different salts

**Key Stretching**
* Wikipedia - Key Stretching: https://en.wikipedia.org/wiki/Key_stretching
* Relevance: Technique to make password hashing intentionally slow
* Why needed: Explains why iterating hash functions 100,000+ times improves security
* Context: Legitimate users barely notice delay; attackers become computationally infeasible

**PBKDF2 Standard**
* IETF RFC 8018: https://www.rfc-editor.org/rfc/rfc8018
* Relevance: Password-Based Key Derivation Function 2
* Why needed: Standard for applying key stretching with salt
* Context: Alternative to bcrypt/Argon2, widely implemented but less secure

**Burnett, M., & Kleiman, D. (2006)**
* *Perfect Passwords: Selection, Protection, Authentication*. Syngress Publishing.
* Relevance: Practical guide to password security
* Why needed: User-facing advice on creating strong passwords
* Context: Explains password complexity vs. length trade-offs

**Ferguson, N., Schneier, B., & Kohno, T. (2010)**
* *Cryptography Engineering: Design Principles and Practical Applications*. Wiley Publishing.
* Relevance: Practical cryptography implementation guidance
* Why needed: Chapter 5 discusses password hashing and key derivation
* Context: Explains why cryptographic hashes shouldn't be used directly for passwords

**Have I Been Pwned**
* URL: https://haveibeenpwned.com/
* Relevance: Database of breached passwords
* Why needed: Real-world source for common passwords in dictionaries
* Context: Demonstrates scale of password reuse problem

**Leetspeak / L33t Speak**
* Wikipedia: https://en.wikipedia.org/wiki/Leet
* Relevance: Common character substitution pattern in passwords
* Why needed: Explains why dictionary includes variations like p@ssw0rd
* Context: Users think substitutions (a→@, e→3, o→0) make passwords secure, but they're easily cracked

---

### General References

**Python Official Documentation**
* Python Software Foundation. (2024). *Python 3 Documentation*.
* URL: https://docs.python.org/3/
* Relevance: Complete reference for Python language features
* Why needed: Understanding language constructs, built-in types, and standard library
* Specific areas:
  * List comprehensions
  * Generator expressions
  * String formatting
  * Exception handling

**Jupyter Notebook Documentation**
* Project Jupyter. (2024). *Jupyter Documentation*.
* URL: https://jupyter.org/documentation
* Relevance: Interactive development environment used for project
* Why needed: Understanding notebook cells, markdown formatting, and code execution

**Git and Version Control**
* Git Documentation: https://git-scm.com/doc
* Relevance: Version control for project development
* Why needed: Tracking changes, collaboration, and code management

---

**Academic Integrity Statement**

All implementations in this project are original work created for educational purposes. While the algorithms follow the FIPS PUB 180-4 specification, the code structure, variable naming, comments, and explanatory text are the author's own. External references are cited where appropriate to acknowledge the sources of algorithms, concepts, and theoretical foundations.

---

**Author**: Nathan Buyrchiyev  
**Course**: Computational Theory  
**Date**: December 2025  
**Institution**: Atlantic Technological University

---

*Note: This implementation is for educational purposes only. For production systems, use well-tested cryptographic libraries such as Python's `hashlib` or dedicated password hashing libraries like `argon2-cffi` or `bcrypt`.*