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
$ sudo nano /etc/mongod.conf

auth = true

$ sudo systemctl restart mongod

$ mongo -u "root" -p "rootpw" --authenticationDatabase  "admin"

> use admin
> 
> exit
> 
$ mongo -u "lefa" -p "lefapw" --authenticationDatabase  "lefa"

> use lefa
> 
> exit

Edit your MongoDB config file. On Ubuntu:

sudo vim /etc/mongod.conf

Look for the net line and comment out the bindIp line under it, which is currently limiting MongoDB connections to localhost:

Warning: do not comment out the bindIp line without enabling authorization. Otherwise you will be opening up the whole internet to have full admin access to all mongo databases on your MongoDB server!


# network interfaces
net:

  port: 27017
  
  bindIp: 0.0.0.0  
  
Scroll down to the #security: section and add the following line. Make sure to un-comment the security: line.
security:
  authorization: 'enabled'
