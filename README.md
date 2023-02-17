# Lab 3

## Table of contents

## Outline
   - [Overview](#overview)
   - [More Exercises](#a-more-in-depth-set-of-exercises)
     - [Creating some tables](#some-simple-tables)
     - [Quick Check](#quick-check)
   - [Anomalies](#anomalies)
     - [Modification Anomaly](#modification-anomaly)
     - [Deletion Anomaly](#deletion-anomaly)
     - [Insertion Anomalies](#insertion-anomalies)
     - [Anomaly Exercise](#anomaly-exercise)
   - [Data Normalization](#data-normalization)
     - [Attributes and Functional Dependence](#attributes-and-functional-dependence)
     - [A Dash of Dependency](#a-dash-of-dependency)
     - [A Clutch of Keys](#a-clutch-of-keys)     - [Prime and Non-Prime Attributes](#prime-and-non-prime-attributes)
   - [The Normal Forms](#the-normal-forms)
     - [First Normal Form](#first-normal-form)
     - [Second Normal Form](#second-normal-form)
     - [Third Normal Form](#third-normal-form)
     - [Boyce Codd Normal Form](#boyce-codd-normal-form)
     - [Summary](#summary-of-normal-forms)
   - [Checklist](#checklist-of-what-to-do)
   

## Overview

In today's lab we are going to get to know more about the `SELECT` statement and a bit about Database Design Theory.  At this point you should be able to do `SELECT`s that involve multiple tables and join them together with a `WHERE` clause, or explicitly with a `LEFT` or `RIGHT JOIN`.   You should also be able to do some nontrivial filtering involving the `WHERE` clause.


You are going to be making several tables to play with, so just in case you make a few mistakes and don't want to `DROP` the table and start over use the method we talked about last week where you can make a copy of the table as a backup. 

As you play with the following commands think about what it would take to design a database application that would be used by a store.  Such an application would need to be able to keep track of what happens at each cash register and have a back-office function that allowed reports about sales and the current inventory.  

Here are a few questions to ask yourself:

* How will inventory be tracked?  
* Should the system keep track of vendors?  
* What about the people working the tills (cash registers)?
    * Should the system track when they log into a till? 
    * Should the system track how long an interaction takes place (efficiency?).  
    * Should the system try to help with up-selling? (suggesting other merchandise... would you like a drink with that?)
* How should it deal with 
    * items sold by unit
    * items sold by volume
    * items part of a combination deal
    * sale items
    * promotional items
    * coupons
    * refunds
    * mistakes
* How independent should the individual tills be?
    * Can the tills operate if the local network is down?
    * What is stored on the server?
    * What is stored on the till?

That's a lot to think about, and we are only going to focus on a small subset of a complete system... but it would be nice to design our subsystem carefully enough that it could be integrated into something larger without too much trouble.

On the same Google Doc that has the info on which database your work in is, I would like each group to write a page or so providing a rough outline of what you think would be necessary – we'll flesh this out later. You will then submit the link to that GDoc on Canvas.

## A more in-depth set of exercises

You are now going to create several tables that will help you develop a deeper understanding of what you will need to do to develop a better Point-Of-Sales System.

### Some simple tables

As this is Lab 3 don't forget to create your Lab03 schema to house your tables.

Create the Schema and tables in 1 person in the groups database and then grant access to the other group members to modify tables in the Lab03 Schema.

Granting Access on Schema https://learn.microsoft.com/en-us/sql/t-sql/statements/grant-schema-permissions-transact-sql?view=sql-server-ver16

Let's create a table called `master` that will keep track of each transactions on a global level. You should make it with the following fields:

field    | data-type      | notes
---------|----------------|-----------
start    | datetime       | to track when the interaction began
stop     | datetime       | and to track when it was completed
tid      | integer        | unique id for keeping track of transactions (should be the primary key in this table)
register | integer        | numeric id of cash register where transaction occurred
user     | nvarchar       | name of clerk who was running the till
total    | decimal with two places | the total amount of money received (or refunded) in this transaction
	
Let call the table for keeping track of the items in a transaction `t_items`:

field    | data-type               | notes
---------|-------------------------|-------------------------------------------
tid      | same as master table    | allows the individual item to be associated to the transaction in the `master` table
pid      | same as inventory table | allows the individual item to be associated with their price, description, etc.
price    | decimal                 | price for this (set of) item(s)
gid      | integer                 | group id for combos
amount   | decimal                 | how many of this item were purchased in this transaction 

In some ways we're breaking normalization by including the `price` field because that could be calculated using the `amount` field and information in the `prices` table. (See the material on normalization below for more on these kinds of _functional dependencies_.) We'll include it, though, because a bottle of pop might cost \$1.92 one day, while it's on sale for \$0.75 another day, so there is an argument for having this field.

The `gid` field allows us to create _combos_ where a group of items are sold together for a (presumably) reduced price. Consider a table called `combos` (read a bit further before creating this):

field    |Description
---------|---------------------
uid      |  unique id (primary key) 
comboName|  name of the combo
item     |  should correspond to a product id in inventory
price    |  the amount of the combo
comboCode|  a number unique to the combo

We could keep track of combos with a table like this:

uid | comboName         |   item   | price  | comboCode
----|--------------------|----------|--------|---------------   
1   | Double Slam       |         2|    1.99| 1
2   | Double Slam         |3|1.99|1
3   | Quacker Jack special|2|2.15|2
4   | Quacker Jack special|17|2.15|2
5   | Quacker Jack special|191|2.15|2

The sale of, for example, three `Double Slam`s would create a single entry in `master` but _two_ entries in `t_items`. Assuming the transaction id (`tid`) for this transaction was 57, the two entries in `t_items` might look like:

tid | pid | price | gid | amount
----|-----|-------|-----|-------
57  |   2 |  1.99 |   1 | 3
57  |   3 |  1.99 |   1 | 3

This design violates some of the normal form rules discussed later in this lab, so go ahead and build that table but give it a name like `poorDesign`; you'll tidy that up in subsequent labs. Add the 5 records indicated above, but use id's from your own inventory table.

### Quick Check 

You are not turning this in... but you still need to do it.  Before we put things in order, let's check your understanding of the material from the previous lab.

Start by constructing a query that links together data from the `poorDesign` table, and from your `inventory` table in order to provide the following information:

* `comboName` 
* `comboPrice`
* `item`

So, for example, if item 2 was `Brawndo`, and item 3 was `Human Kidneys`, then your query should return the following:

```
Double Slam	1.99	Brawndo
Double Slam	1.99	Human Kidneys
```

I want the columns of your query to have the names `combo`, `price`, `item`.  You may find it helpful to review the usage of the keyword `AS` in the documentation on the `SELECT` statement.

You really must be tired of all those `INSERT` statement by now (I know I am). See if you can find documentation on Microsoft's SQL Server Books Online on how to import from at csv or txt file.  That could help alleviate needing to manually insert all the time.

Now let's get even MORE CREATIVE.  Create a query that links information from `poorDesign`, `inventory`, and `price` to display the following information:

* `comboName` 
* `comboPrice` 
* `item`
* `itemPrice`

This is the first time you've done a query that uses three tables.  

Once you get the query to run correctly, you have implicitly begun to understand the secret to something called **data normalization**.  What you have done with your query is shown that redundant (or duplicated) information in your output can be generated, as necessary, by a good query.  

If you don't normalize your data (more on that very soon), then you run the risk of creating an **anomaly**, where data that's duplicated in different places can get "out of sync". Even if your framework is amazing and double checks everything very carefully bad things can still happen accidentally.  Suppose, for example, your internet connection is cut off in the middle of updating. If your data-design is good (which often means, among other things, that there is no redundant information in any of your tables), then anomalies can't happen because the essential information appears in only one place and it only takes one atomic update to make a change.

## Anomalies

**Anomalies** occur when the data in your tables is not consistent (either internally or with the real-world system it is describing).  

Different authors define terms a bit differently but broadly speaking, anomalies fall into the following general categories based upon the cause of the problem [^anomalies] (examples will follow the definitions):

* **Modification anomalies** (some call these **update anomalies**) occur when duplicated data is updated... but not every copy is changed.
* **Deletion Anomalies** occur when removing a record from a table permanently removes information that should be retained.
* **Insertion Anomalies** occurs when new information that *should* be included in the data can't be added without the presence of other attributes.

To make sense of these examples consider the following table:

StudentNum | CourseNum | Student Name | Address    | Course
-----------|-----------|--------------|------------|-------------
S21        | 9201      | Jones        | Edinburgh  | Accounts
S21        | 9267      | Jones        | Edinburgh  | Physics
S24        | 9267      | Smith        | Glasgow    | Physics
S30        | 9251      | Richards     | Manchester | Computing
S30        | 9322      | Richards     | Manchester | Maths


### Modification Anomaly

Notice that the student `Jones` appears in two rows.  If we updated his address in one row, but not both, then that would be a modification anomaly.  As previously mentioned, this is also called this an **update anomaly** by some authors.

### Deletion Anomaly

Assuming this table is our only source of information, then if student `S30` drops out we no longer have a record of the 'Computing' or 'Maths' course.  The ability to list the courses being offered should not depend upon a how many students are enrolled in the class.

### Insertion Anomalies

We will look at two insertion anomalies.  The first is related to the deletion anomaly:  Suppose you want to introduce a course in 'Quantum', but nobody is enrolled in the class yet.  The only way to do it requires using a NULL value to (essentially) leave some attributes blank:

StudentNum | CourseNum | Student Name | Address    | Course
-----------|-----------|--------------|------------|-------------
NULL       | 00X3      | NULL         | NULL       | Quantum

(It **can** be done... but it is very unsatisfying and requires that extra steps be taken to tell the difference between a "normal" record and one purely related to which courses exist.)

Another problem is related to the update anomaly:  Suppose you want to add a new student to the database.  The student is enrolled in `CourseNum` '9322'.  We would have to make certain that the `Course` and `CourseNum` match in all records.  If we did not do this correctly that would also be an Insertion Anomaly.

### Anomaly Exercise

**Exercise**:  Each group should create the table above (structure *and* data), and then update *one* record in such a way as to induce a modification anomaly.

## Data Normalization

**Data Normalization** is the process of organizing the fields and tables (of a relational database) to minimize redundancy and dependency.[^normalization]  The goal of normalizing your data is the following:

* No information redundancy
* Not storing unrelated data in the same table (this helps protect against the *anomalies* we just discussed)

### Attributes and Functional Dependence

Columns or fields are typically called **attributes** in database design books.  An attribute is said to have a **functional dependency** upon another attribute if knowing the value of the second attribute allows one to determine the value of the first.  This is less complicated than it sounds.

Let's consider the course table we were using up above. Notice that the attribute `Course` is functionally dependent upon `CourseNum`, because if I know `CourseNum`, then I should be able to deduce the `Course`.  We write this as:

`CourseNum->Course`.  

In my head I say:  "If I know `CourseNum` then I can deduce `Course`", or even more briefly, "`CourseNum` *determines* `Course`".  Just remember (and this always trips me up) that saying "the attribute `CourseNum` determines `Course`" means the same thing as saying "`Course` is functionally dependent (some books just say *has a dependence*) upon `CourseNum`.  The arrow does NOT mean *functionally depends on* (at least not if you're reading it left-to-right).

Often the relationship is not symmetric (ie, it does not go both ways). The attribute `CourseNum` is **not** functionally dependent upon `Course`,  because knowing the value of `Course` does not determine the value of `CourseID`.  At most universities (like our own) there are many classes with the same name, but different IDs.  Take, for example, 'Calculus I'.  Each section of the course has its own unique ID.  The fact that *some* classes are uniquely paired with their IDs doesn't matter for this definition; functional dependence requires that the relationship holds for _all_ instances.
	
The idea of functional dependence extends to **sets of attributes**.  In the table above we abbreviated address to just a city. If, however, the attribute `Address` was actually a complete address then `StudentNum` is functionally dependent upon the pair `{Address, Student Name}` assuming there are no two students with the same name at a given address. We would write this concisely as `{Address, Student Name} -> {StudentNum}`.

Depending upon the size of the university it is possible that `StudentNum` is functionally dependent upon `Student Name` and vice-versa, but honestly, it would be better to head off any future problems by assuming that `StudentNum->Student Name` and not the other way around.  However, the ultimate arbiter of functional dependency is the situation being modeled, i.e., the data that's actually in the table(s). Notice that it is also true that `StudentNum->{Student Name, Address}`.

There are two important ideas to keep in mind when thinking about these issues:

1. When does repetition in the values of the attributes under consideration make functional dependency impossible?  This can sometimes be determined by looking at data that is going to be in the database.
2. What does the situation being modeled have to say about any functional dependencies?

Consider the following table that relates various preferences between individuals.  Assume that it is an accurate reflection of the real world and that meaningful functional dependencies can be inferred purely from the data in the table (we are pretending that point 2 does not enter into consideration, i.e., that there won't be any future data that might mess things up):

favorite color    |favorite animal |number |name
---------------|---------|--------|-----------
gray           |cats     |      13|Peter
gray           |snakes   |       7|Heather
black          |snakes   |      21|Katya
black          |cats     |       2|Zet
	
The attribute `name` is **NOT** functionally dependent upon color nor animal, but the combination of color **and** animal **does** determine name uniquely:

```
{color, animal}->name
```

Adding more attributes on the left will also be accurate (although redundant):

```
{color, animal, number}->name
```

### A Dash of Dependency

There is clearly a difference between the two dependencies just described.  We say that the attribute `name` has a **full functional dependency** upon `{color,animal}` because dropping any attribute in `{color, animal}` destroys the dependency; neither `color` nor `animal` alone uniquely determines `name`.
	
You might be beginning to see the mathematics supporting the structure behind the database system.  Let's let `A`, `B`, and `C` stand for sets of attributes.

Hopefully it will seem fairly obvious that if `A->B` and `B->C` then `A->C`.  (You may recognize this as **transitivity**)

In database design we say that attribute `C` is **transitively dependent** upon `A` if

1. `A->B`
2. **AND** It is not true that `B->A`
3. **AND** `B->C`

It is always that case that `A->B` and `B->C` implies `A->C` (which is what a mathematician recognizes as transitivity), but in database design the phrase *transitively dependent* **also** means that `B` does **not** determine `A`.

We could be a bit more concise and say that `C` is **transitively dependent** upon `A` if there are three sets of attributes `A`, `B`, and `C` satisfying:

1. `A->B` is a non-symmetric dependence
2. `B->C`

### A Clutch of Keys

A **superkey** is a combination of attributes that can uniquely identify a database record.  Since we don't normally allow duplicate records (we will go into that below), the set of ALL attributes will form a superkey in any reasonable table.

In the color/animal table above we have several superkeys: `{name}`, `{number}`, `{color, animal}`, `{name, color, animal}`, `{number, color, animal}`, `{name, number}`, `{name, color}`, … etc. In fact all combinations of the 4 attributes EXCEPT `{color}` and `{animal}` by themselves will work.

A **candidate key** is a superkey with no extraneous information (you can't drop any of its attributes and remain a superkey)  In our case there are 3:

* `{color, animal}`, 
* `{number}`, and 
* `{name}`.

Since we have, at a bare minimum, the set of all attributes as a superkey (see above for the caveats), we are also guaranteed to have at least one candidate key – we start with all the attributes and just drop attributes until we can't anymore.  If the set of all attributes can't lose any attribute and remain a super key then it is already a candidate key.  All candidate keys are, necessarily, superkeys.

Note that superkeys (and thus candidates keys) have the property that **every** individual attribute is functionally dependent upon the key.

It's possible for a candidate key to be comprised of a single attribute.

### Prime and Non-Prime Attributes

A **non-prime attribute** is one that does not occur in ANY candidate key.  The above table does not have any non-prime attributes.  Let's add one:

favorite color |   favorite animal |number|name   |bonus
---------------|-------------------|------|-------|----
gray           |cats               |    13|Peter  | a
gray           |snakes             |     7|Heather| a
black          |snakes             |    21|Katya  | a
black          |cats               |     2|Zet    | c

The attribute `bonus` is non-prime.  Let me reiterate what the means:  There is no candidate key containing the attribute `bonus`.  To see why this is so, consider potential candidate keys containing `bonus`.  We can immediately rule out any set of attributes with `name` or `number`:  They are candidate keys in their own right so any set containing **more** attributes has superfluous information.  So let's consider potential candidates keys containing `bonus` but not containing `number` or `name`:

set of attributes | discussion
------------------|--------------
`{bonus, color}`  | Not even a super key:  first two records share the same values for `bonus` and `color`
`{bonus, animal}` | Not even a super key:  records two and three share the same values for `bonus` and `animal`
`{bonus,animal,color}`| This **is** a super key… but not a candidate key.  The attribute `bonus` is redundant – dropping it produces a candidate super key.
	
The opposite of a non-prime attributes is a **prime-attribute**.  A prime-attribute occurs in at least one candidate key.

We are now ready for the icing on the cake:  A **primary key** is a candidate key that is considered to be the most-est special-est of all the candidate keys.   You get to decide what that means.  The other candidate keys can be called **alternate keys**.

I *think* the idea is that the primary key seldom (if ever) gets changed.  Many databases have mechanisms for making it easy to guarantee that a table has a primary key (that's what NOT NULL IDENTITY is for).

A **partial dependency** occurs when a *non-prime* attribute is functionally dependent on part of some *candidate key*.  (We will need this definition to explain some things below).  This is a very *specific* definition.  We are not looking at the relationship between any two generic sets of attributes – we are very specifically looking at *candidate keys* and their relationship with *non-prime attributes*.
	
## The Normal Forms

Data can be categorized based upon the amount of redundancy contained within the tables.  

### First Normal Form

Data is in **First Normal Form (1NF)** if the fields don't contain multiple values and if the rows are not duplicated (i.e., the rows form a _set_).  This is frequently described as **data is atomic** (there is some controversy over this definition).  This is best understood with an example

Here is a table in first normal form:

id      |first  |last |book
--------|-------|-------|-------
1       |Peter  |Dolan |Foundation
2       |Heather|Waye  |American Gods

Here is one that is not:

id      | first   | last   | book
--------|---------|--------|-------
1       | Peter   | Dolan  | Foundation<br>Jon Carter, Warlord of Mars<br>...
2       | Heather | Waye   | American Gods

Notice that the field `book` contains multiple examples.  This violates the conditions of first normal form.

We can however put the data into 1NF by making separate tables for each set of related attributes and giving each table a primary key:

id     |first  |last
-------|-------|-----
1      |Peter  |Dolan
2      |Heather|Waye

id  | book
----|------------
1   | Foundation
1   | Jon Carter
1   | ...
2   | American Gods
2   | etc.

#### Easy Conversion to First Normal Form

There are standard techniques for _normalizing_ database designs, i.e., changing them so that they are in first (or second or third) normal form.[^how-to-normalize] To convert a design so that it is in 1NF, for example, you want to _eliminate repeating groups_, i.e., get rid of situations where a single column is being used to hold one or more pieces of data, like "_Foundation_, _Jon Carter_, _Warlord of Mars_" above. We acoomplish this by making a separate table for each set of related attributes and give each table a primary key.  The separate tables allow the non-atomic data in an attribute to be separated into different records, and the primary key ensures that no two records are repeated in their entirety.

### Second Normal Form

Tables in **Second Normal Form (2NF)** are also in first normal form.  In addition, "no non-prime attribute is dependent on any proper subset of any candidate key of the table" (or, more concisely, there are no partial dependencies in the table).  

Another way to think about it is that the *entire* candidate key should be necessary to deduce any non-prime attributes.

Again, this is a bit easier to see with an example.  Here is a table that is NOT in second normal form:

<table>
<tr><th colspan=4 align="center">Employees' Skill</th></tr>
<tr><th>Employee</th><th>Skill</th><th>Current Work Location</th></tr>
<tr><td>Brown</td><td>Light Cleaning</td><td>73 Industrial Way</td></tr>
<tr><td>Brown</td><td>Typing</td><td>73 Industrial Way</td></tr>
<tr><td>Harrison</td><td>Light Cleaning</td><td>73 Industrial Way</td></tr>
<tr><td>Jones</td><td>Shorthand</td><td>114 Main Street</td></tr>
<tr><td>Jones</td><td>Typing</td><td>114 Main Street</td></tr>
<tr><td>Jones</td><td>Whittling</td><td>114 Main Street</td></tr>
</table>

This is clearly in First Normal Form:  The attributes are atomic and there are no repeated rows.   To figure out if the table is in second normal form we need to identify any partial dependencies.  We start by finding all 
non-prime attributes (if any) by first identifying all the candidate keys.  For this data there is only one: `{Employee, Skill}`.  That means `Current Work Location` is a non-prime attribute.  (Tables without non-prime attributes are automatically in second normal form). 

Having identified all non-prime attributes, we check to see if any are functionally dependent upon *any* subset of *any* candidate key.  This check is pretty easy – there's only one non-prime attribute (`Current Work Location`), and only two (proper) subsets (`{Employee}` and `{Skill}`).  It's pretty clear that `Employee->Current Work Location` (knowing `Employee` allows us to determine `Current Work Location`) so `Current Work Location` is functionally dependent upon `Employee`.  On the other hand, `Current Work Location` is NOT functionally dependent upon `Skill` since the skill 'Typing' occurs with both '73 Industrial Way' and with '114 Main Street'.  The functional dependency of a non-prime attribute (`Current World Location`) upon a proper subset of a candidate key (`{Employee}`) means the table is **not** in second normal form.

It's important here to understand the difference between using data and using "situational knowledge" to make inferences about functional dependencies. In our discussion of the `Employees' Skill` table above, we are using the table (data) to infer something about the functional dependencies by assuming that "if the data doesn't violate the rule – no new data will every violate it".  This may, or may not, be a good idea.  If the company employing this database has employees working in only one assigned location, then the situation supports the conclusion that `Current Work Location` depends on `Employee`.  On the other hand, if some employees work in different locations during different shifts, then there is **not** a functional dependency of the form `Employee->Current Work Location`.  I'm not trying to overwhelm you – I just want to make certain you are clear about the subtleties.

More informally, we look to see if a non-prime attribute depends upon only part of a multi-valued key and remove it into a separate table if it does:

<table>
<tr><th colspan=2 align="center">Employees</th></tr>
<tr><th>Employee</th><th>Current Work Location</th></tr>
<tr><td>Brown</td><td>73 Industrial Waye</td></tr>
<tr><td>Harrison</td><td>73 Industrial Waye</td></tr>
<tr><td>Jones</td><td>114 Main Street</td></tr>
</table>

<table>
<tr><th colspan=2 align="center">Employees' skill</th></tr>
<tr><th>Employee</th><th>Skill</th></tr>
<tr><td>Brown</td><td>Light Cleaning</td></tr>
<tr><td>Brown</td><td>Typing</td></tr>
<tr><td>Harrison</td><td>Light Cleaning</td></tr>
<tr><td>Jones</td><td>Shorthand</td></tr>
<tr><td>Jones</td><td>Typing</td></tr>
<tr><td>Jones</td><td>Whittling</td></tr>
</table>
	
#### Easy conversion to Second Normal Form

If a non-prime attribute depends on only part of a multi-valued candidate key, move it to a separate table.

### Third Normal Form

Tables in **Third Normal Form (3NF)** are also in 2NF.  Their defining characteristic is that *every non-prime attribute is non-transitively dependent upon every super key*.  That was one of the original definitions; I think the following equivalent definition is a bit easier to think about, and quite a bit easier for checking:

For **every** functional dependency `X->A`, **at least one** of the following holds:

* X (as a set of attributes) is a superset of A (this is a trivial functional dependency)
* X is a superkey
* Every attribute _a_ ∈ A - X is a prime attribute 
   * Here A-X is the _set difference_ between A and X, i.e., the elements of A not in X.

Here are a few things to help you make sense of these requirements.  First, recall that the components of a functional dependency of the form `X->A` are *sets* of attributes.  In other words `X` is a set of fields and `A` is a set of fields.

The first possibility in the list above means that all the attributes in `A` are also in `X`.  (So you can ignore functional dependencies that are set inclusions).

The second possibility in the list occurs when the attributes in `A` are functionally dependent upon `X`. This means that all the attributes in `A` are determined by (a subset of) the fields in `X`; if they weren't then `X` couldn't be a superkey. This is equivalent to saying that we can ignore all functional dependencies where distinct values of attributes in `X` are unique to the records.

The third possibility is saying that attributes in `A` that are NOT in `X` are all prime. Note: That having tables without non-prime attributes automatically makes this true.

Here is an example of a table that is in second normal form but that is NOT in third normal form:

<table>
<tr><th colspan=4 align="center">Tournament Winners</th></tr>
<tr><th>Tournament</th><th>Year</th><th>Winner</th><th>Winner Date of Birth</th></tr>
<tr><td>Indiana Invitational</td><td>1998</td><td>Al Fredrickson</td><td>21 July 1975</td></tr>
<tr><td>Cleveland Open</td><td>1999</td><td>Bob Albertson</td><td>28 September 1968</td></tr>
<tr><td>Des Moines Masters</td><td>1999</td><td>Al Fredrickson</td><td>21 July 1975</td></tr>
<tr><td>Indiana Invitational</td><td>1999</td><td>Chip Masterson</td><td>14 March 1977</td></tr>
</table>

As always, the key to checking normality lies in figuring out the candidate keys.  No individual attribute suffices for this table, so we turn our attention to pairs.  Let's start by consider pairs containing `Tournament`.  The only replication keeping `Tournament` from being a key in its own right is 'Indiana Invitational', which occurs twice.  

The pair `{Tournament,Year}` is a candidate key.  The data potentially allows for other candidate keys, but situationally speaking, this is the only candidate key.  From this point of view, that means that the attribute `Winner Date of Birth` is NOT part of any candidate key and hence a non-prime attribute.  Notice the following:

* `{Tournament, Year} -> Winner`
* `Winner does NOT determine {Tournament,Year}`  
* `Winner -> Winner Date of Birth`.

Thus the `Winner Date of Birth` is *transitively dependent* upon the candidate key, and the table is not in Third Normal Form.

I'm going to quote Wikipedia's article on this topic:

> The breach of 3NF occurs because the non-prime attribute `Winner Date of Birth` is transitively dependent on the candidate key `{Tournament, Year}` via the non-prime attribute `Winner`. The fact that `Winner Date of Birth` is functionally dependent on `Winner` makes the table vulnerable to logical inconsistencies, as there is nothing to stop the same person from being shown with different dates of birth on different records.

#### Easy Way to make a table Third Normal Form

If attributes do not contribute to a description of the key, remove them to a separate table. In this case the key is `{Tournament, Year}` is the key. `Winner Date of Birth`, however, doesn't tell us anything (directly) about either the `Tournament` or the `Year`; it describes the `Winner`, which isn't part of the key. So let's move that info to a separate table:

<table>
<tr><th colspan=3 align="center">Tournament Winners</th></tr>
<tr><th>Tournament</th><th>Year</th><th>Winner</th></tr>
<tr><td>Indiana Invitational</td><td>1998</td><td>Al Fredrickson</td></tr>
<tr><td>Cleveland Open</td><td>1999</td><td>Bob Albertson</td></tr>
<tr><td>Des Moines Masters</td><td>1999</td><td>Al Fredrickson</td></tr>
<tr><td>Indiana Invitational</td><td>1999</td><td>Chip Masterson</td></tr>
</table>

<table>
<tr><th colspan=2 align="center">Winner Dates of Birth</th></tr>
<tr><th>Winner</th><th>Winner Date of Birth</th></tr>
<tr><td>Al Fredrickson</td><td>21 July 1975</td></tr>
<tr><td>Bob Albertson</td><td>28 September 1968</td></tr>
<tr><td>Chip Masterson</td><td>14 March 1977</td></tr>
</table>

### Boyce-Codd Normal Form 

This is also known as 3.5NF.  This form is only slightly stronger than Third Normal Form, but all redundancy based upon functional dependency has been removed.

The key idea is that a table is in BCNF if and only if for every dependency `X->Y`, at least one of the following conditions holds:

* `X->Y` is a trivial functional dependency (X ⊃ Y)
* `X` is a super key.

Notice that this definition contains the first two items in the list that defined 3NF.  Each of the list items was an allowed *possibility*.  We have removed one of the possibilities, so the requirements are more stringent and harder to meet.

There are a lot of subtleties about this particular form; see [Wikipedia's discussion](https://en.wikipedia.org/wiki/Boyce%E2%80%93Codd_normal_form) for more.

### Summary of Normal Forms

There is a rather pithy way to help remember the first three (and a half) normal forms.  It's attributed to Bill Kent: 

> [Every] non-key [attribute] must provide a fact about the key, the whole key, 
> and nothing but the key... so help me Codd"

Let's break that down:

* **the key** refers to first normal form:  The table must have a key (and the data  must be atomic)
* **the whole key** refers to second normal form:  every non-key attribute (non-prime) must depend upon the **entire** candidate key
* **And nothing but the key** refers to third normal form:  the non-prime attributes need to depend *only* upon the keys and not upon something else. (The non-symmetry in the first step of a transitive dependence `A->B->C`, ensures that `B` is not a candidate key, so the existence of such things violates 3NF).
* **So help me Codd** refers to BCNF:  Everything else has to hold, but it must hold for ALL attributes, not just non-prime ones.

## Checklist of what to do

Things to include on a group Google Doc turn in on Canvas:

- [ ] Indicate which database your group used for your answers
- [ ] Share your group's thoughts on potential DB design
- [ ] Make sure everyone in your group has edit permissions on the document
- [ ] Submit a link to the document on Canvas that provides **comment priviledges** to people with that link

Things we'll look for in the database:

- [ ] All members of your group have been `GRANT`ed all privileges on the DB you used for this lab
- [ ] The results of the [anomaly exercise](#anomaly-exercise)
- [ ] Each group should create their own tables with the following properties:
    - [ ] A table in first normal form, but not second normal form
    - [ ] A table in second normal form, but not in third normal form
    - [ ] A table in third normal form, but not in BNC form

Things you should do, but where there's nothing to "turn in":

- [ ] [The quick check exercises](#quick-check)
- [ ] Type in all (or most) of the examples in this lab

[^anomalies]: [SQA's _Fundamentals of Database Design_](https://www.sqa.org.uk/e-learning/MDBS01CD/index.htm) and Elmasri and Navathe's _Fundamentals of Database Systems_  
[^normalization]: [Wikipedia's entry on "Database normalization"](https://en.wikipedia.org/wiki/Database_normalization)
[^how-to-normalize]: [DataModel.org's "Rules of Data Normalization"](http://web.archive.org/web/20080805014412/http://www.datamodel.org/NormalizationRules.html)
