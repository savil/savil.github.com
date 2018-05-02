---
layout: post
title: Stanford BPASE 2018 Day One
---


You can find the program for the conference and all slides and videos [here](https://cyber.stanford.edu/bpase18).

I've written this as a companion to the talks, mainly as a tool for me to ensure i absorb the material, and to refer back to later. If it helps the broader community, and i hope it does, then awesome! Please direct any feedback to me on [twitter](https://twitter.com/savils).

Table of Contents:
-------

* [Talk 1: Game theory and network attacks: how to destroy bitcoin](#talk1)
* [Talk 2: State of the art attacks on secure hardware wallets](#talk2)
* [Talk 3: Smart contracts for bribing miners](#talk3)
* [Talk 4: Formal Barriers to Proof of Stake Protocols](#talk4)
* [Talk 5: Programming incentives: an intro to cryptonomics, on Casper POS in Ethereum](#talk5)
* [Talk 6: ThunderToken: blockchains with optimistic instant confirmation](#talk6)
* [Talk 7: Smart signatures, experiments in authorization](#talk7)
* [Talk 8: Hardening Lightning](#talk8)
* [Talk 9: Enter the Hydra: towards principled bug bounties and exploit-resistant smart contracts](#talk9)

Intro talk {#intro}
-------
by [Byron Gibson](https://twitter.com/byrongibson)

This conference deals with a few kinds of security:
* data integrity
* fiduciary security
* system security

Some themes that will be covered:
* adversarial thinking
* cryptographic primitives
* programming interfaces/languages
* consensus and secure scaling

Talk 1: Game theory and network attacks: how to destroy bitcoin {#talk1}
-------
[Max Fang](https://twitter.com/MaxFangX). blockchain at berkeley: CS+Econ at UC Berkeley.
[slides](https://cyber.stanford.edu/sites/default/files/20180124_bpase_game_theoretical_attacks_on_bitcoin.pdf).

This talk is a survey of attacks. For an excellent overview of mining attacks with beautiful illustrations, [see this pdf](https://hacken.io/wp-content/uploads/The-Rush-for-Hashpower.pdf).

### selfish mining ###
* paper: ["Majority is not enough"](https://www.cs.cornell.edu/~ie53/publications/btcProcFC.pdf)  

* this is a block-withholding scheme.
* normally, a miner would publish the block as soon as they find a valid block. 
* but they could keep it secret!
* goal: find two blocks before the network finds the next one
* the rest of the network, on the honest chain, thinks it is on the longest chain. But the miner has a longer chain, and this gives the miner some advantages.
* when the miner finds the second block, they broadcast both the blocks => miner's chain is longer => win!
* works because this miner got "free time" mining on network (for second block) while rest were finding first block
* this way they gain larger effective hashrate => greater expected profits!

Variation: what happens if the network found a valid block, before this miner finds the second block?
* this leads to a race-to-propagate:
  * if miner has 25% network hashrate, and assuming 50% of network chooses attacker's chain, then malicious strategy is more profitable.
  * if miner has 33% network hashrate, then malicious strategy is more profitable, regardless of which chain the network chooses.
  * why: intuitively we can consider that 
    * (a) the attacker has had more time to mine the second block on its chain 
    * more importantly, (b) the honest network's hashrate gets divided into two. So, if all the attacker's (~25%) hashrate is focussed on the attacker's chain then it can make progress faster.

### stubborn mining model ###
* model for thinking about [selfish mining, by: Nayak, Kumar, Miller, Shi](https://eprint.iacr.org/2015/796)
* basic idea: can tell rest of network faster than others 

formal model:
* let's have Alice as the attacker (i.e. attacker's hashrate), and Bob as the innocent miner (i.e. honest network's hashrate).
* lead: defined as how much of Alice's chain is ahead of Bob's. Let's call this: N.
* N': there is a fork
  * revealed portion of the fork is equal length
  * bob's mining power is split by these two forks.

* Can make a markov model to represent the "lead".
* The honest network's hash power can get split between honest network and selfish network, assuming "high connectivity" by the attacker.

Stubborn strategies:
  * **lead stubborn mining**: if private-chain has a lead, and honest chain mines a block, then private chain only reveals one block and not all of its lead. This way honest network may keep mining (partly, because there are now two competing chains and hash power will get divided), and the private chain gets a chance to invalidate several honest blocks.
  * **equal fork stubborn**: if private-chain is one ahead, and honest chain mines a block, and such a race is happening, then when private-chain mines a new block it withholds it until the honest chain also mines a new block. This way, can again hope to invalidate multiple honest blocks.
  * **trail stubborn**: even if the honest network is one ahead, the attacker continues trying to mine on its private chain. This strategy is abandoned when the private chain is two blocks behind the honest chain.

### selfish mining defenses ###
* uniform tie breaking:
  * for two blocks that you hear about. Randomly choose between them. 
     * Prevents attacker from benefiting from additional network connectivity.
     * some disagreement in literature on profit threshold. Original authors claim this raises threshold from 0% to 25%, others say: 23.2%.

* unforgeable timestamps (Ethan Heilman 2014)
  * each miner uses block with fresher timestamp
  * raises profit-threshold to 32%

* note: these strategies above only apply to block-propagation races, not to longer chain attacks.

* publish or perish defence: zhang and preneel 2017
  * backwards compat, no hard fork needed
  * can still disincentivize selfish mining even with longer chain
  * replace bitcoin fork resolving policy (FRP) with weighted FRP
  * basic idea: weight depends on being published in a certain time-window. 
  * secret block that is withheld doesn't get any weight.

  * limitations:
    * assumes synchronous model for bitcoin blockchain. But bitcoin is async => this defence is useless.

### Game Theory based censorship ###
want to prevent someone's transactions. 
Let's imagine we have GJ = gary johnson = a prototypical libertarian. 
and we have a Censor = e.g. China.

* first try: blacklist
  * only works if have 100% power on network.
  * even 90% only causes delays
 
* second try: punitive forking
  * have > 51% hashrate
  * mandate that chinese pools do not include GJ's tx
  * if non-chinese miners include tx, then chinese miners fork and make longer chain
  * other miners eventually give up!

* blacklisting via "feather forking":
  * attacker will attempt to fork, but if it doesn't work out, then give up and go back to main chain
  * give up after k transactions
  * q: network hashrate that attacker has, where 0 < q < 1
   * if k = 1, give up after one confirmation (one block more) => chance of successfully orphaning Johnson block = q^2
  * q = 0.2 (20% network hashrate), q^2 = 4%
  * now, other miners are aware of q^2 chance of their block being rejected
  * `E[include GJ tx] = (1-q^2) * block_reward + GJ tx free`
  * `E[don't include] = block_reward`
  * hence, GJ must pay `q^2 * block_reward` in fees for tx to be included
  * concretely, `4% * 12.5 BTC = 0.5 BTC` => GJ must pay $5,500 in fees
  * but don't see instances of this attack on the network, yet!

### 51% attacks and collusions ###
  * Martin Koppelmann, SF bitcoin devs
  * Lemma 1: mining reward = mining cost (b/c mining needs electricity)
  * Lemma 2: cost of acquiring 51% is approximately 0, and is less than mining cost (this is marginal cost, basically need more capital to mine more)
  * Lemma 3: value of 51% attack > mining reward
    * can effectively get all minnig rewards, because can continue on your own chain
    * q = 51% => 49% of blocks orphaned
    * q = 80% => 20% of blocks orphaned, most btc users not affected

  * Conclusion: value of 51% attack > cost of acquiring 51% hashrate. Very profitable!

* btc mining is zero sum, must exclude others
* members-only mining
  * let hashrate join a collusion until 80% of network is in, then exclude rest
  * no incentive not to join 
    * attack succeeds => get more rewards
    * fails => conduct attack in a way it won't start until threshold is reached
  * game theory -> this should happen!

* naive example:
 * 3 pools collude => more than 51%
 * if more 10th block of another pool?
 * how to detect?

practical caveats:
* need lots of risk or capital
  * need tons of mining hw 
* hard to write and deploy exploit software
* insufficiently motivated miners
takeaway: social structure is important. world is not rational.
* need to look at behavioral economics, sociology, psychology to ask why this hasn't happened yet! 

Talk 2: State of the art attacks on secure hardware wallets {#talk2}
------------
by Charles Guillemet, CSO at Ledger. [video](https://www.youtube.com/watch?v=VgIPR37glPM&feature=youtu.be)

I'd recommend just watching this talk. The speaker was slow and clear, and gave a great overview of the security threats. Recently, there was an [interesting attack](https://saleemrashid.com/2018/03/20/breaking-ledger-security-model/) published on the Ledger wallet, whose details are a handy companion to this talk.

Outline:
* intro
* software attack vectors
* physical attacks
  * principles
  * ink cartridge attacks, ounterffeiting
* perturbation attacks
  * principles
  * set top box examples
* side channel attacks
...


Ledgers have:
* secure element (used in smartcards)
  * used for sim, passport, banking cards, TPM (Trusted Platform Module)
  * store private key, and signature computed within here
* pin code : 4-8 digits
* malware proof: designed to protect on untrusted computer

Reminder: Secure software is super hard

* Smartcard:
  * put all security part in a dedicated IC
  * attacker may have physical access
  * big actors: Samsung, etc.
  * framework, attacker has physical access:
    * sw attack
    * physical 
    * pertubation
    * side channel

* sw attacks
  * hard to secure: run code on untrusted platforms, or with root privileges
  * hardware devices have interfaces
  * IOs secured against classical attacks (like buffer overflow)
     * sometimes possible to load and run code
   * isolation must be ensured (Javacard firewall, custom OS with isolation using hw)
  * attack surface of sw attacks is small by design

* physical attacks
  * modify circuit to gain access to secrets or counterfeit certificate
  * Focalized Ion Beam (FIB), circuit modification
     * locally edit circuit
     * signals (like key registers) are probed
     * cost of a FIB machine is $1k/day, or $1.5M total
     * lots of time and expertise needed
     * lots of countermeasures exist
     * there is some way today to access flash memory

  * reverse engineering IC, using Scanning Electron Microscope (SEM) and chemicals
     * chemical deprocessing layers of circuit
     * pictures taken, semiautomatic process to reverse circuit
     * very costly, high know-how, time consuming
     * this attack is used in video game industry and ink cartrdiges
       * ink cartridge: mutual authentication between printer and ink cartridge to avoid counterfeiting. 
       * Several Billion $ market.
       * solutions use secure element: obfuscation, encrypted nvm, active shield.
       * but attackers have cracked this
  
  * x-ray circuit modification (paper: nanofocussed x-ray bean to reprogram secure circuits)

* pertubation attacks
  * use laser, emi (electromagnetic injetion), glitches via lasers
  * lasers shot on circuit => produce induced current that switches values of bits used in circuit.
    * laser is efficient because local
    * code rerouting: false pin or wrong signature acceptance
    * attacks on cryptography related computations
      * differential fault analysis - RSA (Bellcore's attack)
      * DFA on AES (Piret and Quisquater's paper)
    * in both attacks above, one fault gives away the private key
    * very efficient attacks but there are countermeasures one can employ:
      * prevention: shielding IC (but not always possible), packaging, randomization make synchronization difficult
      * detection: light detectors / CRC / double (or more) computations / control flow
      * reaction:  at fault detection => erase keys or reset or destroy device

    * famous example: pay tv (20 years ago).
      * simple, lots of money
      * set top box + smart card
      * gets scrambled message, smart card returns "control word", outputs descrambled content
      * once attacker gets control word, it is shared over internet!
      * today:
        * encrypted(CW) with pairing key that changes every 10 seconds. This E(CW) is returned from smart card.
        * BUT: every 10 seconds, full attack is done (takes ~100ms), session key is found and shared on internet
        * BUT BUT: can also get the pairing key, and now no timing constraints!
           * techniques: 
             * ...see side channel attacks

* side channel attacks
  * measure power consumption/EM during crypto computations
  * record traces
  * post processing traces
  * conduct side channel analysis
  * attacks since 90s, using ML based attacks in 2015-16

  * timing attacks & Simple Power Analysis (SPA)
    * retrieving sensitive info based on duration of computation
    * e.g. password checking with early return
    * SPA: example of "square and multiply".

  * DPA: statistical attack
    * targets one intermediate value (a bit) which depends on key.
    * exploit a statistical difference: if power consumption or EM is different when this bit is 1 versus 0.
  * CPA: generalizes DPA attack assuming power consumption model. 
    * C = a.HW(x) + b, where C = measured power consumption, and HW is hamming weight of sensitive value x.
  
  * template attacks: 
    * instead of assuming a leakage model, model is observed on a sample with a known key
    * two stages:
      * stage 1: profile on known value to build model
      * stage 2: using the model on targetted value
    * european certification requires protection against template attacks up to several million traces for stage 1 and 1 million for stage 2.
    * secure products add countermeasures but others (like yubikey, or tv set top box) don't.
    * attack is cheap and efficient
  
  * ML based side channel attacks
    * classification problem
    * template => supervised learning 
    * CPA, DPA => unsupervised learning
    * Resynchronization effort is minimised
    * countermeasure is to insert a "jitter" (?) in the trace
    * not too many articles published on this, but protection against is required by some certification authorities.
    
Conclusion:
* lessons
  * if stakes are high, expect attackers will have high potential
  * crypto is a big new target (like passport, banks, ink cartridges, set top tv from before)
  * software solutions are hard to secure
  * dedicated hardware is a better approach

Talk 3: smart contracts for bribing miners {#talk3}
--------
by Patrick McCorry: [@paddyucl](https://twitter.com/paddyucl). [Paper](http://homepages.cs.ncl.ac.uk/patrick.mccorry/minerbribery.pdf)

useful background info:

  * miner incentives:
    * long term: want network to succeed
    * short term: may deviate for profit => tragedy of the commons!
      * example:
        * founder of BTC.TOP offered $100 million to kill some fork of btc.

    * whale tx
      * a tx that has a large fee for the miners
      * problem: asymmetrical trust for briber or bribed miner => briber has to trust miner, or miner has to trust briber.

    * script puzzles
      * divert hashrate from mining to solving the puzzle.
      * problem: weaken longest chain overall weight

    * proof of stale block
      * ethereum contract rewards miners for withholding blocks in btc
      * problem: contract cannot verify block's correctness

* can smart-contract impact nakamoto-style consensus?

  * censorshipCon: briber seeks to censor tx
    * attacker to miner: please pursue uncle block mining strategy
      * uncle block: stale blocks later introduced in blockchain
      * ethereum rewards via "uncle block reward policy"
         * You can find Ethereum's design rationale for uncle blocks [here](https://github.com/ethereum/wiki/wiki/Design-Rationale#uncle-incentivization).
    * this way, network funds bribery attack.
    * assumptions:
      * attacker has convinced sufficient % of miners to join
      * attacker will include all uncle blocks and accept bribe tx -> this subsidises her
    * step 1: let's assume uncle block was mined and included in blockchain
    * step 2: miner sends contract that has 3 diff block headers. Namely: uncle block, competing block (same level as uncle block), publisher block (has header of uncle block).
      * contract checks:
        1. verify that Attacker's competing block and the publisher block are in the blockchain.
        2. the publisher block has indeed included the header of the uncle block
        3. the competing block and uncle block extend the same parent block i.e. prove they are at the same level.
      * c_payout = c_bribe + c_block - c_uncle
        * the c_uncle that is issued to the miner is instead consumed by the attacker, thereby subsidizing her.
      * ethereum limits number of uncle blocks to 2, per block  
      * for each uncle block:
        * network pays 2.625 eth
        * Alice pays...

  * historyRevisionCon: 
    * reverse tx to enable double spending.
    * assume:
      * no asymetrical trust assumptions between the briber and the miner. Neither party needs to trust the other.
        * if miners don't trust the briber => then briber must sign a list of tx in advance with incrementing time-locks (to ensure only a single bribe is included per block)
        * if briber doesn't trust the miners => can include a whale tx (with incentive of large fee) after each block in the alternate chain.
      * contract's integrity not affected, by forks.
    * lets have three accounts:
      * A1 = 10 coins. creates and funds contract. 
      * A2 = 100 coins to double spend. Spends coins in the longest chain.
      * A3 = 0 coins. to receive double-spend coins. Receives coins from A2 in the alternate-chain.
     * wait until A2 is confirmed in blockchain.
     * Alice publishes two tx:
       1. double spend tx. send 100 coins from A2 to A3.
       2. tx to create the historyRevisionCon contract.
     * first block: both tx from alice + accept-bribe tx. Makes new fork.
       * in every block in this alternate chain, the accept-bribe tx must be included. It will check that a bribe has not been paid for this block, and send the bribe-money to the miner.
     * analysis. Notice there are two modes of payment:
       1. every block that gets included in the final blockchain.
       2. or if not included, then every uncle block (similar to CensorshipCon)
       * win win for miner
     * Alice doesn't need any hashrate for this attack
     * more than double-spending:
       * inspect state of other contracts - reverse computations and state contracts

  * GoldfingerCon: 
     * reduce utility of competing cryptocurrency (e.g. make btc less useful)
     * assumptions:
       * no asymetrical trust assumptions
       * briber remains online to publish bribed blocks
     * method:
       * have miner mine empty blocks in btc, and then prove in ethereum contract that they did this.
       * miner publishes empty block in btc. This comprises of the coinbase transaction.
       * in ethereum, he publishes a transaction for "accept bribe", that:
         * has the btc-block-header, and the coinbase-tx
         * calls GoldFingerCon contract's AcceptBribe function that needs to verify the btc-block is empty.
           * the block is proved empty by checking that the merkle tree root is the same as the coinbase-tx hash.
             * TODO savil. how does ethereum look up properties of the btc-blockchain?
         * the contract then sends the reward to the ethereum address of the miner.
	   * this ethereum address is calculated by: extract public-key from coinbase-tx output, and compute the ethereum address for this public-key. 

future work:
  * ramp up: assume majority of miners to participate
    * to incentize this, can set time-limits for certain events before voiding the contract.
      * For example, in GoldfingerCon, could specify that x% of blocks must be mined by time t1, then 2x% of blocks by time t2, and so on.
  * impact of selfish mining in combination with bribery attacks for attacking and defending network
  * can bribery ctx by used in proof-of-stake, and are there fundamental diff in style of bribery?
     * same coin can be used for both voting rights and paying bribes.

Talk 4: Formal Barriers to Proof of Stake Protocols {#talk4}
------------
By [Jonah Brown-Cohen](https://people.eecs.berkeley.edu/~jonahbc/) et al

Proof of Stake:
* random miner chosen probabilistically proportional to wealth.
* one coin, one vote, rather than one cpu, one vote
* can similar security to pow be achieved?
* this talk does an incentive-driven analysis: identify incentive problems not in pow

Talk outline:
1. a model for analyzing incentives in pos
2. properties such that every protocol in model satisfies one of these properties
3. for each property, incentive driven attacks against the protocol that satisfies this property

disclaimer: attacks are well understood, but for a broad class of protocols these attacks work better than pow

intuition:
* need source of pseudorandomness to pick random miner
* pow: comes from brute force guessing of nonce
* pos: comes from protocol itself

Jonah proposes the following model 
1. use some method to pick coin
2. use some method to pick existing block
  * TODO savil: won't this always be the last "mined" block?
3. owner of coin gets to add new block
4. repeat

Formal notation:
* B, is a block that has: t(B) timestamp, c(B) coin, miner(B), contents
* A = Pred(B), the previous block in the chain
* for a block B, history of tx in B and Pred(B) define ownership of coins marked as owner(c(B)).

assumptions:
1. chain dependence: validity of B at t depends only on t and Pred(B)
2. monotonicity: if B is valid at time t, then valid at all future times t' > t
  * even if not on longest chain, block is still "valid"
These assumptions hold for pow protocols like btc. These assumptions means we don't consider ["eclipse-attacks"](https://www.usenix.org/node/190891) as threats in this model (although they exist in real life).

POS, has two functions:
1. validating function V
  * efficiently computable by every protocol participant
  * `B` with `A = Pred(B)` is valid at t if and only if 
      1. `V(B) = 1`
      2. `Miner(B) = Owner(c(B))` at A
      3. `t(A) <= t(B) <= t`
2. mining function M: inputs are B, coin c, timestamp t => outputs new block
  * M(A, c, tv) is efficiently computable by owner 
  * if there is a valid block, then mining function should actually mine something

Longest-chain protocol

* Properties:
  * Property 1: D-Locally Predictable

    * For coin c, owner(c) can predict D blocks in advance that she is eligible to use c to mine a block
    * every POS protocol is 1-locally predicatable.
    * but 1-local predictability can be chained on a private fork, letting one predict far in advance how many future blocks one can mine.


  * Property 2: D-Globally predictable 

    * every protocol participant can predict whether owner(c) is eligible to use c to mine, D blocks in advance.


  * Property 3: D-recent

    * miner of C cannot efficiently predict D blocks in advance if she is eligible to mine a block.
    * negation of D-locally predictable
    * blocks [1, D-1] provide the source of pseudo-randomness

  * Two extreme examples:
    
    1. `V(B) = 1 <=> Hash(c(B), t(B)) < Threshold =>` globally predictable for all D
      * this was original POS proposal, where this Hash function replaces the btc-nonce.
    2. protocol where:
      * every block contains signature s(B)
      * `M(A, c, t) = B` such that `s(B) = SIG(Hash(s(A), t)`, where SIG is signed by private key of owner(c)
      * `V(B) = 1 <=> H(s(B)) < T`
      * this is not globally predictable for anyone. It is 1-locally predictable.
      * based on Algorand's protocol
      * here s(A) becomes the source of pseudorandomness

Attack: predictable selfish mining:
* withhold a newly mined block B and secretly try to mine on top of it
* if you mine another B' => then you have a chance to push a longer chain
- with global predictability => no risk! can predict precisely when you can mine k blocks faster than rest of miners
- with 1-local predictability => reduced risk! can predict how fast you can mine k blocks, compared to average rate

Attack: predictable double spending
* you want to predict private fork, but do so to cancel some tx
* predict you can produce secret fork of k blocks
* buy $$, if delivered before k blocks are mined then include conflicting tx on secret-fork, then publish the fork!
* D-predictability for large values of D => gives time to prepare for launching an attack
* could be useful to prepare for stranger attacks by offering to take a bribe

Attacks: undetectable nothing-at-stake
* for d-recent protocols, blocks A and B at two ends of length D fork have "independent pseudorandomness"
* trying to mine on both sides of fork, doubles chances of successfully mining
* if all coins are held in diff accounts, can make attack undetectable

* Existing protocols:
* D-globally predictable:
  * cardano, large D
  * peercoin, for all D
  * tezos, for large D
* 1-locally predictable and 2-recent 
  * algorand (would be vuln to nothing-at-stake attack but not longest chain)

Conclusion:
* Predictability versus recency is a tradeoff: if choose one, then protocol designers must add mitigating measures or protection against attacks from other.
* 1-local predictability is necessary
* global-predictability is not necessary => can eliminate at the cost of a digital sig at every ...stake?

q by ken huang: for casper
* opinion: be recent
* casper is going to be recent and bounty-hunting for nothing-at-stake
* need to measure how much benefit to selfish mining increases by bounty-hunting ("slashing")

q: alternatives to pos
* rely on some other source of randomness in the world

q: ?
* penalties for mining on wrong fork
* stop accepting blocks mined older than .5 seconds. How to protect against network latency or attacks?

q: smaller the D, may affect entropy of seed that is used to randomly select miners?
* depends on how protocol is implemented. 
  * e.g. look back 1000 blocks, and use all of that randomness. That is 1000-predictable in model. 

q: asynchrony plays a role.
the more synchronicity you assume, the more you can eliminate selfish mining

q: check out Dfinity (not a question!)
* but [the paper is here](https://medium.com/dfinity/dfinity-white-paper-our-consensus-algorithm-a11adc0a054c)

Talk 5: programming incentives: an intro to cryptonomics {#talk5}
--------
by [Karl Floersch](https://twitter.com/karl_dot_tech), Ethereum Foundation

On casper protocol

pow:
has incentive to mine on the longest chain => casper needs to solve this

problems:
1. waste of energy
2. vuln to asic and centralization
  * makes chain reversion relatively low cost
    * no inprotocol penalty for reversion
3. no "finality"
  * UASF exploited this vuln
4. no clear validator set
  * useful for: sharding, main chain consensus (casper), cryptoeconomic light clients (e.g. want to sync chain, get block from validator who signs that if block is not in main chain then loses deposit)

hybrid casper:
* FFG (Friendly Finality Gadget): adding finality to eth main chain
  * every 50 blocks vote on whether block is in main chain
  * reduces energy => lower block reward
* any eth holder can become validator by depositing eth in casper smart contract
* staking pools can run casper themselves 

why deposits?
* see validators as evil. 
* give larger incentives to work with. 
* impose large penalty on bad actors.

mechanism:
* chain is chunked in 50 block segments called epochs
* every 50th block => checkpoints
* finality = 2/3 votes from validators
* waits till two checkpoints in a row
* every vote = prep of current epoch, and confirmation of previous epoch

slashing conditions
* solves nothing-at-stake problem
* if there are two chains that are finalized, then 1/3 of validator deposits are lost/"slashed"
  * no double votes => validator cannot vote on two conflicting chains
  * no surround vote => cannot vote on two chains such that there's a gap in between
* see video about this!

designing casper FFG by Vlad and Vitalik
* "minimal slashing conditions" => 4 conditions
* came with formal verification 
* but we should use formal-verification when human intuitions are not enough, not a replacement for intuition!

"parameterizing casper: decentralization/finality tradeoff"

Q: ken huang
problems:
1. ddos against validation pool
2. collusion in validation pool (67% attack)
Answer:
  * slashing means minority loses deposit. There is some paper about this.
  * to mitigate: can fork off censoring chain (???)

  * in validation pool, run decentralized protocol to decide. Doesn't need to be a central server.

q. is there a complete final write up of the protocol? for academic contributions
* go to ethereum website on casper basics

q. planning to swap out consensus mechanism for $100B currency, does it worry you?
* have mandate to move forward. ties in with sharding and plasma vision.
* schedule: have casper test net, have sharded eth node that is being developed, have plasma minimal viable. 3 fronts. no date.

q (DHVC): path from hybrid to full?
* one simple option: turn casper validator votes into blocks themselves. issue: have a 1000 validators, can one have a 1000 blocks to reach finality?
* casper ffg is a simplified expression within vlad's framework. which is a 1 vote is 1 block view.

q: smart contracts, have access to block info, and will there be access to epic/finalization info?
* for plasma chains, this is important. we will probably see access to such apis.

q: what is the simulation environment or research process?
* risks of censorship or finality reversion. based on these risks, what rewards should we offer validators, or failure rates can we tolerate?

Talk 6: ThunderToken: blockchains with optimistic instant confirmation {#talk6}
-----
By [Elaine Shi](https://twitter.com/ElaineRShi), Cornell and Thundertoken

This talk was interesting because it bridges the world of traditional distributed consensus systems, and finds a way to use a blockchain to improve it. A neat idea that could be useful in smaller, permissioned networks.


* what this talk is about:
  * Doing fast, block confirmation times. 
  * on core consensus and crash course on distributed consensus
  * not about: incentives and governance policy

* motivation: 
  * consider losses by airlines in reservation system downtimes
  * need: replication and robustness
  * this inspired dist. systems

* state machine replication: linearly ordered log, and consensus
  * consistency: honest nodes agree on log
  * liveness: txs are incorporated quickly
  * implementation: "chubby lock service for loosely coupled dist systems" mike burrows, google -> apache zookeeper

* traditional consensus: single org, dozen servers, fast lan
* cryptocurrencies: aspire to large-scale deployments
  * in china, scale is 100 banks and 1000 nodes
  * large scale dist protocols: chain.com, hyperledger, etc.

So, why is this not a solved problem? why roll one's own?
* Classical: e.g. pbft, paxos. Are fast BUT complex.
* Blockchains: e.g. "mathematics behind blockchains"[GKL'15][PSS'17]. 
  * Are simple, robust, but slow.
  * are often wasteful
  * assume: can remove pow from blockchains ("sleepy consensus" - PS16)
  * some of the slowness is inevitable - there's some paper about this.


* voting based protocol
  * leader proposes (Seq, txs)
  * everyone votes, but one can have a bad tx in it?
  * honest nodes vote uniquely
  * honest leader = consistency and liveness
  * but dishonest leader = consistency but no liveness
* in classical consensus, this voting has a simple normal path, but when something goes wrong, the recovery path is very complicated
  * e.g. chain.com only implemented the simple normal path, but not practical when so many actors involved (100 banks) to recover with.
* high-level idea with thundertoken is to replace complicated recovery path with a blockchain.
  * simple and robust as a blockchain
  * in optimistic case, need 2-3 "actual network rounds"
  * in attack case, fallback to underlying blockchain

* ethereum scenario: 
  * leader, blockchain and committee (e.g. stakeholders, or banks in a trusted scenario)
  * assumptions: assume miners are majority honest, and committee is majority honest (but need not be online)
  * attempt 1: 
    * vote on blocks, confirm a block upon collecting enough votes by the committee.
    * problem: let's imagine blockchain forks in the day (due to network delay), so people who made fork A are now inconsistent with fork B that later won out. So, even if it worked, must wait for 1 block interval.
  * attempt 2 (actual proposal): 
    * leader proposes (seq, txs) and everyone votes on it
    * for a tx to become "notarized", it needs 3/4 of committee to vote
  * let's imagine 
    * tx1, tx2, tx3 are notarized <-- "confirm maximal lucky sequence"
    * tx4 is not yet
    * tx5, tx6 are notarized
  * crux of the problem: how do we solve the liveness issue if leader is corrupt or offline, or committee is offline?
  * blockchain collects evidence of an attack in "fast path", and upon detecting that we "enter the slow mode" via the blockchain. Once in the slow-mode, can use a smart-contract to elect a new leader and switch back to the fast-path.
  * how to detect failure of the "fast path"? 
    * the faulty nodes must not implicate honest leader. should be robust.
    * miners must tell blockchain all the notarized and unnotarized tx, not just the confirmed tx as in traditional blockchains.
    * if in a block, one observes an unconfirmed tx, then inspect k more blocks: and see that tx3 has still not appeared in a "lucky sequence" => fast-path has failed. If leader had been honest and online, and also the committee, then the unconfirmed-tx would have been confirmed by now => are in failure mode.
  * how does one enter slow mode?
    * nodes have different logs when entering slow mode. need to decide what is the cutoff. its an "agreement problem" in itself.
    * introduce a "grace period" of k blocks (e.g. k = 6 or 10) ... and some more details

Summary:
1. when things are good: single round of voting when things are good
2. when things are bad: use blockchain to "view change" when things go bad 

Insights gained:
1. claim a new theoretical paradigm. 
  * In the old consensus, we have an asynchronous world, where the "bad path" makes solving this problem hard. 
  * with blockchain, we use a synchronous protocol.
2. "block interval must be constant time (O(1)) larger than max network delay"
  * "permissionless consensus has to be synchronous"

* traditionally, we overlook synchrony because it is slow as far as common wisdom goes.
* but this may not be correct. Via thundertoken you can live in async "fast" land normally but can fallback into simple and robust synchronous protocol.

q. what happens when underlying blockchain gets forked in the grace period?

a. grace-period should contain enough blocks (k should be sufficiently large). blockchain protocol is such that if you wait long enough then this is okay.

Talk 7: smart signatures, experiments in authorization {#talk7}
------
By [Christopher Allen](https://twitter.com/ChristopherA), formerly blockstream

* signatures: demonstrate validity of a message 
  * examples = using RSA,  X.501 (Standard)
* Trust policy: defined and limited by third-parties like a Certificate Authority and an app/browser/os
* modern crypto allows many variations like multi-sig, ring sig, blind sig.
* traditional signatures: authenticate who signed message, and certify that the signed part is authorized to do some task.

smart signatures: core use - authorization
* additional parties can be authorized
* more operators like OR and AND

Main difference from traditional signatures: 
 * trust policy not interpreted by a CA or code by app/browser/os
 * trust policy is embodied by the signer into the signature itself.

inspiration: 
  * btc tx signature
  * has "Script" a stateless predicate language. many use-cases.

use-cases: 
* multifactor expressions
* signature delegation, and limit the time for it, or limit to quantity of money, 
and optionally permanently pass control (e.g. employment changes)
* dev release/CI toolchain
* transactional support: 
  * signatures are part of a larger process: provided specific tx states exist or test against oracles. 
  * For instance: prove provenance of art (via transaction history)

Language requirements:
* **composable**: need simple data structures (stacks, lists, etc.), constrained set of operations to allow security review. Like in Forth, Scheme, Haskell.
* **inspectable**: auditable by a programmer.
* **provable**: should be formally analyzable and support tools to discover hidden bugs.

System requirements:
* **deterministic**: script must produce same result.
* **bounded**: execution should not exceed cpu or memory limits, and size should be minimal and bounds should be deterministic
* **efficient**: no requirements on difficulty to create signature, but cannot be costly to verify.

What about privacy as a requirement?

  * there is a tradeoff between flexibility and privacy. A Smart signature may allow correlation. Reduces substituability and may break fungibility and bearer aspects.
  * advice: limit sharing and execute off-chain. Be transparent and deliberate.

experiment: Bitcoin Script

  - (+) deterministic, bounded, efficient
  - (?) is it composable, and inspectable?
  - (-) not provable 
  - no standalone version

experiment: Ivy, by chain.com

  - compiles to btc script. easier syntax, static types.
  - (+) inspectable -> compiles to btc Script
  - (-) same limitations as btc Script
  - whitepaper, full playground available

experiment: Dex

  - Determistic Predicate by Peter Todd
  - Scheme-like lambda calculus
  - blogpost, no whitepaper or code
  - (+) composable, determistic, efficient, bounded
  - kinda: inspectable, provable

experiment: Simplicity, by Russell O'Conor

  - Sequent Calculus
  - (+) provable, deterministic, bounded, efficient, composable
  - kinda: inspectable
  - whitepaper, no code

experiment: Sigma-State

  - Alexander Chepurnoy, optimized for zk-proofs, ring and threshold sigs, strongly typed
  + inspectable, composable, deterministic, efficient
  kinda: provable, bounded

experiment: Michelson

  - by Tezos, inspired by OCaml, is Stack-based like Script
  - (+) composable, inspectable, efficient
  - kinda: provable, bounded, deterministic
  - whitepaper, playground

experiment: crypto conditions

  - by Ripple for Interledger
  - not a language, a JSON description
  - easier testing, limited flexibility
  - (+) boundded, efficient, deterministic
  - kinda: inspectable
  - composable, provable


watching: bamboo on evm

  - jsript like
  - explicit state transitions
  - avoids reentrancy

open questions:
* context
  * internal references? lists, trees, acyclic graphs
  * run time context?
  * external process state?
* oracles:
  * preserving execution boundedness
  * what are simple mvp oracles?
* revocation
  * proof of non-revocation?
  * make scripts have a short life versus revocation?

moar open questions:
* object capabilities
  * are "ocap" and Least Authority architectures another use-case?
* Cryptographic primitives
  * HD keys?
  * Poelstra's "Scriptless Scripts": embedding multiple signatures into a single signature. gives privacy and other benefits.
* Smart Contracts
  * is there value in non-predicate scripts? providing true or false, ... or a hash?
  * none of the experiments above are not Turing complete, but where exactly is the line between a SmartSignature contract and a regular SmartContract? is turing completeness the only criteria, or are there others?

"Smart signatures" paper

Language geeks: go to #RebootingWebOfTrust event march 6-8 in Santa Barbara

q. (ken huang) can one extend X.501 to have these scripts?
a. there has been some talk about this.
more interested in "verifiable credentials or claims" community, using JSON that can be signed as a graph-signature.

q. (ken huang)...?
a. idea behind "self sovereign" identity. something something.

q. confused, you said "a smart signature is a predicate language" but Simplicity is more than that?
a. not turing complete, it unwinds all loops etc. 
and people want to use it for provable predicate scripts, but clearly can be used for more.
still suffers from re-entrancy

q. what was your role in "sovereign foundation"? you'd mentioned it and i missed it.
"sovereign foundation" are part of RebootingWebOfTrust community and working with W3C who have given permissions to move it forward as a "work item". 


Talk 8: Hardening Lightning {#talk8}
---------------------------------
[Olaoluwa Osuntokun](https://twitter.com/roasbeef), Lightning Labs. [video](https://www.youtube.com/watch?v=V3f4yYVCxpk&feature=youtu.be). If you are like me, then you'll watch this video at 0.75x speed.

To see the mechanism behind the Lightning Network, see this excellent [Bitcoin Magazine article](https://bitcoinmagazine.com/articles/understanding-the-lightning-network-part-building-a-bidirectional-payment-channel-1464710791/). TLDR; payment channels, connect them, route transactions across them.

This talk is the only one dealing with a production system, and goes into many improvements that could be done to many aspects of the LN. Its dense: each improvement has an underlying concept and current implementation that one needs to understand first.

Also, see a more recent proposal similar to this called [Eltoo](https://blockstream.com/2018/04/30/eltoo-next-lightning.html)

Talk Outline:
1. Overview of Lightning's Security Model
2. Hardening Contract Breach Defense - what to do in the face of a big backlog of transactions?
3. Reducing Client Side History Storage
  A. a new channel design! It makes channels more succint, storing less history, makes outsourcing more efficient.
4. Scaling Outsourcing (WatchTower++)
5. On-chain Succintness
  A. Better Fee control

Think of Lightning as four layers:
1. Blockchain Layer: bitcoin!
2. Channel Link Layer: opening and updating some channel between me and Bob
3. End-to-End Routing Layer: HTLCs (Hash Time Lock Contracts)
4. Application Layer: exchanges, etc. i.e. [LApps](https://dev.lightning.community/lapps/)

Security Model:
  * Bitcoin is the dispute mediation system:
    * contract creation+enforcement happens onchain
    * contract execution happens offchain
    * secure chain acts as the "trust anchor"

  * The easy way or the hard way:
    * easy way: optimistically we only require on-chain enforcement in the case of a dispute, and everything else can happen offchain
    
  * For a dispute, we assume we can write to the chain **eventually**(T, a time parameter)
    * T = csv_value (see CheckSequenceVerify time-lock in the Bitcoin Magazine article above)
    * Configurable, determines chain watching frequency
    * this is the time-out period within which any dispute must be handled 

  * We assume participants don't collude with pool-operators or miners. Strong non-censorship assumption.

Hardening Contract Breach Defense (Strategy):
  * Active breach defense:
    * A cheater may make invalid state attestations on the main blockchain, so LN users need to actively watch the main blockchain
    * Defenders need to provide evidence to the blockchain showing a violation of signed contract

  * in face of a large backlog of transactions, or low fee rate:
    * its possible one is unable to confirm the "justice tx" in time
    * failure to provide evidence, allows cheater to succeed! :-(

  * Some properties of LN we should bear in mind:
    * Cheater is locked into fixed-fee due to commitment structure
    * Cheater only has access to their active balance
    * Defender has access to entire balance in channel
  * Scorched-Earth approach
    * strategy: iteratively siphon cheater's funds to be mining fees!
    * end game: all cheater's funds go to miners, defender made whole
      * cheater needs to pay fees that are more than the balance to succeed
  * as a result of sending the cheater's funds as miner-fees, the transaction gets prioritized by miners over the rest of the backlog

Reducing ClientSide History Storage
  * Lightning's contract execution is local
    * clients need to store current-state
    * clients need to store prior states for possibly fixing breaches onchain
    * "shadow chain" only manifested on mainchain in event of a conflict. 
      * the "shadow chain" is this set of alternate transactions, which let a Defender regain the funds in the channel, against a Cheater.

  * Channels' state transitions
    * update to current commitment (add/remove HTLCs, new contracts, etc.)
    * each state transition produces a new state
    * need to keep track of portion of prior state in order to be able to enforce

  * Goal: reduce per-channel state for scalability
    * allows for "lighter" light clients
    * high-throughput backbone nodes may see considerable savings

  * Minimally, one needs to store the current state, which is required to be able to go onchain for enforcement/delivery

Overview of Commitment invalidation:
  * each state transition, invalidates the prior state
  * this is critical for safety of bi-directional channels
  * incentives promote forward-progression
    * if you violate the contract, you get slashed!
    * worst-case enforcement: party attempts to claim the prior state
  * naive method: keep all prior states
    * in high-velocity scenario, quickly becomes untenable
    * would then require frequent resets to abandon prior state -> unnecessary on-chain control traffic!

History of Prior Succint Commitment Invalidation
  1. decrementing sequence-locks (using [BIP 68](https://github.com/bitcoin/bips/blob/master/bip-0068.mediawiki)).
     * how: use relative time-locks such that the latest state can go in before prior states
     * downside: this design limits the number of possible updates

  2. Commitment invalidation tree.
     * used in [Duplex Payment Channels](https://www.tik.ee.ethz.ch/file/716b955c130e6c703fac336ea17b1670/duplex-micropayment-channels.pdf)
     * how: commitments are structured in a tree such that the parent must be broadcast before the leaf.
       * in this design, the Roots have decrementing time-lock with a "kick off" that allows for indefinite lifetime
     * downside: has a big on-chain footprint
  3. Commitment Revocations: hash or key-based, is the current channel design
     * how: every state has a public-key, and to move on to the next state, one must reveal the secret-key of the prior state.
       * they've figured out a way to generate this public-key in a deterministic way: can walk down the client's constant-size state to generate secrets, and the receiver can collapse these down. (gah, need more details here!!) 
     * downside: must critically store O(log n) secrets of the remote party. More complex key derivation.
       * Also, has asymmetric state between the two channel parties. This is because of the way "blame" can be ascribed to avoid cheating. This is bad because if there is a network with N participants, then keeping track of state for N grows combinatorially.
  * GOAL:
    1. symmetric state
    2. constant storage for prior states
  
Commitment invalidation in Lightning 1.0 (current Lightning Network):
  * Lightning 1.0 uses key-based commitment invalidation
    * each party has multiple static EC points aka "basepoints".
    * Static points are tweaked with fresh randomness for each state
    * the randomness is derived from a verifiable PRF:
       * currently using a multidimensional hashchain: shachain(k,i): a structure such that element number 10 can derive everything before it. 

  * <insert formulas for doing Key revocation and validation>
 
  * Downsides:
    1. Client storage: C + O(log k)
    2. Outsourcer Storage: O(M) + O(N) + O(log(k))
    3. Complex key derivation
    * where
      * N = num states
      * M = number of historical HTLCs
      * k = number of state updates ...ever!
      * C = current-state

OP_CHECKSIGFROMSTACK
  * strawman proposal for a new Bitcoin opcode
  * main idea is to give the ability to validate signatures from arbitrary messages. Can do delegation ("this is a public key signed by Bob and we're going to use it"), or Oracles ("this is an integer signed by Bitfinex and we're going to use it"), or have "blessed message structures" ("protocol has an opaque message but having structure, and shows as evidence that this must be signed by these two participants in the future")
  * current bitcoin signature validation scheme already has a much simpler implicit message that this concept extends.

Signed Sequence Commitments:
  * Definition:
    * a generic commitment state invalidation scheme
    * openings of signed commitments replace revocation-keys
  * state transition:
    1. Reveal prior commitment opening:
      a. open(c) = (n, r), where n = state number, r = randomness
    2. embed `c` within commitment output script
    3. sign the next commitment: sig(A+B, commit(r, n++)), where A = Alice's signature, B = Bob's signature
  * To enforce:
    * Present (sig, c, n, r) to say "i know of n opening to a signed commitment of a newer sequence-number"
    * each state creates new signed-sequence, only need to store the latest one!
  * Advantages of this design:
    1. O(1) constant size storage for client and outsourcer
    2. simpler client-side implementation
    3. uses same [BOLT #2 state machine](https://github.com/lightningnetwork/lightning-rfc/blob/master/02-peer-protocol.md)
      * BOLT is the spec for the Lightning Network

Scaling Outsourcing: a review of state outsourcing
  * LN's security model assumes mining is decentralized, and there is on-chain liveness.
     * on-chain censorship is a major threat
     * CSV value T acts as a time-based security parameter for the Defender against the Cheater. This is configurable on a per-channel basis.
  * For participants who are not eternally vigilant, which is most folks, they can outsource to a WatchTower
    * under current design: 
      * for commitments:
        * send initial base-points. This is needed to construct the "witness script template": 
        * For each state, send a new signature for the "justice transaction". This is the transaction that protects against the Cheater.
        * send description of "justice transaction", assuming both sides are using [BIP 69](https://github.com/bitcoin/bips/blob/master/bip-0069.mediawiki)
          * BIP69 ensures a deterministic ordering of tx inputs and outputs, originally meant for ensuring tx-privacy in bitcoin wallets.
      * for HTLCs:
       * HTLCs require a new signature for every single HTLC that is there.
       * So, encrypt half the blob of the transaction with txid[:16] (which cannot be predicted), and WatchTower can check if this decrypts with the txid on the chain.
    * possible mechanisms:
      1. authentication: could use ZKP's 
      2. compensation: pay-per-state, only provide bonus ipon action, subscription, etc.

Scaling Outsourcing: lighter outsourcers
  * With Signed Sequence Commitments
    * only need single (c,n,s,r) tuple per-client. O(1) constant size space. 
    * also able to skip outsourcing states. The older shachain approach needed every single state to enforce. Now, the LN client can choose to send every 10th state (or any N'th state) and pick some risk-level of being cheated, of their choice.
  * BUT: the new design, as is, lets outsourcer take all the funds of the Cheater. 
    * However, one may want to add flexibility here to let the Cheater's funds be split in some proportion between the Client (i.e. the Defender) and the Outsourcer.
    * can also cleverly use covenants to split the Cheater's funds.  

Scaling Outsourcing: outsourcer incentivization
  * two ways: pay-per-state or retribution bonus
  * BUT: retribution bonus would hopefully be minimal since breaches should be unlikely given the HTLC design
  * pay-per-state:
   * each outsourcer has a specific kind of e-cash token, and use these to pay the outsourcer 
   * this decoupling lets the outsourcer payment be done independently from the initial payment
   * and this opens up other possibilities of business-models for the outsourcer.

Scaling outsourcing: outsourcer static backups
  * LN wallets have additional storage requirements:
    1. Static state: parameters for each open channel like who are the recipients, cryptographic keys involved, etc.
    2. Dynamic state: recovation-keys etc. which we would delegate to Outsourcers.
  * We could encrypt this wallet-state and also send it to the Outsourcer
  * And then can have cryptoeconomic schemes to ensure the Outsourcer is being honest:
    1. proof-of-retrievability protocols can be used in challenges that the Outsourcer needs to satisfy at some random frequencies.
    2. can have some Providers who regularly issue these challenges and notify the client if the Outsourcer goes rogue.

Onchain Succintness: 2-stage HTLCs
  * in current commitment design ([BOLT #3](https://github.com/lightningnetwork/lightning-rfc/blob/master/03-transactions.md))
    * tries to solve an issue were when the CSV is large, the CLTV must be much larger 
    * solved by making HTLC-claiming a 2-stage state machine:
      1. have convenants that are multi-signature and off-chain
      2. attest (broadcast) -> delay (csv) -> claim (sweep)
   * downsides:
     1. requires a distinct transaction for each HTLC
     2. must store signature for each HTLC
     3. new state updates require signing+verifying N signatures, one for each HTLC
  * Solution:
   * use actual covenants in HTLC outputs! This eliminates sig+verify for each commitment creation, and eliminates signature storage.
   * BUT: these don't exist in bitcoin :-(
  * workaround solution:
   * make more liberal use of bitcoin flags: sighash_single || sighash_anyonecanpay
     * sighash_single signs this input, its corresponding output and other inputs partially.
     * sighash_anyonecanpay signs only the current input.
     * combining both flags: signs this one input and its corresponding output, and allows anyone to add or remove other inputs.
     * this allows 2-stage HTLC transitions to effectively happen in a single onchain transaction.

Onchain succintness: multi-party channels
  * design questions in multi-party channels involve:
   1. who pays the penalty, 
   2. if one wants to update their own state, do they need to wait for everyone else to be online 
  * recent work on this in this paper [Scalable Funding of Bitcoin Micropayment Channel Networks](https://www.tik.ee.ethz.ch/file/a20a865ce40d40c8f942cf206a7cba96/Scalable_Funding_Of_Blockchain_Micropayment_Networks%20(1).pdf)
  * creates a hierarchy of multi-sig and regular bi-di channels. Can basically embed many channels within other channels using signature-aggregation of the participants in that channel.
  * downside: with key-based revocations design, it leads to a large number of onchain transactions in worst-case
  * Solution: reuse Signed Sequence Commitments
    * has symmetric state, and can maintain constant-sized revocation state independent of number of participants in the channel itself. 

Onchain succintness: Fee control
  * today, all the fee in the channel is established at the time of channel-creation
  * but puts burden on the channel participants to anticipate how fees will look in the future when they want to close the channel
  * instead, can use liberal sighash flags (like sighash_noinput) to allow for fee paying inputs.
   * the original sighash_noinput proposal is [here](http://bitcoin-development.narkive.com/ByYWXcxA/sighash-noinput-in-segregated-witness)
   * sighash_noinput only signs the script and not the inputs or outputs. :etting one bind transactions with matching scripts together. This should (how?) enable setting the fees later on.

Talk 9:  Enter the Hydra: towards principled bug bounties and exploit-resistant smart contracts {#talk9}
---------
Florian Tramer et al:
www.thehydra.io making a ethereum contract development framework

bug bounties leverage crowd to find issues, rather than rely on small internal team of engineers to do so.
problems:
* unaligned incentives. exploit $$$ > bounty $
* time lag between reporting and action
* no fair exchange: bounty admin may not pay! don't know ahead of time what the value of bounty is, so how do you know how many resources to put into it?

perfect bug bounty should satisfy:
* strong exploit gap: small bounty incentivises disclosure for valuable program
  * rational attacker's game:
    * no bounties => found exploit => no dilemma, must exploit!
    * classic bounty, unknown payout => doesn't change much
    * classic bounty, known payout => depends if attacker can earn more from exploiting
    * known bounty, use "exploit gap" => use multi-version programs => from space program => have diff implementations of same spec => run all in parallel and check they agree => if one has a bug, check if majority of systems are correct => if majority have mistake, then we're in trouble => if all don't make same mistake in same way, can detect if something is wrong => if all fails in same way, then we're screwed
      * assumes full independence of faults
      * tried at nasa, not cost effective. Used 3x versions, got 4x fewer faults.   
   * but smart contracts are diff:
     * in classical work, availability >> reliability
     * but in smart contracts: we don't care if not available. We want N-out-of-N agreement, better "no answer" than "wrong answer".
     * "exploit gap" means that attacker must find bug simultaneously affecting all deployments. As soon as one is known to be "bad", then we shut it all down.
 
  * smart contracts, have small contracts with lots of value per line:
    * omisego: 396 line of code
    * tether: 423 lines
    * EOS: 584 lines
    * wtf

* automatic remediation: immediate intervention in affected software
  *  put money in escrow or return to sender, and figure out how to fix bug

* automatic payout: must be censorship resistant, verifiable
  * use smart-contract to payout bounty, if we detect at runtime that there has been an exploit.

challenges 
* of coordinating multiple smart contracts:
  * coordinator needs to be a simple proxy, be bug free
  * maintain consistent blockchain state
  * how to recover from a discovered bug => escape hatches

* frontrunning
  * attacker can break the "exploit gap" by withholding bugs
  * search for full exploit until someone tries to claim a bounty
  * solution: "submarine sends" see article on hackingdistributed.com

Bug witholding and commit-reveal:

  - Solution 1: to claim bounty at time T, must commit to bug at T-1 time
    - Problem: attacker commits in every round, and only reveals if someone else does

  - Solution 2: to commit, must pay $$ (in some verifiable way)
    - Problem: if committments are visible on chain, then attacker waits for someone else to commit and does so at the same time.

  - Solution 3: hide commitments (e.g. proof of burn to random address)
    - Problem: is wasteful

**submarine sends** (post-metropolis version)

goals:
1. only allow committed-users to send a transaction to C
2. being eternally committed is expensive
3. attacker can't know if someone has committed
4. money is not wasted

submarine sends:
 * Phase 1: compute: addr = Hash(ContractAddr, nonce, code) and send $$ to addr
   * to an attacker, this looks like some random address on the network, a fresh address that hasn't existed before. this kind of tx happens a lot in ehtereum so cannot ditinguish this send from other txs.

Phase 2: reveal addr to contract
  * C verifies that addr got $$ in phase 1
  * C creates a contract with specified nonce, and code.
  * C collects $$ and allows transaction
