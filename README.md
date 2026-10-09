# MongoDB Replica Set with Live Replication

A 3-node MongoDB replica set (1 primary, 1 secondary, 1 arbiter) set up on a Linux machine, with a test showing data written on the primary is replicated.

> Local lab setup (not a production deployment).

**Stack:** MongoDB 8.0.17, mongosh 2.6.0, Linux (Ubuntu/Linux Mint)

## Architecture

| Member | Port | Role |
|---|---|---|
| localhost:37017 | 37017 | Primary |
| localhost:37018 | 37018 | Secondary |
| localhost:37019 | 37019 | Arbiter (votes only, holds no data) |

The arbiter lets the set keep a 2-of-3 majority for elections without a third data-bearing node.

## Steps

1. Start three `mongod` instances, each with its own data directory and port, all with the same `--replSet rs1` name.
2. Connect to the first node:
   ```bash
   mongosh --port 37017
   ```
3. Initiate the replica set:
   ```js
   rs.initiate({
     _id: "rs1",
     members: [
       { _id: 0, host: "localhost:37017" },
       { _id: 1, host: "localhost:37018" },
       { _id: 2, host: "localhost:37019", arbiterOnly: true }
     ]
   })
   ```
4. Check health and member states:
   ```js
   rs.status()
   ```
   Result: 37017 = `PRIMARY`, 37018 = `SECONDARY`, 37019 = `ARBITER`, all `health: 1`.
5. Test live replication: insert documents on the primary and read them back (see `screenshots/05-live-replication-test.png`):
   ```js
   use livedb
   db.users.insertOne({ name: "LiveReplicationTest", city: "Mumbai", time: new Date() })
   db.users.find().pretty()
   ```

## Screenshots

See the `screenshots/` folder: connection, `rs.initiate` and `rs.status()` output, member states, and the replication test.

> Note: the screenshots come from two runs of the same setup: replica set `rs1` on ports 37017-37019, and `rs0` on ports 27017-27019.

## What I learned

- How replica sets elect a primary (election timeout, majority votes) and what `rs.status()` reports for each member.
- Why an arbiter is used: it provides a vote for elections but stores no data.
- Startup warnings MongoDB reports (access control disabled, XFS recommended, ulimit/swappiness) and why they matter for production.

## Next steps

- Enable authentication and keyfile-based internal auth.
- Test failover by stopping the primary and watching a new election.
- Connect an application using the replica set connection string.
