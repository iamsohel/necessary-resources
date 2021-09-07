https://awswithatiq.com/how-to-install-mongodb-in-ubuntu-inside-aws/ 
 
 sudo apt install mongodb
 
$ mongo

> show dbs
> 
> use admin
> 
> db.createUser({  user: "root",  pwd: "rootpw",  roles: [ "root" ]  })  // root user can do anything
> 
> use lefa
> 
> db.lefa.save( {name:"test"} )
> 
> db.lefa.find()
> 
> show dbs
> 
> db.createUser({  user: "lefa",  pwd: "lefapw",  roles: [ { role: "dbOwner", db: "lefa" } ]  }) // admin of a db
> 
> exit
> 
$ sudo vim /etc/mongodb.conf

auth = true

$ sudo systemctl restart mongodb

$ mongo -u "root" -p "rootpw" --authenticationDatabase  "admin"

> use admin
> 
> exit
> 
$ mongo -u "lefa" -p "lefapw" --authenticationDatabase  "lefa"

> use lefa
> 
> exit


