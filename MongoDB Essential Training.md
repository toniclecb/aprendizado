# MongoDB Essential Training
Naomi Pentrel

released 2009 -> 1.0
open source

2022 -> 5.0

## Self-hosted vs. managed MongoDB

responsibilities:
self: maintaining hardware, security, monitoring, backups, versions, etc.
MongoDB Agent

Managed: IBM Cloud, ScaleGrid, DigitalOcean, MongoDB Atlas

## Enterprise vs. Community

MongoDB Enterprise offers advanced security options, additional storage engines, management tools, the MongoDB Enterprise operator for Kubernetes, and the MongoDB Connector for BI

## Install on Windows

Using npm, we installed mondodb database,
in powershell we installed mongo shell and mongodb data tools (sudo apt-get install mongodb-database-tools)

> mongod --version

> mongosh --version

> montoimport

## Mongod

A mongod is a daemon process for MongoDB. You could say it is the core unit of MongoDB. It handles data requests from the MongoDB shell or the MongoDB drivers, manages data access, and performs background management operations. Let's look at it in the shell.

To get a MongoDB database up and running locally, all you need to do is run the command mongod and specify a directory for storage. You do that by parsing the command line argument dbpath and then specifying a folder for where the data can be stored.

> mongod --dbpath mongod_only

### Default settings:

port: 27017
data storage: /data/db or C:\data\db*

## Replica set

When multiple Mongo D instances work together and maintain the same data set they are called a replica set. 
provide: redundancy and high availability
Let's say you configured your MongoDB deployment to be a three member replica set with three normal replica set members. In a replica set, the Mongo D processes take on different roles. One replica set member in each replica set is elected to be the primary. The primary receives all right operations. The rest of the replica set members are secondaries. Secondaries replicate right operations from the primary asynchronously to maintain exactly the same data set. This is how MongoDB uses replication to achieve redundancy, meaning the data is replicated in multiple places.

This is how MongoDB achieves high availability

#### Set up a replica set from the command line

For our production environment, it is recommended to use configuration files

> mkdir replica_set_cmdline
> cd replica_set_cmdline
create a key file:
> openssl rand -base64 755 > keyfile
adding permissions:
> chmod 400 keyfile

And now you can run MongoDB without enabling authentication, but it is fairly easy to enable key file authentication which then provides a minimum of security between members of the replica set. For production environments, it is recommended you use X.509 certificate instead.
Now we're going to lock down permissions on this file, so only the current user has read permissions on the key file.

we're going to create some sub-directories for each mongod process to store their individual set of data files and log files.
creating 3 folders:
> mkdir -p {1,2,3}/db

let's start the mongod processes
we're going to specify a replica set. We'll do that by using the command line option replSet

> mongod --replSet myReplSet --dbpath ./m1/db --logpath ./m1/mongodb.log --port 27017 --fork --keyFile ./keyfile

So now we're going to run this entire command two more times. But we are going to have to change some of these arguments. folders and port!

we'll need to initiate a replica set and add the instances to it. To do that, we'll have to connect to one of the instances, and we'll do that with the MongoDB Shell.

> mongosh
  > rs.initiate()
  use admin
This changes the database we are using to use the admin database which contains configuration options for your database

Using this configuration database, we can now create our first user. And we are able to do that because of something called the localhost exception which allows you to enable access control and create the first user or role in the system. After that, you have to authenticate as a user with privileges to create other users in order to create more users.

  > db.createUser({user: 'naomi', pwd: passwordPrompt(), roles: ["root"]})
  
  > db.getSiblingDB("admin").auth("naomi", passwordPrompt())

Now the next thing that we're going to do is we're going to add the other two replica set members.
  > rs.add("localhost:27018")
  > rs.add("localhost:27019")

The next thing I'm going do is I'm going check the status of the replica set.
  > rs.status()
or
  > db.serverStatus()["repl"]

So here we can also see all of the host names, and that means those are all of the replica set members that we have in this replica set.

#### Set up a replica set with config files

Create a keyfile and change permissions!

> touch m1.conf

```
storage:
	dbPath: m1/db
net:
	bindIp: localhost
	port: 27017
security:
	authorization: enabled
	keyFile: keyfile
systemLog:
	destination: file
	path m1/mongod.log
processManagement:
	fork: true
replication:
	replSetName: mongodb-essentials-rs
```

Create two more copies of this file above
change paths, port

Now that we have created all the configuration files, one thing that remains to do is to start the mongod processes. And the way you start a mongod from a configuration file is by specifying the dash f option and then specifying the configuration file.

> mongod -f m1.conf
Do the same with others files.

Now we have three individual mongod processes. To make them work together, we're going to need to initiate a replica set and add the instances to it.

> mongosh
  > use admin
  > config = {_id: "mongodb-essentials-rs", members: [{_id: 0, host: "localhost:27017"}, {_id: 1, host: "localhost:27018"}, {_id: 2, host: "localhost:27019"},]}
  > rs.initiate(config)

The next thing we are going to do is we're going to create our first user

  > db.createUser({user: 'naomi', pwd: passwordPrompt(), roles: ["root"]})
    
  > db.getSiblingDB("admin").auth("naomi", passwordPrompt())

Now we can check the status of the replica set if we want to. To do that, we can issue the command rs dot status. The rs dot status command reports on health of replica set members.

  > rs.status()

There is one more way of obtaining information about a replica set, and that is to run db dot serverStatus and then get the repl field value from it, like this.

  > db.serverStatus()["repl"]
  
#### Import the sample data

When you installed MongoDB earlier, you also installed the MongoDB database tools. They include **mongostat** to get statistics on a running mongod, **mongodump** which allows you to export dump files from MongoDB collections in BSON, **mongorestore** which allows you to import dump files from MongoDB collections that were stored in BSON, **mongoexport** which allows you to export collection data to JSON or BSON. And the one we are looking for, **mongoimport**, which allows you to import collection data from JSON or CSV files.

So because we have created a user on our database deployment, we're going to have to specify our username.

> mongoimport --username="naomi" --authenticationDatabase="admin" --db=sample_data inventory.json
> mongoimport --username="naomi" --authenticationDatabase="admin" --db=sample_data movies.json

#### Debug

If your database is failing to start, first, check the log files at the location that we specified, in our case m1/mongod.log, et cetera.

The next thing that you can do is to check the oplog. The way you do that is with this command.
So that is db.oplog.rs.find. 

The next thing that you can do is you can try to increase the log level. The way you would do that is you would, first of all check what the current log level is. You do that with getlLogComponents

you would then run a command on the admin database, which you can do also by saying db.adminCommand


## The document model

One of the main features that differentiates MongoDB from many of the databases is that it is built to natively work with JSON documents.

MongoDB actually uses a binary encoded serialization of JSON-like documents called BSON for storing data as well as for their drivers and tools. BSON was designed to be especially lightweight, traversable, and efficient. BSON also brings support for additional types such as images, timestamps, and longs, and much more.

## Namespaces, collections, documents

From a technical point of view, this represents a MongoDB deployment with one replica set with three MongoDBs. 

you have one database deployment and inside your deployment, you can then have one or more databases.

A collection is a grouping of MongoDB documents.

Inside each collection are documents. Documents form the basic unit of data in MongoDB and each document contains one individual record.

Be aware that there is a maximum size for each document, which is 16 megabytes.

The easiest way to create a collection, is to just insert a document into the collection as though it already existed. So we're going to do just that.

> use blog
> show collections
[empty]
> db.authors.insertOne({"name":"Naomi Pentrel"})

If you don't assign one, which in this case we didn't do, MongoDB will automatically create one.

So the way we did that was we said db.authors to specify the namespace

## MongoDB Query Language

how to access that data. That's what you use the MongoDB query language for. Be aware that you may also find references to MQL as the MongoDB query API. MQL allows you to perform, create, read, update, and delete operations, also called CRUD.

There are MongoDB drivers available for almost all programming languages out there

commands:
insertOne, insertMany
findOne, find
updateOne, updateMany
deleteOne, deleteMany

## Indexes

An index provides an organized way to look up data by storing a subset of the data with pointers to the location of the full records. If the data you are querying is present in the subset of data in an index, the database only needs to check the index. 

Queries that can be answered completely with just the index are called covered queries in MongoDB. And these queries are very performant.

 Indexes are very useful and you should generally create an index when you frequently query on the same fields.
 
 However, indexes come at a price. Indexes need to be maintained by the database, which adds about 10% write overhead to your write operations. So having indexes is a trade-off. You can get faster data access but any writes take a little bit longer.
 
 In MongoDB, there are many types of different indexes, single field indexes, which create an index on just one field, partial indexes, where you can add an option to the index to tell the database to only match documents that have a value satisfying a certain condition for the index, compound indexes, which create an index on a combination of fields.
 
 There are also multi-key indexes. That's an index on up to one array value. It can't be more than one array value because the size of the index would otherwise grow really quickly. There are also text indexes which allow you to search within text fields. And there are wild card indexes. This is a special kind of index that is indexed on a field or set of fields for which you don't know the name because the schema changes dynamically. It is useful for very diverse data but it should not be used for any other cases.
 
 There are also geometric indexes, which are, for example, 2D sphere indexes, 2D or geoHaystack indexes. And lastly, there are hashed indexes. Hashed indexes can reduce the index size if original values are very large but they are not performant for ranged queries.
 
 method:
 createIndex({name: 1})
 
 ## Durability
 
 When inserting data into a database, a concept that you should be aware of is durability. Durability is a property that guarantees that acknowledged writes are permanently stored in the database, even if the database or parts thereof become temporarily unavailable. In MongoDB, you can configure the level of durability. You can choose to have high durability, but then writes will be slower, or you can choose to have low durability, but faster writes. The way you configured durability in MongoDB is by specifying a writeConcern.
 
 You can specify the writeConcern as the second argument in the insertOne or the insertMany command.
 

## Consistency and availability

For find one and find many, you may need to know about the read concern for consistency and availability guarantees.

it may be important to only see data that is majority committed as compared to data that is written to the primary but has not yet propagated to a majority of nodes. You can configure the read behavior by specifying the read concern.
and then just adding at the end .readConcern and then specifying the read concern majority.

- It can be local, which is the default and returns the most recent data available on the MongoD
- available, which is the same as local but used for sharded clusters
- majority, 
- linearizable which returns majority committed data only but waits for any concurrent writes to complete before reading and returning the data.

There is also the option of specifying a read preference if you are using a MongoDB driver, which allows you to then read data from secondaries if you want to. This can allow you to get faster reads. The options are **primary**, which is the default. So then all of the reads come directly from the primary. Primary preferred which allows reads to be routed to secondaries but the primary is still the preferred option. **Secondary**, which means reads are routed directly to secondaries. Secondary preferred which means they will only go to a primary in certain circumstances. And then **nearest** which is then the node that has the lowest latency to where you are querying from.

What does {w: 1, j: true} mean?
One replica set member must write to its journal before the write is acknowledged as having succeeded. If the replica set member temporarily becomes unavailable before the write can be replicated, the write could be rolled back.


## readConcern and readPreference

Specifying a readConcern allows you to define the level of consistency you require when reading data. The default is local, and you can change it to available, majority, or linearizable.

Specifying a readPreference allows you to read data from secondaries. The default is primary, and you can change it to primaryPreferred, secondary, secondaryPreferred, and nearest.

## Comparasion Operators

$eq
=
$gt
>
$gte
>=
$lt
<
$lte
<=
$ne
<>
$in
IN
$nin
NOT IN

> db.inventory.findOne({"variations.quantity": {$gte: 22}})

## Logical operators

$and
$or
$nor
$not

## Sort, skip, limit

find().sort({title: 1})
find().sort({title: -1})

**skip method**. If you want the results on a sorted order, but you want to skip the first 10 or however many results, you can do that. The way you do that is, you just add the skip method at the end, and then you pass on a number of how many documents you want to skip.

**limit**: So on top of sorting and skipping, you can also limit the results that you get. And to that, you just add the limit method at the end.

In what order are the sort, skip, and limit operations performed by MongoDB?
sort, skip, limit

## Update Operators

$inc
$mul
$max
$min

$push
$addToSet
$pop
$all

**$set and $unset**

the $unset operator allows you to remove a field. You still have to specify the field and you still have to specify a field value.

## Transactions

In MongoDB, an operation on a single document is atomic. If person A issues a command to change some values in a document and person B issues a command to read the same document, there are two things that can happen. Either the read will happen before the document has changed or the read will happen after the document is changed. You will not receive the document with partial changes.

> session = db.getMongo().startSession({readPreference: {mode: "Primary"}})
> session.startTransaction()
> session.getDataBase("blog").authors.updateMany({}, {$set: message: "transaction occured"})
> session.commitTransaction()
> session.endSession()

You should only use transactions when you really need them. Overuse of transactions can lead to performance degradation.

## $expr

there is one call operator that can do things like comparing different document values. That's the $expr operator

db.movies.find({$expr: {$gt: [{$multiply: ["$ratings.mndb", 10]}, "$ratings.soft_avocados"]}})

Now, we got back all of the documents where the MNDB value, once it was scaled, because it was multiplied by 10, was higher than the soft avocados rating. So that means on this last document here, the MNDB value is four, four times 10, 40, 40 is bigger than 17, so that's why this is in the results sets. So this is one of the example use cases where $expr can come in handy

## Aggregation pipelines

MongoDB's aggregation framework allows you to manipulate and transform data in ways that are not possible with the normal queries that we've discussed so far. If you are familiar with Unix pipes, you can think of aggregation pipelines in a similar way.

### $group

Grouping data is a common pattern we need when accessing or manipulating data. Let's look at this in the terminal. In MongoDB, you can group data by using the $group stage in an aggregation pipeline.

group for count:
> db.inventory.aggregate([{ $group: { _id: "$source", count: { $sum: 1}}}])

### Othres aggregations

$bucket: the dollar bucket stage allows you to put documents into groups, but instead of grouping them by one value that has to be the same for all documents, you define bucket boundaries and ranges for a document value.

> db.inventory.aggregate([(Sbucket: [groupBy: "Syear', boundaries: [1980, 1990, 200 0, 2010, 2020], default: "Other" > }])

$unwind: If you have a document with an array you can use the $unwind stage to create one output document for each array element.

$out: allows you to store the results of an aggregation pipeline in a new collection. This is awesome if you often run the same aggregation because you can store it once and then query the stored results instead.

$merge: command also allows you to merge results into an existing collection.

$function: which you can program to be whatever you need it to be. allows you to write JavaScript functions that operate on the field values in your documents.

$lookup stage allows you to perform joins. That is to pull in data from another collection based on a matching field.

there are two things that are very important to consider when you are using joins. The first thing is that you absolutely need to have an index on the foreignField if you're going to perform this as a common query. Otherwise, the performance of these queries will not be good because the lookup query will have to perform a collection scan for each document, meaning that it needs to, for each order document, check the entire inventory collection to check every single document whether it is a match.

The second thing that you should know with $lookup is that your common query patterns should very rarely require joins. So if you find yourself using a lot of lookups, this is a telltale sign that you are very likely not structuring your data well for MongoDB. MongoDB is not optimized to perform a lot of joins.

## Performace

we are calling the **explain** method and we are passing it a string that says executionStats. That will tell MongoDB that we are not actually interested in the results of the aggregation pipeline 

When you execute an aggregation pipeline the MongoDB profiler comes up with multiple plans for how to execute this query using different indexes if multiple indexes are available.

Since we passed in the executionStats argument to the explain method, we now get all of this information about how the profiler went about this and which query plan won

You can enable a **Native Profiler** to collect data for operations slower than a specified limit, by setting the profiling level to one and specifying a time limit.

use **$project** as the final stage. Contrary to what you may hear somewhere else or read, it is not better to project early in your pipeline. You will often hear that projecting away fields you don't need as early as possible will make your pipelines perform better, but this is an anti-pattern that may result in the database not being able to use indexes.

if you run aggregation pipelines and you notice directly that your deployment is slowing down, you may have to kill that operation. To do that, you use db.currrentOp(true), to figure out which operations are currently running, and that will also return an operation number for each operation. Then you can use the db.adminCommand to perform a kill operation for that running operation. Use the killOp command.

## Relational vs. document

In relational databases, you generally store JSON inside a JSON blob. In MongoDB, you just store it.

Relational databases are optimized for working with tables and performing joins between tables fast. You could store data in tables in MongoDB as well. And you can also perform joins with MongoDB using the $lookup aggregation state, which is reasonably fast if you use indexes. However, MongoDB is not optimized for this data access pattern.

Different people have different opinions on which way of querying the data is simpler. Many folks, used to querying data with SQL, don't mind. I am firmly in the camp that querying data with MongoDB is easier. So it depends. What most people don't realize is you can actually query MongoDB with SQL. There is a BI connector for that. So if you prefer the MongoDB way of querying, go with MongoDB.

If you scale both databases, joins can become a problem because the joins for your data will only perform fast if all the tables you need to access to perform a query are co-located. And there is a limit to how big a single machine can get. The fact MongoDB is natively built around documents that contain all the data that is generally required when you perform a query, makes it easier and more performant to scale horizontally.

## Data modeling

The main principle to keep in mind when modeling data is that data commonly queried together should live as closely together as possible. That means, in the same document or collection. While pulling in data from another collection is possible, it is not the most performant way to use MongoDB. 

The first one is if there is a one-to-one relationship between a document and another piece of data, like with a document about a person and a date of birth, store them together.

The second guideline is if there is a one-to-many relationship, like between an author and something like books. Consider the amount. If it is one to few, like with authors and books, you can store both in one collection using nesting or an array. If it's more than a few, it's likely best to store them in separate collections and link them with IDs.

it's a many-to-many relationship, avoid embedding this information. Instead, store IDs in either or both collections.

Keep arrays reasonably small, meaning less than 100 elements

## Flexible schema

MongoDB does not require you to define a schema for you to store data. This makes it easy to iterate and change your schema. And that is something that comes in handy when you are building a new application, and changing things a lot. However, your documents should still have a common structure they adhere to.

MongoDB is schemaless and does not enforce any structure on your data, aside from requiring that each object must have an _id field. If no _id field is provided, MongoDB will add one of you. However, MongoDB supports schema validation and you can enable it for a collection at any point.










