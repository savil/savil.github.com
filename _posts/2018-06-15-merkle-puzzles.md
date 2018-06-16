---
layout: post
title: Merkle Puzzles
---

Merkle Puzzles is a simple, clever scheme that Merkle came up with at a seminar course, during his undergrad in Berkeley, sometime in the 1970s.

It enables two parties to establish a shared secret, while communicating over an insecure channel i.e. where an eavesdropper can listen in on all the communication. The security of this scheme relies on making it computationally-hard for the eavesdropper to figure out this shared secret, while remaining much easier for the two parties. This shared secret can be a [symmetric-encryption-key](https://en.wikipedia.org/wiki/Symmetric-key_algorithm). Such a symmetric-encryption-key enables the two parties to communicate completely securely for any future messages. 

Here's how it works. We'll have Alice trying to communicate the shared-secret to Bob, with Eve trying to be the eavesdropper.
1. Alice makes 2^32 ciphertexts as `ciphertext = Encrypt(key, message)` with 
  * `key` = `0^96 | b1 b2 ... b32`. A standard size for an encryption key is 128 bits. So, the first 96 bits are set to 0. The remaining 32 bits are uniquely chosen for each ciphertext.
  * `message` = `"#i: <i'th shared secret>"` for the i'th ciphertext. Each shared-secret is unique. 
  * `Encrypt` is function that has two properties:
    1. there is a corresponding function `Decrypt` such that `message = Decrypt(key, Encrypt(key, message))`
    2. the output of `Encrypt` appears completely random.
2. Alice stores a table with two columns: index number, shared secret. This will be useful in the last step later.
3. Alice sends all 2^32 ciphertexts to Bob, in random order. Recall that each ciphertext looks completely random, and garbled, to Eve.
4. Bob picks one of these ciphertexts at random. And then decrypts it by trying all possible 2^32 keys for it. This gives Bob the number `i` and its corresponding shared-secret.
5. Bob then sends this number `i` to Alice as plaintext i.e. Eve can tell what this number is.
6. Finally, Alice can look up the shared-secret for this number `i` in her table, and now both Alice and Bob have the same shared-secret!

Okay, so why is this secure? 

Notice that Alice needed to make 2^32 puzzles, and Bob took 2^32 tries to decrypt one of them. Eve doesn't know which puzzle Bob chose, so would attempt to decrypt all the puzzles: `2^32 puzzles * 2^32 work to decrypt each puzzle = 2^64 steps`. In computer-science speak, Bob and Alice did `O(n)` work while Eve would need to do `O(n^2)` work.

Please note this is not a very practical scheme, and we have much more efficient schemes today. What really speaks to me is the power of such thought experiments: Merkle Puzzles were one of the earliest public-key crypto schemes.
