# go-artisanal-integers-summitdb

No, really.

Assuming the following per the [SummitDB documentation](https://github.com/tidwall/summitdb#getting-started):

```
$> ./summitdb-server
$> ./summitdb-server -p 7482 -join localhost:7481 -dir data2
$> ./summitdb-server -p 7483 -join localhost:7481 -dir data3
```

Then:

```
$> ./bin/int -engine summitdb -dsn 'redis://localhost:7481'
2
$> ./bin/int -engine summitdb -dsn 'redis://localhost:7481'
4
$> ./bin/int -engine summitdb -dsn 'redis://localhost:7481'
2017/03/27 14:58:55 dial tcp 127.0.0.1:7481: getsockopt: connection refused
$> ./bin/int -engine summitdb -dsn 'redis://localhost:7482'
2017/03/27 14:58:57 TRY 127.0.0.1:7483
$> ./bin/int -engine summitdb -dsn 'redis://localhost:7483'
6
```

The SummitDB engine attempts to handle `TRY` messages automagically so really it looks like this:

```
$> /bin/int -engine summitdb -dsn 'redis://localhost:7482'
summitdb told me to try redis://127.0.0.1:7483 instead, so here we go...
22
```

It will also attempt to fail over to whichever peer takes over if and when the leader goes down. For example, let's say you did this and then shortly afterwards stopped the SummitDB server listening on port `7482`. You'd see something like this:

```
$> ./bin/int -engine summitdb -dsn 'redis://localhost:7482' -continuous
10282
10284
10286
10288
10290
10292
couldn't connect to leader so trying to see if the peers are rebalancing themselves (1/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (2/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (3/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (1/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (2/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (3/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (4/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (5/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (6/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (7/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (8/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (9/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (10/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (11/200)...
couldn't connect to leader so trying to see if the peers are rebalancing themselves (12/200)...
10294
10296
10298
10300
10302
10304
10306
10308
```

_Note the use of the `-continuous` flag to just keep generating integer after integer after integer..._

## Performance

Running `intd` backed by SummitDB (running with [high consistency](https://github.com/tidwall/summitdb#read-consistency)) on a vanilla Vagrant machine (running Ubuntu 14.04) on a laptop against 100 concurrent users, using siege:

```
$> siege -c 100 http://localhost:8080
** SIEGE 3.0.5
** Preparing 100 concurrent users for battle.
The server is now under siege...^C
Lifting the server siege...      done.

Transactions:			418 hits
Availability:			100.00 %
Elapsed time:			44.57 secs
Data transferred:		0.01 MB
Response time:			9.13 secs
Transaction rate:		9.38 trans/sec
Throughput:			0.00 MB/sec
Concurrency:			85.65
Successful transactions:	303
Failed transactions:		0
Longest transaction:		22.80
Shortest transaction:		0.07
```

_Note: This pegged the (single) CPU on the virtual machine._

## See also

* https://github.com/aaronland/go-artisanal-integers
* https://github.com/tidwall/summitdb
