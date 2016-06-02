# GrapheneJS (graphenejs-lib)

Pure JavaScript Bitshares/Graphene library for node.js and browsers. Can be used to construct, sign and broadcast transactions in JavaScript, and to easily obtain data from the blockchain via public apis.

Most of this code was written by [jcalfeee](https://github.com/jcalfee), my work was mostly just repackaging to a discrete npm package.

[![npm version](https://img.shields.io/npm/v/graphenejs-lib.svg?style=flat-square)](https://www.npmjs.com/package/graphenejs-lib)

## Setup

This library can be obtained through npm:
```
npm install graphene-crypto
```

## Usage

Three sub-libraries are included: `ECC`, `Chain` and `Serializer`. Generally only the `ECC` and `Chain` libraries need to be used directly.

### Chain
This library provides a websocket rpc connection and other utility functions to handle blockchain state. To obtain a websocket connection to the Openledger API and subscribe to any object updates, use the following code:

```
var {Apis} = require("graphene-crypto");
Apis.instance("wss://bitshares.openledger.info/ws").init_promise.then((res) => {
    console.log("connected to:", res[0].network);
    Apis.instance().db_api().exec( "set_subscribe_callback", [ updateListener, true ] )
});

function updateListener(object) {
    console.log("set_subscribe_callback:\n", object);
}
```
The `set_subscribe_callback` callback (updateListener) will be called whenever an object on the blockchain changes. This is very powerful and can be used to listen to updates for specific accounts, assets or most anything else, as all state changes happens through object updates.

The Chain library contains a complete state container called the ChainStore. The ChainStore will automatically configure the `set_subscribe_callback` and handle any incoming state changes appropriately. It uses Immutable js for storing its state, so all objects are return as immutable objects. It has its own `subscribe` method that can be used to register a callback that will be called whenever a state change happens.

The ChainStore has several useful methods to retrieve, among other things, objects, assets and accounts using either object ids or asset/account names. These methods are synchronous and will return `undefined` to indicate fetching in progress, and `null` to indicate that the object does not exist.

```
var {Apis, ChainStore} = require("graphene-crypto");

Apis.instance("wss://bitshares.openledger.info/ws").init_promise.then((res) => {
    console.log("connected to:", res[0].network);
    ChainStore.init().then(() => {
        ChainStore.subscribe(updateState);
    });
});

let dynamicGlobal = null;
function updateState(object) {
    dynamicGlobal = ChainStore.getObject("2.1.0");
    console.log("ChainStore object update\n", dynamicGlobal ? dynamicGlobal.toJS() : dynamicGlobal);
}

```

### ECC
The ECC library contains all the crypto functions for private and public keys as well as transaction creation/signing.

As a quick example, here's how to generate a new private key from a seed (a brainkey for example):

```
var {PrivateKey, key} = require("graphene-crypto");

let seed = "THIS IS A TERRIBLE BRAINKEY SEED WORD SEQUENCE";
let pkey = PrivateKey.fromSeed( key.normalize_brainKey(seed) );

console.log("\nPrivate key:", pkey.toWif());
console.log("Public key :", pkey.toPublicKey().toString(), "\n");
```

## ESDoc (beta)
```bash
npm i -g esdoc esdoc-es7-plugin
esdoc -c ./esdoc.json
open out/esdoc/index.html
```