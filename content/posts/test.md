+++
date = "2019-07-02T19:00:00+02:00"
draft = false
title = "Advanced Segwit - How Segwit solved the quadratic sighash problem"
slug = "advanced-segwit-how-segwit-solved-the-quadratic-sighash-problem"
tags = ["segwit","bitcoin", "draft"]
image = "images/2014/Jul/titledotscale.png"
comments = false
share = true        # set false to share buttons
menu = "blog"           # set "main" to add this content to the main menu
+++

### Why write about Segwit in 2019?

Segwit is one of the most discussed topics in the history of
Bitcoin and rightfully so. The solution to a multitude of Bitcoin's
problems was surprisingly effective. The activation, on the other hand,
was a rollercoaster. Additionally, it is a very complex topic that
requires those interested to spend a significant amount of time with in order to
get a complete understanding of all the details around mechanics and
implications of it.

One of the several issues that were solved by Segwit is the quadratic
sighash problem. This seems to have been missed quite frequently by mainstream
and even Bitcoin-specific media, since information on Segwit's
reasoning seemed to focus mostly on its fix of malleability and
the expansion of available block space. Since there are not many
resources going into details on the topic, I will lay them out here and might follow
up with other articles on other frequently missed aspects of Segwit.

### The quadratic sighash problem

Let's start by locating the problem: the quadratic sighash problem
arises while hashing the transaction data. Why do we need to hash it?
We need the hash of the transaction data to create
a signature that can verify that an output is spendable by the person
who created the transaction, i.e. that person
has control over the private key. The unspent outputs we are spending in
a new transaction become inputs in that new transaction and each input needs
to be signed to ensure that each old output is actually spendable.

Now, the impact of doing this when creating a transaction is
not very dramatic, since there is only one person (the creator of
the transaction) doing it only once. But Bitcoin's security promise
stems from the fact that any participant in the network can verify
that all transactions in the history of the blockchain are correct.
In fact, every full node must verify any transaction that ends up in
a Block that has been mined. So the impact of high computational
complexity is much more serious in the validation of transactions.

On the issues itself, as the name already implies, there is something quadratic going on
while verifying a transaction (CHECKSIG, etc.). Quadratic complexity
in an algorithm is certainly [not something anyone
wants to see](https://en.wikipedia.org/wiki/Big_O_notation#/media/File:Comparison_computational_complexity.svg)
in performance critical code.

This matters to the Bitcoin network in particular because we do want
to allow as many people as possible to participate in the network,
even if the computational power they can afford is rather low. Ignoring
this could theoretically be exploited by miners who could have the interest
to reduce the number of full nodes in the network by slowing them
down to a degree that they can not stay up-to-date with the current tip
of the blockchain, factually preventing them from being a participant in the
consensus process. It is currently possible to run a full node on
an older generation Raspberry Pi for example but this could be jeopardized by such an attack.
This does not require a hash power of anywhere near 51% of the network,
a miner would only need to mine a malicious block with a high enough
frequency that low-powered nodes cannot catch up. An increased block
size would, as was in a heated discussion in 2017, would
increase the chance of the happening as it would leave more space
for malicious transactions in a single block.

While the issue had been [reported already in 2013](https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures#CVE-2013-2292)
before, it became most apparent when f2pool mined a block containing
a single transaction, which was taking up the whole block space of
1MB. Rusty Russell wrote a blogpost on it the following day, naming
it the [mega transaction](https://web.archive.org/web/20190524044433/https://rusty.ozlabs.org/?p=522).
This was not a malicious transaction but rather
sweeping small spam outputs from weak 'brain wallets'*, as you can observe
in a [block explorer](https://www.blockchain.com/btc/tx/bb41a757f405890fb0f5856228e23b715702d714d59bf2b1feb70d8b2b4e3e08).
Rusty also noted in his post that a malicious transaction could
be designed to do even worse, estimated over 3 minutes on comparable
hardware.

The more significant part of this story is the time it took to validate
this transaction: a Reddit poster reported that it took [25 seconds
to validate](https://www.reddit.com/r/Bitcoin/comments/3cgft7/largest_transaction_ever_mined_999657_kb_consumes/csva1ei/?context=8&depth=9).
He also mentioned that an electrum server would take about
4 minutes. The poster did not provide hardware specs but we can assume
that blocks usually will take less than a second to validate on up-to-date
hardware. This was clear evidence at the time, that the quadratic sighash
problem was not just theoretical.

Q: Include comment about Tx not not propagated throught the network??

Let's look at what is actually causing the quadratic time increase
in the signature hashing algorithm before Segwit.
We are talking about O(n^2), or O(n) * O(n), so what is the first n and
what is the second n?

For each input in the transaction, we have to go through the following steps (and that
means here we also have our first n):
1. We have to strip the other inputs from the transaction (and here we
already have the second n).
2. Replace the input script with the script of the output
we are trying to spend.
3. We double-SHA256 hash the resulting transaction construct.
4. We check that the signature correctly signed the hash result.

Q: Add schema of old transaction including signature in input script?
Q: Show code?

### How Segwit solved it

The breakthrough of Segwit lies in the name: Segwit stand for
segregated witness and, if that sounds strange to you,
I should add that witness stands for a proof that a statement is true
in the field of cryptography.
In the case of bitcoin transactions, the statement is that whoever wants
to spend an input is the rightful owner, i.e. has power over the private key. The
proof is the signature that is attached to the input. In other words,
Segwit stands for the separation of the signature. As you have read above, the cause
for the quadratic sighash problem is, in fact, that we needed to remove
signatures out of the input in order to perform our hashing operations
for every input.

In Segwit transactions, the input script data is not included in the input
part of the transaction. In its place we have
an empty field. Hence we do not need to to go through the tedious step
of stripping that data out of the input field for every single output (the
second n in our list above). This means that the number of operations for each
input now only grows linearly as step 1 of the 4-step process described
above simply does not exist. This reduces the computational complexity of
signature verification to O(n).

Q: Add schema of new transaction including signature in input script?
Q: Show code?

### Other proposed solutions

For the sake of completeness, I want to mention that there were other proposals
to deal with this issue. Most prominently the suggestion to limit the
transaction size to minimize the damage that could be done by larger transactions.
Particularly Gavin Andreesen suggested a transaction size limit of [100,000 bytes](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-July/009494.html).
The proposal was however dismissed by large parts of the community as it
did not solve the root cause of the issue and thus also left Bitcoin vulnerable
to exploitations of the problem.


### Conclusion

It is still fascinating to me how Segwit solved so many problems at the same
time. While malleability was probably the main motivation for it, the quadratic
sighash problem was a vulnerability that had been known for a long time and
would not have been too hard to exploit, given a significant amount of mining power.

Removing the witness data from the transaction solved the root cause of the issue,
lowering the complexity of the algorithm from O(n^2) to O(n), making it an
absolute non-issue, even in the event of future block increases.

### Further reading
[BIP143: Transaction Signature Verification for Version 0 Witness Program](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki)
[Segwit benefits](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)
[Wallet implementation hints](https://bitcoincore.org/en/segwit_wallet_dev/)

\* Brain wallets are wallets where the mnemonic phrase for the wallet is designed to
be easy to remember for a user. Examples of particularly weak ones are short
and uncreative phrases, like 'password' for example.

