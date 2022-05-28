# Imports and exports

- Connect to the cluster: pass the uri connection string : `--uri mongodb+srv://user:password@clusterURI.mongodb.net/database`
- `mongodump`: exports data in BSON format: `mongodump --uri mongodb+srv://user:password@clusterURI.mongodb.net/database`
- `mongoexport` : exports the data of a given collection in json format. By defaults print to stdout. We can specify a file to save it: 
`mongoexport --uri mongodb+srv://user:password@clusterURI.mongodb.net/database --collection=mycollection --out=myfile.json`
- `mongorestore`: imports data in BSON dump: `mongorestore --uri <..> --drop dump`: note using drop allows to drop the collection and replace with the dump
- `mongoimport`: imports data in json: `mongoimport --uri <..> --drop <filename>.json --collection <mycollection>.json`: if collection is not specified, uses the collection with the same name as filename

# Basic queries
## connect to mongo
- connect with `mongo "mongodb+srv://user:password@clusterURI.mongodb.net/admin"`: by default admin will contain infos about users:
## basic views in the mongo shell
- `show dbs`: shows the list of databases with their size. In addition to what i see on the atlas cloud, I also see a `local` and an `admin` table
- `use dbname` to indicate we work with a specific db. You can verify the current db with `db.getName()`
- to view the collections in this db: `show collections`
## get queries
- simple query: `db.<collection_name>.find({key:"val"})`: will load 20 docs and then we can type `it` to iterates through the cursor (a pointer [= direct address to memory location] to a result set of a query)
- count results: `db.<collection_name>.find(<query>).count()`
- pretty results: `db.<collection_name>.find(<query>).pretty()`
- pass an empty query if you want to iterate through everything
- get a random doc: `db.<collection_name>.findOne()`

## insert
- insert a doc: `db.<collection_name>.insert(<doc>)`: pass an array to insert several
NOTE: `db.<collection>.insert([{"_id":1, "test":1},{"_id":1, "test":1},{"_id":3,"test":1}])` will produce a write error when inserting the second doc and the third doc will not be inserted. To insert all docs whose unique id is not in the db: `db.<collection>.insert([{"_id":1, "test":1},{"_id":1, "test":1},{"_id":3,"test":1}], {"ordered":false})`

## update

- update many docs: `db.<collection>.updateMany(<query>, {"<op>":{<field1>:<value1>, <field2>, <value2>}})`. Example: `db.zips.updateMany({"city":"HUDSON"}, {"$inc":{"pop":10}})`. List of operators:
    - https://www.mongodb.com/docs/manual/reference/operator/update/#id1

- update one doc: should be used when matching the `_id` field because we are sure there is only one match: `db.zips.updateOne({"_id": <id>}, <updateOperation>)`

- all update operations:
    - `{"$currentDate":{<field>:true}}` same as  `{"$currentDate":{<field>:{"$type":"date}}}`: stores current date as Date
    - `{"$currentDate":{<field>:{"$type":"timestamp"}}}`: : stores current date as timestamp
    - `{"$inc":{<field>:<value>}}`: increases field by value. WIll fail if field is not a number
    - `{"$min":{<field>:<value>}}`: updates filed if value is less than existing field value. NOTE: if field is not nb, will set the field as value
    - `{"$max":{<field>:<value>}}`: updates filed if value is more than existing field value. NOTE: if field is not nb, will set the field as value
    - `{"$mul":{<field>:<value>}}`: multiplies field by value. WIll fail if field is not a number
    - `{"$rename":{<field>:<newname>}}`: renames field
    - `{"$set":{<field>:<value>}}`: sets the value of a field
    - `{"$setOnInsert":{<field>:<value>}}, {upsert:true}`: sets the value of a field only if the operation resulted in an insert
    - `{"$unset":{<field>:<value>}}`: unsets a field. As what we pass as the value does not impact the output, we can pass "" in the value

## delete
- `db.<collection>.deleteOne(<query>)`
- `db.<collection>.deleteMany(<query>)`
- drop a collection from the mongo shell: `db.collection.drop()`