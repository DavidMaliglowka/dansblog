---
layout: post
title: 'Web Assembly on EOS - 50,000 Transfers Per Second'
date: 2018-01-01
author: Dan Larimer
cover: ''
tags: eos webassembly wren blockchain
---
We had always intended to use a small, tight scripting language to write out contracts for EOS, and the initial choice was Wren. A few weeks back I managed to run a test with an empty contract. This revealed performance of about 1000 transactions per second; far too slow for our target performance.

Over the past couple of weeks the EOS development team has pivoted away from the [Wren](https://wren.io/) programming language and embraced [Web Assembly](http://webassembly.org/). Today we would like to update everyone on the progress and initial results we have achieved.


## Background on Web Assembly
Web Assembly is an emerging industry standard backed by Microsoft, Google, and Apple. The goal of this standard is to make it possible to run untrusted high-performance code in your browser. [Web Assembly is a game changer](https://medium.com/mozilla-tech/why-webassembly-is-a-game-changer-for-the-web-and-a-source-of-pride-for-mozilla-and-firefox-dda80e4c43cb), it will enable high performance web applications such as video and image editing and games.

<iframe type="text/html" width="100%" height="385" src="https://www.youtube.com/embed/MaJCfdmr9Wg" frameborder="0"></iframe>

WebAssembly provides a universal compile target that enables applications to be developed in any language. Currently there are compilers for C, C++, and Rust. There is even work going on to compile Solidity to Web Assembly.

## EOS Integration
A few weeks ago we introduced a [hypothetical currency smart contract written in Wren](https://steemit.com/eos/@eosio/implementing-a-hypothetical-currency-application-on-eos). Today we will show an actual working implementation written in "C" and compiled to Web Assembly (WASM). We then used EOS transactions to create an account ([@simplecoin](https://steemit.com/@simplecoin)) and upload the WASM code to the test blockchain.

At this stage of development everything is still in rapid flux and there are some rough edges that will be smoothed out before our official public test network this summer. For example, the message definitions and deserialization boiler plate can be replaced with a simple code generator that takes input similar to our [hypothetical example](https://steemit.com/eos/@eosio/implementing-a-hypothetical-currency-application-on-eos).

That said, here is an example of what the contract looks like in C today:

```c
typedef struct  {
  AccountName from;
  AccountName to;
  uint64_t    amount;
  String*     memo;
} Transfer;

void Transfer_unpack( DataStream* ds, Transfer* transfer )
{
   AccountName_unpack( ds, &transfer->from );
   AccountName_unpack( ds, &transfer->to   );
   uint64_unpack( ds, &transfer->amount );
   String_unpack( ds, &transfer->memo );
}

typedef struct {
  uint64_t    balance;
} Balance;

/** Constructor called once when code is first uploaded */
void onInit() {
  static Balance initial;
  static AccountName simplecoin;
  AccountName_initCString( &simplecoin, "simplecoin", 10 );
  initial.balance = 1000*1000;

  store( &simplecoin, sizeof(AccountName), &initial, sizeof(Balance));
}

/** Message handler when Transfer message is delivered to @simplecoin */
void onApply_Transfer_simplecoin() {
   static char buffer[100];
   int read   = readMessage( buffer, 100  ); /** load message content */
   static Transfer message;
   static DataStream ds;
   DataStream_init( &ds, buffer, read );
   Transfer_unpack( &ds, &message );  /** unpack it */

   static Balance from_balance;
   static Balance to_balance;
   to_balance.balance = 0;

   read = load( &message.from, sizeof(message.from),
                &from_balance.balance, sizeof(from_balance.balance) );

   assert( read == sizeof(Balance), "no existing balance" );
   assert( from_balance.balance >= message.amount, "insufficient funds" );

   load( &message.to, sizeof(message.to),
         &to_balance.balance, sizeof(to_balance.balance) );

   to_balance.balance   += message.amount;
   from_balance.balance -= message.amount;

   if( from_balance.balance )
      store( &message.from, sizeof(AccountName),
             &from_balance.balance, sizeof(from_balance.balance) );
   else
      remove( &message.from, sizeof(AccountName) );

   store( &message.to, sizeof(message.to),
          &to_balance.balance, sizeof(to_balance.balance) );
}

```


This code was compiled using the [WasmFiddle](https://wasdk.github.io/WasmFiddle//?1841vb) interface to generate the WebAssembly (WASM). It uses several simple API calls such as `readMessage`, `load` and `store` to fetch and store information from the blockchain.

This particular contract will create 1 million coins and allocate them to the account [@simplecoin](https://steemit.com/@simplecoin), then it will allow [@simplecoin](https://steemit.com/@simplecoin) to transfer these coins to other accounts which in turn can transfer them to others.

## Initial Benchmarks
I created a unit test that would load this [contract and then construct individual transactions to transfer funds from @simplecoin to @init1 1000 times](https://github.com/EOSIO/eos/blob/master/tests/tests/block_tests.cpp#L888).

```c
auto start = fc::time_point::now();
    for( uint32_t i = 0; i < 1000; ++i )
    {
       eos::chain::SignedTransaction trx;
       trx.emplaceMessage("simplecoin", "simplecoin",
                          vector<AccountName>{"init1"}, "Transfer",
                          types::Transfer{"simplecoin", "init1", 1+i, "memo"} );
       trx.expiration = db.head_block_time() + 100;
       trx.set_reference_block(db.head_block_id());
       db.push_transaction(trx);
    }
    auto end = fc::time_point::now();
    idump((  1000*1000000.0 / (end-start).count() ) );

```

The final result yielded about 50,000 transfers per second average over many different runs. Keep in mind that these early results have many things that impact performance for better and for worse. It is too early to draw conclusions on the final chain performance, but 50,000 sequential actions per second is a lot closer to the area we want to be - Facebook and Visa and so on.

This performance was measured on a 2014 iMac with a 4Ghz Intel Core i7 CPU.

## Comparison to Wren
The reason Wren was slower was it had to compile the code every time and there was no easy way to cache the compiled results. It turns out that the speed of Wren is highly dependent upon running a program for a long period of time relative to its initial startup cost. This is the opposite of what we want for a smart contract platform that runs many quick programs.

The WebAssembly library we are using compiles Web Assembly (WASM) into native x86 instructions using the LLVM compiler library. This enables WASM to run at up to 80% of native speed, which is far faster than any interpreted language, and certainly faster if the Just-in-Time (JIT) compilation optimization kicks in for every single run.

## The Path Forward

Now that we have proven the concept of WebAssembly based contracts and verified that their single threaded performance is already industry leading, we will continue to flush out the APIs we expose to Web Assembly and look at adding support for higher level languages and tools to make it easier for developers than writing in C.

Our initial test networks will focus on stability of code and APIs and will execute contracts within a single thread; however, the design of EOS.IO software architecture will allow us to switch to multi-threaded execution without having to hard fork the blockchain. As you can see from our initial benchmarks, even a single threaded implementation of EOS is still industry leading with plenty of headroom for today’s applications.

## Stay Informed
Sign up to our mailing list at [https://eos.io](https://eos.io) to stay informed about the latest developments and the coming EOS Token Sale.
