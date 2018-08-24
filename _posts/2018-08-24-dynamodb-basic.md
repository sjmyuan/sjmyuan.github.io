---
title: DynamoDB Basic Knowledge
tags:
  - AWS
  - Database
---

DynamoDB is a no-sql database supplied by AWS, we will talk about some basic knowledge and best practice about it.

## Table

- Standard Table

  Amazon DynamoDB is a fast and flexible [nonrelational database](https://aws.amazon.com/nosql/) service for all applications that need consistent, single-digit millisecond latency at any scale. It is a fully managed cloud database and supports both document and key-value store models.

  Supported Attribute Type:

  - **Scalar Types** – A scalar type can represent exactly one value. The scalar types are number, string, binary, Boolean, and null.
  - **Document Types** – A document type can represent a complex structure with nested attributes—such as you would find in a JSON document. The document types are list and map.
  - **Set Types** – A set type can represent multiple scalar values. The set types are string set, number set, and binary set.

- Global Table

  [ Global Tables](https://aws.amazon.com/dynamodb/global-tables/) builds upon DynamoDB’s global footprint to provide you with a fully managed, multi-region, and multi-master database that provides fast local read and write performance for massively scaled, global applications. Global Tables replicates your Amazon DynamoDB tables automatically across your choice of AWS regions

## Key

- Partition Key(HASH)

  DynamoDB uses the partition key's value as input to an internal hash function. The output from the hash function determines the partition (physical storage internal to DynamoDB) in which the item will be stored.

  The only data types allowed for primary key attributes are string, number, or binary

  In a table that has only a partition key, no two items can have the same partition key value.

  Supported operator:

  - =

- Partition Key and Sort Key(RANGE)--*composite primary key*

  All items with the same partition key are stored together, in sorted order by sort key value.

  In a table that has a partition key and a sort key, it's possible for two items to have the same partition key value. However, those two items must have different sort key values.

  Supported operator by Sort Key:

  - =, >, >=, <, <=, <>
  - contains, begins_with, between-and, in

## Secondary Indexes

- GSI(Global Secondary Index)

  An index with a partition key and sort key that can be different from those on the table.

  Actually, the behaviour of GSI just look like a new table, it will need separate resources(WCU,RCU etc.)

- LSI(Local Secondary Index)

  An index that has the same partition key as the table, but a different sort key.

You can define up to 5 global secondary indexes and 5 local secondary indexes per table

AWS will synchronize base table and Secondary Index automatically, but this process is asynchronous, which means you need to wait a second to see the data in index after you put data in base table.

## Auto Scaling

The scaling policy also contains a target utilization—the percentage of consumed provisioned throughput at a point in time. Application Auto Scaling uses a target tracking algorithm to adjust the provisioned throughput of the table (or index) upward or downward in response to actual workloads, so that the actual capacity utilization remains at or near your target utilization.

![](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/auto-scaling.png)

## Consistency Model

- Eventually consistent reads (Default)

  The eventual consistency option maximizes your read throughput. However, an eventually consistent read might not reflect the results of a recently completed write. Consistency across all copies of data is usually reached within a second. Repeating a read after a short time should return the updated data.

- Strongly consistent reads

  In addition to eventual consistency, Amazon DynamoDB also gives you the flexibility and control to request a strongly consistent read if your application, or an element of your application, requires it. A strongly consistent read returns a result that reflects all writes that received a successful response prior to the read.

## Backup

- Same Region

  - On-demand
  - Continues

- Cross Region

  There is no official support for cross-region backups, there are some workarounds

  - Global Table
  - [dynamodb-continuous-backup](https://github.com/awslabs/dynamodb-continuous-backup)

## Cost

The cost of dynamodb mainly consist of three parts:

- The number of WCU(write capacity unit)

  One write per second for an item up to 1 KB in size. If you need to write an item that is larger than 1 KB, DynamoDB will need to consume additional write capacity units

- The number of RCU(read capacity unit)

  One strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4 KB in size. If you need to read an item that is larger than 4 KB, DynamoDB will need to consume additional read capacity units

- The size of Storage

## Best Practice

In DynamoDb, you have to give the **Partition Key** when you want to query something and the **Partition Key** just support the equal operator, so if you have more than one query scenarios, you'd better create a GSI for each scenario and use an unique id as the partition key of main table, then your database schema will be more flexible.

The default behaviour of update operation is unconditional, the last operation in parallel will win and all operation will return success

![](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/update-no-condition.png)

If you only want one parallel update success, you need to use the conditional update

![](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/update-yes-condition.png)

DynamoDB have a list of [reserved words](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ReservedWords.html), you can't use them in the expression. If your attribute name is reserved words, you have to give it an alias name.



