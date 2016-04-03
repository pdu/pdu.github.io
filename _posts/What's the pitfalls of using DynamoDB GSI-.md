---
layout: post
title:  "What's the pitfalls of using DynamoDB GSI?"
date:   2016-04-03 11:04:52
categories: database
---

# What's the pitfalls of using DynamoDB GSI?
## What's DynamoDB GSI?
- [DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html) is a NoSQL database service of AWS, based on the classic paper of [Dynamo: Amazon’s Highly Available Key-value Store](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [GSI](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html) stands for Global Secondary Index. Since you may have many query requirements on non-key fields, create GSI can improve the query performance.
## What's the pitfalls of using GSI?
Consider the following use case:
1. After you inserted some data into the table, then you got the response that the insert was succeeded.
2. Of course you thought that the related GSI should already had the inserted data.
3. Query GSI for the newly inserted data, but sometimes you may got empty response.
**Oops, your queried data was not promised to be up-to-date.**
## Why you can't get the latest data from GSI?
After reviewing the [GSI Writes](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html#GSI.Writes) document, you can find out that:
> When you put or delete items in a table, the global secondary indexes on that table are updated in an eventually consistent fashion. Changes to the table data are propagated to the global secondary indexes within a fraction of a second, under normal conditions. However, in some unlikely failure scenarios, longer propagation delays might occur. Because of this, your applications need to anticipate and handle situations where a query on a global secondary index returns results that are not up-to-date.

**Eventually consistent** is the key.
I suppose the table updates will be sent to some queues, and GSI will incrementally build the index from queues with some scheduling algorithms.
## Why AWS team design GSI like this?
The key feature of DynamoDB is high performance key-value database, GSI only supports the complex SQL-like queries.
Updating GSI can be very heavy disk-IO operations, if you are writing a very large table. If the synchronization between table and GSI is in strong consistent fashion, the write performance of DynamoDB can be very unstable.  
## How to avoid the pitfalls?
- Don't blame AWS team for this, it's a very good design. 
- Make sure not to make decisions based on the queries from GSI, that could be  very bug prone.
- If you have noticed very large gap between table and GSI, send AWS team a support ticket.

---
> Written with [StackEdit](https://stackedit.io/).
