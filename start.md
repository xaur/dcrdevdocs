## On 51% attacks

(from chat on [2019-04-11](https://matrix.to/#/!NKtIRqGOEGaZvSQkKl:decred.org/$155495621013062aqmFL:decred.org))

degeri: (...) in PoS/PoW once someone 51% control they are in control forever. Is this argument sound?

davecgh: In a pure PoS with no other facts at play, yes. In Decred, or any other hybrid PoW/PoS that is setup to dilute influence, No.

All you have to do is think about how many tickets you could buy with your coins a year ago versus now. Assuming you were staking the same amount of coins then as now, your influence is most definitely less now than it was then, so even if you, perhaps, had 51% a year ago, you definitely wouldn't now.

Furthermore, 51% stake alone is not enough to "be in control". Whoever made that argument needs to spend a little more time digging into the math behind it.

degeri: That would be me. What is the percentage for POS "total control" ? Ie.On average 3 out of 5 tickets in ticket pool are yours. So that would be 60% ?

davecgh: It's a curve that also depends on the PoW. The crux of it is that there isn't a single number. https://medium.com/decred/decreds-hybrid-protocol-a-superior-deterrent-to-majority-attacks-9421bf486292

Scroll down to proof. One key observation is "If an attacker has around 50% of the stake/tickets, they would also need 1 times (100%) the honest hashpower." Notice that at 51% is still near 100% hash power needed.

jz: Assuming you could buy 12K DR5 units from Bitmain (you can't) it would set you back around USD $16M.

Then you need to fight it out in two other markets, the DCR market and the ticket market which both have constrained liquidity.

50% of the tickets represents USD $54M of DCR, but if you had to buy 2M DCR in the open market I would expect it to cost many multiples of that, and then you need to bid up the tickets, that's where the sdiff algo will send you back to buy more DCR in short order as ticket price ratchets up.

davecgh: I didn't calculate that 12K DR5, but if you're just taking the current hashpower and divdiing by the hash rate of a DR5, keep in mind that isn't correct because you are doubling the hash power in the process which means you only have 50%, not 100%.

Keeping the numbers easy, assume a global hash rate of 100PH/s. If you buy 100PH/s, there is now 200PH/s. So 100 / 200 = 50%.

jz: 12K DR5 is about equal to current total hash power I believe.

davecgh: Well, let's see. A DR5 is ~34TH/s.

```
$ ./dcrhashps.sh
From block difficulty: 460.41 PH/s
From last 120 blocks:  428.81 PH/s
```

We'll go with the lower value of ~429000 TH/s. 429000 / 34 ~= 12618. So yeah, that's what I figured.

degeri: So to summarize ... That argument only holds up to pure POS and not decred.

davecgh: Yes, it's true in pure PoS.

degeri: Someone needs both pow and POS super majority to keep others from taking back control .

jz: I don't see how you can prevent control from being regained. I can bid up hash and tickets too.

davecgh: Pure PoS with no other factors in play as well, since it's not impossible to devise a scheme such that there is some form of relative influence dilution there either, although it is much more difficult to do that securely, and, in fact, there is, as far as I know, no currently known way to solve the security proof for it.

The minority will eventually get a run of blocks that undoes everything the majority was trying to do at a rate of 4x.

jz: Right 20 tickets can be added and only 5 are coming out of the pool.

You would really need to dominate the PoW side to mitigate that effect.

davecgh: Assuming you could even buy the ~12618 DR5s, doing so would double the hash rate from the current ~429k TH/s to ~858k TH/s, means that 429k TH/s you just bought puts you at 50%. Same as the math above showed because it's a percentage.

What if you bought, 2x the current hash rate? 429k * 3 = 1287k. Then 858k / 1287k ~= 66%. See the formula? It's `n / (n+1)` where n is the ratio of the total hashpower being purchased. e.g. n=1 is 1x the current hashpower. n=2 is 2x the current hashpower, etc. Now, we can solve for something high like say, 90% and find that you'd need to buy 9x the current hash rate to end up with 90%.

(also see https://github.com/decred/dcrdata/issues/1022)

## Learning to code

(from chat on [2019-02-01](https://matrix.to/#/!HEeJkbPRpAqgAwhXWO:decred.org/$154901997725062UzBvj:decred.org))

[Go by Example](https://gobyexample.com/), [Effective Go](https://golang.org/doc/effective_go.html), and digging into existing quality code such as that in dcrd is about all you need to get up to speed on Go. That is probably actually Go's greatest strength. It is a super straight-foward language.

Unfortunately, that can also be its weakness when you're a much more experienced developer because it doesn't offer a lot more advanced things that make life easier such as immutability by default, zero-cost abstractions, and well-supported functional programming primitives. It's fantastic for readability though, and that is super important in a project like this.

## Mining

(from chat on [2019-03-28](https://matrix.to/#/!NNzHoaSdnsbZDQOXJr:decred.org/$155380788346482zglLE:decred.org))

There is only a single round of blake256 hashing required in Decred versus 2 in sha256 because blake does not have the vulnerabilities that require the double hashing. Then there is the fact that the Decred header provides ample nonce space so it's not necessary to completely recalculate the merkle root every 2^32 nonces (which an ASIC blows through in a few microseconds). The combined result is that it takes much less energy to find a solution for a given hash rate.

## Signatures

(from chat on [2019-02-05](https://matrix.to/#/!dhHYPTtCtvPSUfTepT:decred.org/$154933621028906vRveM:decred.org))

Signature hash calculation for transaction signing: https://github.com/decred/dcrd/blob/59ed4247a1d5816070852a332dcddff9322b9722/txscript/sighash.go#L225-L448

It uses blake256r14. The signature hash is then signed. The most common (and thus compatible with Bitcoin) type is indeed secp256k1 with ECDSA. The address is blake256r14+ripemd160+base58, but has a prefix of 2 bytes as compared to BTC's one byte. Here are the intermediate preimages to help with address costruction:

```
privKey: 0000000000000000000000000000000000000000000000000000000000000001
 compressedPubKey: 0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798
ripemd160Preimage -- blake256(compressedPubKey): ba6f7ba0201fa407e7f6d48dc5d10f685c72b625f1d17442e986018d551836f6
pkHash -- ripemd160(ripemd160Preimage): e280cb6e66b96679aec288b1fbdbd4db08077a1b
checksumPreimage -- (prefix || pkHash): 073fe280cb6e66b96679aec288b1fbdbd4db08077a1b
checksum -- blake256(blake256(checksumPreimage))[:4]: d4de2763
base58Preimage -- (checksumPreimage || checksum): 073fe280cb6e66b96679aec288b1fbdbd4db08077a1bd4de2763
address -- base58(base58Preimage): DsmcYVbP1Nmag2H4AS17UTvmWXmGeA7nLDx
address: DsmcYVbP1Nmag2H4AS17UTvmWXmGeA7nLDx
```

The prefix for mainnet addresses based on secp256k1 with ECDSA is here: https://github.com/decred/dcrd/blob/59ed4247a1d5816070852a332dcddff9322b9722/chaincfg/mainnetparams.go#L211

## Lottery

https://github.com/decred/dcrd/blob/master/blockchain/stake/lottery.go

It uses a deterministic PRNG based on blake256 hashes that is seeded with the serialization of the header of the block that is being voted on suffixed with a constant derived from the hex representation of Pi which acts as a publicly verifiable NUMS number. From there, all of the eligible tickets (also referred to as live tickets) are sorted lexicographically by their hash to generate a total order. Finally, uniformly random values are produced (which obviously means it uses the total number number of eligible tickets as an upper bound to be able to properly produce them) and used as indices into the total order.

If you're comfortable looking at code, this test harness code is pretty well documented and makes it clear what's going on. The aforementioned NUMS constant is defined here.

https://github.com/decred/dcrd/blob/e9b2b4854f6e9f70907477a4cf689c1bcfbab327/blockchain/chaingen/generator.go#L1092-L1224

https://github.com/decred/dcrd/blob/e9b2b4854f6e9f70907477a4cf689c1bcfbab327/blockchain/chaingen/generator.go#L25-L29

matheusd covered it in a video as well: https://www.youtube.com/watch?v=eysGWVhDFWY

## Websockets

(from chat on [2019-03-13](https://matrix.to/#/!HEeJkbPRpAqgAwhXWO:decred.org/$15524874501636tvwxN:zettaport.com))

https://github.com/jrick/wsrpc may be useful for working with the dcrd or dcrwallet JSON-RPC servers. It has a CLI tool to call RPCs, but with some differences from dcrctl. It's not Decred aware, and that allows you to make calls with arbitrary methods and args. Also supports websockets.

## Links

https://faucet.decred.org/ usually can send you some testnet coins.

https://docs.decred.org/wallets/cli/dcrctl-basics/

https://testnet.dcrdata.org testnet explorer

#### RPC docs

https://github.com/decred/dcrd/tree/master/rpcclient

https://docs.decred.org/wallets/cli/dcrctl-rpc-commands/

https://github.com/decred/dcrwallet/tree/master/rpc/documentation

https://github.com/decred/dcrwallet/blob/master/rpc/documentation/api.md

https://github.com/decred/dcrdata#apis

#### reference implementations for Pythons, Node.js, C#, etc

https://github.com/decred/dcrwallet/blob/master/rpc/documentation/clientusage.md

#### Decred Change Proposals (DCPs)

https://github.com/decred/dcps

## Premine

(from chat on [2019-03-31](https://matrix.to/#/!lbzTjhzNbIaDbuAxkS:decred.org/$15540662511937zsfnk:decred.org))

jy-p:

The justification for the airdrop went like this: it took several years of software dev FTE to get Decred to the point it could be launched, which had a real fiat cost. CS/C0 had spent quite a bit on btcsuite prior to Decred's launch, which I did not think was fair to attempt to recoup in the initial premine. The total amount spent to do bringup on Decred was roughly USD 300K from C0, and devs earned USD 115K for sweat equity and direct buy in. We wanted to premine as little as possible to avoid giving the dev premine too many coins.

We put the figure for a maximum of the premine at 10% (2.1M DCR) and worked backwards from that. We wanted for the dev/airdrop split to be more like 40/60, but the staking system dictated that we needed to 50/50 split to ensure nobody could kill the chain early on.

Further, some of that 4% that went to the dev premine had to be staked to be certain nobody could easily attack the chain.

We were able to cut it from 10% to 8% by estimating a value of USD 0.49 per DCR.

Bitcoin is special b/c nobody knew that BTC was going to get big back then, so there is an effective premine of 1M btc. Whether it ever gets spent - who knows.

With PoS you need an "initial set of stakers" as you cannot stake with 0 coins. If miners=stakers it wouldn't create diversity among stakeholders. Hence the airdrop to the community/contributors to create the initial group of stakers. This made it possible to start with a multi-stakeholder situation.

Trying to compare PoW and PoW/PoS launches is hard b/c no PoS component changes the consensus algorithm in a major way. With PoS, if someone buys up all the coins and just doesn't participate in the pos system, the chain could die.

Airdrop created interest amongst a lot of ppl. ~2900 ppl registered. If you got your 282 DCR airdrop and just staked it all continuously, you'd have 1500+ DCR right now.

In many cases, we have worked backwards from a deliverable set we want to have to figure out how we should do it, versus 'forward' engineering everything. e.g. we collectively noticed that consensus rule changes were the most contentious changes to make in a cc project, so we then worked backwards from that to determine how we should go about making those changes. This is covered pretty extensively on the forum and the old bitcointalk thread.

People had to confirm their emails, we vetted the list manually for fraud and various games. There were some pretty creative games played by malicious actors.

Premine code:

https://github.com/decred/dcrd/blob/master/chaincfg/params.go#L478-L481

https://github.com/decred/dcrd/blob/47ade78c1a4e350313e9b5d9f5556f707ad22d0d/chaincfg/premine.go#L10
