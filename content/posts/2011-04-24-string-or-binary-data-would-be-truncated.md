+++
title = 'String or binary data would be truncated'
date = 2011-04-25T16:12:00-04:00
categories = 'Bugs'
tags = ["SQL"]
+++

![String or Binary](/assets/img/StringOrBinary.png)

This, in my opinion, is the most frustrating error you can get in SQL Server. It's the error you get when you're trying to do an `"INSERT INTO (...) SELECT (...)"` to push a bunch of records from a query result into a table. If one of your `(n)varchar` fields is too small to hold one of the values, you get this lovely error. It's nice that SQL Server won't truncate your fields for you, but frustrating that you get this sad little error with no details. You don't know which field is causing the problem, and if you're inserting into a large number of fields, it's completely frustrating to figure out what caused the issue. Especially when SQL Server could easily have done that heavy lifting for you and told you in the error details.

I've hit this error enough that I have developed a simple technique for troubleshooting this. It's not fancy, and you could probably think of something better, but this works and gets me to my resolution in a few seconds so I thought it would be worth sharing. The trick is to turn your `"INSERT INTO (...) SELECT (...)"` into a `"SELECT (...) INTO _BadDataTable_ (...)"` statement. The `SELECT INTO` will create a new table for you on the fly, assuming that you use the `AS `clause in your `SELECT `to name your fields the same as the destination table you're looking to `INSERT INTO`.

From there, you can then run this simple SQL script changing the values of `@prodTable` and `@invalidTable` into the real names of the destination table and the one you made on the fly containing the bad data. Kinda hackish, but it works and that's all I'm looking for. Note that you have to make a real table with the fields named the same for this to work. Feel free to take and modify to suit your needs, and here's to hoping the next release of SQL Server will fix this completely arcane error:

```sql
-- setup
declare
    @prodTable varchar(100) = 'MyDestTable', -- CHANGE THIS!!!
    @invalidTable varchar(100) = '_BadDataTable_', -- AND THIS!!!
    @c cursor,
    @column_name varchar(255),
    @max_len int

if object_id('tempdb..#max_len') is not null drop table #max_len
create table #max_len (max_len int)

-- get schema for prod table
if object_id('tempdb..#prod_table_schema') is not null drop table #prod_table_schema
select
    ordinal_position,
    column_name,
    data_type,
    character_maximum_length
into
    #prod_table_schema
from
    INFORMATION_SCHEMA.COLUMNS a
where
    a.table_name = @prodTable
order by 1

-- get schema for table made from invalid data with the trucate error
if object_id('tempdb..#invalid_table_schema') is not null drop table #invalid_table_schema
select
    ordinal_position,
    column_name,
    data_type,
    cast(null as int) as character_maximum_length,
    case when character_maximum_length is null then 0 else 1 end as is_char_field
into
    #invalid_table_schema
from
    INFORMATION_SCHEMA.COLUMNS a
where
    a.table_name = @invalidTable
order by 1

-- we need to chase after the max(len(COL)) info with a dynamic query
set @c = cursor local fast_forward for
    select a.column_name
    from #invalid_table_schema a
    where a.is_char_field = 1
    order by a.ordinal_position
open @c

fetch next from @c into @column_name
while @@fetch_status = 0 begin
    -- get the maximum data length of the field from the table
    delete #max_len
    insert into #max_len
    exec('select max(len(' + @column_name + ')) as max_len from ' + @invalidTable)
    select @max_len = max_len from #max_len

    update #invalid_table_schema
    set character_maximum_length = @max_len
    where column_name = @column_name

    fetch next from @c into @column_name
end
close @c
deallocate @c

-- Tell me which fields have the problem
select
    a.column_name,
    a.character_maximum_length,
    b.character_maximum_length as actual_length_of_data
from
    #prod_table_schema a join
    #invalid_table_schema b on
        a.column_name = b.column_name
where
    a.character_maximum_length < b.character_maximum_length
```
