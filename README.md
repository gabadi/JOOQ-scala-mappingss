# JOOQ-scala-mappingss

All the [JOOQ](http://www.jooq.org) power, in the [scala](http://www.scala-lang.org/) world.

### Why?
-----------

JOOQ is a great tool for accessing a SQL store. It's expressive, powerful, etc. The single thing that i would like to improve is to facilitate the most simple use cases (more or less like an ORM does). And for that this project uses scala macros.

### Installation
------------

TODO: Publish maven central
TODO: Depends on jooq 3.6.1 or greater with scala codegen

### Functionalities
---------------

 1. [Record to entities mappings](#record-to-entity-mapping)
 2. [Entity to record mappings](#entity-to-record-mapping)
 3. [Scala option support](#scala-option-support)
 3. [Implicit type conversion](#implicit-type-conversion)

More to come...

### Record to entity mapping

Given this mysql table:
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(50) NOT NULL,
  `last_name` varchar(50) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
If i want to map this with my case class
```
case class User(id: java.lang.Long,
                firstName: String, 
                lastName: String) {
}
```
That is as simple as doing:
```
    val r = dsl.newRecord(Tables.USER)
    r setId 1l
    r setFirstName "name"
    r setLastName "last"

    val meta = com.github.gabadi.scalajooq.JooqMeta.metaOf[tables.User, UserRecord, User]
    
    val user = meta toEntity r

```
Note: **tables.User** and **UserRecord** were autogenerated by the jooq scala codegen.

Note: To generate the dsl instance, see the JOOQ documentation

This is not magic, this is not reflections, this is a macro.
But what do i win? 
The answer is that now, all the runtime exceptions will be compilation exceptions. So, if for example, you have a bug in the user like this:

```
case class User(id: java.lang.Long,
                firsName: String, 
                lastName: String) {
}
```
During compilation you'll see a message like this:

```
Mappings error:
 com.github.gabadi.scalajooq.User.firsName expects a db.test.public.tables.records.UserRecord.FIRS_NAME column, but doesn't exists
```

### Entity to record mapping
Now in the other way this is valid too
```
    val entity = User(0, "name", "last")
    
    val meta = com.github.gabadi.scalajooq.JooqMeta.metaOf[tables.User, UserRecord, User]
    
    val record = meta toRecord entity
    record.store()

```
Note: This the method toRecord depends in an implicit DSLContext instance

### Scala option support
One important feature in scala, that it's necessary yo wait till JOOQ version 4.0 for a correct support are Option values.
But with JOOQ-scala-mapping, giving this class:
```
case class UserOption(id: java.lang.Long,
                      firstName: Option[String], 
                      lastName: Option[String]) {
}
```
Anyone can make a mapping like this:

```
    val r = dsl.newRecord(Table<s.USER)
    r setId 1l

    val meta = com.github.gabadi.scalajooq.JooqMeta.metaOf[tables.User, UserRecord, User]
    
    val user = meta toEntity r
    println(user)
```
With this result:
```
    User(1, None, None)
```
And in the other hand, JOOQ-scala-mappings will never let you have an Entity with a null field. Instead of that, an exception will be thrown when you try to generate that entity.
###Implicit type conversion
The current user as we have defined it, has a little issue. If you see the id field, you'll see that it's not a scala Long, it's a java Long.
So, what if you want a standard scala **Long**?, or **Int**? or **Boolean**?
For that, you have implicit conversions. What that means is simple that for **JOOQ-scala-mappings** this two entities are valid representations for the **user** table

```
case class User1(id: java.lang.Long,
                 firsName: String, 
                 lastName: String) {
}
case class User2(id: Long,
                 firsName: String, 
                 lastName: String) {
}

```

###Next functionalities
--------

 1. base DAOs
 2. Embedded entities
 3. Joined entities

###Contributing
------------
Please see [CONTRIBUTING](CONTRIBUTING.md)
