# Nic tries to understand normal forms

In class it was clear that _lots_ of people were confused about normal forms, especially 3NF and BCNF. As I attempted to better explain things, I felt like I was personally getting more and more confused, and that I certainly wasn't helping folks understand these important concepts.

When I got back to my office I grabbed a bunch of DB books off the shelf and spent some time trying to clarify my own understanding in the hopes that I could better explain these ideas.

First, I'm increasingly suspicious of [the Wikipedia page on "Database normalization"](https://en.wikipedia.org/wiki/Database_normalization). There's a flag on that page from March, 2018, essentially asking for a DB expert to look things over, and I'm definitely not convinced that all the examples that are currently on that page are in fact correct.

Second, reading different presentations of normal forms in books, blog posts, etc., can make one feel like everyone's speaking totally different languages. It is _remarkable_ how different the language and terminology is from one writer to the next, which makes it very hard to understand how different presentations of the same ideas connect, or even decide which ones are "correct". In a perfect world I'd go back to the original papers by Codd, Boyce & Codd, etc., and see how _they_ actually defined these things, but I don't have time for that.

I suspect part of the problem is that in some ways the "natural" definitions are mathematical (these are, after, essentially set-theoretic properties of mathematical entities), but many authors try to avoid formal math, especially when they feel that there's a more "intuitive" way to present things. To make it worse, I think that at least some presentations are people trying to "math up" an "intuitive" version based on some other "math-y" version, and we end up with a game of definitional telephone. It's a bit like letting Google Translate something from English to Hindi, and then translate the resulting Hindi into Russian, and translating that to Hebrew, and then Japanese, Swahili, French, etc., before finally returning to English. Here, of course, it's all human translators, and they all no doubt believe that they're making things clearer in some way, but frankly I find it a bit of a mess.

All that said, here's my attempt at clarifying the first few normal forms.

– @NicMcPhee, 6 Feb 2019

---

# First normal form (1NF)

In some sense this one is easy – if you can get your favorite DB software to accept your table and data, it's probably in first normal form.

There are still some subtleties and variations of opinion, though. The one practical bit is no column should contain what are effectively lists. So we don't want a column "Favorite books" that contains several values.

So a table like:

| Reader | Member since | Favorite Books |
|--------|--------------|----------------|
| Nic    | March 2013 | "The Mind's I", "The Immortal Life of Henrietta Lacks", "Understanding Comics", "The Lorax", "The Stone Sky" |
| Susan  | August 2014 | "Trans Like Me", "Little Fires Everywhere", "The Goblin Emperor", "The Stone Sky" |
| Thomas | November 2012 | "A Clockwork Orange", "The Laramie Project", "The Only Harmless Great Thing", "Her Body and Other Parties", "The Stone Sky" |
| Paddington | Last week | "The Lorax" |

isn't in 1NF because we're cramming all kinds of things into the "Favorite Books" column. (If you find yourself wanting to use a plural in an attribute name, that's often a sign that you're at risk of violating this rule.)

We fix this by splitting the list data off into it's own table, with a shared key to connect the two tables; I'll use the "Reader" as the shared key here:

| Reader | Member since |
|--------|--------------|
| Nic    | March 2013 |
| Susan  | August 2014 |
| Thomas | November 2012 |
| Paddington | Last week |

| Reader | Favorite book |
|--------|---------------|
| Nic    | "The Mind's I" |
| Nic    | "The Immortal Life of Henrietta Lacks" |
| Nic    | "Understanding Comics" |
| Nic    | "The Lorax" |
| Nic    | "The Stone Sky" |
| Susan  | "Trans Like Me" |
| Susan  | "Little Fires Everywhere" |
| Susan  | "The Goblin Emperor" |
| Susan  | "The Stone Sky" |
| Thomas | "A Clockwork Orange" |
| Thomas | "The Laramie Project" |
| Thomas | "The Only Harmless Great Thing" |
| Thomas | "Her Body and Other Parties" |
| Thomas | "The Stone Sky" |
| Paddington | "The Lorax" |

# Second normal form (2NF)

We'll start with a somewhat simplified version: For a table to be in 2NF we need every attribute to depend on _all_ the attributes in the primary key.

Consider this table:

| Reader | Favorite book | Publisher |
|--------|---------------|----------|
| Nic    | "The Mind's I" | Basic Books |
| Nic    | "The Immortal Life of Henrietta Lacks" | Broadway Books |
| Nic    | "Understanding Comics" | William Morrow Paperbacks |
| Nic    | "The Lorax" | Random House |
| Nic    | "The Stone Sky" | Orbit |
| Susan  | "Trans Like Me" | Seal Press |
| Susan  | "Little Fires Everywhere" | Penguin |
| Susan  | "The Goblin Emperor" | Tor |
| Susan  | "The Stone Sky" | Orbit |
| Thomas | "A Clockwork Orange" | W. W. Norton & Company |
| Thomas | "The Laramie Project" | Dramatists Play Service |
| Thomas | "The Only Harmless Great Thing" | Tor |
| Thomas | "Her Body and Other Parties" | Graywolf Press |
| Thomas | "The Stone Sky" | Orbit |
| Paddington | "The Lorax" | Random House |

Let's assume our primary key is `{ Reader, Favorite Book }`; that pair is certainly unique for each row and is in fact the only candidate key.

The problem is that the `Publisher` column in fact only depends on the `Favorite book` column, so it only depends on _part of the key_. This is what we mean by a _partial dependency_, and 2NF is about eliminating partial dependencies.

2NF is often defined specifically in terms of the _primary_ key, but it should in fact hold for _all_ candidate keys, i.e., for any candidate key, we need all other other attributes to depend on _all_ of that candidate key, and not just part of it. If all we paid attention to was the primary key, we could "fix" 2NF problems by just adding a unique ID attribute and using that as the primary key. That wouldn't eliminate the duplication above, though, which is way we really need it to hold for all candidate keys.

Violations of 2NF are usually because we're storing two essentially different things in a single table; in this example we're storing publisher information in the "Favorite Books" table, and it really doesn't belong there. So we split it out into its own table to fix the problem:

| Title | Publisher |
|-------|-----------|
| "The Mind's I" | Basic Books |
| "The Immortal Life of Henrietta Lacks" | Broadway Books |
| "Understanding Comics" | William Morrow Paperbacks |
| "The Lorax" | Random House |
| "The Stone Sky" | Orbit |
| "Trans Like Me" | Seal Press |
| "Little Fires Everywhere" | Penguin |
| "The Goblin Emperor" | Tor |
| "A Clockwork Orange" | W. W. Norton & Company |
| "The Laramie Project" | Dramatists Play Service |
| "The Only Harmless Great Thing" | Tor |
| "Her Body and Other Parties" | Graywolf Press |

Now there's just one place where, for example, we record that "The Stone Sky" was published by Orbit, where before that was in three places, creating all sorts of opportunities for anamolies.

# Third normal form (3NF)

Here's where things start to get weird and where different sources say things in _very_ different ways. Probably the simplest description is that to be in 3NF there can be no transitive dependencies where `A -> B -> C` for some candidate keys `A`, `B`, and `C`.

Consider, for example,

| Name | Zip code | City | State |
|------|----------|------|-------|
| Nic  | 76302    | Wichita Falls | Texas |
| Thomas | 56267 | Morris | Minnesota |
| Misty | 54901 | Oshkosh | Wisconsin |
| Susan | 56267 | Morris | Minnesota |

Here our only option for a candidate key is `{ Name }`; everything else is duplicated on the `Thomas` and `Susan` rows. The problem is that Zip codes determine the city and state, so we have a transitive dependency 

```{ Name } -> { Zip code } -> { City }```

There's a similar transitive dependency ending in `{ State }`.

That transitive dependency is bad because, again, it leads to duplicate data (the city and state for 56267).

The more formal definition provided by Peter (and included in many other sources is that if `A -> B` holds then at least one of these holds:

   * `A` is a superset of `B`
   * `A` is a superkey
   * Each attribute p in `B - A` is _prime_, i.e., a member of at least one candidate key in this table.

Here we have that `{ Zip code } -> { City }` is true but:

   * `{ Zip code }` obviously isn't a superset of `{ City }`
   * `{ Zip code }` isn't a superkey (it doesn't uniquely determine all the rows)
   * The one attribute in `{ City } - { Zip code }` is `City`, and `City` isn't in any candidate key for this table since the only candidate key is `{ Name }`.

Thus all three conditions fail, and this table is not in 3NF.

In Peter's presentation of the transitive dependency condition, he had the requirement that not only should `A -> B -> C`, but that it _not_ be the case that `B -> A`. In our address example this holds because `{ Zip code } -> { Name }` isn't true. What does an example look like where this doesn't hold?

There's an example below of a table that's in BCNF (and thus 3NF) where there are transitive dependencies, but `A` is always a superkey so everything's OK. 

Can we find an example where the first two fail, but the third condition succeeds, giving us 3NF? Consider the following table:

| First name | Last name | Street | City | Position in household |
|------------|-----------|--------|------|-------------|
| Susan | Gilbert | Nevada | Morris | 1 |
| Nic | McPhee | Nevada | Morris | 2 |
| Thomas | McPhee | Nevada | Morris | 3 |
| Thomas | The Tank Engine | Track 2 | Big Station | 1 |
| Edward | The Blue Engine | Track 2 | Big Station | 2 |
| Gordon | The Big Engine | Track 2 | Big Station | 3 |

Let's assume for the purposes of the example that 

```{ First name, Last name }``` 

will always be unique, and thus can be a candidate key. Similarly, let's assume that 

```{ Street, City, Position in household }``` 

is always uniqe, and thus _that_ can be a candidate key as well.

There are a variety of dependencies in this table; one is:

```
{ Last name } -> { Street }
```

(This probably wouldn't last as a dependency as we add more data, but it's true for this set of data.)

The first two 3NF cases fail:

   * `{ Last name }` is not a superset of `{ Street }`
   * `{ Last name }` is not a superkey

What about the third case? The only attribute in

```{ Street } - { Last name }```

is `Street `. It _is_ a prime attribute, however, because it's part of the candidate key

```{ Street, City, Position in household }.```

Thus this particular dependency doesn't violate 3NF. In fact this table is in 3NF because _every_ attribute in the table is prime (part of one of the two candidate keys), so the third condition will _always_ hold. That seems troublesome, though, because we can see lots of duplication in this table; BCNF will fix that.

# Boyce-Codd Normal Form (BCNF)

Where for 3NF every dependency `A -> B` must satisfy at least one of:

   * `A` is a superset of `B`
   * `A` is a superkey
   * Each attribute p in `B - A` is _prime_, i.e., a member of at least one candidate key in this table.

in Boyce-Codd Normal Form (BCNF) we eliminate the third option, requiring that every dependency `A -> B` must satisfy either of these two conditions:

   * `A` is a superset of `B`
   * `A` is a superkey

This suggests that our previous address example might be in trouble because we _needed_ the third condition to "save the day" and allow the dependency

```{ Last name } -> { Street }```

to be "accepted". Since BCNF doesn't allow that third condition, our address table is _not_ in BCNF. This seems reasonable since there does seem to be two separate (but related) sets of data being stuffed in this table:

   * Info about people (names, position in household)
   * Location of household (street, city)

So we probably want to split that up:

| ID | First name | Last name | Household ID | Position in household |
|----|------------|-----------|--------------|---------|
| 1 | Susan | Gilbert | 32 | 1 |
| 2 | Nic | McPhee | 32 | 2 |
| 3 | Thomas | McPhee | 32 | 3 |
| 4 | Thomas | The Tank Engine | 17 | 1 |
| 5 | Edward | The Blue Engine | 17 | 2 |
| 6 | Gordon | The Big Engine | 17 | 3 |

| Household ID | Street | City |
|--------------|--------|------|
| 32 | Nevada | Morris |
| 17 | Track 2 | Big Station |

Now consider another example that's in BCNF to begin with:

| Name | Employee ID | Zip code |
|------|-------------|----------|
| Nic  | 42 | 76302    |
| Thomas | 87 | 56267 |
| Misty | 16 | 54901 |
| Susan | 22 | 56267 |

If we make the fairly ludicrous assumption that names will always be unique, then both `{ Name }` and `{ Employee ID }` are candidate keys. Since they each determine all the info in the row, both of these transitive dependencies hold:

```
Name -> Employee ID -> Zip code
Employee -> Name -> Zip Code
```

Because the dependency between `Name` and `Employee ID` goes both ways, though, this does _not_ violate 3NF. 

How does this play out in the other, case-based definition? There are four dependencies in this table:

   * `Name -> Employee ID`
   * `Name -> Zip code`
   * `Employee ID -> Name`
   * `Employee ID -> Zip code`

None of these have the property that `A` is a superset of `B`. _All_ of them have the property that `A` is a superkey, though, because both `Name` and `Employee ID` are candidate keys (and thus superkeys) on their own. Therefore this is in BCNF (and in 3NF).

---

That was an interesting (and time consuming) journey. I hope this helps some. – Nic
