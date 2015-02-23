> While I read the paper, I kept wondering whether about how these methods address transactions. In particular, the issue of durability and consistency. I feel that the challenges described in this paper might be deceivingly simple, and might hide difficulties, specially when dealing with complex issues of
consistencies such foreign key constraints, as the authors claim. In other words, before I am presented with a tool to address this issues, I would like to see, from a theoretical point of view, why such method guarantees ACID properties of a database, specially from the point of view of failure and rollbacks. I personally don't see why the issue of foreign keys is much more difficult than otherwise. Foreign key constraints in the ghost tables should be treated as any other constraint (eg. NULL, or Primary key). Also, by definition of the operations, if a tuple in main table has satisfied the constraint, hasn't also the ghost table satisfied the constraint? I imagine that any modifications of the ghost tables (done in the stored procedures by the triggers) will happen in a transaction. Therefore the ghost tables will be consistent as long as the main table is (even if the ghost table does not have ANY constraint).

The reason this is a non-trivial problem is a combination of the blocking nature of modifying tables and constraints, and the foreign key constraints which put restrictions on data and tables which must exist. Also triggers in Postgres ensure that everything happens within the same transaction. Perhaps clarify this, and the problem even further.

> Turning the ghosts tables into main tables should also be a transaction (locking out any other transaction that operates on those tables...).

Nope, not in our approach. In our approach there's no need to make the eventual switch, since the clients translate to the correct table name. If we eventually do decide to switch original and ghost table, then yes we need to do this in a transaction.

> So this probably means that to wrap this in a transaction, the constraints have to be dropped, the old table deleted, the new table created, and the constraints added. Example, imagine that the authors table in Figure 1 is the one that needs to be modified. One cannot drop this table unless books no longer has FK constraint to it. So one needs to drop constraints from books, drop authors, create authors using authors_ghost (there is no way to rename a table in SQL, to my knowledge), add the constraints back to book, then commit. Only this way it would guarantee consistency, assuming that authors and authors_ghost are consistent with each other.

Nope. An entirely different set of tables would be mirrored with ghost tables in the case of modifying the "authors" table. In that case the reviewer's statement doesn't make much sense. Renaming tables is possible in pretty much every SQL database.

> The example of Figure 1 is currently much simpler than the true problem of FK constraints, because one can simply drop books without any constraint violation, and then "rename the ghost table (with the FK constraints in place) to the old name.

No this is not possible, since the "publications" table referres to the "books" table.

> Concentrates too much on performance, when consistency and durability are equally, if not more important.

Perhaps shift focus more away from performance...

> The second paragraph of Section B is somehow confusing. Make it clear what “This” in the second sentence refers to. 

See what we can do here...

> The authors write in point 2 of their vision (Section A) that “Their ‘view’ should only contain tables, columns, foreign keys, etc which are present in their chosen version of the database schema, but contain the same data as is present in all other ‘views’.” “contain the same data as is present in all other ‘views’” is not correct. A primary objective of making database schema change is to store different data. For example, the new version may have a new table or new columns to store new pieces of data. The new version of the application will make use of the new data.

Some slight rephrasing should be enough...

> The paper criticized existing approaches for their focus on MySQL databases only. The proposed QuantumDB also only supports one database, PostgresSQL, which is probably less popular than MySQL. Although the authors do claim that QuantumDB should prove easily extendable to other popular SQL databases. The same claim probably could be equally made for many other existing tools as well. The authors did not explain why other tools could be harder extendable to other SQL databases than QuantumDB. “prove easily extendable” is probably an over claim.

Yup, but the existing tools are part of MySQL DBA toolkits, whereas QuantumDB is not...

> More explanation on how the lack of foreign key support impacts the usage of related tools could be beneficial

See what we can do here...