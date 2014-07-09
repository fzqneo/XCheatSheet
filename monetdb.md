#MonetDB

Reference: [monetdbd](https://www.monetdb.org/Documentation/monetdbd)

##monetdbd

Each monetdb demon is associated with a **dbfarm**. Only one instance of *monetdbd* can be running on a dbfarm.

1. Create a dbfarm: `monetdbd create ~/my-dbfarm`
2. Show properties: `monetdbd get all ~/my-dbfarm`
3. Start the demon on a dbfarm: `monetdbd start ~/my-dbfarm`

##monetdb

1. Create a new database: `monetdb create tpch`
A newly-created DB is in ``locked'' state.
