# Ubuntu

### MongoDB
#### Installation

First install mongodb:
```
$	sudo apt-get install mongodb
```

#### Administration & Security


mongodb service:
```
$	sudo systemctl start mongodb
$	sudo systemctl enable mongodb
```

To enable authentication, edit `/etc/mongodb.conf` and add/edit the following configruations, then restart the daemon:
```
noauth = false
auth = true
```

#### Usage

To login to the database, with the user `root`,  which is stored in the database `admin` :
```
$	mongo -u root -p --authenticationDatabase admin
```

To create the new user `root` in the db `admin`:
```
> use admin
switched to db admin
> db.createUser({user:"root", pwd:"Holiday", roles:[{role:"root", db:"admin"}]})

Successfully added user: {
	"user" : "root",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}

```

to change a the user 

to list all databases:
```
>	show dbs
admin  0.078GB
local  0.078GB
```

to select the database `admin`:
```
> use admin
```

To list all of the users in the current database:
```
> show users
{
	"_id" : "admin.root",
	"user" : "root",
	"db" : "admin",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}
```

To view all roles for the current database:
```
> show roles
```

To change the password of th user `root`, in the db `admin`, to the password `Holiday7`:
```
> db.changeUserPassword('root', 'Holiday7')
```

to change the role of the user `michael` to `dbAdmin`, stored in the db `admin`:
```
>	db.updateUser("michael", {roles: [{role: "dbAdmin", db: "admin"}]})
```

to delete the user `michael`:
```
>	db.dropUser("michael")
```

or, if you are on older systems:

```
>	db.removeUser("michael")
```

to create a database `tux`, select it, then insert a document :
```
>	use tux
>	db.tundra.insert({"beast":"bear"})
```

