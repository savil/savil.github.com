---
title: Stanford BPASE 2018 Day Two
layout: post
---

Table of Contents
----------------

1. [Talk 1: Cryptocurrency privacy: research-based guidelines for protocol designers](#talk1)
2. [Talk 2: Simplicity: new language for blockchains](#talk2)
3. [Talk 3: Schnorr signatures for bitcoin: challenges and opportunities](#talk3)
4. [Talk 4: Spectre, Phantom: BlockDAG Protocols](#talk4)
5. [Talk 5: A new design for a Blockchain Virtual Machine](#talk5)
6. [Talk 6: Beyond Hellman's time-memory trade-offs with applications to proofs of space](#talk6)
7. [Talk 7: Verifiable Delay Functions: Applications and Candidate Constructions](#talk7)
8. [Talk 8: Settling Payments Fast and Private: Efficient Decentralized Routing for Path-based transactions](#talk8)
9. [Talk 9: Network and Revive: secure payment hubs supporting Off-Blockchain Rebalancing](#talk9)

Talk 1: Cryptocurrency privacy: research-based guidelines for protocol designers {#talk1}
----------------
By Malte Moser, Arvind Narayanan

Anonymity and privacy enhancing techniques

Overview
* adoption of strong privacy techniques is low
* privacy in cryptocurrencies is hard, and current techniques have weaknesses
* path forward: guidelines for protocol designers

Anonymity: 
- definition: "is the state of not being identifiable within a set of subjects, the anonymity set" - pfistzman and kohntopp 2001
- depends on the context

observations from looking at old techniques 
![image-title-here](/assets/2018-01-25-stanford-bpase-twooheighteen-day-two/talk1_privacy_currencies.png)

* the speaker draws a graph of amount-of-cryptography over time
* number of techniques has increased over time
* most techniques in bitcoin go towards obfuscation, while stronger cryptography is used more in altcoins

reasons for low adoption:
* perf: add overhead to normal tx behavior
* complexity: 
  * coordination and preventing loss of funds in an adversarial environment.
     * for example, consider the CoinJoin technique. It needs to find other people to hide behind.
     * JoinMarket is a marketplace for CoinJoin, where there was a barrier to have so many people, so they offered a list of peers everyone could use, but this gives away the game. Trade off!
  * hurts adoption and speed of development. e.g. CoinSwap needs a setup like refund-transaction to ensure people get money back
* competition
  * PETS address different threat models (PETS = privacy enhancing techniques)
  * negative side-effect: splits anonymity set. Consumers get divided, and hurts the overall protection they offer each other by potentially hiding others.
* traceability: tradeoff to help law-enforcement versus privacy
  * law enforcement can be in an elevated position (e.g. requesting more data from exchanges about actors)
* disincentive for regulated players to adopt strong techniques
  * need to comply with rules like money laundering

CryptoCurrency inherits the worst of:
* data anonimyzation:
  * blockchain data is public and weaknesses might be exploited retroactively
* communication anonymity
  * behavior of some users influence anonymity of users
  * "anonymity loves company"

Monero:
* based on a new design called [CryptoNote](https://cryptonote.org)
* every input links to many outputs (called "mixins"), not just one
* initially, didn't enforce mandatory number of mixins. So some users may say "right now, not worried about privacy" so may skip this. But this can break anonymity of other users:
  * if a block has two inputs, but one of those inputs only points here, then we can invalidate the other input. 
  * Because clearly the input which only points here must be the true input, otherwise it is never used!
* analysis:
  * 66% of inputs had no mixins
  * 62% of inputs are deducible
  * fix: 
    * enforce 2+ mixins since march 2016, 4+ since sept 2017. 
    * Chose new transaction format that protects. Old txs still remain vulnerable.

CoinJoin:
* multiple users provide inputs
* tradeoff: a small tx has a limited amount of indistinguishibility, while large tx has unlinkability.
* JoinMarket: has participants (takers) who want to do a CoinJoin and makers who will help the takers by being inputs
* found: default number of parties was 2, and later increased to 4. Most chose default.
  * certain text combinatorial attacks become feasible.
  * values of inputs and outputs allow attacker to identify the roles (maker versus taker). Usually taker is losing value and has higher-value inputs/outputs. 
  * possible for 67% of all txs, and may allow for tracing through cascades

Payment with BTC ("when cookies meets the blockchain" by Goldfeder et al):
* e-commerce sites have this flow: product -> cart -> checkout -> payment processor -> receipt
* every step can leak info to third-parties (e.g. google ads tracking)
* timing and values allow to later determine corresponding transactions made.
* can use this correlation to uncover CoinJoin inputs via "intersection attacks"

5 principles for designers:
  1. Be cautious about privacy claims:
     * claims purely based on protocol design may overestimate privacy. 
     * Depends on usage patterns, and external environment. 
     * validity can only be evaluated empirically.
 
  2. invite and incentivize researchers to perform independent measurement studies
     * example1: zcash foundation, and monero funds protocol research

  3. respect power of defaults:
     * "anonymity loves company"
     * user's anonymity depends on behavior and choices of others insystem

  4. allow parameters to change
     * revisit from time-to-time and allow to change based on empirical evaluations

  5. articulate trade-offs
     * these are inevitable, vis-a-vis perf and usability
     * detectability versus unlinkability

q. LN privacy implications, and improvements
- tradeoffs can be there. depends on if network is centralized or decentralized.

q. 2 years back people were using miner-fees to do CoinJoin obfuscation. Thoughts? they would launder money through the fees.
- some txs have really high fees, so this could be that scenario. Don't have concrete insights into that.

q. analysis not including Exchanges. is there a formal way to include the influence of the exchanges?
- exchanges are responsible for a large number of txs, and if they can add privacy techniques that is great, but they may have regulatory limitations

q. implications of designs like MimbleWimble?
- not sure

q. lot of parameters may change from empirical analysis. Is there a criteria for how to select these parameters?
- good question. depends on context.

Talk 2: Simplicity: new language for blockchains {#talk2}
---------
By Russell O'Connor, Blockstream. very readable [Paper](https://blockstream.com/simplicity.pdf). Philip Wadler [is a fan](http://homepages.inf.ed.ac.uk/wadler/simplicity-and-michelson.html), with some valuable feedback. Last, you can also find an awesome implementation of Simplicity in haskell [here](https://medium.com/@danrobinson/understanding-simplicity-implementing-a-smart-contract-language-in-30-lines-of-haskell-827521bfeb4d).

A language for the consensus layer.

* why not use javascript?
  * ethereum's solidity is syntactically similar. compiled to EVM. Lots can go wrong!
    * difficult to assign costs to primitive operations 
      * DOS attacks. TXs that consume lots of resources (IO, CPU).
    * difficult to bound cost of programs 
      * for Turing Complete language, this is impossible. 
      * can have gas, but can enter contracts that have insufficient gas to complete
    * complex and informal semantics makes reasoning difficult
      * could argue it is the participants' responsiblity to ensure correctness. 
      * sad :-( would be nice to ensure this By Design
* Bitcoin's Script
  * not expressive
    * multiplication disabled since 2010. Satoshi disabled many ops (speculate: because some attack was seen early on).

Tony Hoare: 
> we have a choice: make one's program so simple, there are obviously no errors. Or, make it so complex there are no obvious errors. 

We want the former!

Simplicity:
  * low-level language for user-defined programs in an adversarial environment
  * features:
    * typed-combinator language
    * Finitarily complete, but not turing-complete
      * only finite number of inputs and outputs
    * simple, formal denotational semantics
    * formal operational semantics
    * easy static analysis of computational costs

Type System:
  * 1 - unit type
  * A + B - sum types (or union types) or an AlgebraicDataType in Haskell
  * A * B - product types (like a record type)

Expressions: are functions
Core typing rules: have nine combinators
  * identity
  * unit
  * composing
  * injl
  * injr
  ...

Denotational Semantics for this fits on a t-shirt
based on Sequent-Calculus and not Lambda-Calculus
  * problem is with "function types"
  * conjecture: static analysis in presence of function-types that the upper-bounds are impractically large
  * its possible that the "front end language" uses function-types and they can be "undone".

2 = 1 + 1 (a bit)
2^2 = 2 x 2 (a two-bit word)
2^4 = 2^2 x 2^2 (a four-bit word)
...

Simplicity in a Blockchain:
* recursively hash the DAG of the program, and compute a merkle root that commits to the program.
  * i.e. when receive coins, can just check the hash
  * when redeeming the coins do you need to show the actual program 
  * this is basically how PushToScriptHash works in bitcoin

* has a builtin "witness" combinator that directly produces inputs like digital signatures

* during redemption, provide full Simplicity DAG including witness values
  * system checks the merkle root matches the commitment
  * then evaluates the Simplicity program until it succeeds

* other features in roadmap:
  * signature aggregation
  * covenants 
  * delegation
    * in principle, this would enable a Solidity program that produces a trace of output and can use Simplicity to verify it

* denotational semantics are great for reasoning about programs but cannot tell us about the operational costs. This is important to protect against DOS attacks.

* operational semantics:

  * to model it, use an abstract machine called a Bit Machine
    * has two stacks: read frame stack and write frame stacka

  * cost is determined by: size of the program's DAG, number of steps the BitMachine takes, amount of memory needed by the BitMachine
    * this cost can be bound by static analysis
  * Jets: optimization that recognizes common subexpressions and run the corresponding C code
    * disadvantage: added bunch of complexity to the simple language of nine-combinators!
    * advantages:
      * provide a formal spec of their behavior. Because must match the behavior of the Simplicity code.
      * themselves cannot include new effects   
      * are transparent when reasoning about behavior
 
Formal verification:
  * very desirable for custom programs on a public blockchain dealing with lots of value i.e. $$$. These programs may only be run once.
  * Plan: SimplicityLanguage --[VST]--> C implementation --[CompCert]--> x86 assembly
  * Bigger Plan: FormalSecurityProperties --[FCF:FoundationalCryptographyFramework]--> SmartContractProtocol --> FrontEndLanguage --[Compiler]--> (above plan)
    * many of these use Coq (proof assistant)
    * this gives an end-to-end verification framework


Talk 3: Schnorr signatures for bitcoin: challenges and opportunities {#talk3}
------
By Pieter Wuille et al, Blockstream

some reading for [context on Schnorr Signatures in Bitcoin](https://bitcoinmagazine.com/articles/the-power-of-schnorr-the-signature-algorithm-to-increase-bitcoin-s-scale-and-privacy-1460642496/).

* smaller onchain size
* faster validation 
* better privacy

threshold signatures require multiple sigs
k-of-n threshold: publish n keys and require k sigs for a valid sig

* the main advantage of schnorrs is that one can have native multi-sigs 
  * without requiring all keys that need to be stored as part of the transaction
* batch validation
* non malleable: third-party cannot manipulate signature without invalidating it

bitcoin currently uses ECDSA scheme. Seems like one could add new opcodes to bitcoin that accept schnorr instead of ECDSA.

other advantages:
* **taproot**: proposal that realizes that all transactions with multi-parties can be classified as: either everyone-agrees or some more complicated state. Taproot encodes a public-key and the hash-of-a-script that goes on to the blockchain, both inside another public-key that goes on the blockchain. Looking at this latter key, one cannot tell if it is a simple public-key or also encodes a script in it.
* **scriptless scripts**
  * see [talk at Real World Crypto by Poelstra](https://www.youtube.com/watch?v=ovCBT1gyk9c)
  * "cross chain atomic swaps": all coins get locked, when first party retrieves their coins, this reveals a hash-preimage that enables the other party to retrieve their coins. Schnorr sigs make this more efficient.

cross-input aggregation:
* why stop at 1 signature per input?
..... bunch of explanation about some attacks and schemes to overcome them

* Rogue Key Attack:
  * let Alice have key A, Bob have key B.
  * Bob claims he has key B' = B - A
    * Observe: a naive multisig for (B', A) would use sum of keys: is now just B!
  
  * this is usually prevented by having keys sign themselves.
    * while claiming a public-key, one must prove they have its private-key.
    * this works fine for the multi-sig within a single input approach, since the people who care about it are the participants themselves. They can internally prove to each other they can validly claim the public key. 
    * BUT: for cross-input aggregation, the signer-set is controlled by the attacker -> keys cannot sign themselves
    * need security in the plain public key model i.e. no key setup procedure beyond users claiming they own a pk. They can even lie about it!

* Bellare-Naven in 2006 have solved the plain pk model!
  * Schnorr multi sig uses: `s*G = R + H(X,R,m)*X, where X = X1 + X2 + ...
  * Bellare-Naven "introduces a separate hash for every signer, and into the hash goes the set of all the signers"
  * wide security proof!

* MultiSignature versus Interactive Aggregate Signature:
  * multisig: multiple signers, one message
  * IAS: multiple signers, multiple messages

  * when spending multiple outputs, each input has their own message
  * BN: has a solution i.e. m = m1 || m2 || ...

* Russell's attack
  * Alice has two outputs (O1 and O2). Bob has O3.
  * m1 is a message authorizing a spending of O1, m2 is ...
  * Alice wants to spend O1 in a CoinJoin with Bob, but not O2.
  * Bob can claim he has the same key as Alice (allowed!), and choose as message m2 (instead of m3)
  * Bob can hence duplicate Alice's message, and steal O2.
  * Solution: include the messages themselves in the hash-of-the-commitments-of-all-the-participants

* concretely, on integrating
  * benefits: can turn all keys in a particular input into one, using multisig and threshold sigs, and can reduce further by doing aggregation over multiple inputs, 
  * for cross-input aggregation, we want one signature overall
  * solution:
    1. have CHECKSIG always succeed, but remember pubkey/msg
    2. Transaction valid if all input predicates succeed
       AND an overall Bellare-Neven IAS is valid for all delayed checks.

* ongoing work:
  1. BIP for Bellare-Naven based IAS
  2. BIP for incorporating cross-input capable CHECKSIG
  3. recommended approaches for doing various kinds of threshold signing

Talk 4: Spectre, Phantom: BlockDAG Protocols {#talk4}
-------
By Yonatan Sompolinsky, Hebrew Univ of Jerusalem, DAGLabs

motivation:
* graph algos are fun
* blockchain does not scale
* BlockDAG: generalization of satoshi's paradigm
* 144 blocks per day -> 1 million blocks per day

Very similar to most of Satoshi's paradigm. One deviation.

Blockchain with 10 blocks per second
  * will have many orphans
  * may have many double-spends, and need to specify how to resolve them

DAG mining protocol:
  * rule 1: reference all tips of DAG, as opposed to tip of longest chain
  * rule 2: publish block immediately

key observations:
1. two honest blocks unconnected in the DAG only if created at approx same time
2. rarely more than k honest blocks created at approx same time (because use proof of work), where k is ... a parameter we can play with
3. atttacker can create arbitrary structures, but < 50% of 

* if throughput is very low (like btc), then k = 0 and we have a chain
* if k = 2, we have a DAG with some width and any block can have k "sibling" blocks
* if k = 10, then every block is unaware of 10 other blocks
importantly, we can know k in advance, but having a bound on network propagation delay D and the block rate R. k <= 2.D.R

BlockDAG is a paradigm not a solution, still need to resolve conflicts like double-spends

Phantom protocol 
  * goal: for recognizing a cluster of honest blocks in a DAG, and discarding/penalizing the rest. How:
  * definition: a "k-cluster" is a cluster of blocks that is "well connected" where every block in it is unconnected to at-most k blocks in the set.
  * claim (good news): honest blocks always form a k-cluster. because:
    * two honest blocks are unconnected in a DAG if created at approx same time
    * 
   * claim (bad news): attacker can form a k-cluster as well
   * claim (good news): attacker's k-cluster is bigger, because:
      * attacker controls < 50% of mining power

Phantom protocol:
1. search for largest k-cluster
2. order its blocks via some topological ordering
  * solving consensus is same as agreeing on order of messages.
3. iterate over blocks in the prescribed order, and accept transactions consistent with history
* Note: this protocol is NP hard for step 1.
* why is this secure:
  * hold this thought

satoshi's design was intended to identify which blocks in the tree belong to honest nodes and which do not
  * longest chain = set of honest blocks in tree. Holds true because k is low (block creation rate is slow). So, we eliminate the possibility of forks from propagation delays. Most likely it stems from attacker deliberately forking the chain.
  * key observations (with parens to point out generality):
   1. two honest blocks unconnected in the chain (i mean, DAG) only if created at approx same time
   2. rarely more than 1 (i mean, k) honest block created at approx same time
   3. attacker can create artbit structures, but < 50%
   in satoshi, longest chain = largest 0-cluster.

Satoshi first decided on longest chain and set throughput to lowest level that satisfies protocol. Phantom flips this on its head: set throughput and find k.

theorem:
if attacker < 50%, then under any throughput, the largest k-cluster was created by honest nodes, ...

To get around NP-Hard issue:
  * count no orphans. why not just pick longest chain in DAG. Because attacker can make a slightly longer one (hmmmhow?)
  * why not count all orphans?
    * ethereum tried this with GHOST.
    * but here the attacker could add references from its chain to orphan blocks, and can defeat this scheme as well
  * count "some" orphans: those that are "well connected" to the longest chain.
    * intuition is that orphans will not have incoming edges to the attacker's chain
    * definition: k-uncle of a chain is the number of blocks in the cluster that are not connected to the block.
      * Phantom greedy version, replace step 1 with:
        "search for chain wtih largest number of uncles of degree <= k, the chain + uncles are honest"

* security guarantee:
  1. (almost) all honest block are uncles of degree <= k of heaviest chain
  2. (almost) no attacker blocks are uncles of degree <= k of heaviest chain
  3. any topological order over honest blocks will converge

Phantom:
* pros: 
  * throughput not limited by protocol. Adjust k accordingly.
  * easy to implement efficiently
* cons: 
  * poor waiting times when conflicts are visible. If it takes time for largest k-cluster to finalize, then need to wait.  
  * in payments, double-spenders make conflicts => attackers
    * but in smart contracts, anyone can embed conflicting txs, so will likely have to wait

throughput: 

protocol | (throughput limited by) | (confirmation times w/o conflict) | (w/t conflict) | ordering
BTC | latency | slow | slow | linear
Spectre | capacity | very fast | not guaranteed| pairwise
Phantom | capacity | slow | very slow | linear
Spectre + Phantom | capacity | very fast | very slow | pairwise

Phantom recognizes "blue set" (i.e. honest nodes)
and instead of doing topological ordering, run "spectre" inside it and get fast confirmation time but lose linear ordering.

q. "chain quality":

q. if throughput is high, 

q. can one prune uncles from a year ago?
- protocol

q. if topology, where some nodes are very isolated, can they be mistaken to be attacking, or will they get rolled in?
- both. definition of "honest" means "well connected" and that is up to the implementer of the protocol. will probably be late in the order.


Talk 5: A new design for a Blockchain Virtual Machine {#talk5}
--------
By Cathy Yun, Dan Robinson et al, at Chain

background:
* chain build ledger services for financial inst
* tx on blockchain
* want to support high transaction volumes, issued assets and smart contracts

similar goals to Simplicity. But approach it from a solution-oriented, empirical versus formal, math approach.

* Bitcoin: has declarative txs. where inputs and outputs are applied deterministically to the UTXO set. Localized effects. But scripts are limited in capability: cannot introspect or coordinate with other scripts
* Ethereum: txs and contracts are imperative, and resulting state is unknown until tx is published and run. Great because very expressive but hard to reason about what state the contract will have when executing. Difficult to reason about effects and security.

TxVM:
* deterministic and isolated
* expressive language
* safe environment

Tx is the program: 
* executes contracts, controls v
* produces deterministic tx log (inputs and outputs) that are applied to UTXO set.

language:
* first class values and contracts in type system
* values
* contracts is a program that can also hold things in storage including values

two laws:
1. constrained ops that preserve a "law of conservation" cannot create or destroy value without satisfying some conditions.
2. must be cleared from vm at end of execution

example for first-class values: Example ride-sharing tx
* $15 input from rider
* split it into $10 for driver (contract) and $5 for company (another contract)
* both laws satisfied

example 2: by careless engineer
* $5 from rider
* $5/3 = $1 to company and $2 to rider
* BUT: violates law 1.

example 3, same as example 2:
* $5 from rider
* $5/3 = $1 to company and $2 to rider, and $2 remaning
* But: violates law 2

* to make these valid, the remaining contract of $2 must be used (e.g. as tip to driver)

Internals:
* VM parts
  * stacks
  * txlogs
  * run limit
  * code
* VM rules
  * empty stacks: value input must match output. 
  * tx is finalized. Tx log must be frozen.
  * runlimit not exceeded.
  * no failures.
* Blockchain updates
  * all effects in tx log
  * remove inputs
  * add outputs

* walkthrough: see video.

diving into contracts:
  * ...

* language has specialized op-codes for working with contracts (which basically, deal with ops on values)

q. model a tx with contract. Does that enable formal verification, compared to Solidity?
- not sure

q. is there persistent store of memory like eth?
- no global storage. modelled closer to btc with inputs and outputs

Talk 6: Beyond Hellman's time-memory trade-offs with applications to proofs of space {#talk6}
-------
By Bram Cohen

Outline:
* proofs of space
* previous constructions
* simple construction based on function inversion

goal: prover must convince the verifier that it is using N space.
* verifier has O(1) communications with prover, ideally, for challenge-response protocol
* prover uses O(n) space and O(1) communication
* verifier then uses O(1) cpu to verify the proof.
* attacker could use T time instead of S space.
* security follows if:
  * either, S = N space before exec
  * or, T = N time in exec

* known proofs of space:
  * graph pebelling. is asymptotically optimal
     * is complicated, cannot be made non-interactive when used in blockchains
 
* this work: inverting random functions
  * simple and practically efficient
  * proof holds unconditionally in the random oracle model
  * disadvantage: not asymptotically optimal

* simple ways of doing proofs of space:
  * verifier makes a random table, sends to prover. Then verifier sends an index into table, and prover responds with entry in table.
    * but: lots of communication complexity and space inefficient on verifier.

  * simpler: verifier picks random function F (like a secure hash function i.e. SHA-256), and sends to prover who makes table, and verifier picks index and prover sends inverse.
    * intuitive but Hellman found attack where prover can use sqrt(n) space and time.
       * prover saves sqrt(n) states and remembers the order. When index comes from verifier, prover finds the position in the list and walks it linearly.

  * two observations:
    1. for hellman's attack to work, function needs to be easy to evaluate in forward direction.
    2. for PoS: sufficient that function table is computable in linear time. Don't need this forward direction.

  * explanation of proof: ....omitted. 


Talk 7: Verifiable Delay Functions: Applications and Candidate Constructions {#talk7}
-------
By Ben Fisch, Stanford University

Related Work: see [Simple Proofs of Sequential Work](https://eprint.iacr.org/2018/183) by Bram Cohen. Best paper at EuroCrypt 2018.

VDF Definition:
  * F is a function that has a long walltime to compute. Not just in terms of runtime instructions.
  * it needs to let one quickly verify that y is the output of F(x)
  * slow to compute: sequential, non-parallelization
  * fast to verify
    * short proof of correctness
    * efficient, not simply fast due to parallelization
  * publicly verifiable
  * tunable
    * works for good range of time-factors


time-lock puzzles
  * private verifier prepares one-time-use puzzle
  * not publicly verifiable. Needs a trusted setup for this.
  * new setup for each puzzle
    * wall-clock timer starts once puzzle is released

proofs of sequential work
  * publicly verifiable
  * no trusted setup
  * not a function, output for a given challenge is not unique

Application 1: publicly verifiable random beacon
  * e.g. if someone is running lottery, how do participants know that winner was truly randomly selected?
  * ideal service that regularly publishes random values which no party can predict or manipulate
  * approaches:
    * multi-party protcols 
       * multiparty coin tossing (1/3 honest participants....are needed?)
       * commit-and-reveal (colluding fraction of people can manipulate by withholding reveal?)
    * public displays of randomness
      * not easy to verify
      * difficult to display on scale (e.g. do you trust the video feed?)
    * public source of entropy
       * stock market prices (low order bits)
          * HFT
       * blockchain headers (PoW solutions)
          * miners may withhold unfavorable solutions
            * possible solution: add some delay to derivation of random beacon from block (i.e. wait for N blocks more to be confirmed)
 
Application 2: consensus from any Proof of Resource
  * combines proofs of space with VDF ---> see Bram Cohen's talk and paper.
  * PoR: proves miner owns X% of total resources
  * want property that: miner with X% of resource should in expectation mine X% of blocks in any chain window (referred to as "chain quality")

  * break into X proofs of 1% of resources: Proof_i for i in [1,X]
  * each proof gives one independent random trial: R_i = Hash(Proof_i)
  * miner finds R = min(R_1, R_2, ..., R_n)
  * miner evaluates a VDF with a time-delay proportional to R, on an unpredictable challenge derived from Proof and previous block
  * miner wins block if it samples the lowest delay parameter (i.e. R)
  * would expect: 
    * miner who makes X% of all random samples, will get the minimum (delay parameter) of all random samples X% of the time.

VDF Formalism:
* setup(lambda, t) -> pp
  * outputs public params (pp) including challenge space C and solution space S
  * variants: trusted versus transparent setup

* Eval(pp, c) -> s, proof
  * takes challenge c and public params pp, and outputs solution s and proof
  * runs in time t, even if run in parallel processors each takes time t

* Verify(pp, c, s, proof) -> {Accept, Reject}
  * runs in complexity poly(log(t), lambda). So, there is an exponential gap between

* uniqueness: Eval defines a function on C to S
* sigma-sequentiality: no adversary with poly(t, lambda) processors is able to compute output on Eval on random challenge in less than (t - sigma(t)) steps. Why?

Construction steps:
 
* HashChain: c -> H(c) -> H(H(c)) -> ... = s
  * not efficiently verifiable => verification takes as long as evaluation => not a VDF!
* hashChain with snark: 
  * i.e. a succint proof that hash-chain computed correctly on c to derive s
  * fast to verify
  * but proof-computation takes a long time AND is very parallelizable
  * this fails sigma-sequentiality: non-optimized Eval takes > 2t steps, and highly parallel Eval takes t steps.
* HashChain with "incrementally verifiable" SNARK
  * theoretical VDF
  * basic idea: proof validates that the i'th block of steps is computed correctly
  * trusted setup relied on for verifier-efficiency, but not security
    * anyone can redo evaluator's computation to detect faulty proofs

In this work, we will be practical! square roots modulo a prime: 
  * has a gap between evaluation and verification
  * Challenge c is in Z_p
    * public parameters specify prime modulus p = 3 mod 4
  * Eval(p, c):
    * find x:
      * if c is a quadratic residue (i.e. is a square-root in some Z_p), then solve x^2 = c mod p
      * else, solve x^2 = -c mod p (which we somehow know to exist)
    * x = c^((p + 1)/4) mod p 
      * canonical way of choosing solution s = +/- x
      * needs log(p) sequential operations
  * Verify(c, s):
    * Check s^2 = c mod p
    * requires 1 squaring operation

  * Two ways to increase Eval time
    1. increase p 
      * increases log(p) sequential squarings for Eval 
      * caveats:
        * introduces parallelism for log(p) bit multiplication
        * VDF solution size is log(p) bits
    2. chain square-root computations
      * c_i = sigma(sqrt(c_(i-1) mod p)): where sigma is a simple permutation
      * scales verification and evaluation time equally

    * BUT: optimized Eval and Verify both take O(log^2(p)) steps

* Generalize polynomial inversion
  * f(x) = x^2
  * finding root of x^2-c is slow, but verifying is fast
  * ... see talk for better explanation
  * but this gives us the exponential-gap we seek: Eval is O(d) and Verify is O(log(d))

* Candidate family: <insert specific BIG polynomial here>
  * but this is not a well-studied problem in crypto, so be cautious 

* HashChain with "incrementally verifiable" SNARK
  * replace hash-function with snark-friendly VDF
  
Talk 8: Settling Payments Fast and Private: Efficient Decentralized Routing for Path-based transactions {#talk8}
----------
By Stefanie Roos

* limitations of blockchains
  * scalability (btc is 7tx/second)
  * privacy, due to traceability (same public key appears in all the txs)

* offchain txs
  * create payment channel: A can send X funds to B
  * with unidirectional channel: A can send Y funds, and channel then has X - Y funds
  * with bidirectional channel: A can send X - Y funds, and B can now send Y funds to A. 
  * bidirectional is more common, but in talk will refer to unidirectional (mainly due to brevity in talk)

  * path based transactions (PBTs)
    * idea: if A and B don't have a direct channel between them, we find a path of nodes which are connected and update their channel balances.
    * problem: the total amount you can send is limited by the min(channel fund) for any path
      * can use multiple paths to get around this

   * algorithms:
     1. routing: 
       a. finding paths
       b. assigning which paths get how much funds to route
     2. payment
       a. all nodes along path must update values
     3. accountability
       a. must ensure two nodes are in agreement about what the balances are for them in their channels

     focus on the routing algorithm, since the others are more "solved" problems in this space

     goals:
       * privacy
         * value privacy: guessing how much money was sent.
         * sender/receiver privacya
      * scalability
        * effectiveness  => high probability of having a successful payment by finding these paths
        * efficiency => avoid delay, and ensure not very costly (e.g. don't involve every node in the network to find a path)
      
     * related work
        * Canal/PrivPay: central server (not desirable)
        * Max-Flow algorithms: inefficient for large-scale distributed system that is changing a lot. hard to keep op top of all changes
        * flare: keep track of all links, and keep track of path to. Hard to keep up to date with network dynamics (links changing etc.)
        * SilentWhispers (closest related work):
          * main idea is to use Landmark Based Routing. A landmark is a dedicated node in network 
            * usually has high bandwidth
 
          * might choose two landmarks and for each you build a spanning tree. Still need to keep on top of network dynamics.
             * periodically rebuild them
          * so, how do we route to these networks?
             * look at each landmark, get path from sender to landmark, and then landmark to receiver. number of paths = O(number of landmarks).
          * how much to transfer on each path?
            * get minimum funds on channel in each path = this is the maximum that can be sent on that path
          * scales linearly to depth of spanning tree, which is logarithmic
          * problem:
             * the spanning tree nodes might overload. 
             * Only using payment channels in the trees, which is wasteful of payment channels in rest of network. Can we improve on this?
                 * main idea (SpeedyMurmurs): 
                   * for each node, add (parent number, self number), where number = number in all nodes on same level in tree 
		     * have dynamic and local reconstruction on leave/join
		     * lets one calculate distance between nodes (TODO savil. is this true?)
        * how:
  		    1. divide funds randomly before finding paths
          2. select neighbour such that (a) neighbor is closer to receiver (b) link has atleast c_i funds
             * for value privacy:
               * value c is hidden from nodes not on path
               * or if there is an adversary on one of the paths. Then if A sees value=5, and knows there are two landmarks, can estimate that Expected[value] = value*num_landmarks = 5*2 = 10
                 * cannot know exact value, but can derive estimates
             * for sender/receiver privacy:
                * network embeddings have been used in context of anonymous routing (old result) 

security:
* confidentiality: just encrypt values enroute
* integrity: how do we ensure nodes on the path do not mess with values.
  * if payment is received at receiver, it checks it and aborts 
* availability: this is the big problem. Someone can cause receiver to abort tx, which can eventually stop these txs.

Talk 9: Network and Revive: secure payment hubs supporting Off-Blockchain Rebalancing {#talk9}
----------
By Rami Khalil, Arthur Gervais

current PoW Blockchains won't scale beyond 100 tx/s
with:
1 minute block interval and 1 Mb blocksize => 

options for scaling:
* consensus algorithm: alternatives to proof of work like proof-of-stake, or proof-of-space
* sharding: partitioning
* offchain (this talk): delayed onchain settlement
  * 2 party channels
    * unidirectional
     * bidirectional 
     * linked payments
     * secure payment hubs
   * N party channels: will cover later as well

 off-chain payment lifecycle:
  * establish channel
  * p2p payment tx execution
  * broadcast channel resolution. Enforce latest state, and this tx is mined and channel closed.

* unidirectional:
  * one entity deposits collateral
* bidirectional 
  * both parties deposit collateral
  * important that previous offchain txs are invalidated.
* linked payment channels
  * if peers are not directly connected (see previous talk)
  * considerations:
    *  needs to find path, 
    * and maintain the channel (as the deposits of the channels are consumed or replenished). Doing this topup onchain can be expensive.
    * and tx security
    * offchain congestion. If only have limited amount of time to claim that an old tx is invalid. if congested, then cannot refute old claim (this is dangerous and adds a loophole). This is for "timeout based" payment channels.

* skewed channel balances: topic of this talk.
  * i.e. channel maintenance
  * may find a longer route: time to find route, higher fees, and need to do more onchain topup.

* REVIVE: protocol to make payment channels more scalable.
  * works on two-party payment channels
  * enables rebalancing collaterals off-chain, by finding routes with sufficient channel funds and moves them to the shorter path's funds.

how:
* rebalancing the tx set generation algorithm:
  * input: channel balances, user preferences (like ?)
  * output: payments to be executed
* protocol for atomic enforcement of this tx set:
  * input: payments to be executed
  * output: execution or failure of all payments
* PoC on ethereum using Sprites channels, [is on github](https://github.com/liquidity-network/revive)

Transaction Set Generation Algorithm:
  * linear program (allows us to model an amount from U to V as Delta_U_V)
  * constraints: 
    * sum of all deltas is zero
    * must satisfy user preferences
    * cannot exceed channel balances
  * optimization objective: 
    * maximize the total amount of funds rebalanced (Sum of Delta_U_V)
  * problems:
	  * problem with linear programs is that there may be minuscule losses due to numerical precision from rounding.
	  * running time of linear programs is hard to calculate. But in practice seems okay.

Setup:
* round-robin leader election: leader notifies rebalance-init, with a particular strategy respecting user preferences.
* freeze and generate: leader receives channel balances and participants freeze channels and leader computes tx set
* signature aggregation: participants review tx set, approve and leader collects sigs and leader distributes full sig set
* dispute: rebalance is challenged on chain. Needs full signature set to be broadcast.

guarantees for honest parties: 
  * balance is conserved
  * rebalance objectives are fairly satisfied
  * no guarantees about privacy, because leader is collecting all balances of participants and so knows it all. Future work.

usability-adoption consideration:
* reputation and relationship: probably want to do rebalancing with peers you trust
* number of participants doesn't scale, because disputes can slow it down. More participants mean that more expensive the disputes get.
* parallel channels: having multiple channels of one's own, makes it more efficient for oneself. Could be some optimizations here.
* feasible optimization objectives (because linear program, and not somehting more complex)

must need a cycle in topology to rebalance. a tree will not do.

2-party payment channel hubs:
* need to maintain channels
* frozen deposits
* hub needs to do onchain top-up.

to make this efficient, proposing a liquidity network. efficient offchain payment hubs.
n-party payment hubs

main architecture:
* blockchain + (LiquidityNetwork smart contract + LiquidityNetwork server)
   * is centralized? open and anyone can join the network. funds are owned by users via private key. server can never access the funds.

   * can manage collaterals better
   * offchain registration
     * upon registration, can get offchain transfer from peer and forward offchain to another peer (wat)

@liquiditynet
https://liquidity.network
Development now
Eth testnet in march 2018
[Paper published](https://eprint.iacr.org/2017/823.pdf).

q. what is the trust model for n-party payment model?
- payment hub is not trusted

q. would have expected revive to be a series of rebalances, rather than offchain resolution
- could go from set-of-txs to set-of-paths to rebalance, but need an algorithm to do this mapping.

* secure payment hubs
