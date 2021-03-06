<p align="center">
  <a href="https://app.circleci.com/pipelines/github/KtorZ/cardano-ogmios"><img src="https://img.shields.io/circleci/build/github/KtorZ/cardano-ogmios/master?color=%23cc3898&style=for-the-badge&token=544a094c627f593d00aa74b5a099e81b5cc6c66f" /></a>
</p>

<p align="center">
  <img src=".github/ogmios.png" height=400 alt="ogmios" />
</p>

# Overview

**Ogmios** is a translation service written in Haskell running on top of [cardano-node](https://github.com/input-output-hk/cardano-node/). It offers a [JSON-WSP](https://en.wikipedia.org/wiki/JSON-WSP) interface through WebSockets and enables clients to speak [Ouroboros' mini-protocols](https://hydra.iohk.io/build/1070091/download/1/network.pdf#chapter.3) via remote procedure calls over JSON.

```
            CARDANO RELAY                                     OGMIOS SERVER   
                                                                           
                  ◦                                                _=_       
<-------->    ◦   ○   ◦                                          q(-_-)p     
               ○ ◯ ◯ ○                                            _\=/_      
<--------> ◦ ○ ◯     ◯ ○ ◦  <----Mini-Protocols / CBOR---->      /     \     <----WebSocket / JSON---->
               ○ ◯ ◯ ○                                         _(<\   />)_   
<-------->    ◦   ○   ◦                                       (__\_\_/_/__)  
                  ◦
```

Ouroboros' mini-protocols are described as state-machines. **Ogmios** currently supports the following mini protocols:

<details>
  <summary>Chain Synchronisation Protocol</summary>

```
 *-----------*                                              | Clients wishing to synchronise blocks from
 | Intersect |◀══════════════════════════════╗              | the chain can use the Chain Sync protocol.
 *-----------*         FindIntersect         ║              | 
       │                                     ║              | The protocol is stateful, which means that
       │                                *---------*         | each connection between clients and ogmios
       │ Intersect.{Found,NotFound}     |         |         | has a state: a  cursor locating a point on
       └───────────────────────────────╼|         |         | the chain.
                                        |   Idle  |         | 
    ╔═══════════════════════════════════|         |         | Typically, a client will  start by looking
    ║            RequestNext            |         |⇦ START  | for an intersection  between its own local
    ║                                   *---------*         | chain and  the one from the  node / ogmios.
    ▼                                        ╿              |  
 *------*       Roll.{Backward,Forward}      │              | Then, it'll simply request the next action
 | Next |────────────────────────────────────┘              | to take: either rolling forward and adding
 *------*                                                   | new blocks, or rolling backward.
```
</details>

<details>

  <summary>Local Transaction Submission Protocol</summary>

```
 *----------*                                                | Transaction submission is pretty simple &
 |   Busy   |◀══════════════════════════════╗                | works by submitting an already serialized 
 *----------*            SubmitTx           ║                | and signed UTxO transaction as one single
      │                                     ║                | message. 
      │                                *---------*           | 
      │                                |         |           | In case of  success, the server returns an
      │                                |         |           | empty  response. Otherwise, it  returns an
      │          SubmitTxResponse      |   Idle  |           | error  with some details  about what  went 
      └───────────────────────────────╼|         |           | wrong.
                                       |         |⇦ START    |
                                       *---------*           | Clients must thereby know how to construct
                                                             | valid transactions.
```
</details>

### Why Bother?

Ogmios brings the Ouroboros mini-protocols to the Web, effectively allowing light clients to interact with 
a relay node in a very simple and efficient way. Ogmios is very lightweight too and can be deployed alongside 
relays to create entry points on the Cardano network for various type of applications (e.g. wallets, explorers,
chatbots, dashboards...).

Here is a little [demo](https://ktorz.github.io/cardano-ogmios/demo) of a simple frontend app using Ogmios
to display, in-real-time, the current tip of MainNet.

# Getting Started

[![DockerHub Badge](http://dockeri.co/image/ktorz/ogmios)](https://hub.docker.com/r/ktorz/ogmios/)

```
🐳 docker-compose up
```

:point_up: This will run and connect together:

- An OBFT Byron cardano-node, connected to `MainNet`.
- An Ogmios server, listening to `localhost` on port `:1337`.

Check out the [demo source](https://github.com/KtorZ/cardano-ogmios/blob/master/demo/demo.js) for a detailed albeit simple client example!

<hr/>

<p align="center">
  <a href="https://ktorz.github.io/cardano-ogmios/api-reference">:book: API Reference</a> 
  |
  <a href="docs/COMMAND_LINE.md">:computer: Command-Line Interface</a>
  |
  <a href="docs/AWS_DEPLOYMENT.md">:package: Deployment Guide</a>
  |
  <a href="CONTRIBUTING.md">:gift: Contributing</a>
  | 
  <a href="CHANGELOG.md">:floppy_disk: Changelog</a>
</p>

<p align="center">
  <a href="https://github.com/KtorZ/cardano-ogmios/blob/master/LICENSE"><img src="https://img.shields.io/github/license/KtorZ/cardano-ogmios.svg?style=for-the-badge"/></a>
</p>
