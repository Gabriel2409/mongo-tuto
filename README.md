# Imports and exports

- Connect to the cluster: pass the uri connection string : `--uri mongodb+srv://user:password@clusterURI.mongodb.net/database`
- `mongodump`: exports data in BSON format: `mongodump --uri mongodb+srv://user:password@clusterURI.mongodb.net/database`
- `mongoexport` : exports the data of a given collection in json format. By defaults print to stdout. We can specify a file to save it: 
`mongoexport --uri mongodb+srv://user:password@clusterURI.mongodb.net/database --collection=mycollection --out=myfile.json`
- `mongorestore`: imports data in BSON dump: `mongorestore --uri <..> --drop dump`: note using drop allows to drop the collection and replace with the dump
- `mongoimport`: imports data in json: `mongoimport --uri <..> --drop <filename>.json --collection <mycollection>.json`: if collection is not specified, uses the collection with the same name as filename

# MQL: Mongodb Query Language
## Basic queries
### connect to mongo
- connect with `mongo "mongodb+srv://user:password@clusterURI.mongodb.net/admin"`: by default admin will contain infos about users:
### basic views in the mongo shell
- `show dbs`: shows the list of databases with their size. In addition to what i see on the atlas cloud, I also see a `local` and an `admin` table
- `use dbname` to indicate we work with a specific db. You can verify the current db with `db.getName()`
- to view the collections in this db: `show collections`
### get queries
- simple query: `db.<collection_name>.find({key:"val"})`: will load 20 docs and then we can type `it` to iterates through the cursor (a pointer [= direct address to memory location] to a result set of a query)
- count results: `db.<collection_name>.find(<query>).count()`
- pretty results: `db.<collection_name>.find(<query>).pretty()`
- pass an empty query if you want to iterate through everything
- get a random doc: `db.<collection_name>.findOne()`

### insert
- insert a doc: `db.<collection_name>.insert(<doc>)`: pass an array to insert several
NOTE: `db.<collection>.insert([{"_id":1, "test":1},{"_id":1, "test":1},{"_id":3,"test":1}])` will produce a write error when inserting the second doc and the third doc will not be inserted. To insert all docs whose unique id is not in the db: `db.<collection>.insert([{"_id":1, "test":1},{"_id":1, "test":1},{"_id":3,"test":1}], {"ordered":false})`

### update

- update many docs: `db.<collection>.updateMany(<query>, {"<op>":{<field1>:<value1>, <field2>, <value2>}})`. Example: `db.zips.updateMany({"city":"HUDSON"}, {"$inc":{"pop":10}})`. List of operators:
    - https://www.mongodb.com/docs/manual/reference/operator/update/#id1

- update one doc: should be used when matching the `_id` field because we are sure there is only one match: `db.zips.updateOne({"_id": <id>}, <updateOperation>)`

- all update operations:
    - `{"$currentDate":{<field>:true}}` same as  `{"$currentDate":{<field>:{"$type":"date}}}`: stores current date as Date
    - `{"$currentDate":{<field>:{"$type":"timestamp"}}}`: : stores current date as timestamp
    - `{"$inc":{<field>:<value>}}`: increases field by value. Will fail if field is not a number
    - `{"$min":{<field>:<value>}}`: updates filed if value is less than existing field value. NOTE: if field is not nb, will set the field as value
    - `{"$max":{<field>:<value>}}`: updates filed if value is more than existing field value. NOTE: if field is not nb, will set the field as value
    - `{"$mul":{<field>:<value>}}`: multiplies field by value. Will fail if field is not a number
    - `{"$rename":{<field>:<newname>}}`: renames field
    - `{"$set":{<field>:<value>}}`: sets the value of a field
    - `{"$setOnInsert":{<field>:<value>}}, {upsert:true}`: sets the value of a field only if the operation resulted in an insert
    - `{"$unset":{<field>:<value>}}`: unsets a field. As what we pass as the value does not impact the output, we can pass "" in the value

### delete
- `db.<collection>.deleteOne(<query>)`
- `db.<collection>.deleteMany(<query>)`
- drop a collection from the mongo shell: `db.collection.drop()`

## Advanced queries

### comparison operators:
Used inside a query
- `{<field>:{<operator>:<value}`
- `<op>` can be: 
    - `"$eq"`: equal
    - `"$ne"`: not equal
    - `"$gt"`: greater than
    - `"$gte"`: greater than or equal
    - `"$lt"`: lower than
    - `"$lte"`: lower than or equal
- `{<field>:<value}` is equivalent to `{<field>:{"$eq":<value>}}`

### logic operators:
Combines multiple queries
- `{<op>:[{<statement1>}, {<statement2>}, ...]}`
    - `$and`: matches all the specified query clauses
    - `$or`: matches at least one specified query clauses
    - `$nor`: negates or: returns docs that fail to match all queries

Not operator: inverts the condition, can not be used as a top operator 
- `$not`: returns docs that dont match the query: `{<field>:{"$not":<operator-exp>}}`. Ex: `db.inspections.find({"date":{"$not":{"$eq":"Feb 21 2015"}}})`

implicit AND: 
- `{<statement1>, <statement2>, ...}` is the same as`{"$and":[{<statement1>}, {<statement2>}, ...]}`
- when querying on the same field: `{"population": {"$gt":15, "$lt":100}}` is the same as `{"$and":[{"population":{"$gt":15}},{"population":{"$lt":100}}]}`
- be careful with implicit AND : `db.testdb.find({"$or":[{"b":"I"},{"b":"II"}], "$or":[{"a":1},{"a":3}]})`: here, `$or` is declared twice in the same object so only the second `$or` will be considered => correct way to do it: `db.testdb.find({$and:[{"$or":[{"b":"I"},{"b":"II"}]}, {"$or":[{"a":1},{"a":3}]}]})`

## Expressive $expr
Allows the use of aggregation expressions withing the query language
- For ex: comparison of fields inside the same doc: `db.collection.find({$expr:{$eq:["$<field1>", "$<field2>"]}})`

NOTE: `db.trips.find({"tripduration":{$gt:1200}, $expr:{$eq:["$start station id", "$end station id"]}}).count()` is the same as `db.trips.find({$expr:{$and:[{$gt:["$tripduration", 1200]},{$eq:["$start station id", "$end station id"]}]}}).count()`

## Array Operators
- `$push`: adds element to array or turns a field into an array field if it was previously of a different type
- when doing a find query on an array field, such as `db.listingsAndReviews.findOne({"amenities":"Shampoo"})`: the query will return all the docs where Shampoo is in the amenities array.
- Now if the query is `{"amenities":["Shampoo", "TV"]})`, we will get docs where the array field is exactly `["Shampoo", "TV"]` in that order.
- `$all`: we get all elements where array field contains at least the given elements (order is not important):  `{"amenities":{$all:["Shampoo", "TV"]}})`
- `$size`: filter by array size: `{"amenities":{$size:2, $all:["Shampoo", "TV"]}})`

## projections:
- after a query: pass another object such as `{bedrooms:1, amenities:1,  _id:0}` to only show the fields we want to get: for ex: `db.listingsAndReviews.find({"amenities":{$all:["Free parking on premises", "Air conditioning", "Wifi"]}},{bedrooms:1, amenities:1,  _id:0})`
- array field projections: here, scores array contains subdocuments: `db.grades.find({"class_id":431},{scores:{$elemMatch:{score:{$gt:85}}}})` will only show the scores field if one of the elem of the array has a score above 85. Then it will only show the relevant elements
- if `$elemMatch` is used in the query, the result is the full doc where at least one element of the array matches the condition

## query subdocuments
- dot notation: `db.collection.find({"<field1>.<attr1>":<value>})`
- for array: .0, .1...

## Aggregation framework
Everything we do using the query language can also be done using the aggregation framework

For ex:
- standard MQL : `db.listingsAndReviews.find({amenities:"Wifi"},{_id:0, address:1, price:1}).pretty()`
- with aggregation framework: `db.listingsAndReviews.aggregate([{$match:{amenities:"Wifi"}}, {$project:{_id:0, address:1, price:1}}]).pretty()`

Aggregation framework allows us to stack operations on data. It can allow us to create full pipelines. Example operators:
- `$group`: takes incoming data stream and siphons it into multiple distinct reservoirs. Example: group by countries: `db.listingsAndReviews.aggregate([{$project:{_id:0, address:1}},{$group:{_id:"$address.country", count:{$sum:1}}}])`

## Sort and limit

- `db.zips.find({},{pop:1}).sort({pop:-1}).limit(2)` : limits to 2 results, sort by pop in descending order. Note: sort arg can be 1 (ascending) or -1 (descending)
- Multi sort: `<...>.sort({pop:1, "loc.y":-1})`
- can be used in aggregation framework: `db.listingsAndReviews.aggregate([{$project:{_id:0, room_type:1}}, {$group:{_id:"$room_type", count:{$sum:1}}}, {$sort:{count:-1}}, {$limit:2}])`

Note: when using sort and limit, regardless of their order, mongodb will always sort first

## Introduction to indexes
- Index makes a query more efficient
- In a database, an index is a special data structure that stores a small portion of the collection's data set in an easy to traverse form
- `db.trips.createIndex({"birth year":1})`
- compound index: `db.trips.createIndex({"start station id":1, "birth year":0})`
- See all indexes: `db.trips.getIndexes()`
- drop an index: `db.trips.dropIndex(<name>)`
- Note: if we pass -1 instead of 1, the index is created in descending order (note that it does not really matter for single field index)

## Introduction to data modelling
- Data modeling is a way to organize fields in a document to support your application performance and queryting capabilities
- Data should be stored in the way that it is used: everything that is queried together should be stored together

## upsert (update or insert)
- upsert is a hybrid command between update and insert and should only be used when needed: `db.collection.updateOne({<query>},{<update>}, {"upsert":true})` => if there is a query match, it will update it. If not it will insert a new document
