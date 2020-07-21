## Common Commands



### Mongo

+ Login as user to a database: `mongo -u admin -p passsword -authenticationDatabase db`
+ Show users of one particular database: 
    + `use my_db;`
    + `show users;`

### Collection

+ Find all: `db.collectionName.find({})`


### Backup a database
+ Dump the orginal database: ``` mongodump --db=dev_db --username=dev --password="dev" ```
+ Zip it: ```zip -r dump.zip dump/```
+ Download to local
+ Restore: ```mongorestore --username=dev --password="password" --authenticationDatabase=your-db dump/```
