# Week 1


# Week 2

## Code vs Cipher

- *Code* - Data representation
- *Cipher* - Code where data derivation is difficult

## Caesar Cipher
Caesar cipher shifts all letters to the right by _n_ positions i.e. a -> d, b -> e with $n=3$

^**Easy to break** 


> [!note] Kerckhoff's principle
> Cipher secure even if attacker knows everything apart from key

## Frequency Analysis

Counts the number of times:
- Each symbol occurs
- Pair of symbols occurs
- etc
to draw conclusions. 

Useful against _substitution ciphers_

## **_Symmetric Cryptography_**

> [!tip] XOR
> $\forall bitstrings \space M, \ M \oplus 0 = M$
> $\forall bitstrings \ M, \ M \oplus M = 0$

### One Time Pads

XOR message and key.
Length(key) == Length(Message)

Has _perfect encryption_ => Don't learn anything about plaintext from ciphertext

> [!abstract] Theorem
> Given a plaintext of length $n$, all ciphertexts of same length are
> equally probable, without knowing the key

> [!danger] Problems
> - Key must be as long as message
> - Key must only be used once

## Block Ciphers

Cipher that works on **a fixed size** length/block of plaintext, not just single symbol by using *permutations and substitutions* using a key

## Advanced Encryption Standard (AES)

AES state-of-the-art block cipher
Works on 128 bit blocks
Generates 10 round keys from single 128 bit key
**1 permutation** => *ShiftRows*
**3 substitutions** => *SubBytes, MixColumns, AddRoundKey*

No formal proof for security but key aspects are:
- Shuffling ensures small change in input => big change in output
- Requires at least one non-linear operation to make it difficult to crack/reverse the encryption

### Data Encryption Standard (DES)
- Old standard that can be broken by hashcat easily in about 15 days on average GPU
- Triple DES uses three keys and expected to be good until 2030, used in RFID chips

### Padding
- Block ciphers only work on **fixed size blocks** so padding ensures message can fit a block (if the last block is too small)
- Receiver needs to tell between padding and message

Different potential paddings:
- Adding random bytes? **no**
- Add zeroes? **no**

### PKCS 5/7
If there are n bytes of space left, write the value n, n times
e.g. 
1 byte left => 01
2 bytes left => 0202
3 bytes left => 030303

If message perfectly fits block size, add extra block just block size.
i.e. 16 byte block => 16 16 times

PKCS7 => 16 byte blocks
PKCS5 => 8 byte blocks


## Block Cipher Modes

Block ciphers can only operate on single blocks. Need a method to encrypt a message of multiple blocks in length.

There are multiple different modes of operation which dictate how and when to apply the block cipher for each plaintext block.

### Electronic codebook (ECB)
- each block encrypted individually and assembled in the same order
- Repeated plaintext blocks are reflected in the cipher text

### Cipher Block Chaining (CBC)
- Each **plaintext block is XOR'd with prev ciphertext** block before block cipher encryption
- First **plaintext block is XOR'd with IV** which should be **random**
- IV helps overcome replay attack/overcome when same message is encrypted multiple times. Since IV should be random, it will ensure that **subsequent encryptions are different**
- Decryption is just the reverse process
- *First block is susceptible to change in the decryption since IV is usually sent in plaintext so can change IV to make only the first block decrypt to something else*
![[Pasted image 20230515121519.png]]

![[Pasted image 20230515121555.png]]

CBC is an example of **probabilistic encprytion** since it uses random elements to make every encryption different by using random IV.
This allows encryption of same message with same key without attacker realising

### Counter Mode (CTR)

- Nonce/IV is xor'd with each counter value (which specifies block index but may or may not be starting from 0) and encrypted by block cipher
- The result of that encryption is xor'd with the plaintext block to create ciphertext
- In this way, we create a keystream and technically convert it into a stream cipher. The keystream can be thought of as a onetime pad for each block. The last block does not require padding as we can just xor the required symbols without the need of padding

![[Pasted image 20230515123234.png]]

### Known Plaintext attacks

Decryptions can be run on any block

CTR is vulnerable to this type of attack. 
Can change **ciphertext** to **decrypt** to **anything** given the **plaintext** without knowing the key by following operation
![[Pasted image 20230515123730.png]]


# Week 3

## Public Key Cryptography

Want 4 things:
- *Confiedentiality*
- **Message Integrity**
- **Sender Authentication**
- **Sender Undeniability (non-repudiation)**

**Public Key** to *encrypt* and to *verify*
**Private Key** to *decrypt* and to *sign*

For public cryptography, also need some way to authenticate a message. We use **signatures** for this. 
![[Pasted image 20230515125458.png]]

To facilitate public key cryptography, need some **public key infrastructure**, where a user *registers* with a *registration authority* to get a *certificate* from a *certificate authority (CA)* which genereates certificate (with public key) so anyone can request for one's certificate from from the certificate directory (managed by CA)


## Secure Key Exchange

### MultiRound 

Multiround basically has a "two sided lock box"/buffer where each person unlocks with their own key and to send a message, Alice unlocks the box and inputs message and the other unlocks the box to receive it

![[Pasted image 20230515130112.png]]



> [!abstract] Diffie Hellman Key Exchange
> Choose prime $p$ and number $g<p$ such that $gcd(g, p-1) = 1$
> Alice chooses secret $a \in_{R} \{ 2, \dots , p-2 \}$ and calculates $A = g^a \ mod \ p$
> Bob chooses secret $b = \in_R \{ 2, \dots, p-2 \}$ and calculates $B = g^b \ mod \p$ 
> Both send each other their $A$ and $B$
> Key is then calculated as $B^a \mod \ p = A^b \ mod \ p= g^{ab} \ mod \ p$


> [!question] Diffie Hellman Assumption
> No polytime algo to compute $g^{ab}\ mod \ p$ from $g^a \ mod \ p$ and $g^b \ mod \ p$

This is **NOT** enough on its own. An attacker can intercept and disrupt key agreement since they can make it so the attacker can set up a key with each person whilst each person still thinks they are talking only to each other

To solve this, add authentication to the message to ensure the $A$ or $B$ is from the correct person


## Digital Signatures

They should be **Unforgeable** and **Correct**

## RSA

Three algos `GEN`, `ENC`, `DEC`

### GEN
on security parameter $\lambda$
- Generate two primes $p$, $q$ of same bit size $\lambda$
- Compute $N = pq$ and $\phi(N) = (p-1)(q-1)$
- Choose random integer $e \ (1 < e < \phi(N))$ s.t. $gcd(e, \phi(N)) = 1$
- Compute $d$ s.t. $ed \equiv 1 \ mod \ \phi(N)$
- Pub Key $PK = (e, N)$
- Priv Key $SK = (e, d, N)$

### ENC
- To encrypt a message $m$, the ciphertext $c = m^e \ mod \ N$

### DEC
- To decrypt a ciphertext c, the message $m = c^d \ mod \ N$



# Week 4

## Access Control Matrix (ACMs)

ACM is a matrix of all principals (users) and objects (resource i.e. file)
x: execute, r: read, w: write, -: no permission
Problem: maintenance, corruption => control lost

## Access Control Lists (ACLs)

Store each column of the corresponding ACM with only rows that have permissions


## UNIX ACL

![[Pasted image 20230515135042.png]]
![[Pasted image 20230515135056.png]]


## Directory Access Control

r: contents are read only
x: traversal allowed, e.g. switching and running

## Program Access Control

x permission controls who can run program
s permission indicates that program runs with permission of owner


## User ids
Different user ids
- Real uid (ruid) owner of process
- Effective uid (euid) access checks
- file system uid (fsuid) access checks and ownership of files
- saved uid (suid) when euid changed, old euid => suid. Unprivileged process may change euid only to ruid or suid

Provides flexibility for granting temporary privileges
**Make sure process have as low a level as possible to prevent security issues**

## Hashes

- Hash => string generated from message which is always the same and hard to go back to message
- Unlikely for two messages to generate same hash

### Uses
- Verification of message (careful)
- Tying parts together
- Hashing message and sign the hash (electronic signatures)
- Protect passwords

### Attacks
- **Preimage attack** - find the message for a hash (hard)
- **Collision attack** - find two messages with same hash
- **Prefix collision attack** - collision attack where attacker prefixes the message

### Examples

MD5, SHA family of hashes (SHA1, SHA2 (SHA256, SHA512), SHA3)


## Message Authentication Codes (MACs)

MAC sometimes used for authentication that a message has not be altered

A possible attack can do a **length extension attack** if the MAC just uses a certain type of hash

## CBC MAC

![[Pasted image 20230515142552.png]]


## Inefficient and broken hash/MAC functions
![[Pasted image 20230515142612.png]]

we could try to make a MAC by hashing key and message but leads to extension attack

![[Pasted image 20230515142647.png]]


## Authentication Encryption Modes

Authentication encryption modes add message authentication and can only form valid ciphertext with key

Common way is to add a MAC to ciphertext

## CCM mode encryption

- Calculate AES CBC-MAC
- Then encrypt message followed by MAC using same key in CTR mode


# Week 5

## Ports

TCP uses ports to allow multiple connections

**Socket** defined by => (dest IP, dest port, src IP, src port)
and dest port depends on service
src port normally random

## Protocol Stack

- **Application**
- **Transport**
- **Network/Internet**
- **Link/Hardware**

## Mac and IP address

Every machine has unique MAC address
Every computer on Internet has IP address
NAT address and 192.168 are not unique local addresses


## Dynamic Host Configuration Protocol 

Router assigns each device in a network an IP address from a pool of IP addresses when added to network. Temporary

## Address Resolution Protocol (ARP)
Allows router to know which device uses which IP address
ARP spoofing allows impersonation in same network


## Nonce (Number used once)

A nonce is often used in challenge/response setting
To ensure that the other user is active and wants to talk to you but not enough on its own to prevent from attacks


## Key Establishment Protocol

- Principals often set up a session key using a **Key Establishment Protocol**
- Often need a **Trusted Third Party (TTP)** to send public keys (+ adds authentication to public key)

## Needham-Schroeder Public Key Protocol

### Flawed:

$$
A \to B: E_{B}(N_{a}, A)
$$$$
B \to A: E_{A}(N_{a}, N_{b})
$$$$
A \to B: E_{B}(N_{b})
$$
This has a vulnerability of a mitm attack. If the attacker can convince A to communicate with them, they can impersonate A and talk to B and B will be convinced they are talking to A

![[Pasted image 20230515151301.png]]


### Fixed


$$
A \to B: E_{B}(N_{a}, A)
$$$$
B \to A: E_{A}(N_{a}, N_{b}, B)
$$$$
A \to B: E_{B}(N_{b})
$$

## Forward Secrecy

Keeps message secret from attacker who has **long term (private) keys** and a **recording** of protcol run

Protects against governments who can force handing over keys or attackers who steal them

## Station-To-Station Protocol (STS)

![[Pasted image 20230515151930.png]]

STS has **forward secrecy**

## Full Station-To-Station Protocol (Full STS)

![[Pasted image 20230515151951.png]]
Adds certificates signed by trusted TTP so Alice and Bob don't need to know each other's public key

## Certificates

Certificates used to send public keys to those who don't know them. 
**Trusted Third Party TTP** signs a client's identity and public key


## Needham-Schroeder key establishment protocol

![[Pasted image 20230515152235.png]]

And in this way, Alice and Bob can reuse the same key later, but still need to do challenge/response

![[Pasted image 20230515152509.png]]

## Mutual Belief in Key

![[Pasted image 20230515152544.png]]

- **Fresh Key**: Key established is new (from new nonce or TTP)
- **Key Exclusivity**: Key only known to intended principals
- **Far-end operative**: A knows "B" is active. E.g. might make B return signed nonce from A
- **Once authentication**: A knows "B" wishes to communicate with A. E.g. might have B return the identity of A in message

# Week 6

## SSL/TLS Protocol


# Week 7

# Week 8

# Week 9

# Week 10



