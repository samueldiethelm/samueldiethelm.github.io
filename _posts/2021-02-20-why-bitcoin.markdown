---
layout: post
categories: crypto
title: Why Bitcoin
tags: [crypyocurrencies, bitcoin, blockchain]
---
Many times I have spoken to friends about Bitcoin and the blockchain to try to explain why it is so revolutionary as a concept. It is for some people a complex topic to understand so I decided to write this little article explaining it a bit and expressing my general opinion on the matter.
<!--more-->
## What is Bitcoin?
In essence, it is a currency that is uncensorable and mostly not controlled by any single entity. It is economic freedom reclaimed. It might still seem a complicated system, but so are email, the smart grid, cellphone networks and banking systems to name a few and this doesn't stop us from using those technologies every day. 

## Why is it secure?
Bitcoin is based on a concept called public key cryptography. You might not realize it but you are using this technology every day. When you visit almost any website, you will see the secure `https://` protocol. What is happening here on a very simplistic level is that the website you visit is sending you their public key and telling you "encrypt your traffic with this key". When you encrypt any message with a public key you are making sure that only the creator of that public key can decrypt it by using what is called the private key, and as its name implies, it must remain private. Those public and private keys are always created in pairs and are the basis for all the internet security nowadays. In addition to this, the private key owner can digitally _sign_ a message with it, and anybody can verify the authenticity of the signature with the public key.

## How does a transaction work?
When you first jump into Bitcoin, you will get a public and private key pair. With the public key, you get a Bitcoin address which would be like your bank account number. At this point, you have no balance yet. Let's say now someone pays you some Bitcoin, what they are effectively doing is signing a message which says "I send 0.004 Bitcoin to Alice". This message is then broadcasted and eventually included into a block of the Bitcoin blockchain. 

As you might see, your balance is simply an accumulation of the transactions involving your address. You could try signing a message which states you transfer 10BTC to Alice, but if you never received that amount of BTC, that transaction will not be valid and will never be included in the blockchain. This validation is part of the mining process.

## What is mining?
Mining is the act of creating blocks to be included in the blockchain, and in the case of Bitcoin, those blocks consist of transactions. Every validator working on the network, contains a local copy of the blockchain, and when transactions are sent or _broadcasted_, miners _listen_ for those transactions, they validate them against their copy of the blockchain and compose a block with many transactions to be added as the next block.

Up to this point anyone could compose a block but all validators need to agree on which block goes next in the blockchain. For a block to be accepted into the blockchain by all the validators, it must contain valid transactions but must also contain what is called _Proof of Work_ (PoW), which is a way to prove that you have spent time and resources on mining that block.

## Proof of Work and Hashing
This section might get more complicated, but it is fundamental to how the blockchain works, and this concept is the real revolutionary innovation in the original Bitcoin whitepaper.

In addition to containing valid transactions, a valid block must also comply with a consensus rule that determines that the hash of a block must start by a certain number of zeros. Hashing is an algorithm that given some data, generates a hash which is a fixed length string of text. This hash is unique to the given data, and will be completely different with any minor modificacion to this data. It is incredibly useful for data integrity, since you can validate that an arbitrarily large amount of data is uncorrupted by calculating its hash. For example, if you download a big file from the internet, you can compute the hash of the downloaded file and ask the file creator to confirm the hash of the data at origin. If both hashes are equal, you can be certain that the downloaded file has not been corrupted or even altered by a middleman.

### SHA256 algorithm

SHA256 is the hashing algorithm used by Bitcoin, and as an example, the SHA256 hash of "hello world" is `b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9`. If you change just one letter the hash will be drastically different, for example, the SHA256 hash of "Hello world" (with capital "H") is `64ec88ca00b268e5ba1a35678a1b5316d212f4f366b2477232534a8aeca37f3c`. These examples are hashes of short texts, but you could generate the hash of any data and again, changing just one byte will render a different hash.

### Nonce
Hashing algorithms are also one way function, which means, there is no way of knowing which content generated a given hash without trying every single combination of input data. Right now, we know that a block contains transactions, so we can obtain its hash, which will be random string of data depending on the transaction contents. In order for this block to be valid, the miner must add, in addition to the transactions, a random string of data (called the _nonce_) in order to obtain a hash that starts in a given number of zeros. The miner must try all possible _nonce_ variations which takes time and computing power. This can be considered as a form of lottery, and the winner is the first one to find a valid nonce which generates a compliant hash.

Once a miner finds a nonce that generates a valid hash, the block can be broadcasted to the network. The rest of the miners can easily validate the transactions, validate the provided nonce and add the block to their local copy of the blockchain. All the other miners should then create a new block with new transactions and start mining again to try to get their block included next.

### Difficulty
The number of preceding zeros is defined by the difficulty parameter, and this system ensures that blocks are mined every 10 minutes on average. This works by averaging the time elapsed during the last 2016 blocks and adjusting the difficulty level. If more people start mining, the number of blocks produced will increase and therefore the difficulty will automatically increase, making it harder to mine blocks, and keeping the 10 minute average in check.

## Immutability
Another well known feature of Bitcoin and blockchains in general is their immutability. This means that once a block has been generated, it is very hard to roll back any transaction which was included in it. Moreover it becomes virtually impossible to change the contents of a block which is several blocks back in the chain. This is achieved by chaining the blocks with their hashes. Each block also contains as part of its content the hash of the previous block, which means that if you were to remove or change any block in the chain, you would need to replace the following block's hash and to do so you would have to mine yourself that block again and the following one and so on.

Effectively, you would be fighting on your own against the cumulative mining power of the rest of the Bitcoin network who keep adding blocks at the end of the legitimate blockchain. This is computationally prohibitive and might only be possible to do if you have vast amounts of computing power but still in that case you might only be able to alter the last block in the chain.

It can happen though that two competing miners successfully mine a block and broadcast it at the same time. In that case other miners might chose one or another block to continue mining upon. As soon as the next miner adds a block at the end of one of those competing blocks, this longer blockchain would automatically be considered as the valid one. For this reason, many merchants or Bitcoin exchanges today only consider a transaction as valid once it has received a few _confirmations_, meaning a few additional blocks have been added to the chain that contains your transaction. In practice, unless you are intentionally trying to trick the system, this circumstance should not affect you when you send a regular transaction, as it will likely be included in both competing blocks.

## Mining reward and Halving
There is one more fundamental piece of data in a valid block which is the reward address. The miner includes here a special transaction that specifies a reward for having mined this block. When Bitcoin started, this reward was set at 50BTC per block, but over time it is halved every 210,000 blocks or roughly every 4 years. It currently is 6.25BTC.

Since transactions are only possible if you previously have received Bitcoin, it ensures that BTC is introduced into the system in a distributed manner without relying on a single authority issuing BTC. Having a central authority would be a vulnerability and by design no single entity should be able to alter the system at will. The mining process together with halving also contributes to Bitcoin's scarcity. Effectively only 21 million BTC will ever be in circulation and this scarcity sometimes has had Bitcoin compared to Gold.

## Block size and transaction fees
Blocks initially were capped at 1MB in size. This avoids them growing uncontrollably in size and ensures that the blockchain grows at a consistent rate, remember that miners must each store a copy of the blockchain. This initially was not a problem since 1MB every ten minutes was enough space for the few transactions that were occuring and those transactions had effectively no fees. However, as Bitcoin's popularity grew over time, more and more transacions were being broadcasted and eventually not all could fit on the next block. This is how transaction fees started to be commonplace. 

When you make a Bitcoin transaction, you also set a transaction fee associated to it. This is an amount of Bitcoin which is sent to whoever mines the block which includes your transaction. Given the limited space available in each block, this is now an added income stream for miners and a bidding system for users to get their transaction included since miners will prefer picking those transactions which give them more rewards. In times of high traffic volumes, those transaction fees can get quite high, leading to congestion in some times and has been recurrently a debate on Bitcoin's scalability.

## Freedom reclaimed
In my opinion, Bitcoin is the basis for a stronger economic freedom. I would like to equate it to other freedoms that we currently take for granted such as freedom of speech or freedom of thought. The potential of Bitcoin is that if you own the private keys which I discussed at the beginning, you own the funds associated to them. Your funds can't be frozen and there can be no corrupt political system that can lock you from accessing your assets. Similarly, since the asset has inherent scarcity and there is no central authority, no actor such a central bank can _print_ more BTC and devaluate its value leading to an inflation and the loss of value of your savings.

Additionally, Bitcoin is a global currency, so even though it might not be _instant_ and since it is not tied to any geographical location, it takes virtually the same time to make a transfer to your neighbor than to someone at the other end of the world. This in regular banking systems is not feasible and that is why international transactions can take days to settle and can have comparable transaction fees.

Finally, apart from Bitcoin, there are many other revolutionary innovations that have been possible thanks to blockchain technology that are not necesarily related to monetary transaction which I would like to touch upon at some point.