## Common Commands



### Mongo

+ Login as user to a database: `mongo --host localhost -u admin -p passsword -authenticationDatabase db`
+ Show users of one particular database: 
    + `use my_db;`
    + `show users;`

### Database
+ There is no CREATE DATABASE statement in MongoDB like there is in SQL. To create a database in MongoDB, simply switch to a non-existent database, then insert data into it: `use homstead`

### User

+ Create an user for a specific database:

+ Switch to the db
```use my_db```

+ Create the user 
```
db.createUser(
   {
     user: "homestead",
     pwd: "secrete",
     roles: [ "readWrite", "dbAdmin" ]
   }
);
```

### Collection

+ List all collections of a database: `show collections;`
+ Find all: `db.collectionName.find({})`
+ Format result: `db.collectionName.find({}).pretty()`


### Backup a database
+ Dump the orginal database: ``` mongodump --db=dev_db --username=dev --password="dev" ```
+ Zip it: ```zip -r dump.zip dump/```
+ Download to local
+ Restore: ```mongorestore --username=dev --password="password" --authenticationDatabase=your-db dump/```
