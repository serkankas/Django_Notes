## REDIS

Main course [link](https://www.youtube.com/watch?v=jgpVdJB2sKQ)

In this section we will see and discuss about the Redis technology.<br>
Redis is alternative to message broker, which means alternative to kafka or rabbit-mq. However in most django application, that's the official one which supported by channels and celery. Since it's the common one, it's good idea to take a look at it.

> What is REDIS

REDIS is an in-memory data structure store, like NoSQL structure. However, Redis doesn't works with tables, instead it works with key-value pairs like json.<br>
It uses storage for cache memory. Which means, if your system shut, you end up with losing all your data. Unless you back up that data. Its kinda memory storage, already boot up. __That might be an issue for low equiped system like raspberry pi since they have limited memory__.

```console
$ sudo apt install redis 
```
Check status with
```console
$ sudo systemctl status redis 
```
Running and Checking server
```console
$ redis-server
```
Now, it may gives an error when we type ```redis-server```. However, you can control with ```systemctl status redis```

Connection redis via
```console
$ redis-cli
```

Now if you type ```quit``` inside redis, it's quit.<br>
it gets query like
```redis
> SET name kyle
> GET name
"kyle"
> DEL name
> EXISTS name
```
|Command |Used For
|--- |---
|SET | For Creating variable
|GET | Getting key and paired value from redis
|DEL | Deleting key and paired value
|EXISTS| Check if key is exists.
|KEYS *| Check all the table
|flushall | Remove all content in REDIS
|EXPIRE <key> <second>| Setting Expire time for that key
|TTL <key>| Check if the key has been expired. (-1 means doesn't have expiration, -2 means it's gone!)
|SETEX <key> <second to expire> <value>| Combination of SET and Expire
|lpush <list_name> <value/s>| Creating/adding List of Items (left)
|rpush <list_name> <value/s>| Same as lpush, but (right)
|lpop <list_name>| Remove item from Left
|rpop <list_name>| Remove item from right
|lrange <list_name> <lower baund> <upper baund>| Shows the all the item in boundary
| SADD <sets_name> <value/s> | Sets add (This Set is unique java like. The members has to be unique no matter what!)
| SMEMBERS <sets_name> | Sets member showing.
| SREM <sets_name> | Removing the sets.
| HSET <hash_parent_name> <key> <value>| Key Value Pair (HASH) SET Hash
| HEXISTS <hash_parent_name> <key> | Control the Hash
| HGETALL <hash_parent_name> | Get All (with key and pair)
| HGET <hash_parent_name> <key> | Get one
| HDEL <hash_parent_name> <key> | Hash Delete 
