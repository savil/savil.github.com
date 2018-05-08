---
layout: post
title: Stanford BPASE 2018 Day Three
---

Bulletproofs
----
Benedikt Buenz
http://crypto.stanford.edu/bulletproofs

initially designed to help with confidential txs, but can be applied to arbit computations

validity of btc tx:
1. sig is correct
2. inputs are unspent
3. sum of inputs > sum of outputs

But is neither confidential (tx amounts not hidden) nor anonymous (can see addresses coming from and to).
Confidentiality is important e.g. for businesses to not reveal suppliers and customers.

idea:
* instead of sending amounts, we send cryptographic-commitments (which hides the actual values)
  * e.g. Pedersen commitments
  * but then how to check that sum of inputs >= sum of outputs.
  * turns out need to check that outputs are positive. (why?)

Confidential txs [Maxwell 2016]
  * use zero-knowledge proof-of-knowledge
  * peggy: have commitment to a positive number
   * verifier: prove it!
   * verifier: here's a challenge!
   * peggy: here's a response to prove it positive and Peggy knows
     * uses Range Proof, x is in [0, 2^52]
   * can extend to NIZK (non interactive proof of knowledge)
   * how to do range proof?
     * can decompose number into 52 bits 
     * peggy commits to all bits of number...explanation of Linear Range proof.
     * bad: range proofs have 4kb for 64bit range proof, but good: don't need trusted setup

  * Preprocessing SNARKs with Trusted Setup (GGPR 13)
     * prover computes short proof (always 188 bytes regardless of thing being proofed), that is fast to verify.
     * problematic: need to do setup. If malicious person does setup, then can be used to make "cheating proofs".
       * cannot prove that there was a malicious setup. Any bad behavior is undetectable at this point.
       * Low flexibility: any changes need new setup.
     * proof time is k and so is verifiation time.  

   * STARKS: CS Proofs (computationally sound)
     * very elegant but not practical, because proof-size becomes really large.

   * Bootle et al '16: verification time is linear but proof time is log-sized.

   * Bulletproofs (also applies to MimbleWimble): 
     * can do proofs on committed values
     * has lots of good properties: relies on discrete-log problem being hard (well tested) and can be used in any elliptic-curve problem.
     * want it such that: can have multiple provers making a single proof without revealing their secrets to each other.
     * designed a multi-party computation protocol for bullet proofs: which lets one combine the proofs into a single one.
       * either log(n) rounds and computation
       * or ... 
     * reduces the UTXO set size by order of magnitude
     * builtin CoinJoin protocol for combining confidential transactions
     * 
     * Why not use BP for ZCash? problem: verification time for SNARK is constant, but for BP grows linearly.

     * Applications:
     * can use it for Solvency Proofs: Mt Gox must prove they are solvent
       * can provide a zero-knowledge proof that they are solvent
          * doesn't reveal what each customer has, or even what they have in total
       * with BP proof size goes down from 60GB to 60KB (or so, handwaved)

     * for smart contracts:
       * no trusted setup, for artbitrary computation
       * proofs are short so tx size is kept small
       * but verification time is linear, so may not be practical
          * workaround: refereed-delegation model (what TrueBit uses)
         
     * for verifiable shuffles:
       * set of senders (with emails, phones, btc addresses) -> want to shuffle -> and want to send to recipients  

     * found improvements when implemented in btc-security-lib
       * improving verification: can verify with one big "multiexponentiation"
         * well studied problem: it is sublinear N/log(N)
         * BP are faster to verify than a SNARK (for small ciruit times)
         * verifying takes 4ms. ECDSA sig verification is 5 times faster.
         * batch verification also reduces down to doing one bug multiexponentiation
           * 4ms for each verification + 200mus for each additional proof
         * implies is practical for btc in terms of verification time. So, no real barrier to add this to btc.
         * other problems: quantum unsoundness, and some more.



Chainweb: a Braided Parallel-Chain PoW architecture for massive throughput
------
Will Martino, Stuart Popejoy: Kadena

* same energy, higher throughput, lower latency

1. peer chains cross referecee each other's merkle root
2. <missed>

Prior Art:
* blockrope/betacoin
  * cross-referencing chains twisted together
* GHOST/Decor+
  * include orphaned header's input for security

two-step trustless cross-chain descructive coin transfers:
1. delete coins on chain 1
  * delete acc ton chain 1 and amount 
  * create chain and create acct
2. create requires a merkle proof of deletion
  * smart contract runs spv, gets amount and acct from proof
  * consume delete's tx id

* use the concept of a "base graph" to estimate the size of graph made from N chains.
  * "diameter": distance between any two nodes.
* "merkle cones": pretty neat idea that any nodes in the graph in the history or future cone can be checked with an efficient spv.
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


Mobius: privacy layer for ethereum
--------
Rebekah Mercer

unlinkability: have many inputs which can giv eplausible deniability into who the sender was

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
     * secuiryt is extremely important
   * method
     * sc, senders deposit tx, and receivers withdraw tx
     * sender tx: cannot specify recipient
     * challenge 1: hide link between sender and receiver
       * derive key:
         * publish long-term master (public) key, instead of publishing address (hash of public key)
         * others can derive child public keys for you
         * only you can derive private key
         * only one communication is needed per pair (ever).
         * use Diffie-Hellman key exchange
       * key registry
       * senders use derived keys to deposit

     * challenge 2:  withdrawal must hide which receiver it is
       * ring signature: three algos for setup, sign and verify
       * receiver: need to prove that i deserve money
         * zcash uses the blockchain via zk-snarks to prove this
         * in this limited setting, look at public keys taht have money on them, receiver controls private key for one of them
         * but ring sigs cannot be used to authorize money transfers, because sign step has private randomness which can make double-spending easily
         * can use a "linkable ring signature"
      * good summary slide at the end  
  
Trustchain: micro economy for bandwidth tokens
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

Dfinity: Threshold Relay
------
Timo Hanke

goal for PoStake:
* fast consensus (5s): short block time, instant finality
* scalability (1m people)
* randomness: 
  * for scalability in terms of sharding. Also, expose randomness from consensus algo into app layer via an op-code. (why?)
  * should be unpredictable, unmanipulatable
* security (provable)
  * no selfish-mining, nothing-at-stake problems

tadeoff between finality and scalability
e.g. PBFT chain (has finality after one block)
pow (finality probabilistically over time, manby nodes)

derive randomness from chain?
  * chain is manipulatable
  * assumes everyone agrees on chain (but forkable!)
  * hence:
  
"last actor" bias:
  * last actor has info advantage, can see randomness that may be output and could abort. e.g. one actor, and she needs to roll dice in cup, but could peek under it and see 6 and reject it.
  * any fallback mechanism (after abort) introduces bias
  * e.g.: miner discards block, commit-reveal schemes (moved problem to when reveal happens that acotr could be bad)

PoS attacks:
* causes of selfish mining and nothing at stake: sig are timeless and

Obs 1: "threshold groups" have no last actor. a k-of-n threshold group have k actors who are necessary and sufficient.
Obs 2: "consensus without consensus":
  * BLS threshold sigs
     * private keys are split with groups. To combine, they make sig-shares which then combine (handwave) without revealing full key
     * properties:
       * distributed key generation: no trusted dealer required.
       * unique: for every message only one signature verifies. Note: this doesn't imply deterministic.
       * non-interactive: signature shares are created independently. Schnorr doesn;t satisfy this, but BLS does
Obs3: committee-size has ramifications, should be < 50%.
Obs4: enforced publication
  * notarization: majority sig by a committee
  * this notarization also serves as a timestamp, assuming each committee only active at a certain blockheight
    * but: a block could get half-notarized, if there is a timeout. An adversary can withhold sig-shares, and hold back the notarization.
    * to fix: have block ref previous notarization (rather than prev block itself). this results in just one withhelf block but cannot have a withheld child-chain.
    
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

how do they regulate new participants joining?
they define block epochs (a seq of blocks, bookended by two "key frame" blocks)
     









RapidFire Talks
----

Power Fault Tolerance: a new framing for consensus protocols
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

what if: user owned data but couild control who had access.
moreover: can we use incentives that enable renting space on harddrives.

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

protocol needs blockchain for:
* currency, balances, 
* market orders: bids, ask, 
* allocations: which miners store what pieces
* proofs: verifiable record of correct beaviors
* contracts: state and execution of smart contracts

miner's power is proportional to useful-storage they provide network

markets for puts and gets can happen offchain, but for now is onchain to assist price-discovery

type DSN interface {
  Put
  Get
  Manage
}

DSN properties: 
1. complete, 
2. secure if assures data-integrity and retrievability, 
3. tolerates management faults ((f, n)-tolerant if f nodes accept n faults), 
4. tolerates storage-faults (f,m)-tolerant if Put stores in m independent storages can tolerate f faults, where m <<< n 
5. publicly verifiable
6. auditable, via trace of operations
7. incentive-compatible




Proof of Replication using Depth Robust Graphs
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

proof of replication:
 * must incorporate the time-axis
 * 


TLS-N: Non-repudiation over TLS enabling Ubiquitous Content Signing
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







NuCypher KMS: Decentralized Key management system
-----
MacLane Wilkison et al



