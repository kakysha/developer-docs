---
description: >-
  Obyte is cryptocurrency platform, which is written with NodeJS. This site is
  for developers and should help developers to get started using Obyte platform
  in their projects.
---

# Introduction

All operations on Obyte platform are done using Obyte wallet, which is a Node.js standalone app or module, to run it you need a Node.js installation.

To interact with this wallet, you have two options:

1. if you are writing Node.js app, you can just require 'headless-obyte' module and call exported methods of the wallet directly from your code.

2. or run a 'headless-obyte' app as a separate Node.js process and communicate with it using JSON-RPC. Your code in any programming language is now separated from wallet code.

Whichever method of integrating Obyte wallet you choose, the first thing you should consider is whether you want it to operate as light node or full node. The difference is simple: full node downloads and stores full DAG with every transaction in it, while light node only stores transactions somehow related to your wallet addresses (txs made to you and from you). Full node size is around 40GB at the moment and requires a noticeable amount of time to be fully synced (hours). Light node's database has 0 transactions at start and requires no syncing at all. If your app doesn't need information about transactions of other people, you can switch to light node.

Lets [Quick Start](quick-start.md).