---
title: MongoDB Gotchas & How To Avoid Them
date: 2012-11-05
author: russ
categories: [opinion, technology]
hackernews: http://news.ycombinator.com/item?id=4745067
tags:
  - mongodb
---

A lot of people hate on MongoDB. In my opinion they're misguided - the main reason so many people think [like this](http://diegobasch.com/ill-give-mongodb-another-try-in-ten-years) is a lack of understanding. Everyone should be able to benefit from MongoDB's power and simplicity, and so as a follow up to [David's article](http://blog.serverdensity.com/does-everyone-hate-mongodb/) I have outlined some common and not-so-common things that hackers should know about MongoDB.

First though, why should you listen to me? I used to be a consultant specializing in Ops and helping companies (The Guardian, Experian) scale large web applications. As well as co-founding the offical [MongoDB London User Group](http://www.meetup.com/London-MongoDB-User-Group/), I am a [MongoDB Master](http://www.mongodb.org/display/DOCS/MongoDB+Masters#MongoDBMasters-RussellSmith) and have worked on installations from single servers all the way to projects with 30k queries per second and over a terabyte of active data. I have learnt all of the following from experience.

## 32-bit vs 64-bit
Most modern servers are either running [32-bit](http://en.wikipedia.org/wiki/32-bit) or [64-bit](http://en.wikipedia.org/wiki/64-bit) operating systems. Most modern hardware supports 64-bit operating systems, which are better because they permit more addressable memory space, i.e. more RAM.

MongoDB ships with two versions - 32-bit and 64-bit. Due to the way MongoDB uses [memory mapped files](http://blog.mongodb.org/post/137788967/32-bit-limitations) 32-bit builds can only store around 2G of data. For standard replicasets MongoDB only has a single process type - mongod. If you're intending on storing more than 2G of data you should use a 64-bit build of MongoDB. For setups that are sharded, you can use 32-bit builds for the mongos.

tl;dr - Just use 64-bit, or understand the limitations of 32-bit

## Document size limits
Unlike a Relational Database Management System ([RDBMS](http://en.wikipedia.org/wiki/Relational_database_management_system)) which stores data in columns and rows, MongoDB stores data in documents. These documents are [BSON](http://en.wikipedia.org/wiki/BSON), which is a binary format similar to JSON.

Like most other databases, there are limits to what you can store in a document. In older versions of MongoDB, documents were limited to 4M each. All recent versions support documents up to 16M in size. This may sound like an annoyance, but 10gen's opinion on this is that if you are hitting this limit then either your schema design is wrong or you should be using [GridFS](http://www.mongodb.org/display/DOCS/GridFS), which allows arbitrarily sized documents.

Generally, I would suggest avoiding storing large, irregularly updated objects in a database of any kind. Services such as [Amazon S3](http://aws.amazon.com/s3/) or [Rackspace Cloudfiles](http://www.rackspace.com/cloud/public/files/) are generally a much better option and don't load your infrastructure unnecessarily.

tl;dr - Keep documents under 16M each and you'll be fine!

## Write failure
MongoDB allows very fast writes and updates by default. The tradeoff is that you are not explicitly notified of failures. By default most drivers do asynchronous, 'unsafe' writes - this means that the driver does not return an error directly, similar to INSERT DELAYED with MySQL. If you want to know if something succeeded, you have to manually check for errors using [`getLastError`](http://www.mongodb.org/display/DOCS/getLastError+Command).

For cases where you want an error thrown if something goes wrong, it's simple in most drivers to enable "safe" queries which are synchronous. This makes MongoDB act in a familiar way to those migrating from a more traditional database.

If you need more performance than a 'fully safe' synchronous write, but still want some level of safety, you can ask MongoDB to wait until a journal commit has happened using [getLastError with 'j'](http://www.mongodb.org/display/DOCS/getLastError+Command#getLastErrorCommand-%7B%7Bj%7D%7D). The journal is flushed to disk every 100 milliseconds, rather than 60 seconds as with the main store.

tl;dr - use safe writes or use `getLastError` if you want to confirm writes

## Schemaless does not mean you have no Schema
RDBMSs usually have a pre-defined schema: tables with columns, each with names and a data type. If you want to add an extra column, you have to add a column to the entire table.

MongoDB does away with this. There is no enforced schema per collection or document. This makes rapid development and changes easy.

However, this doesn't mean you can ignore schema design. Having a properly designed schema will allow you to get the best performance from MongoDB. Read the [MongoDB docs](http://www.mongodb.org/display/DOCS/Schema+Design), or watch one of the many videos about schema design to get started:
- [Schema Design Basics](http://www.10gen.com/video/mongosf2011/schemabasics)
- [Schema Design at Scale](http://www.10gen.com/video/mongosf2011/schemascale)
- [Schema Design Principles and Practice](http://www.10gen.com/presentations/mongosv-2011/schema-design-principles-and-practice)

tl;dr - design a schema and make it take advantage of MongoDB's features

## Updates only update one document by default
With a traditional RDBMS, updates affect everything they match unless you use a LIMIT clause. However MongoDB uses an equivalent of a 'LIMIT 1' on each update by default. Whilst there is no way to do a 'LIMIT 5', you can remove the limit completely by doing the following;

    db.people.update({age: {$gt: 30}}, {$set: {past_it: true}}, false, true)

There are similar options in all the official drivers - the option is usually called '[multi](http://www.mongodb.org/display/DOCS/Updating#Updating-update%28%29)'.

tl;dr - specify multi true to affect multiple documents

## Case sensitive queries
Querying using strings may not quite work as expected - this is because MongoDB is by default case-sensitive.

For example, `db.people.find({name: 'Russell'})` is different to `db.people.find({name: 'russell'})`. The solution is to make sure your data is in a known case - which is the ideal solution. You can also use regex searches like `db.people.find({name: /russell/i})` although these aren't ideal as they are relatively slow.

tl;dr - queries are case sensitive

## Type sensitive fields
When you try and insert data with an incorrect data type into a traditional database, it will generally either error or cast the data to a predefined value. However with MongoDB there is no enforced schema for documents, so MongoDB can't know you are making a mistake. If you write a string, MongoDB stores it as a string. If you write an integer, it stores it as an integer.

tl;dr - make sure you use the [correct type](http://www.mongodb.org/display/DOCS/Advanced+Queries#AdvancedQueries-%24type) for your data

## Locking
When resources are shared between different parts of code sometimes locks are needed to ensure only one thing is happening at once.

Older versions of MongoDB - pre 2.0 - had a global write lock. Meaning only one write could happen at once throughout the entire server. This could result in the database getting bogged down with locking under certain loads. This was improved significantly in 2.0, and again in the current stable 2.2. MongoDB 2.2 solves this with [database level locking](https://jira.mongodb.org/browse/SERVER-4328) and is a big step forward. The next step, which I expect will be another large improvement is [Collection level locking](https://jira.mongodb.org/browse/SERVER-1240), which is planned for the next stable version.

Having said this, most applications I've seen were limited by the application itself (too few threads, badly designed) rather than MongoDB itself.

tl;dr - use a [current stable](http://www.mongodb.org/downloads) to get the best performance

## Packages

A lot of people have had issues with out of date versions of MongoDB being shipped in the standard repositories of common distributions. The solution is simple: use the offical 10gen repositories which are available for [Ubuntu and Debian](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-debian-or-ubuntu-linux/) as well as [Fedora and Centos](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-redhat-centos-or-fedora-linux/).

tl;dr - use [official packages](http://www.mongodb.org/downloads#packages) for the most up to date versions

## Using an even number of Replica Set members
Replica Sets are an easy way to add redundancy and read performance to your MongoDB cluster. Data is replicated between all the nodes and one is elected as the primary. If the primary fails, the other nodes will vote between themselves and one will be elected the new primary.

It can be tempting to run with two machines in a replica set; it's cheaper than three and is pretty standard way of doing things with RDBMSs.

However due to the way voting works with MongoDB, you must use an odd number of replica set members. If you use an even number and one node fails the rest of the set will go read-only. This happens as the remaining machines will not have enough votes to get to a quorum.

If you want to save some money, but still support failover and increased redundancy, you can use arbiters. Arbiters are a special type of replica set member - they do not store any user data (which means they can be on very small servers) but otherwise vote as normal.

tl;dr - only use an odd number of replica set members and be aware that arbiters can reduce the costs of running a redundant setup

## No joins
MongoDB does not support [joins]( http://en.wikipedia.org/wiki/Join_%28SQL%29); If you need to retrieve data from more than one collection you must do more than one query.

If you find yourself doing too many queries, you can generally redesign your schema to reduce the overall number you are doing. Documents in MongoDB can take any format, so you can de-normalize your data easily. Keeping it consistent is down to your application however.

tl;dr - no joins, [read how to design a schema](#schemaless_does_not_mean_you_have_no_schema)

## Journaling
MongoDB uses [memory mapped files](http://www.mongodb.org/display/DOCS/Caching) and flushes to disk are done every 60 seconds, which means you can lose a maximum of 60 seconds + the flush time worth of data.

To reduce the chance of losing data, MongoDB added journaling -since 2.0 it's been enabled by default. Journaling records changes to the database to disk every 100ms. If the database is shutdown unexpectedly, this will be replayed before starting up to make sure the database is in a consistent state. This is the nearest thing in MongoDB to a commit in a more traditional database.

Journaling comes with a slight performance hit - around 5%. For most people the extra safety is well worth the overhead.

tl;dr - don't [disable journaling](http://www.mongodb.org/display/DOCS/Journaling)

## No authentication by default
MongoDB doesn't have authentication by default. It's expected that `mongod` is running in a trusted network and behind a firewall. However authentication _is_ fully supported, if you require it you can [enable it really easily](http://docs.mongodb.org/manual/administration/security/).

tl;dr - secure MongoDB by using a [firewall](http://docs.mongodb.org/manual/administration/security/#networking-risk-exposure) and binding it to the correct interface, _or_ enable authentication

## Lost data with Replica Sets
Running a replica set is a great way to make your system more reliable and easier to maintain. An understanding of what happens during a node failure or failover is important.

Replica Sets work by transferring the oplog - a list of things that change in your database (updates, inserts, removes, etc) - and then replaying it on other members in the set. If your primary fails and later comes back online, it will rollback to the last common point in the oplog. At any point during this process, newer data that may have existed will be removed from the database and placed in a special folder in your data directory called 'rollback' for you to manually restore. If you don't know about this feature, you may find that data goes missing. Each time you have a failover you should check this folder. Manually restoring the data is really easy with the standard tools that ship with MongoDB.

There is a far more [detailed explanation in the official docs](http://www.mongodb.org/display/DOCS/Replica+Sets+-+Rollbacks).

tl;dr - 'missing data' after a failover will be in the rollback directory

## Sharding too late
Sharding is a way of splitting data across multiple machines. This is usually done to increase performance when you find a replica set is too slow. MongoDB supports automatic sharding.

MongoDB allows migrating to a sharded setup with very little effort. However if you leave it too late it can cause headaches. To shard, MongoDB splits the collections you choose into chunks based on a 'shard key' and distributes them amongst your shards automatically. Splitting them and migrating chunks takes time and resources, and if your servers are already near capacity could result in them slowing to a standstill right when you need them most.

The solution is simple; use a tool to keep an eye on MongoDB, make a best guess of your capacity (flush time, queue lengths, lock percentages and faults are good gauges) and shard before you get to 80% of your estimated capacity. Exampe tools include [MMS](https://mms.10gen.com/user/login), [Munin](http://munin-monitoring.org/) (+ [Mongo plugin](https://github.com/erh/mongo-munin)), [CloudWatch](http://aws.amazon.com/cloudwatch/).

If you know you are going to have to shard from the outset, a nice alternative if you're using AWS or similar is to start off sharded - but on smaller servers. Stopping and resizing machines is much quicker than migrating thousands of chunks.

tl;dr - [shard](http://docs.mongodb.org/manual/core/sharding/) early to avoid any issues

## You cannot update a shard key in a document
For sharded setups, shard keys are what MongoDB uses to work out which shard a particular document should be on.

After you've inserted a document you cannot update the shard key. The suggested solution to this is to [remove the document and reinsert it](https://jira.mongodb.org/browse/SERVER-1893) - which will allow it to be allocated to the correct shard.

tl;dr - you can't update a shard key

## You cannot shard a collection over 256G
Going back to leaving things too late - MongoDB won't allow you to shard a collection after it has grown bigger than 256G. This used to be [a lot lower](https://jira.mongodb.org/browse/SERVER-2271). This will eventually be removed completely. There is no solution other than recompiling or avoiding trying to shard collections larger than this.

tl;dr - shard collections before you reach 256G

## Unique indexes and sharing
Uniqueness is not enforced across shards as the enforcement is done per-shard and not globally, except for the shard key itself.

tl;dr - Read [this](http://docs.mongodb.org/manual/tutorial/enforce-unique-keys-for-sharded-collections/)

## Choosing the wrong shard key
MongoDB requires you to choose a key to shard your data on. If you choose the wrong one, it's not a fun process to correct. What is the wrong shard key depends on your application - but a common example would be using a timestamp for a news feed. This causes one shard to end up 'hot' by having data constantly inserted into it, migrated off and queried too.

The common process for altering the shard key is simple; [dump and restore the collection](http://www.mongodb.org/display/DOCS/Changing+a+Shard+Key).

MongoDB will support hashing a key for you in the next release (see [SERVER-2001](https://jira.mongodb.org/browse/SERVER-2001)) which will make our lives easier if you need to shard on a certain key that happens to be sequential.

tl;dr - [read this before you choose a shard key](http://www.mongodb.org/display/DOCS/Choosing+a+Shard+Key)

## Traffic to and from MongoDB is unencrypted
The connections to MongoDB by default aren't encrypted, which means that your data could be logged and used by a third party. If you're running MongoDB in your own non-public network, this is unlikely to happen.

However, if you're accessing MongoDB over a public connection you may want to encrypt the traffic. The public downloads and distributions of MongoDB do not have SSL support enabled; luckily it's quite easy to compile your own version. Subscribers to 10gen support get this enabled in their build by default. Also, luckily most of the official drivers support SSL out of the box so there should be little hassle there too. Checkout the [docs here](http://docs.mongodb.org/manual/administration/ssl/).

tl;dr - if you connect publicly, be aware stuff is unencrypted

## Transactions

MongoDB only supports single document atomicity, rather than a traditional database such as MySQL which allows [longer sequences of changes to either completely succeed or fail](http://en.wikipedia.org/wiki/Database_transaction). This means it can be hard with out being creative to model things which have shared state across multiple collections. One way of getting around this is by [implementing two phase commits](http://docs.mongodb.org/manual/tutorial/perform-two-phase-commits/) in your application...however this is not for everyone; it may be better to use more than one datastore if this is required.

tl;dr - there is no built in support for transactions over multiple documents

## Journal allocation times

MongoDB may report to be ready to work, but in fact be allocating the journal still. If you have machines provisioned automatically, as well as a slow filesystem or disks, this may be an annoyance. Normally this won't be an issue - but if it is, you can use the [undocumented flag --nopreallocj](https://github.com/mongodb/mongo/blob/e6b9b76748d8051ef6616f7fc0a8e0cfbf38d0d1/src/mongo/db/db.cpp#L721) to disable pre-allocation.

tl;dr - if you have slow disks or certian file systems, journal allocation may be slow

## NUMA + Linux + MongoDB

"Linux, NUMA and MongoDB tend not to work well together. If you are running MongoDB on numa hardware, we recommend turning it off (running with an interleave memory policy). Problems will manifest in strange ways, such as massive slow downs for periods of time or high system cpu time."

tl;dr - [Disable NUMA](http://www.mongodb.org/display/DOCS/NUMA)

## Process Limits in Linux

If you experience segfaults under load with MongoDB, you may find it's beacuse of low or default open files / process limits. 10gen [recommend setting your limits to 4k+](http://www.mongodb.org/display/DOCS/Production+Notes#ProductionNotes-GeneralUnixNotes), however this may need to be varied depending on your setup. Read up on [ulimit](http://www.linuxhowtos.org/Tips%20and%20Tricks/ulimit.htm) and its meaning here.

tl;dr - Permanently increase hard and soft limits for open files / user processes for Mongo on Linux

## Thanks to

[Alvin Richards](https://twitter.com/jonnyeight), [mason55](http://news.ycombinator.com/item?id=4745157), [Pewpewarrows](http://news.ycombinator.com/item?id=4745191), [T-R](http://news.ycombinator.com/item?id=4745277), [rgarcia](http://news.ycombinator.com/item?id=4745282), [stbrody](http://news.ycombinator.com/item?id=4745344), [23david](https://news.ycombinator.com/item?id=4745720) and [timdoug](http://news.ycombinator.com/item?id=4745219)