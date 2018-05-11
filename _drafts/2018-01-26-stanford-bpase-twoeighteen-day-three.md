---
layout: post
title: Stanford BPASE 2018 Day Three
---

You can find the program for the conference and all slides and videos [here](https://cyber.stanford.edu/bpase18). Notes for day-one are [here](/2018/05/02/stanford-bpase-twoeighteen-day-one.html), and day-two [here](/2018/05/02/stanford-bpase-twoeighteen-day-two.html).

Table of Contents
------------------

* [Talk 1: Bulletproofs](#talk1)
* [Talk 2: Chainweb: a Braided Parallel-Chain PoW architecture for massive throughput](#talk2)
* [Talk 3: Mobius: privacy layer for ethereum](#talk3)
* [Talk 4: Trustchain: micro economy for bandwidth tokens](#talk4)
* [Talk 5: Dfinity: Threshold Relay](#talk5)
* [Talk 6: Power Fault Tolerance: a new framing for consensus protocols](#talk6)
* [Talk 7: Proof of Replication using Depth Robust Graphs](#talk7)
* [Talk 8: TLS-N: Non-repudiation over TLS enabling Ubiquitous Content Signing](#talk8)
* [Talk 9: NuCypher KMS: Decentralized Key management system](#talk9)


Talk 1: Bulletproofs {#talk1}
----
Benedikt Buenz
http://crypto.stanford.edu/bulletproofs

initially designed to help with confidential txs, but can be applied to arbit computations

validity of btc tx:
1. sig is correct
2. inputs are unspent
3. sum of inputs > sum of outputs. Difference -> miner fees.

But is neither confidential (tx amounts not hidden) nor anonymous (can see addresses coming from and to).
Confidentiality is important e.g. for businesses to not reveal suppliers and customers.

Confidential Transactions idea:
* instead of sending amounts, we send cryptographic-commitments (which hides the actual values)
  * e.g. Pedersen commitments: `Commit(x;r) = g^xh^r
  * but then to check that sum of inputs >= sum of outputs, need to ensure that outputs are positive. 

Confidential txs [Maxwell 2016]
  * CT txes are structured like btc txes
  * tx amounts are hidden
  * tx graph is still public
  * public verifiability of tx validity

To verify tx validity:
  * use zero-knowledge proof-of-knowledge!
  * peggy to victor: (prover to verifier):
  * prover to verifier: `c = g^xh^r` is a commitment to a positive number x
  * verifier: prove it
  * verifier: here's a challenge!
  * prover and verifier do an interactive challenge-response protocol
  * peggy: here's a response to prove it is positive and that Peggy knows it, but doesn't tell victor what x specifically is
    * uses Range Proof, x is in [0, 2^52]

NIZK: non-interactive zero-knowledge proof of knowledge
  * more applicable to blockchains. In this, peggy creates the proof on her own, and can send the proof to victor or any public verifier.

Concrete range-proof using bit-commitments
  * idea: can represent the number in 52 bits 
  * peggy commits to all bits of number...explanation of Linear Range proof.
    * for each bit of number, add a commitment
    * and also develop a proof that: 
      * each commit-value is in [0,1] i.e. is a "bit"
      * and, these are "homomorphic commitments" i.e. satisfy the property that C(a) + C(b) = C(a+b), and so verifier that validate that their sum equals the commitment to the original value

Linear range proofs:
  * based on sigma-protocols with Fiat-Shamir heuristic. This is what CT was originally proposed with.
  * recent optimizations by Poelstra et al 2x improvement
  * 4kb for 64-bit range proof  ---> way too large for a transaction
  * linear in bit-length of range (bad)
  * no trusted setup is needed (good)

Preprocessing SNARKs with Trusted Setup (GGPR '13)
  * goal: reduce the size of the proof
  * they pre-compute "queries" for the prover, and pre-compute "answers" for the verifier.
  * prover computes short proof (always 188 bytes regardless of thing being proved), that is fast to verify.
  * problematic: need to do setup. If malicious person does setup, then can be used to make "cheating proofs".
    * cannot prove that there was a malicious setup. Any bad behavior is undetectable at this point. 
    * can be alleviated by having multiple parties create the setup together, which is what ZCash did. But still have downsides.
      * cannot disprove someone spreading FUD about a "bad setup".
    * low flexibility: any changes to the proof-system needs new setup.
  * proof time is k (constant) and so is verification time.  

STARKS: CS Proofs (computationally sound) [Ben-Sasson '17]
  * Based on PCP theorem and Fiat-Shamir heuristic
  * no trusted-setup, but log-sized proofs and verifications
  * very elegant but not practical, because proof-size becomes really large: > 200kb proof-size, and prover overhead is massive (needs lots of RAM for 16-bit circuit)
 
Bootle et al '16: log-sized proofs for Arithmetic Circuits
  * verification time is linear but proof time is log-sized.
  * interactive, multi-round protocol
  * public-coin (verifier's messages are random)
  * for arbitrary arithmetic circuits
  * assumption: discrete-log, only
  * proof size: O(log(n)) elements. This is pretty short. this is what we really care about in a replicated-network like a blockchain.
  * proving and verifying is linear in circuit
  * no proofs on committed values (important for range proofs)

Bulletproofs (also applies to MimbleWimble): 
  * improves on Bootle et al's protocol
  * can do proofs on committed values
  * Fiat-Shamir heuristic to create a NIZK
  * has lots of good properties: 
    * relies on discrete-log problem being hard (well tested) 
    * and can be used in any elliptic-curve problem, unlike SNARKS which rely on other assumptions beyond discrete-log.
  * proof-size: 
    * O(log(n)) elements for the range-proof
    * to prove that multiple outputs are in the range, bulletproof's size increases very marginally compared to a linear range-proof whose size will increase xN, for N outputs.

Multi-Party Protocol for Bulletproofs
  * its not common for multiple outputs in a transaction, so how can we incentivize this? CoinJoin!
  * want it such that: 
    * can have multiple provers making a single proof without revealing their secrets to each other.
    * trivial solution: concatenate the proofs, but that is linear again so not great.
  * designed a multi-party computation protocol for bullet proofs: which lets one combine the proofs into a single one.
    * works if circuits are disjoint i.e. n range proofs for n provers
    * simply aggregate proofs in each round, and compute Fiat-Shamir challege
    * either log(n) rounds and log(n) communication
    * or 3 rounds and O(n) communication
  * This way CoinJoin helps with not just anonymity but also efficiency. There are economic incentives to adopt it!


Bulletproofs for CT or MimbleWimble
  * reduces the UTXO set size by order of magnitude 
    * 4kb to 0.67kb per range-proof
    * UTXO set size or MimbleWimble size: 160 GB -> 17 GB
  * builtin CoinJoin protocol for combining confidential transactions
  * with quantum computers, anonymity could be broken, but soundness cannot. i.e. cannot create money out of thin air.

  * Why not use BP for ZCash? 
    problem: verification time for SNARK is constant, but for BP grows linearly.

...insert image for comparing proof systems...

Applications:
  * can use it for [Solvency Proofs](http://www.jbonneau.com/doc/DBBCB15-CCS-provisions.pdf) 
    * Mt Gox must prove they are solvent
    * can provide a zero-knowledge proof that they are solvent
      * doesn't reveal what each customer has, or even what they have in total
   * with BP, proof-size goes down from 18GB -> 60MB!


  * for smart contracts:
   * no trusted setup, for arbitrary computation
   * proofs are short so tx size is kept small
   * but verification time is linear, so may not be practical
      * workaround: refereed-delegation model (what TrueBit uses). 
        * Protocol:
          * Prover claims Proof is correct i.e. V(proof) = 1
          * Lay out verification circuit, and name i'th gate V_i
          * Challenger claims proof is invalid
          * can do a binary search to find the divergence point in their claims
          * Smart Contract just needs to verify the final gate
        * Cost:
          * O(log(n)) cpu to prepare proof
          * O(log(n)) communication
          * 1 gate computation in the smart contract
     
  * for verifiable shuffles:
   * set of senders (with emails, phones, btc addresses) -> want to shuffle -> and want to send to recipients  

  * found improvements when implemented in btc-security-lib
    * constant time prover
    * fast verifier

   * improving verification: 
     * if one unrolls the interactive protocol, then turns out one can verify with one big "multiexponentiation"
     * well studied problem: it is sublinear N/log(N) i.e. gets better with bigger N
     * BP are faster to verify than a SNARK (for small ciruit times)
     * verifying takes 4ms. ECDSA sig verification is 5 times faster.
     * batch verification also reduces down to doing one bug multiexponentiation
       * 4ms for each verification + 200mus for each additional proof
     * implies is practical for btc in terms of verification time. So, no real barrier to add this to btc.
     * other problems: quantum unsoundness, and some more.


Talk 2: Chainweb: a Braided Parallel-Chain PoW architecture for massive throughput {#talk2}
------
Will Martino, Stuart Popejoy: Kadena

ChainWeb:
* same energy, higher throughput, lower latency

Key Features:
1. peer chains cross reference each other's previous merkle roots
2. trustless, destructive cross-chain coin transfers through SPV at the smart-contract level

Prior Art:
* blockrope/betacoin
  * cross-referencing chains twisted together
* GHOST/Decor+
  * include orphaned header's input for security

two-step trustless cross-chain destructive coin transfers:
1. delete coins on chain A. Specify:
  * delete account on chain 1 and amount 
  * create chain and create acct
2. "Create" requires a merkle proof of deletion
  * smart contract runs spv, gets amount and acct from proof
  * consume delete's tx id

* use the concept of a "base graph" to estimate the size of graph made from N chains.
  * "diameter": distance between any two nodes.
* "merkle cones": pretty neat idea that any nodes in the graph in the history or future cone can be checked with an efficient spv.
    * security hypothesis:
      * once a block's future merkle-cone is formed, then a transaction is "confirmed".

* considerations:
  * configurable confirmation latency
    * block time * base-graph-diameter

  * bandwidth:
    * heavy: whole chainweb block stream
    * light: single chain 
    * light: only send headers for larger number of chains
    * (headers carry the assurance)

  * subset replication and mining:
    * possible to mine subset of network

  * role of large mining pools
   * hashrate load balancers
   * header stream network providers: hard for attacker. By withholding a block, prevents further mining since other previous headers in history-cone are required.
   * layer consensus providers: incentivized to have everyone see network as they see it to avoid misallocating their mining resources
   * cross-chain account transfers
     * take the same amount of time (diameter) as a same-chain transfer
     * well formed Delete is enough for Bob to confirm the tx will occur, Create at Bob's leisure (he knows its well formed and can do it whenever)
   * Load balancing smart contracts
     * cross-chain transfer tools work for contract assets as well


* also see: Pact, smart contract language. non-turing complete (no recursion, etc.) and designed for non-programmers to write contracts (like people write excel formulas).


Talk 3: Mobius: privacy layer for ethereum {#talk3}
--------
Rebekah Mercer

unlinkability: have many inputs which can give plausible deniability into who the sender was

approach 1: new currency
ZCash: 
Dash: litecoin uses master node to do mixing in some way
monero: ring confidential txs

approach 2:
privacy overlay for an existing currency
* centralized
  * naive solution: run a "bitcoin bath" in between senders and receivers. the "bath" 
    * but btc-bath knows who the (sender, receiver) pair is!
    * also, sender knows receiver's address
    * bad for: availability and theft. the btc-bath can run away with the money!

  * TumbleBit:
    * creates two escrows (Alice to Tumbler, Tumbler to Bob)
    * Tumbler doesn't learn (Alice, Bob) pairing for the tx, only knows it talks to Alice and some other point to Bob.
    * availability problem: if Tumbler goes down, money is not stolen but could be in limbo.
    * lots of exchanges (txs cost $$$)

* decentralized
  * Mixcoin, BlindCoin etc.
  * CoinJoin:
     * btc's fav one. 
     * offchain: find other senders, and then share addresses to want to send to
     * anonymity: senders learn who the receivers are. but receivers don't know who the senders are.
     * availability: if a large group of senders, then one sender could DoS this by claiming to intend to participate and at the last stage they could disappear.
   * Xim:
     * aim to reduce offchain communication, do onchain
     * need to do 7 or so onchain txs

* mobius:
   * main problem: entities participating need to do much coordination
   * availability problem: built on ethereum so sidestep this problem by ...
   * anonymity: recipients can learn sender, but sender doesn't know
     * use-case: employer could pay wages but don't know the employee's addresses and so cannot monitor your use of money
     * use-case: onboarding blockchain (passport etc). Exchange can link onchain address to offchain identity.
   * only two messages to be exchanged ever (offchain) but online (need two messages per tx)
   * ethereum
     * normally, no privacy
     * no local storage
     * no non-deterministic execution
     * no external lookup of data
     * everything is expensive
     * security is extremely important
   * method
     * smart-contract: senders deposit tx, and receivers withdraw tx
     * sender tx: cannot specify recipient
     * challenge 1: hide link between sender and receiver
       * derive key:
         * publish long-term master (public) key, instead of publishing address (hash of public key). The hash loses the nice algebraic structure that is exploited.
         * others can derive child public keys for you
         * only you can derive private key
         * only one communication is needed per pair (ever).
         * use Diffie-Hellman key exchange:
            1. sender generates a fresh random key-pair: R = g^r
            2. recipient derives a long-term key pair: A = g^a
            3. the derived public-key = A.g^(H(A^r)), where H is a hash function
                * note: the Hash could be a PRG which can derive more public-keys
            4. the derived private-key = a + H(R^a)
       * can have a public-key registry to avoid communication between alice-and-bob
       * senders use derived keys to deposit

     * challenge 2:  withdrawal must hide which receiver it is
       * ring signature: three algos for setup, sign and verify
       * receiver: need to prove that i deserve money
         * zcash uses the blockchain via zk-snarks to prove this
         * in this limited setting, look at public keys that have money on them, receiver controls private key for one of them
         * but ring sigs cannot be used to authorize money transfers, because sign step has private randomness which can make double-spending easy
         * can use a "linkable ring signature"
      * good summary slide at the end  
  
Talk 4: Trustchain: micro economy for bandwidth tokens {#talk4}
----------
John Pouwelse, Quintin Stokkink: Ledger Scientist

"TRIBLER: a social based peer to peer system"

Outline:
* science: produce trust with software
* overlay for torrent-search and sharing
* distributed accounting system 2007-today
  * trustchain 10k tps
  * microeconomy for bandwith tokens
* internet of trust for the real economy

currently trust is locked-in, where there is a reputation system owned by airbnb or uber
in the future, we'll have a generic trust system

* "trustchain ietf" draft proposal for standardization (19 pages)
  * scales horizontally, each person tracks just their transactions
  * they don't solve the double-spend problem

* end up building a network phone-to-phone that lets one run a reputation system ala uber or airbnb without having any central server

Talk 5: Dfinity: Threshold Relay {#talk5}
------
Timo Hanke, Dfinity
[Whitepaper for consensus](https://dfinity.org/pdf-viewer/library/dfinity-consensus.pdf)

goal for PoStake:
* fast consensus (5s): short block time, instant finality
* scalability (1m people)
* randomness: 
  * for scalability in terms of sharding. 
  * Also, expose randomness from consensus algo into app layer via an op-code. 
  * need it in consensus for leader-election
  * should be unpredictable, unmanipulatable.
* security (provable)
  * no selfish-mining, nothing-at-stake problems

Challenge 1: consensus and scalability
* tradeoff between finality and scalability
  * e.g. PBFT chain: has finality after one block, but scales to only a few nodes
  * pow chain: probabilistic finality over time, arbitrary many nodes

Challenge: randomness
  1.derive randomness from chain?
    * chain is manipulatable
    * assumes everyone agrees on chain (but forkable!)
    * hence:
      * must not depend on chain, since it is "grindable"
      * must not fork
  
  2. "last actor" bias:
    * last actor has info advantage, can see randomness that may be output and could abort. e.g. one actor, and she needs to roll dice in cup, but could peek under it and see 6 and reject it.
      * any fallback mechanism (after abort) introduces bias
    * e.g.: miner discards block, commit-reveal schemes (moved problem to when reveal happens that actor could be bad)

Challenge: Security - PoS attacks:
* causes of selfish mining and nothing at stake: sig are timeless and cheap. 
  * timeline -> blocks can be withheld
  * cheap -> equivocate

Fundamental Observations:
* Obs 1: "threshold groups" have no last actor. a k-of-n threshold group have k actors who are necessary and sufficient.
* Obs 2: "consensus without consensus":
  * BLS threshold sigs
     * private keys are split with groups. To combine, they make sig-shares which then combine (handwave) without revealing full key
     * BLS has all the following properties:
       * distributed key generation: no trusted dealer required.
          * not easy to do. Possible for discrete-log systems, not easy for RSA or factoring-based systems b/c not easy to create a modulus that is product of two primes such that no individual member knows the two primes. In discrete-log systems, the secret is a scalar element of Z mod p, which is easy to share with Shamir-secret sharing.

       * unique: for every message only one signature verifies. Note: this doesn't imply deterministic. True for RSA, but not true for most discrete-log systems like ECDSA or Schnorr because they are randomized. When the system is initialized, one needs to pick some random k value, and so there are infinitely many signatures for each message.
          * not deterministic: you could say in ECDSA one derives the k-value from the message and the secret-key, but for the verifier it won't work. Because to reveal this k-value to the verifier would mean revealing the secret key.
        
       * non-interactive: signature shares are created independently. Schnorr doesn;t satisfy this, but BLS does

  * why "consensus without consensus"? the key generation process means all parties agree on something, and because it is unique it means they always agree on this message. Now, if the signature is the source-of-randomness, then we've agreed on the random number.

* Obs3: Randomly sampled committees. Committee-size has ramifications, number of bad actors in the universe should be < 50%.
* Obs4: enforced publication
  * notarization: majority signature done by a committee
  * if a committee is a random sample of entire network -> notarization is a proof of publication
  * this notarization also serves as a timestamp, assuming each committee only active at a certain blockheight
  * this scheme implies:
    1. blocks cannot be withheld
    2. impossible to build on old blocks.    
  * BUT: a block could get half-notarized, if there is a timeout. An adversary can withhold sig-shares, and hold back the notarization.
    * to fix: have block ref previous notarization (rather than prev block itself). this results in just one withheld block but cannot have a withheld child-chain.


q. how does one recover if a majority of validators is offline?
- assume majority is honest (implying online). Otherwise, system stalls until enough validators come online.

q. what are the tunable parameters?
this could be important when bootstrapping the network, or managing it over time as more participants join or leave.
1. committee size
2. block time:
  rank block-makers, because we want to give them a higher chance of submitting their block to the notaries.

q. fork choice rule?
1. block maker needs to pick fork on which to build next block (can have various algos to weigh diff forks)
2. notaries need to decide which of the two proposed blocks to confirm.

Second part, by Mahnush

Open Participation
* Until now, we've assumed everyone knows everyone's public-keys and everybody knows the group public-keys.
* open participation: how do they regulate new participants joining?
* they define block epochs (a seq of blocks, bookended by two "key frame" blocks)

to join a group is hard:
1. randomness of the chain defines new groups
2. the group members run distributed key generation
3. they join by providing their group public key

Challenge: how to agree on the group public key
solution: use the consensus mechanism of the dfinity chain at the current epoch to change the behavior of the system itself at future epochs

system contracts:
  * are happening on the chain, and can manipulate the behavior of the system in the future. 
    * Most contracts require agreement, hence happening on the chain
    * result will be used on;y on the side of a fork on which it is used    
  * examples:
    * node registration
    * group registration
    * distributed key generation
    * system parameters
    * BNS
    * DFN payment

DKG contract:
  * for BLS we need signatures, so each person needs a signature
  * (t, n) Distributed Key Generation: create a master public key and set of partial signing keys  * previous studies:
    * extensively studied in crypto literature
    * most use point-to-point communication
    * assume or implement a reliable broadcast channel as a bulletin board. But because byzantine consensus is expensive, they cannot scale well.
    * we use the Dfinity chain as the bulletin board

DKG phases:
  * 4 phases:
    1. dealing: 
      * steps
        * share a secret value using a polynomial. Use Shamir-secret sharing
        * create commitment to the polynomial
        * encrypt the shares of each party and
        * send the vector of shares to the contract
      * all crypto-heavy steps are happening on the client, and chain only used for consensus.
    2. complain
      * steps
        * receive your share and commitment vector
        * check validity of your share
        * send your complaint to the contract if:
          * no share received 
          * share was invalid
    3. justification
      * for each complaint:
        * send the share in clear-text to the contract. Everyone can check that it is valid.
      * for each justification, the contract:
        * checks if the share is valid
        * if it is valid, it removes the complaint and accepts the sharing
    4. registration
      * the group public-key is the sum of the verification vector of each successful dealer
      * successful dealer: not have any unjustified complaints remaining

take away:
1. system contracts are useful. Can be used to manage future behavior of the change
2. carefully design so that heavy part of the code is client-side and only consensus-parts are onchain
3. only the data that need agreement over will pass to the contract 
  * contracts are event-driven, instead of phase-driven








RapidFire Talks
----

Talk 6: Power Fault Tolerance: a new framing for consensus protocols {#talk6}
-------
Juan Benet

Outline
1. context:
2. engineering protocol
3. open problems

types of nodes: malicious, rational (need incentive), altruistic/honest (always follow protocol)
btc works even in presence of malicious nodes. 
Introduce notion of proofs that data is stored.
current cloud is based entirely on replication.
currently users can lose data if the startup/company closes down.

what if: user owned data but could control who had access.
moreover: can we use incentives that enable renting space on hard-drives.

modelled filecoin v2 paper after zerocash paper (zerocash was v2 of zerocoin)
filecoin is built on top of IPFS

stack:
* filecoin: decentralized storage network
* ipfs: dist web protocol
* ipld: auth data model and formats
* libp2p: modular p2p netoworking lib

Filecoin properties:
* must be a market
* must allow price undercutting
* must scale to ZB and beyond
* must provide all tiers of service
* good perf
* get rid of need for business frontend

Novel parts:
* decentralized storage network: need formalization for these protocols
* verifiable markets for storage and retrievable
* proofs of replication and proofs of spacetime
* useful proof of work for consensus

three characters:
1. clients: store files, pay filecoin
2. miners: store data for network collect filecoin
3. network: organizes miners and clients

Network should constantly be requiring proofs of storage. 

protocol needs blockchain for:
* currency, balances, 
* market orders: bids, ask, 
* allocations: which miners store what pieces
* proofs: verifiable record of correct beaviors
* contracts: state and execution of smart contracts

miner's power is proportional to useful-storage they provide network

markets for puts and gets can happen offchain, but for now is onchain to assist price-discovery

type DSN interface {
  Put(Data) Key
  Get(Key) Data
  Manage()
}

DSN properties: 
1. complete, 
2. secure if
  * assures data-integrity 
  * assures retrievability, 
  * tolerates management faults ((f, n)-tolerant if f nodes accept n faults), 
  * tolerates storage-faults (f,m)-tolerant if Put stores in m independent storages can tolerate f faults, where m <<< n 
5. publicly verifiable
6. auditable, via trace of operations
7. incentive-compatible

Proofs-of-Replication & Proofs-of-Spacetime
  * classifying the types of Proofs-of-storage
  * concept & a definition of proofs-of-replication
    * time-bounded PoRep
    * construction using CBC (expensive)

  * concept and definition of Proofs-of-spacetime
    * construction using proof-chains
    * compact construction using SNARKs

Proofs-of-Space comprises Proofs-of-Storage, which comprises (PoRep, PoRet, Proof-of-Data-Possession)

Security game for Proofs-of-Storage are of the form:
1. Verifier sends to Prover: Setup(data)
2. Verifier sends Challenge()
3. Prover does proof <- Prove(challenge, data); and sends proof 
4. Verifier: {0,1} <- Verify(challenge, proof)
This is PDP (proof of data possession).
if proof can return data, then it is PoRet.

Problems:
  1. **What if we want P to store 2 copies of data?**:
    * P could pretend to store 2 copies and only store one.
    * simple fix: verifier pre-encrypts different copies
    * but these preclude prover from repairing copies...
  2. **What if we want P1...Pn to store 1 copy of data each? (Sybil attack)**:
    * P1....Pn may be sybil identities or a coalition
    * They could pretend to store 1 copy each, but deduplicate to store only 1.
  3. **What if P could choose or determine the data?Generation Attack)**:
    * concrete description: P could choose data such that P can generate the data on-demand. P then pretends to store the data, but re-generates it just-in-time to Prove challenges.
    * hard to defend, but in DSN settings its very easy to do by aligning incentives.
    * hard to avoid participation rewards to ensure Generation Attack would be irrational

Proof-of-Replication
  * definition: a proving scheme where a prover P can prove to a verifier V that P is indeed storing a particular replica R^D of data D. No two replicas R^D_i and R^D_k can be deduplicated into the same physical storage.
  * security game:
    1. init: verifier sends: setup(data, ek) and prover does: replica <- Encode(data, ek)
    2. verifier sends "challenge".
    3. prover does proof <- Prove(challenge, replica) and sends proof
    4. verifier does {0,1} <- Verify(challenge, proof)
  * so, if there are two replicas on the same prover:
    1. init: verifier sends: setup(data, ek1, ek2), and prover does: r1 <- encode(data, ek1); r2 <- encode(data, ek2)
    2. verifier sends two challenges c1, c2
    3. prover makes p1, and p2 proofs and sends to verifier
    4. verifier does Verify(c1,p1) and Verify(c2,p2)

Proof-of-Spacetime
  * how do we reduce frequency of verifications?
    * Proofs-of-Storage require the verifier V to do frequent polling to ensure the prover P does indeed continue to store data D.
  * intuitions: 
    * we can aggregate proofs and present them all at once
    * we can use chaining to ensure proofs were produced sequentially, covering a span of time
  * formally:
    * a proving scheme that allows a prover P to aggregate Proofs-of-Space(or Proofs-of-Storage) over time into an auditable record Proof that proves to a verifier V that P was indeed wasting space S (or storing data D) for a certain duration of time T.
  * security game:
    1. verifier sends: Setup(data, r0)
    2. prover does:
      * one:
        * save data
        * c0 <- NextChallenge(r0)
        * p0 <- Prove(c0, data)
      * two:
        * c1 <- NextChallenge(p0)
        * p1 <- Prove(c1, data)
      * three:
        * c2 <- NextChallenge(p1)
        * p2 <- Prove(c2, data)
    3. verifier sends: poll & re-challenge r1
      * prover does:
        * c3 <- NextChallenge(p2, r1)
        * p3 <- Prove(c3, data)
    4. prover sends: (p0, c0, ..., (p3, c3)
    5. verifier does: {0,1} <- Verify(c0,..c3,  p0...p3)

  * Note:
    * This process represents an auditable proofs-chain
    * can poll and add randomness to sequential computation. (Why?)
    * compress: can aggregate the proofs-chain with a SNARK/STARK sp and just send sp as proof

See slides for:
  * Storage Market
  * Retrieval Market
  * Filecoin Consensus and Power Fault Tolerance
  * Secret Leader Election and Expected Consensus
  * Storage Power Consensus: Useful Proof-of-Work for Consensus
  * Open Problems

Users:
  1. IPFS is live
  2. dApps
  3. participants with large datasets: machine learning or scientific


Talk 7: Proof of Replication using Depth Robust Graphs {#talk7}
----------
Ben Fisch

* system can internally ensure files are being stored
* PoStorage:
  * proofs of data possession
  * proofs of retrievability (similar to possession but with many queries can extract file)
  * proofs of space: confirm that some resources are dedicated (could be junk!)
  * proof of spacetime: storing space throughout period of time

Data Generation Attack:
* getting paid to store file, so generate junk data on the fly from some small seed on machine
* want proof: dedicating unique resources to storing data, even if can generate junk data
  * this removes incentive to cheat, so no benefit to gen-ing junk data

Crypto Definition
  * PoRep.Setup(1^l, id, D) -> R_id, commit_D
    * setup generates replica R_id
    * commit_D - (optional) commitment to data D. 

  * PoRep.Prove(R, id, c) -> P_i
    * takes a challenge c
    * outputs proof-of-storage of R_id 
  
  * PoRep.Verify(id, c, commit_D, P) -> {0,1}

1. Leads to an Extractor, proving proof-of-retrievability
2. proof-of-space: any snapshot (at time t) of the prover's space must be of an appropriate size
3. replication: can partition this snapshot into parts so that one can independently derive a replica of the original data.
  * unfortunately, this is very hard to achieve in a cryptographic sense, because whatever strategy an honest-prover uses to verify their snapshot, can be mimic'd be an adversary who stores the data+key and then generates the snapshot such that it has two partitions.
  * so: assume a rational adversary. "Good" strategy satisfies strong-replication. For any adversary strategy, there is an efficient way to switch to a "good" strategy. Hence there is no real incentive to act bad.
 
Proofs of Replication:
  * scalability: maintain security and perf for large files
  * low communication: compact proofs (smaller than file size), and infrequent polling
  * fast verification: low complexity verification, much faster than polling period / setup time.

Basic PoRep:
* main tool: verifiable time-delay encoding. Slow encoding, fast decoding.
* type of Verifiable Delay Function
* could do: AES(KDF(id), D) -> R_id
  * iterate many times
  * slow to decode
* see Ben's talk from day-two for a faster way to do this

What about large files?
* encode separately each block
* apply erasure code to file before breaking blocks
  * guarantees that if you haven't deleted some data, can recover it
* verification: query for randomly sampled number of blocks to gain confidence that 90% of blocks aare being stored
* problem:
  * initialization is expensive: each encoding takes a long time
  * highly parallelizable for adversary

CBC-stream encoding
  * derive encoding on one block, and use as seed for next block
  * adds sequentiality for each encoding, so can adjust the delay on each encoding (i.e. #iterations) so total time for N encodings remains the same
  * verifier still randomly selects number of blocks and decodes
  * BUT: adversary can delete half the blocks, and on the fly derive encodings from predecessar

Solution: use Depth Robust Chaining
  * instead of a line (as in cbc), build a DAG graph
  * to derive encoding of specific block, need encodings of ancestors in graph
  * verification: ask for node and all ancestors

  * properties needed:
    1. depth robust: any large subgraph of nodes contains a long path:
      * and many nodes in the subgraph have a large depth
      * studied problem: there are constructions for such graphs, and randomized constructions that are highly efficient
    2. suppose some d fraction of block encodings are deleted
      * high probability that random query selects node of "high" depth
      * answering query correctly requires recomputing the encodings sequentially of this long path









Talk 8: TLS-N: Non-repudiation over TLS enabling Ubiquitous Content Signing {#talk8}
--------
Hubert Ritzdorf

in TLS:
* Alice can get data from server, 
* but cannot forward to Bob, because it is encrypted with symmetric key encryption between alice and server

may want to do so for e.g. for archiving.
better use case is: blockchain oracles

with smart contract on blockchain want to do something that requires on real-world data, and needs an external data source (called oracle) to help out.



requester/client <-- TLS conversation (http session or else) --> generator/server <.....CA verifies identity of server to client to verify server identity
* can also have "verifier"(thirdparty/blockchain) that wants to learn something about the TLS conversation. (needs to trust CA)
* server sends "evidence" to client. Client makes proof from evidence and plaintext. client sends proof to verifier.

possible problems:
1. request-response can suffer from content reordering issues. e.g. what is price of btc? and response can come in other order.
2. privacy: usually send GET request to server. use access_token. added in plain-text with encryption. but now, this will be exposed. 
  * need redactable proofs!
3. denial of service: if have 1 signature over everything. this may mean server gets very large state => DOS attack.
4. redactability of proofs: if server has to decide what parts to redact. Means computational overhead on server => DOS attack.

goals:
1. small server side state and overhead
2. client-side privacy protection (client decides what parts to redact and which to keep after getting response form server)
3. clear context -> total order on requests

TLS-N overview:
1. start with TLS handshake.
2. generate evidence: process small tls records with small state (on server). on client, needs to store all tls records and TLS-N parameters from handshake
3. client "returns evidence" to server
4. server "signs: evidence-generated + tls private key" + client's evidence -> sends evidence to client.
5. client uses privacy settings to generate its proof, after any redactions it has done. 







Talk 9: NuCypher KMS: Decentralized Key management system {#talk9}
-----
MacLane Wilkison et al

[whitepaper](https://cdn2.hubspot.net/hubfs/2807639/NuCypher%20KMS%20Technical%20White%20Paper.pdf?t=1507918722771)

Why?
* imagine if a sender (patient) is wanting to share data with multiple parties (doctors)
* encrypted multi-user chats
  * encrypt every message for every participant. Doesn't work for new participants who want to see history.
  * share group-key with everyone. But then one leak unravels security.
* decentralized netflix
  * today, the CDN or other centralized services like S3 is trusted to have access to the files


Solution: proxy re-encyption + decentralization

PRE (proxy re-encryption)
* steps
  0. store file as C_A <- encrypt(pk_A, m)
  1. send file to proxy as C_A
  2. proxy asks alice for rk_AtoB 
  3. proxy sends to bob: c_B <- reencrypt(rk_AtoB, C_A)
* this way proxy doesn't get secret data, while bob sees data as if encrypted for him

Use threshold split-key re-encryption (Umbral)
  * splitting works similar to Shamir key-splitting
  * Umbral is spanish for threshold
  * PRE properties: unidirectional, single-hope, non-interactive
  * verification of re-encryption correctness via Non-interactive ZK proofs of knowledge
  

used by:
  * Medical
    * MediBloc: https://medibloc.org/en/
    * IRXO
    * Medixain
    * Wholesome
  * IoT
    * Spherity (with BigChainDB)
  * decentralized marketplaces
    * datum
  * decentralized DBs
    * bluzelle
    * fluence
    * wolk
