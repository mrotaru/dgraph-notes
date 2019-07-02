# Notes on [`dgraph`](https://dgraph.io/)

## Cluster
- Zero - controls the Dgraph cluster, assigns servers to a group and re-balances data between server groups.
- Alpha - hosts predicates and indexes. (❓ what's a predicate)
- Ratel - serves the UI to run queries, mutations & altering schema.

## Running

Everything from the same container:

```sh
mkdir -p /tmp/data
docker run -it -p 5080:5080 -p 6080:6080 -p 8080:8080 -p 9080:9080 -p 8000:8000 -v /tmp/data:/dgraph --name diggy dgraph/dgraph dgraph zero # zero
# ❓ many ports; what does each of them do
# 5080 - Zero ❓
# 6080 - 
# 8080 - Alpha
# 9080 - Alpha
# 8000 - Ratel
docker exec -it diggy dgraph alpha --lru_mb 2048 --zero localhost:5080 # alpha
docker exec -it diggy dgraph-ratel # ratel
```
Now can go to http://localhost:8000 and run queries on the local cluster.

## Seeding

### Data Format

Below is the data in the examples; I dumped it into `movies-data`. Looks weird. It's silly that there is no info at all about this format on the quick-start page, or any links. (❓: data syntax reference) 

```
{
  set {
   _:luke <name> "Luke Skywalker" .
   _:leia <name> "Princess Leia" .
   _:han <name> "Han Solo" .
   _:lucas <name> "George Lucas" .
   _:irvin <name> "Irvin Kernshner" .
   _:richard <name> "Richard Marquand" .

   _:sw1 <name> "Star Wars: Episode IV - A New Hope" .
   _:sw1 <release_date> "1977-05-25" .
   _:sw1 <revenue> "775000000" .
   _:sw1 <running_time> "121" .
   _:sw1 <starring> _:luke .
   _:sw1 <starring> _:leia .
   _:sw1 <starring> _:han .
   _:sw1 <director> _:lucas .

   _:sw2 <name> "Star Wars: Episode V - The Empire Strikes Back" .
   _:sw2 <release_date> "1980-05-21" .
   _:sw2 <revenue> "534000000" .
   _:sw2 <running_time> "124" .
   _:sw2 <starring> _:luke .
   _:sw2 <starring> _:leia .
   _:sw2 <starring> _:han .
   _:sw2 <director> _:irvin .

   _:sw3 <name> "Star Wars: Episode VI - Return of the Jedi" .
   _:sw3 <release_date> "1983-05-25" .
   _:sw3 <revenue> "572000000" .
   _:sw3 <running_time> "131" .
   _:sw3 <starring> _:luke .
   _:sw3 <starring> _:leia .
   _:sw3 <starring> _:han .
   _:sw3 <director> _:richard .

   _:st1 <name> "Star Trek: The Motion Picture" .
   _:st1 <release_date> "1979-12-07" .
   _:st1 <revenue> "139000000" .
   _:st1 <running_time> "132" .
  }
}
```

#### Notes on Data Above

- `_:<foo>`: `foo` is the node name; node is referenced with `.`
- not sure if newlines are significant or just for formatting
- so, first "block" creates 6 nodes, each having a `name` attribute
- second block creates a node with a `name` and a few other string attributes
- also in the second block, some previously created nodes are linked to the current node
- following blocks are similar to the second one
- no examples of numeric/date attributes - just strings and nodes


### Insert Data

We can insert the data using `httpie`:

```
http POST localhost:8080/mutate < movies-data
```

Response:
```
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, X-Auth-Token, Cache-Control, X-Requested-With, X-Dgraph-CommitNow, X-Dgraph-Vars, X-Dgraph-MutationType, X-Dgraph-IgnoreIndexConflict
Access-Control-Allow-Methods: POST, OPTIONS
Access-Control-Allow-Origin: *
Connection: close
Content-Encoding: gzip
Content-Length: 608
Content-Type: application/json
Date: Tue, 02 Jul 2019 06:00:40 GMT

{
    "data": {
        "code": "Success", 
        "message": "Done", 
        "uids": {
            "han": "0x5", 
            "irvin": "0x7", 
            "leia": "0x8", 
            "lucas": "0x6", 
            "luke": "0x4", 
            "richard": "0x1", 
            "st1": "0x3", 
            "sw1": "0x9", 
            "sw2": "0x2", 
            "sw3": "0xa"
        }
    }, 
    "extensions": {
        "server_latency": {
            "parsing_ns": 49941, 
            "processing_ns": 91841132
        }, 
        "txn": {
            "keys": [
                "2c20l9utmu1tn", 
                "1d08efmdj0nz6", 
                "2ighhupgrkqgq", 
                "2jw1e0r30xvgd", 
                "4x3vqo4rlscb", 
                "1yfwi1w117aam", 
                "21zpn4zmq6l3e", 
                "nxwn4ta6z8mk", 
                "3gbgcrjldwxvw", 
                "1xt30nurgc241", 
                "14vev19z7h61y", 
                "np2g1l36tg3j", 
                "3txkhtl7eb6g6", 
                "1pz3zehdqddmg", 
                "3sxceqz6wwhny", 
                "gb3g8h7wtgvx", 
                "6i7bkb45wa6e", 
                "3t5qbcwvj42lm", 
                "29a62xaylc076", 
                "32ui61lf23pad", 
                "h4g99766zia3", 
                "1h50q2ey7n77m", 
                "3bmlta5fbx2vw", 
                "1mtg3lq60wftw", 
                "gp31sv71808r", 
                "2ub3p6c2prhgz", 
                "1l4o14wfafpde", 
                "3fr6hjna9fx2p", 
                "1mb7ydzmbezu", 
                "osqxvsd26sve", 
                "2w0shyotiqaks", 
                "f4nkjpu8s3vj", 
                "t9vua9eljkvn", 
                "175d9ifapl17b"
            ], 
            "preds": [
                "1-revenue", 
                "1-running_time", 
                "1-_predicate_", 
                "1-name", 
                "1-starring", 
                "1-director", 
                "1-release_date"
            ], 
            "start_ts": 11
        }
    }
}
```

## Index

To create and index:

```sh
http POST localhost:8080/alter <<<'
name: string @index(term) .
release_date: datetime @index(year) .
revenue: float .
running_time: int .
'
```

- so in indices, we do use types, but data itself is not typed ? (`string`, `datetime`, etc) (❓)
- need a reference for `@index`, info on `term` and `year` (❓)

## Query

```sh
http POST localhost:8080/query <<< '{ me(func: has(starring)) { name } }'
```