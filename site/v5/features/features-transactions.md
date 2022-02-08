---
title: Transactions 
date: 2014-04-18 11:30:00 
permalink: /v5/features/transactions/index.html
toc: true
eleventyNavigation:
  version: v5
  order: 1115 
  parent: features
  key: features transactions 
  title: Transactions
---


[[TOC]]
## Introduction 

We already discussed the [Mongock process](/v5/technical-overview#mongock-process) and the [migration component](/v5/migration) in previous sections where the transactions are already mentioned. 

As its name suggests, a changeUnit represents the unit of a migration. By default each changeUnit is wrapped in a independent transaction. This can be change by configuration, but it's not recommended.

<div class="tip">
<p>In this section we mention <b>native transactions</b>. By this we mean the transaction mechanism provided by the database.</p>
<p>Mongock always try to provide a transactional environment as much as possible. When native transactions are not possible, it tries to rollback the changes manually with the <a href="/v5/migration#changeunit-methods">@RollbackExecution</a> method.</p>
</div>

## Configuration

There are two points where the transactions are configured to be enforced or disabled, the property `mongock.transactionEnabled`(setTransactionEnabled method in the builder) and the **driver**.

In every driver's page, you will find enough information about how to enable the native transactions. Sometimes they are enabled by default and other times, like in the Springboot ecosystems, the transactions manager needs to be injected in the context.


As explained in the [runner properties table](/v5/runner#Configuration), the Mongock native transactionability follows the next logic:

<div class="success">
<p >When <b>mongock.transactionEnabled</b> is true, it enforces native transactions, throwing an exception is the driver is not capable of it.</p>
<p >When <b>mongock.transactionEnabled</b> is false, it disables the transactions and ignores the transactionability of the driver.</p>
<p >When <b>mongock.transactionEnabled</b> is null, it totally delegates on the driver the transactionability of the migration.</p>
</div>
 

## How it works

The easiest way to understand how Mongock handles the transactions is by looking at [this section](/v5/technical-overview#process-steps).


## Best practices

- Always set explicitly the `mongock.transactionEnabled` property to true/false.
- DDL operations placed in the @BeforeExecution method of the changeUnit.