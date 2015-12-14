# ha-postgres-docker
High-Availability Postgres Docker Image

### Work in Progress

Until a v1.0 release, this work should be assumed to be horrendously unstable
and unsuitable for production use.  Help us get to v1!

### Goals

There does not currently exist a public, easily-tuned, highly-available
deployment option for Postgres.  The goal of this repository is to change that.
The external API of this image should be indistinguishable from Postgres v9.4 -
you should be able to connect an existing app to this container/service and have
it behave identically to a non-HA version.

### Tutum/Rancher Integration

We want to make this image easily integrated with Tutum, Rancher, and other
docker-deployment options.  We also want it to work in docker-compose.
Initially, there are a few deployment options:

1) Easiest: "Stack"/"Compose" deployment, where each container is individually
configured to fulfill a given role.  You would start out with a few instances of
pgpool2 distributing connections to the different databases.  Automatic failover
would occur if the main database does not respond in a timely way to a request.
If we're in a tutum context, a witness server would promote a read replica to
primary, the old master would be shut down, and started back up as a read
replica.  The changeover process would be configurable in terms of duration.

2) Harder, but better: Using tutum's environment variables, we can determine
which host we're on, and how many containers there are. Based on configuration
variables, one of a few different options can be selected for
failover/replication.  Starting point would be for a high-read, fewer-writes DB
setup.  Optimizing for high-read AND high-write in this kind of application
would be difficult, I think - this solution is more for hobbyists and small
startups that do not have the ability to hire DBA staff.  In the fully-automagic
solution, simply scaling the number of instances for the DB service allows you
to create more replication.  Using the "High Availability" option on Tutum would
automatically create read replicas on every node, with automatic restarting and
failover.  Every failure should cause some kind of email or webhook trigger so
that the developer is notified that a failover has taken place.  The cool part
of this setup is that Tutum is used as the "witness" server.
