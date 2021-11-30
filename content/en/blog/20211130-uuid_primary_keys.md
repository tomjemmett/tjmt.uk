---
title: "Why we should use UUIDs for all primary keys"
date: 2021-11-30T10:00:00-00:00
tags: ["SQL"]
---

Recently I've been working with [HES](https://digital.nhs.uk/data-and-information/data-tools-and-services/data-services/hospital-episode-statistics)
data on a project, and the current structure of tables we have is for each of the different datasets (Inpatients,
Outpatients, Emeregency Care, etc.) we have a separate table for each financial year. Within these there will be a
column like `EPIKEY` or `ATTENDKEY` which is the primary key in that table. This key is stored as a `BIGINT`.

This key restarts counting each year... well, sort of. Sometimes the minimum value is 1, other times it's in the
100,000,000's.

What I wanted to do was build a single table that had all of the inpatients records (and later, do the same for the
other datasets). Because the `EPIKEY` values may repeat in different years I need someway to make these unique. One way
of achieveing this would be to create the primary key as a composite of `FYEAR` and `EPIKEY`.

This is what I have seen done in other databases, but (in my opinion) it's a very dangerous approach to take. Why?
Because it's far to easy to write a query that joins to other tables and forget to include all of the columns in the
composite key, for instance:

``` sql
SELECT
  stuff
FROM
  inpatients i
INNER JOIN
  inpatients_diagnoses d
  ON
    i.EPIKEY = d.EPIKEY
```

And I've seen many, many, analysts fall into this trap. It's not always a case of simply forgetting either. Many times
people will approach a new database and start just trying to write a query without looking at a database diagram (if one
exists), or inspecting the primary/foreign key contraints on a table (and all too often I've seen databases created
without these being implemented). I think this is exacerbated because we treat SQL as something that analysts must
learn, but forget to teach them any theory about databases and their structure.

So, what is the solution to this? What I did was rename the `EPIKEY` column to `ORIG_EPIKEY` (rather than dropping it
and finding out we needed it later), and adding a new `EPIKEY` column of type `UNIQUEIDENTIFIER`. Now I have a single
column as the primary key, so my above query works, reducing the risk of incorrect joins.

A [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) (Universally Unique Identifier) is, for practical
purposes, unique. As in, it's unique in any system that may ever exist. This it turns out is a very useful property. If
we build our database tables to use a UUID as the primary key, we do not need it to be composite with any other columns.

And there is an added benefit. Consider we have two tables, `table_a` and `table_b` that both use an incrementing
integer as the primary key. We could join `table_a` and `table_b` together on their primary keys, and the database will
happily spit out results. But they may be completely useless. If however we use a UUID, then the join will only ever
return results where the keys match. If the keys were generated separately, then there is no chance of a collision
between `table_a` and `table_b`, so any query that tries to join these two will result in 0 rows of data (this would be
a good indicator to anyone that the query isn't correct). If, and only if, the keys were generated in one place, say
`table_a`, and then used as a foreign key in `table_b` would our query work. This is a great guarantee that our results
are correct and valid.

Of course, there is a downside to using this datatype. For one, it's slower to generate than an autoincrementing number.
I've never found generating these values to be terribly slow however. The big downside is storage: a typical ID column
may be a `BIGINT` which takes up 8 bytes, whereas a UUID is 128 bytes, 16 times bigger. In certain circumstances these
two may be just enough of a difference to cause an issue.
