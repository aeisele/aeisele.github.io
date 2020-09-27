---
title: "Prevent Hibernate from doing N+1 selects with constructor expression"
date: 2020-09-25T13:00:14+02:00
draft: false
tags: [hibernate, jpa, sql, dto, jpql]
categories: [programming]
resources:
    - name: featured-image
      src: drawers.png
lightgallery: true
code:
    maxShownLines: 50
---

## The Motivation

While basic CRUD operations with _JPA_ / _Hibernate_ are easy, every application sooner or later
needs to introduce _DTO_ style result objects for specific use cases like projections. That is part
of the deal, no OR mapper can do your homework for you.

Luckily _JPA_ gives us a way of specifying exactly what result objects we expect from a query. One
way to do that is to use _JPQL_ with its so called 'constructor expression' in the select part of
the query.

```jpaql
select new com.example.NameDto(e.firstName, e.lastName) from Employee e
```

Note the `new` keyword and the fully qualified class name. This does exactly what you would think it
would do. It creates a new instance of `NameDto` using the appropriate public constructor for each
result row.

## The Problem

Using parts of an entity and projecting them into a _DTO_ is fine and all but what happens if we want
to fetch the whole entity and wrap it into a _DTO_? Imagine the use case where we want to select an
entity but also some auxiliary information that is not stored along with the entity. For example
suppose we want to select "places near me in a 2000m radius". That's only one where condition and
then you have a result set of the actual place entities right? What if we also want to show to
the end user how far away from his position those places happen to be? Why don't we just wrap the
`Place` entity into an appropriate _DTO_ that also holds the distance, say `DistanceResult`?
Let's try it out.

Here we have the `Place` entity, you can see it has a location field indicating where it is located.

{{< image src="place.PNG" caption="Place entity" >}}

And then we a _DTO_ called `DistanceResult` which should just wrap our entity together with how far
it is from the origin of the query.

{{< image src="distance_result.PNG" caption="DistanceResult DTO" >}}

Prepared with that let's fire a simple _JPQL_ query like this:

```jpaql
select new com.example.DistanceResult(p, distance(:center, p.location))
  from Place p
  where dwithin(:center, p.location, :radiusMeters) = true
```

Ignore for a moment that we are dealing with geographic types and functions here. This is actually
part of `hibernate-spatial` and I use it on an instance of `postgis` but this doesn't matter here.

This is what our query log gives us for that query:

```sql
select place1_.id as col_0_0_, st_distance(?, place0_.location) as col_1_0_ from places place0_ inner join places place1_ on (place0_.id=place1_.id) where st_dwithin(?, place0_.location, ?)=true

select place0_.id as id1_0_0_, place0_.location as location2_0_0_, place0_.name as name3_0_0_ from places place0_ where place0_.id=?

select place0_.id as id1_0_0_, place0_.location as location2_0_0_, place0_.name as name3_0_0_ from places place0_ where place0_.id=?
```

We can clearly see that our singular _JPQL_ query resulted in one query that is roughly its _SQL_
equivalent. On top of that we can see additional selects one for each entity instance in our result
set (here the result size is 2). This looks like the dreaded __N+1__ problem.

## Solution 1: result transformers

On one hand someone at the _hiberante_ project clearly had a reasoning behind why using the constructor expression would
not fetch complete entities but rather only the immediate needed parts. On the other hand in the given use case N+1 is 
nothing we want to have. Sadly there is no clear way of telling hibernate to fetch the whole entity. For example a self
join using the `fetch` keyword doesn't work. Also the `fetch all properties` stanza does not affect the resulting _SQL_
in this case.

Luckily there is some other way to do the _DTO_ instantiation we want to have. This is _hibernate_ specific and called
`ResultTransformer`. Given any _hibernate_ query we can attach an instance of this interface and have our results
handled by it.

### Result Transformers with Spring Data JPA

I trust it a large part of _JPA_ and/or _hibernate_ users will probably be using _Spring Data JPA_ to avoid boilerplate.
Here is how to use `ResultTransformer` in that case:

First we have our usual repository interface without anything fancy, note that it extends an additional interface.

```java
@Repository
public interface PlaceRepository extends PagingAndSortingRepository<Place, Long>,
    DistancePlaceRepository {
}
``` 

This is that additional interface, we can see our distance query method is defined here.
```java
public interface DistancePlaceRepository {
    List<DistanceResult> findInDistance(Point center, double radiusMeters);
}
```

But how is this method implemented? We can add another `@Repository` bean into the context that implements only this
interface. _Spring Data JPA_ will actually do some kind of _mixin_ magic behind the covers such that our concrete
repository bean later on uses the following implementation:

```java
@Repository
public class DistancePlaceRepositoryImpl implements DistancePlaceRepository {

    @PersistenceContext
    private EntityManager entityManager;

    @SuppressWarnings({"unchecked", "deprecation"})
    @Override
    public List<DistanceResult> findInDistance(Point center, double radiusMeters) {
        String jpql = "select p, distance(:center, p.location) from Place p where dwithin(:center, p.location, :radiusMeters) = true";

        return (List<DistanceResult>) entityManager.createQuery(jpql)
                .setParameter("center", center)
                .setParameter("radiusMeters", radiusMeters)
                .unwrap(Query.class)
                .setResultTransformer(
                        (ListResultTransformer)
                                (tuple, aliases) -> new DistanceResult(
                                        (Place) tuple[0],
                                        ((Number) tuple[1]).doubleValue()
                                )
                ).getResultList();
    }
}
```

{{< admonition type=warning title="Warning" >}}
I'm using Vlad Mihalcea's `hibernate-types` library here to get access to `ListResultTransformer` (See links below).
{{< /admonition >}}

Now how does our query log look like if we use this transformer?

```sql
select place0_.id as col_0_0_, st_distance(?, place0_.location) as col_1_0_, place0_.id as id1_0_, place0_.location as location2_0_, place0_.name as name3_0_ from places place0_ where st_dwithin(?, place0_.location, ?)=true
```

As you can see its only a single select which is much better. No __N+1__ in sight.

### Drawbacks

The first thing you will note when trying to use `ResultTransformer` is that the actual method to use them
`Query#setResultTransformer` is marked as deprecated. This is actually some form of premature deprecation by the
_hibernate_ developers. Unless you're using _hibernate_ 6.0 there is no way around using this deprecated method. So it
is clearly fine to ignore the warning here (suppress it until 6.0 is GA and then migrate).

The second thing to note is that clearly this solution is _hibernate_ specific. `ResultTransformer` is nothing _JPA_
has an equivalent of (yet). 

## Solution 2: Blaze Persistence entity view

While investigating this problem and related info on the internet I happened upon a stack overflow [answer](https://stackoverflow.com/a/64041288/222272)
leading me to discover so called _"Blaze Persistence Entity Views"_. It looks like we could redefine our _DTO_ with this.
I leave it as an exercise to the reader to find out via their [documentation](https://persistence.blazebit.com/documentation/1.6/entity-view/manual/en_US/#spring-data-features)
how exactly we would achieve the wrapping of the whole entity and also map the _distance_ part of our original query. Its
an extensive library, I'm sure there is a way.

## Solution 3: use jOOQ

Last but not least I still have to mention that of course there is nothing preventing us from just mixing our existing
_hibernate_ mapping with a little bit of [jOOQ](https://www.jooq.org/) on the side. This would give us full control over
what _SQL_ exactly we are executing. And we can easily wrap the resulting tuples into our _DTO_ if we want. Sure we would
have to pay attention that the result are no managed entities but in our use case this isn't necessary in the first place.

I said not to pay attention to the _spatial_ functions earlier but if we actually want to use _jOOQ_ for this kind of
query then we would need support for those _SQL_ functions in the query builder. Luckily there is a project for this
on github by the name of [jooq-postgis-spatial](https://github.com/dmitry-zhuravlev/jooq-postgis-spatial) (in the case of
postgis that is).

## Links
 - Vlad Mihalcea on hibernate result transformers: [Blog Post](https://vladmihalcea.com/hibernate-resulttransformer)
 - Blaze Persistence entity views [documentation](https://persistence.blazebit.com/documentation/1.6/entity-view/manual/en_US/#spring-data-features)
 - Original Question on [stackoverflow](https://stackoverflow.com/questions/64028853/jpa-circumvent-n1-when-using-constructor-expression-in-jpql-to-wrap-singular-e)
 - jOOQ [homepage](https://www.jooq.org/) 
 - Photo by [Jason Leung](https://unsplash.com/@ninjason?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/drawer?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)