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
By Russell O'Connor, Blockstream

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
  * when redeeming the coins do you need to shwo the actual program 

* has a "witness" combinator that direclt produces inputs to digital signatures
* during redemption, provide full Simplicity DAG including witness values
  * system checks the merkle root matches the committment
  * then evalutes the Simplicity program until it succeeds

* other features in roadmap:
  * signature aggregation
  * covenants 
  * delegation
    * in principle, this would enable a Solidity program that produces a trace of output and can use Simplicity to verify it

* operational semantics:
  * to model it, use an abstract machine called a Bit Machine
    * has two stacks: read frame stack and write frame stack
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

* smaller onchain size
* faster validation 
* better privacy

threshold signatures require multiple sigs
k-of-n threshold: publish n keys and k sigs

* the main advantage of schnorrs is that one can have native multi-sigs 
  * without requiring all keys taht need to be stored as part of the transaction
* batch validation
* non malleable

bitcoin currently uses ECDSA scheme. Seems like one could add new opcodes to bitcoin that accept schnorr instead of ECDSA

other advantages:
* taproot
* scriptless scripts
  * "cross chain atomic swaps": all coins get locked, when first party retriesves their coins this reveals a hash-preimage that enables the other party to retrieve their coins. Schnorr sigs make this more efficient.

cross-input aggregation:
* why stop at 1 signature per input?
..... bunch of explanation about some attacks and schemes to overcome them

summary:
* btc tx inputs are independent:
 * evry output acts as a prediate 
 * tx is valid if all inputs are correct



 

Reading:
https://medium.com/@SDWouters/why-schnorr-signatures-will-help-solve-2-of-bitcoins-biggest-problems-today-9b7718e7861c

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

if throughput is very low (like btc), then k = 0 and we have a chain
if k = 2, we have a DAG with some width and any block can have k "sibling" blocks
if k = 10, then every block is unaware of 10 other blocks
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
    * but here the attacker could add references from its chain to orphan blocks, and can defaat this scheme as well
  * count "some" orphans: those that are "well connected" to the longest chain.
    * intuition is that orphans will not have incoming edges to the attacker's chain
    * definition: k-uncle of a chain is the number of blocks in the cluster that are not connected to the block.
      * Phantom greedy version, replace step 1 with:
        "search for chain wtih largest number of uncles of degree <= k, the chain + uncles are hoenst"

* security guarantee:
  1. (almost) all honest block are uncles of degree <= k of heaviest chain
  2. (almost) no attacker blocks are uncles of degree <= k of heaviest chain
  3. any topological order over honest blocks will converge

Phantom:
* pro: throughput not limited by protocol. Adjust k accordingly.
* pro: easy to implement efficiently
* cons: poor waiting times when conflicts are visible. If it takes time for largest k-cluster to finalize, then need to wait.  * in payments, double-spenders make conflcits => attackers
  * but in smart contracts, anyone can embed conflciting txs, so will likely have to wait

throughput: 

protocol | throughput limited by | confirmation times w/o conflict | w/t conflict | ordering
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

Bitcoin: has declarative txs. where inputs and outputs are applied deterministically to the UTXO set. Localized effects. But scripts are limited in capability: cannot introspect or coordinate with other scripts
Ethereum: txs and contracts are imperative, and resulting state is unknown until tx is pbulished and run. Great because very expressive but hard to reason about what state the contract will have when executing. Difficult to reason about effects and security.

TxVM:
* deterministic and isolated
* expressive language
* safe environment

Tx is the program: executres contracts, controls v
* produces deterministic tx log (inputs and outputs) that are applied to UTXO set.

langauge:
* first class values and contracts in type system
* values
* contracts is a program that can also hold things in storage including values

two laws:
1. constrained ops that preserve a "law of conservation" cannot create or destroy without satisfying some conditions
2. must be cleared from vm at end of execution

example:
first-class values Example ride-sharing tx:
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
  * remove inpots
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

* prover
* verifier has O(1) communications with prover, ideally, for challenge
* prover uses O(n) space and O(1) communication

* attacker could use 

* known proofs of space:
  * graph pebelling. is asymptotically optimal
     * is complicated, cannot be made non-interactive
 
* inverting random functions
  * simple and practically efficient
  * proof holds unconditionally in the random oracle model
  * disadvantage: not asymptotically optimal

* simple ways of doing proofs of space:
  * verifier makes a random table, sends to prover. Then verifier sends an index into table, and prover responds with entry in table.
    * but: lots of communication complexity and space inefficient on verifier.

  * simpler: verifier picks random function F, and sends to prover who makes table, and verifier picks index and prpover sends inverse.
    * intuitive but Hellman found attack where prover can use sqrt(n) space and time.
       * prover saves sqrt(n) states and remembers the order. When index comes from verifier, prover finds the position in the list and walks it linearly.

  * two observations:
    1. for hellman's attack to work, function needs to be easy to evaluate in forward direction.
    2. for PoS: sufficient that function table is computable in linear time. Don't need this forward direction.

  * explanation of proof: ....omitted. 

Talk 7: Verifiable Delay Functions: Applications and Candidate Constructions {#talk7}
-------
By Ben Fisch

* F is a function that has a long walltime to compute. Not just in terms of runtime instructions.
* it needs to let one quickly verify that y is the output of F(x)

* slow to compute: sequential, non-parallelization
* fast to verify
  * short proof of correctness
  * efficient, not simply fast due to parallelization
* publicly verifiable
* tunable
  * works for good range of time-factors

* time-lock puzzles
  * private verifier prepares one-time-use puzzle
  * not publicly verifiable. Needs a trusted setup for this.
  * new setup for each puzzle
    * wall-clock timer starts once puzzle is released

* proofs of sequential work
  * publicly verifiable
  * no trusted setup
  * not a function, output for a given challenge is not unique

* Application 1: publicly veriable random beacon
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
 
* Application 2: consensus from any Proof of Resource
  * PoR: proves miner owns X% of total resources
  * want property that: miner with X% of resource should in expectation mine X% of blocks in any chain window (referred to as "chain quality")

  * break into X proofs of 1% of resources
  * each proof gives one indepdendent random trial: R_i = Hash(Proof_i)
  * miner finds R = min(proof_1, ...proof_n)
  * <missed this>
  * miner wins block if it samples the lowest delay param
  * would expect: 

Construction steps:
 
* HashChain: c -> H(c) -> H(H(c)) -> ... = s
  * not efficiently verifiable
* hashChain with snark:
  * fast to verify
  * but proof computation takes a long time
  * computation of snark is very parallelizable
* HashChain with "incrementally verifiable" SNARK
  * theoretical VDF
  * trusted setup relied on for verifier-efficiency, but not security
    * anyone can redo evaluator's computation to detect faulty proofs

In this work, we will be practical!
* square roots modulo a prime
  * specify some prime number
  * evaluation on a given challenge (if is quadratic residue), compute `sqrt(c mod p)`
  * ...
  * can get to same complexity of evaluator and verifier

* Generalize polynomial inversion
  * f(x) = x^2
  * finding root of x^2-c is slow, but verifying is fast

* HashChain with "incrementally verifiable" SNARK
  * replace hash-functin with snark-friendly VDF
  
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
        * effectiveness  => high probabilit of having a successful payment by finding these paths
        * efficiency => avoid delay, and ensure not very costly (e.g. don't involve every node in the network to find a path)
      
     * related work
        * Canal/PrivPay: central server (not desirable)
        * Max-Flow algorithms: inefficient for large-scale distributed system that is changing a lot. hard to keep op top of all changes
        * flare: keept rack of all links, and keep track of path to. Hard to keep up to date with network dynamics (links changing etc.)
        * SilentWhispers (closest related work):
          * main idea is to use Landmark Based Routing. A landmark is a dedicated node in network *usually has high bandwidth
 
          * might choose two landmarks and for each you build a spanning tree. Still need to keep on top of network dynamics.
             * periodically rebuild them
          * so, how do we route to these networks?
             * look at each landmark, get path from sender to landmark, and then landmark to receiver. number of paths = O(nuber of landmarks).
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
  			* 1: divide funds randomly before finding paths
			* 2: select neighbour such that (a) neighbor is closer to receiver (b) link has atleast c_i funds
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
* consensus algorithm
* sharding
* offchain (this talk)
  * 2 party channels
    * unidirectional
     * bidirectional 
     * linked payments
     * secure payment hubs
   * N party channels

 off-chain payment lifecycle:
  * establish channel
  * p2p payment tx execution
  * broadcast channel resolution. Enforce latest state, and this tx is mined and channel closed.

* unidirectional:
  * one entity deposits collateral
* bidirectional 
  * both parties deposit collateral
  * important that previous offchain txs are invalidated.
* linked payments
  * if peers are not directly connected (see previous talk)
  * considerations:
    *  needs to find path, 
    * and maintain the channel (as the deposits of the channels are consumed or replenished), 
    * and tx secuirty
    * offchain congestion. If only have limited amount of time to claim that an old tx is invalid. if congested, then cannot refute old claim (this is dangerous and adds a loophole).

* skewed channel balances
  * may find a longer route: time to find route, higher fees, and need to do more onchain topup.

* REVIVE: protocol to make payment channels more scalable.
  * works on two-party payment channels
  * enables rebalancing collaterals off-chain, by finding routes with sufficient channel funds and moves them to the shorter path's funds.

* how:
input: channel balances, user preferences (like ?)
output: payments to be executed
protocol for atomic enforcement of this tx set

PoC on ethereum using Sprites channels. Is on github.

Transaction Set Generation Algorithm:
  * linear program (allows us to model an amount from U to V as Delta_U_V)
  * constraints that sum of all deltas is zero
   * must satisfy user preferences
  * cannot exceed channel balances
  * optimization objective: maximize the total amount of funds rebalanced (Sum of Delta_U_V)
  * problems:
	  * problem with linear programs is that there may be minuscule losses due to numerical precision from rounding.
	  * running time of linear programs is hard to calculate.

Setup:
* round-robin leader election: leader notifies rebalance-init 
* freeze and genreate: leader receives channel aalances and participatnts freeze channels and leader computes tx set
* signature aggregatino: participatns review tx set, approve and leader collects sigs and leader distributes full sig set
* dispaute: revalance is channelged on chain.

guarantees for honest parties: 
  * balance is conserved
  * rebalance objectives are fairly satisfied
not guarantees about privacy, because leader is collecting all balances of participants and so knows it all

usability-adoption consideration:
reputation and relationship: probably want to do rebalancing with peers you trust
number of aptticipants doesn't scale, because disputes can slow it down
parallel channel s*todo savil
feasbible optimization objectives (because linear program, andn ot somehting more complex)

must need a cycle in topology to rebalance. a tree will not do

2-party payment channel hubs:
* need to maintain channels
* frozen deposits
* hub needs to do onchain top-up.

to make this efficient, proposing a liquidity network. efficient offchain payment hubs.
n-party payment hubs

main architecture:
* blockchain + (LiqudityNetwork smart contract + liqueudityNetwork server)
   * is centralized? open and anyone can join the network. funds are owned by users via private key. server can never access the funds.

   * can manage collaterals better
   * offchain registration
     * upon registration, can get offchain transfer from peer and forward offchain to another peer (wat)

@liquiditynet
https://liquidity.network
Development now
Eth testnet in march 2018

q. what is the trust model for n-party payment model?
- payment hub is not trusted

q. would have expected revive to be a series of rebalances, rather than offchain resolution
- could go from set-of-txs to set-of-paths to rebalance, but need an algorithm to do this mapping.


* secure payment hubs
