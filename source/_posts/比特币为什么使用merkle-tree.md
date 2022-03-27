---
title: 比特币为什么使用merkle tree
date: 2022-03-26 23:43:38
categories:
tags:
- btc
- blockchain
---

## 概述

比特币中每个 block 存储的 transaction 的技术方案是 merkle tree。主要目的是比特币为了实现 SPV，从而节省节点的存储/带宽成本，通过 merkle tree 存储交易数据，可以完成交易验证的同时并且减少消耗的资源。

## 存在的问题

比特币的本质是一个分布式的账本，去中心化的特性要求网络的节点需要存储完整的，全部的交易记录。然而随着链上的数据越来越多，完整的区块链数据库实际需要的存储空间是也会越来越多。如果每个节点参与交易都需要先下载和存储全部交易记录，并且验证交易和区块，所耗费的资源（存储和带宽）的很大的。

## 如何解决

**因此在白皮书的第八章提出了 SPV（Simple Payment Verification）来解决这个问题**。只参与交易的节点（SPV client）在网络中只是作为一个轻节点（仅存储 Block Header），不需要下载整个区块，参与交易时，需要连接到一个全节点（存储了所有交易数据的节点）来检索验证交易。

> SPV allows a lightweight client to verify that a transaction is included in the Bitcoin blockchain, without downloading the entire blockchain.

1. A 用户向商户 B 发起了一笔比特币转帐，而且告知 B，这笔交易的 txid 和 SPV 上面的 block 信息，因而 B 根据 txid 和 block 信息，同时向**多个 FullPeer** 去 confirm 这笔交易是否真的存在。
2. FullPeer 根据 block 的哈希值，先找到这个 block，然后遍历 block 里的 transaction 寻找是否存在对应的 txid。
3. 如果找到对应的 txid，FullPeer **返回相关 merkle tree 中对应 transaction 的兄弟节点以及相关父节点的兄弟节点信息。**
4. SPV 接收到返回后，**根据返回信息就可以计算出 merkle tree 的 root hash，来比对是否一致**。

### 方案细节

#### 1. 为什么向多个 FullPeer 确认？

1. 稳定性的考虑。分布式系统对单点问题的容错/容忍，提高交易验证的稳定性和效率。
2. 减少造假的可能。单个 FullPeer 可能是 attacker。

#### 2. 为什么返回兄弟节点以及相关父节点的兄弟节点让 SPV 计算验证，而不是直接告知存不存在？

FullPeer 可能是 attacker，直接告知 SPV 存不存在是不可信任的。因此需要通过一个办法让其无法造假，因为哈希运算的不可逆性，FullPeer 可以认为是没办法伪造假的信息让这些值和 txid 一起计算出 root hash，实现了防篡改。

#### 3. 为什么用 merkle tree？

减少存储的同时又能做好交易验证，第一反应是把 transaction 的哈希值都合在一起再哈希一下，相当于所有的哈希存储成一个 list 后，再统一哈希一下作为 root hash。

但这样做有个缺点就是验证的成本，hash list 方式验证的时候得获得所有 transaction 来验证，然后计算所有的 transaction 哈希后来验证。如果把 transaction 一个个分块，建成一个平衡树，来验证的话，只需要 $O(logn)$ 的时间/空间复杂度的 transaction 了。

![merkle-tree](./merkle-tree.png)

## 扩展

merkle tree 除了在 btc 中作为实现 SPV 的一个数据结构，在像 P2P BT 网络分块下载中的也有应用，一个东西被分成很多 block，从各个 peer 下载各个 block 并做验证，最后组成一个完整的文件。