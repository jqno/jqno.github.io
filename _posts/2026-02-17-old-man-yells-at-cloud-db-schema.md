---
title: "Old man yells at cloud db schema"
excerpt: In which I explain my personal rules for defining a schema in relational databases
tags:
- database
---
I have four rules that I follow when I define a relational database schema:

1. Implement [Flyway](https://github.com/flyway/flyway) from the start to avoid future regret
2. Slap a separate primary key field on every table
3. Yes, this includes join tables
4. Be consistent

These rules have served me well over the years. Yes, there are always exceptions to any rule, but I believe it's good to follow these unless you have a good reason not to.

![Meme - old man yells at cloud](/images/2026-02-17-old-man-yells-at-cloud-db-schema/old-man-yells-at-cloud.webp)

Coincidentally, these rules have come up surprisingly often for me in recent years in conversations with teammates who tend to be smart and very experienced with tools like Kafka or Redis or Cassandra, but haven't worked with relational database because I'm old and they're not. Let's call these teammates Chad, because addressing a group of people with a single name is a very reasonable thing to do.

Of course Chad can totally pull up a PostgreSQL in `$preferred_cloud_provider` and make it go brr, but they don't have the painful experiences I've had. Thankfully, I very much enjoy nitpicking their PRs and my rules come up every time I do. I've discussed them with three different Chads in the past year alone.

My rules are laughably easy to implement in a fresh database, but extremely difficult and expensive when the database has been running in prod for some time. So let's go over each one.

## 1. Implement Flyway from the start to avoid future regret

[Flyway](https://github.com/flyway/flyway) is a tool that manages schema migrations. Migrations are when you want to make a change to an existing schema, like add a field or a table. There are alternatives to Flyway, like [Liquibase](https://www.liquibase.com), which are all fine.

One time Chad argued that their microservice was very small and wouldn't ever change, but they were wrong. It's software, therefore it will change, and they need to prepare for that. Flyway is that preparation.

Without Flyway, when Chad has to make a change, they'll have to do it manually in test, accept, prod, and oh yeah, the entire team's local machines. Good luck keeping everything in sync Chad.

If Chad decides to add Flyway later, they're in for a lot of pain too, because Flyway will not be able to see any pre-existing schemas. I once had to add Flyway to an existing database. It took me _weeks_. I yelled at many clouds that winter.

## 2. Slap a separate primary key field on every table

Another time, Chad chose a phone number field to be the primary key. Phone numbers are unique, right? We can do that! (This is called a _natural id_.)

I told them to change it and add a separate field, not linked to the data itself, whose only function is to be a primary key. (This is called a _surrogate id_).

I know, _natural id_ sounds wholesome and cuddly and good, while _surrogate id_ sounds distant and cold and, well, unnatural. Still, surrogate ids are better.

Why? Because it's not just software that changes; the world changes too. A phone number seems unique enough, but what if the owner switches providers and the number is assigned to someone else? Or worse: I remember when the GDPR was introduced. We had to find a way to delete the phone number but not the rest of the data. Think about all the foreign keys that point to this phone number field!

If—no, _when_ such a change occurs, Chad suddenly needs to pick a new primary key, change all the foreign keys that point to it, and update all the database code related to that table. That's gonna be painful.

Better to just have an auto-incremented number (don't forget to give it its own dedicated sequence!) or a generated UUID, right?

## 3. Yes, this includes join tables

This also applies to join tables (with a caveat).

Join tables? Why join tables?

People may start using your software in unexpected ways, and what once was a humble join table could become an entity in its own right. If it does, you'll be glad if it already has its own surrogate id.

For example, a join table that links between Employees and Departments might become its own entity in version 2.0: a Project or Assignment, because it's the gig economy where jobs are temporary and people work several different jobs at once. The only possible natural id for a classic join table is the combined primary key of the two fields `employee_id` and `department_id`, and that just _sucks_ to work with. If you have to add a real primary key later, you'll have to update a bunch of client code and, depending on how much data is already in the table, the migration might run for a long time, resulting in down time. Better to have a surrogate id field ready to go!

Is this premature optimization? Maybe. But it's extremely cheap to add a primary key field that you don't really need, and it's extremely expensive to add one later.

When I was but a young Chad, I once had to promote such a join table to an entity in its own right. Fortunately, the company where I worked had a rule that every join table have a surrogate id field. You see, I didn't make this one up! I learn from experienced old geezers around me, and you should too.

The caveat I mentioned, is that JPA doesn't handle link tables with an explicit id field very well: the join table must become an `@Entity` and you can't use `@ManyToMany`. This is a consequence of ORM being a leaky abstraction, but that's a topic for another rant called "Old Man Yells at ORMs". Take a look at [jOOQ](https://www.jooq.org/) or [Jdbi](https://jdbi.org/), or if it's a small service, just rawdog the JDBC, it's not that hard!

So, if you _must_ use JPA, you're excused. You can skip this rule.

## 4. Be consistent

Sometimes when you have multiple tables, it can be useful to write some generic code that works across all tables. But you can only do that if all tables work the same way, even if that way doesn't follow the previous rules.

For example, if you need to set up an audit trail, it's really annoying if all your tables have a `created_at` field except the Account table where it's called `create_time` because it was written by Chad, and Chad didn't pay attention to the other tables that already existed in the database, and now you're stuck with it.

I recently inherited a microservice from a Chad when they left the company, where every table had a string natural id except for one which had an auto-incrementing integer surrogate id. I had to special-case all my shared integration test logic for that one entity and it made me sad and angry.

## Conclusion

There you have it: my personal rules for defining a schema for a relational database. If Chad is you, I hope this rant can help you make good decisions. If Chad is your teammate, I hope it can help you better explain your comments when you need to review their code. And if you disagree with some (or all) of these rules, that's fine. I hope my rant can still help you put your own arguments into words when inevitably you find yourself in a debate with an old cloud-yeller like me.

![Me, an old cloud-yeller](/images/2026-02-17-old-man-yells-at-cloud-db-schema/old-jan.webp)
