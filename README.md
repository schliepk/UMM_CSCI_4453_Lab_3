# Lab 3

## Table of contents

## Outline
   - [Overview](#overview)
   - [More Exercises](#a-more-in-depth-set-of-exercises)
     - [Creating some tables](#some-simple-tables)
     - [Quick Check](#quick-check)
   - [Anomalies](#anomalies)
     - [Modification Anomaly](#modifcation-anomaly)
     - [Deletion Anomaly](#deletion-anomaly)
     - [Insertion Anomaly](#insertion-anomaly)
     - [Anomaly Exercise]#[anomaly-exercise)
   - [Data Normalization](#data-normalization)
     -[Attributes and Functional Dependence](#attributes-and-functional-dependence)
     -[A Dash of Dependency](#a-dash-of-dependency)
     -[A Clutch of Keys](#a-clutch-of-keys)
     -[Prime and Non-Prime Attributes](#prime-an-non-prime-attributes)
   - [Normal Forms](#normal-forms)
     -[First Normal Form](#first-normal-form)
     -[Second Normal Form](#second-normal-form)
     -[Third Normal Form](#third-normal-form)
     -[Boyce Codd Normal Form][boyce-codd-normal-form)
     -[Summary](#summary)
   - [Checklist](#checklist)
   

## Overview

In today's lab we are going to get to know more about the `SELECT` statement and a bit about Database Design Theory.  At this point you should be able to do `SELECT`s that involve multiple tables and join them together with a `WHERE` clause, or explicitly with a `LEFT` or `RIGHT JOIN`.   You should also be able to do some nontrivial filtering involving the `WHERE` clause.  At the moment you have a toy version of the inventory table and a toy version of the prices table.  Today's lab will require SEVERAL tables and mucking about... so you only need **ONE copy per group**.  You will have to be sure to tell me where to look for your data, and who is in your group, so make a note in [[lab 3 group wiki|groups]].

You are going to be making a few tables to play with, so just in case you make a few mistakes and don't want to `DROP` the table and start over, read this before continuing:

<https://mariadb.com/kb/en/alter-table/>

(You don't really know what an index is yet... but you will soon)

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

I would like each group to start a document (about a page long) detailing a rough outline of what you think would be necessary-- we'll flesh this out later.  Make your document link-able from [[Group Planning wiki|planning]] (a link to another Wiki page is fine-- ask if you're not sure how to it)

## A more in-depth set of exercises

You are now going to create several tables that will help you develop a deeper understanding of what you will need to do to develop a better Point-Of-Sales System.

### Some simple tables

Let's create a table called `master` that will keep track of each transactions on a global level

You should make it with the following fields:

field    | data-type      | notes
---------|----------------|-----------
start    | datetime       | for keeping track of when the interaction began
stop     | datetime       | and for keeping track of when it's done
tid      | integer        | unique id for keeping track of items in all transactions (should be the primary key)
register | integer        | numeric id of cash register WHERE transaction occurred
user     | text           | who was running the till
total    | decimal with two places | (the total amount of money received (or refunded))
	
Let call the table for keeping track of the items in a transaction `t_items`:

field    | data-type               | notes
---------|-------------------------|-------------------------------------------
tid      | same as master table    | allows the individual item to be associated to the till transaction
pid      | same as inventory table | allows the individual item to be associated with their description
price    | decimal                 |in some ways we're breaking normalization by doing this, but a coke might cost 1.92 one day or it might be on sale for .75 another day... so arguably we're not.  Plus-- someone might be buying TWO cokes instead of one)
gid      | integer                 | (group id) we need a way to deal with combo's and this will lets us keep track of which items are grouped
amount   | decimal                 |  (how many).  We might need 
	
Consider a table called `combos` (read a bit further before creating this):

field    |Description
---------|---------------------
uid      |  Unique Id 
comboName|  Name of the combo
item     |  Should correspond to a product id in inventory
price    |  The amount of the combo
comboCode|  A number unique to the combo
	
We could keep track of combos  with this table.  It might look like this:

uid |  comboName         |   item   | price  | comboCode
----|--------------------|----------|--------|---------------   
1   |  Double Slam       |         2|    1.99| 1
2   |Double Slam         |3|1.99|1
3   |Quacker Jack special|2|2.15|2
4   |Quacker Jack special|17|2.15|2
5   |Quacker Jack special|191|2.15|2
	
Okay… NOW build that table but give it a name like `poorDesign`.  Add the 5 records indicated above, but use id's from your own inventory table (we will fix things soon… or more accurately *you* will).

### Quick check (you are not turning this in... but you still need to do it)

Before we put things in order, let's check your understanding of last classes material:

Construct a query that links together data from the `poorDesign` table, and from your `inventory` table in order to provide the following information:
* comboName, 
* comboPrice, and
* item.

So, for example, if item 2 was `Brawndo`, and item 3 was `Human Kidneys`, then your query should return the following:
```
Double Slam	1.99	Brawndo
Double Slam	1.99	Human Kidneys
```

I want the columns of your query to have the names `combo`, `price`, `item`.  You may find it helpful to review the usage of the keyword `AS` in the documentation on the `SELECT` statement.

You really must be tired of all those `INSERT` statement by now (I know I am), so read this:

<https://mariadb.com/kb/en/load-data-infile/>

Pay special attention to the effect of the word `LOCAL`... it's the difference between your command work, and failing.

Now let's get even MORE CREATIVE.  Create a query that links information from `poorDesign`, `inventory`, and `price` to display the following information:
* comboName, 
* comboPrice, 
* item, and 
* itemPrice

This is the first time you've done a query that uses three tables.  

Once you get the query to run correctly, you have implicitly begun to understand the secret to something called **data normalization**.  What you have done with your query is shown that redundant (or duplicated) information in your output can be generated, as necessary, by a good query.  

If you don't normalize your data (more on that very soon), then you run the risk of creating an **Anomaly**-- even if your framework is amazing and double checks everything very carefully-- bad things can still happen accidentally;  Suppose your internet connection is cut off in the middle of updating?  However, if your data-design is good (which often means, among other things, that there is no redundant information in any of your tables), then that can't happen-- the essential information appears in only one place and it only takes one atomic update to make a change.

## Anomalies

**Anomalies** occur when the data in your tables is not consistent (either internally or with the real-world system it is describing).  

Different authors define terms a bit differently but broadly speaking, anomalies fall into the following general categories based upon the cause of the problem[1] (examples will follow the definitions):

* **Modification anomalies** (some call these **update anomalies**) occur when duplicated data is updated... but not every copy is changed.
* **Deletion Anomalies** occur when removing a record from a table permanently removes information that should be retained.
* **Insertion Anomalies** occurs when new information that *should* be included in the data can't be added without the presence of other attributes.

To make sense of these examples consider the following table:

StudentNum | CourseNum | Student Name | Address    | Course
-----------|-----------|--------------|------------|-------------
S21        | 9201      | Jones        | Edinburgh  | Accounts
S21        | 9267      | Jones        | Edinburgh  | Accounts
S24        | 9267      | Smith        | Glasgow    | Physics
S30        | 9201      | Richards     | Manchester | Computing
S30        | 9322      | Richards     | Manchester | Maths


### Modification Anomaly

Notice that the student `Jones` appears in two rows.  If we updated his address in one row, but not both, then that would be a modification anomaly.  As previously mentioned, this is also called this an **update anomaly** by some authors.

#### Deletion Anomaly

Assuming this table is our only source of information, then if student `S30` drops out we no longer have a record of the 'Computing' or 'Maths' course.  The ability to list the courses being offered should not depend upon a how many students are enrolled in the class.

#### Insertion Anomalies

We will look at two insertion anomalies.  The first is related to the deletion anomaly:  Suppose you want to introduce a course in 'Quantum', but nobody is enrolled in the class yet.  The only way to do it requires using a NULL value to (essentially) leave some attributes blank:

StudentNum | CourseNum | Student Name | Address    | Course
-----------|-----------|--------------|------------|-------------
NULL       | 00X3      | NULL         | NULL       | Quantum

(It **can** be done... but it is very unsatisfying and requires that extra steps be taken to tell the difference between a "normal" record and one purely related to which courses exist.)

Another problem is related to the update anomaly:  Suppose you want to add a new student to the database.  The student is enrolled in `CourseNum` '9322'.  We would have to make certain that the `Course` and `CourseNum` match in all records.  If we did not do this correctly that would also be an Insertion Anomaly.

### Anomaly Exercise

**Exercise**:  Each group should create the table above (structure *and* data), and then update *one* record in such a way as to induce a modification anomaly.

## Data Normalization

**Data Normalization** is the process of organizing the fields and tables (of a relational database) to minimize redundancy and dependency (thanks wikipedia!).  The goal of normalizing your data is the following:

* No information redundancy
* Not storing unrelated data in the same table (this helps protect against the *anomalies* we just discussed)

### Attributes and Functional Dependence

Columns or fields are typically called **attributes** in database design books.  An attribute is said to have a **functional dependency** upon another attribute if knowing the value of the second attribute allows one to determine the value of the first.  This is less complicated than it sounds.

Let's consider the table we were using up above.

Notice that the attribute `Course` is functionally dependent upon `CourseNum`, because if I know `CourseNum`, then I should be able to deduce the `Course`.  We write this:  

`CourseNum->Course`.  

In my head I say:  If I know `CourseNum` then I can deduce `Course`, or even more briefly, `CourseNum` *determines* `Course`.  Just remember (and this always trips me up) that saying "the attribute `CourseNum` determines `Course`" means the same thing as saying "`Course` is functionally dependent (some books just say *has a dependence*) upon `Coursenum`.  The arrow does NOT mean *functionally depends* (at least not if you're left-to-right)
	
Often the relationship is not symmetric (ie, it does not go both ways)-- The attribute `CourseNum` is **not** functionally dependent upon `Course`,  because knowing the value of `Course` does not determine the value of `CourseID`.  At most universities (like our own) there are many classes with the same name, but different IDs.  Take, for example, 'Calculus I'.  Each section of the course has its own unique ID.  The fact that *some* classes are uniquely paired with their IDs doesn't matter for this definition.
	
The idea of functional dependence extends to **sets of attributes**.  In the table above we abbreviated address... but assuming that the attribute `Address` was actually a complete address then `StudentNum` is functionally dependent upon the pair `{Address, Student Name}`. We would write this concisely as `{Address, Student Name} -> {StudentID}`.

Depending upon the size of the university it is possible that `StudentNum` is functionally dependent upon `Student Name` and vice-versa, but honestly, it would be better to head of any future problems by assuming that `StudentNum->Student Name` and not the other way around.  However, the ultimate arbiter of functional dependency is the situation being modeled. Notice that it is also true that `StudentNum->{Student Name, Address}`.

There are two important ideas to keep in mind when thinking about these issues

1. When does repetition in the values of the attributes under consideration make functional dependency impossible?  This can sometimes be determined by looking at data that is going to be in the database.
2. What does the situation being modeled have to say about any functional dependencies?

Consider the following table that relates various preferences between individuals.  Assume, that it is an accurate reflection of the real world and that meaningful functional dependencies can be inferred purely from the data in the table (we are pretending that point 2 does not enter into consideration):

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

There is clearly a difference between the situations described up above.  We say that the attribute `name` has a **full functional dependency** upon `{color,animal}` because dropping any attribute in `{color, animal}` destroys the dependency.  
	
You might be beginning to see the mathematics supporting the structure behind the database system.  Let's let `A`, `B`, and `C` stand for sets of attributes.

Hopefully it will seem fairly obvious that if `A->B and B->C` then `A->C`.  (You may recognize this as **transitivity**)

In database design we say that attribute `C` is **transitively dependent** upon `A` if

1. `A->B`
2. **AND** It is not true that `B->A`
3. **AND** `B->C`

It is always that case that `A->B` and `B->C` implies `A->C` (which is what a mathematician recognizes as transitive), but in database design the phrase *transitively dependent* **also** means that `B` is **not** determined by `A`.

We could be a bit more concise and say that `C` is **transitively dependent** upon `A` if there are three sets of attributes `A`, `B`, and `C` satisfying:

1. `A->B` is a non-symmetric dependence
2. `B->C`

### A Clutch of Keys

A **superkey** is a combination of attributes that can uniquely identify a database record.  Since we don't normally allow duplicate records (we will go into that below), the set of ALL attributes will form a superkey in any reasonable table.

In the table above we have several superkeys:

`{name}, {number}, {color, animal}, {name, color, animal}, {number, color, animal}, {name, number}, {name, color},` … etc.  

In fact all combinations of the 4 attributes EXCEPT `{color}` and `{animal}` by themselves will work.

A **candididate key** is a superkey with no extraneous information (you can't drop an attribute and remain a superkey)  In our case there are 3:
* `{color, animal}`, 
* `{number}`, and 
* `{name}`.

Since we have, at a bare minimum, the set of all attributes as a superkey (see above for the caveats), we are also guaranteed to have at least one candidate key-- we just drop attributes until we can't.  If the set of all attributes can't lose any attribute and remain a super key then it is already a candidate key.  All candidate keys are, necessarily, superkeys.

Note that superkeys (and thus candidates keys) have the property that **every** individual attribute is functionally dependent upon the key.

It's possible for a candidate key to be comprised of a single attribute.

### Prime and Non-Prime Attributes

a **non-prime attribute** is one that does not occur in ANY candidate key.  The above table does not have any non-prime attributes.  Let's add one:

favorite color |   favorite animal |number|name   |bonus
---------------|-------------------|------|-------|----
gray           |cats               |    13|Peter  | a
gray           |snakes             |     7|Heather| a
black          |snakes             |    21|Katya  | a
black          |cats               |     2|Zet    | c

The attribute `bonus` is non-prime.  Let me reiterate what the means:  There is no candidate key containing the attribute `bonus`.  To see why this is so, consider potential candidate keys containing `bonus`.  We can immediately rule out any set of attributes with `name` or `number`:  They are candidate keys in their own right so any set containing **more** attributes has superfluous information.  So... let's consider potential candidates keys containing `bonus` but not containing `number` or `name`:

set of attributes | discussion
------------------|--------------
`{bonus, color}`  | Not even a super key:  first two records share the same values for bonus and color
`{bonus, animal}` | Not even a super key:  records two and three share the same values for bonus and animal
`{bonus,animal,color}`| This **is** a super key... but not a candidate key.  The attribute `bonus` is redundant-- dropping it produces a candidate super key.
	
The opposite of a non-prime attributes is a **prime-attribute**.  A prime-attribute occurs in at least one candidate key.

We are now ready for the icing on the cake:  A **primary key** is a candidate key that is considered to be the most-est special-est of all the candidate keys.   You get to decide what that means.  The other candidate keys can be called **alternate keys**.

I *think* the idea is that the primary key seldom (if ever) gets changed.  Many databases have mechanisms for making it easy to guarantee that a table has a primary key (that's what NOT NULL AUTO_INCREMENT is for)

A **partial dependency** occurs when a *non-prime* attribute is functionally dependent on part of some *candidate key*.  (We will need this definition to explain some things below).  This is a very *specific* definition.  We are not looking at the relation ship between any two generic sets of attributes-- we are very specifically looking at *candidate keys* and their relationship with *non-prime attributes*
	
## The Normal Forms

Data can be categorized based upon the amount of redundancy contained within the tables.  

### First Normal Form

Data is in **First Normal Form (1NF)** if the fields don't contain multiple values and if the rows are not duplicated.  This is frequently described as **data is atomic** (there is some controversy over this definition).  This is best understood with an example

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

#### Easy Conversion to First Normal Form[2]

Eliminate Repeating Groups:  Make a separate table for each set of related attributes and give each table a primary key.  The separate tables allow the non-atomic data in an attribute to be separated into different records, and the primary key ensures that no two records are repeated in their entirety.

### Second Normal Form

Tables in **Second Normal Form (2NF)** are also in first normal form.  In addition, "no non-prime attribute is dependent on any proper subset of any candidate key of the table" (or, more concisely, there are no partial dependencies in the table).  

Another way to think about it, is that the *entire* candidate key should be necessary to deduce any non-prime attributes.

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
non-prime attributes (if any) by first identifying all the candidate keys.  For this data there is only one: `{Employee,Skill}`.  That means `Current Work Location` is a non-prime attribute.  (Tables without non-prime attributes are automatically in second normal form).  Having identified all non-prime attributes, we check to see if any are functionally dependent upon *any* subset of *any* candidate key.  This check is pretty easy-- there's only one non-prime attribute, and only two subsets.  It's pretty clear that `employee->work location` (knowing `employee` allows us to determine `work location`) so `work location` is functionally dependent upon `employee`.  On the other hand, `work location` is NOT functionally dependent upon `Skill` since the skill 'Typing' occurs with '73 Industrial Way' and with '114 Main Street'.  The functional dependency of a non-prime attribute upon a proper subset of a candidate key means the table is **not** in second normal form.

I would like to, once more, point out the difference between using data and using "situational knowledge" to make inferences about functional dependencies .  We are using the table (data) to infer something about the functional dependencies by assuming that "if the data doesn't violate the rule-- no new data will every violate it".  This may, or may not, be a good idea.  If the company employing this database has employees working in only one assigned location, then the situation supports the conclusion.  On the other hand, if some employees work in different locations during different shifts, then there is **not** a functional dependency of the form `employee->work location`.  I'm not trying to overwhelm you-- I just want to make certain you are clear about the subtleties.

More informally-- we look to see if a non-prime attribute depends upon only part of a multi-valued key and remove it into a separate table if it does:

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
	
#### Easy conversion to Second Normal Form [2]

If an attribute (non-prime) depends on only part of a multi-valued key, remove it to a separate table.

### Third Normal Form

Table in **Third Normal Form (3NF)** are also in 2NF.  Their defining characteristic is that *every non-prime attribute is non-transitively dependent upon every super key*.  That was one of the original definitions… I think the following equivalent definition is a bit easier to think about, and quite a bit easier for checking:

For **every** functional dependency `X->A`, **at least one** of the following holds:

* X contains A (this is a trivial functional dependency)
* X is a superkey
* Every elt in A *set minus* X is a prime attribute

Here are a few things to help you make sense of these requirements.  First, recall that the components of a functional dependency of the form `X->A` are *sets* of attributes.  In other words `X` is a set of fields and `A` is a set of fields.

The first possibility in the list above means that all the attributes in `A` are also in `X`.  (So you can ignore functional dependencies that are set inclusions).

The second possibility in the list occurs when the attributes in `A` are functionally dependent upon `X`.  This is equivalent to saying that we can ignore all functional dependencies where distinct values of attributes in `X` are unique to the records.

The third possibility is saying that attributes in `A` that are NOT in `X` are all prime.

Note:  That having tables without non-prime attributes automatically makes this true.  

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

```
The breach of 3NF occurs because the non-prime attribute Winner Date of Birth is transitively dependent on the candidate key {Tournament, Year} via the non-prime attribute Winner. The fact that Winner Date of Birth is functionally dependent on Winner makes the table vulnerable to logical inconsistencies, as there is nothing to stop the same person from being shown with different dates of birth on different records.
```

#### Easy Way to make a table Third Normal Form [2]
If attributes do not contribute to a description of the key, remove them to a separate table.

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
* `X->Y` is a trivial functional dependency (Y &#8834; X)
* `X` is a super key.

Notice that this definition contains the first two items in the list that defined 3NF.  Each of the list items was an allowed *possibility*.  We have removed one of the possibilities, so the requirements are more stringent and harder to meet.

There are a lot of subtleties about this particular form:

<https://en.wikipedia.org/wiki/Boyce%E2%80%93Codd_normal_form>

### Summary of Normal Forms

There is a rather pithy way to help remember the first three (and a half) normal forms.  It's attributed to Bill Kent: 

```
"[Every] non-key [attribute] must provide a fact about the key, the whole key, and nothing but the key... so help me Codd".
```

Let's break that down:

* **the key** refers to first normal form:  The table must have a key (and the data  must be atomic)
* **the whole key** refers to second normal form:  every non-key attribute (non-prime) must depend upon the **entire** candidate key
* **And nothing but the key** refers to third normal form:  the non-prime attributes need to depend *only* upon the keys and not upon something else (The non-symmetry in the first step of a transitive dependence `A->B->C`, ensures that `B` is not a candidate key, so the existence of such things violates 3NF).
* **So help me Codd** refers to BCNF:  Everything else has to hold, but it must hold for ALL attributes-- not just non-prime ones.

## Checklist of what to do

* Indicate your group name, members in a file named `group.txt` that gets uploaded with your repository
* Create about a page's worth of speculation on a point of sales system in a file named `speculations.txt`
* Do the required checks (not turned in)
* Do the Anomaly Exercises (one per group-- indicate WHO has the answers)
* Each group should create their own table with the following properties:
    * In first normal form, but not second normal form
    * In second normal form, but not in third normal form
    * In third normal form, but not in BNC form


[1] SQA's _Fundamentals of Database Design_ and _Elmasri and Navathe_  
[2] http://web.archive.org/web/20080805014412/http://www.datamodel.org/NormalizationRules.html
