# Square Arches


### What have open source and NI got in common?


### What was our biggest pain point with Arches?

The learning curve


### Why does this matter?

Note: For us to bring devs, infrastructure engineers and data scientists onto Arches projects, we need to have familiar, clean abstractions. Otherwise, we restrict to team members who specialize in Arches, and in its internal structures - realistically, this has been the case, and we are fortunate to have two such developers with us, but for us, practically, familiarity with Arches internals is hard to recruit for, hard to teach and can overspecialise team members. 


### How did we deal with this?

* Web: ArchesORM [beta]
* IT Admin: Casbin Permissions Manager [alpha]
* Data: Arches GraphQL Server/Client [alpha]
* Infrastructure: Arches Kubernetes Toolkit [beta]

With particular thanks to NI HED, and our other Arches clients

Note: by implementing familiar abstractions.



### ArchesORM

#### Motivation

* Webdevs are used to Object Relational Mappers
* e.g. `LINQ`, `Fluent`, `SQLAlchemy`
* Easier to read, write and maintain business logic in a project
* Runs within an Arches project -- Django and `arches.*` importable
* _Can assume a model has not changed since process restart_


#### What have we done?

* In `settings` we give graph IDs of **well-known resource models** (WKRMs)
* On start, we map the WKRMs by nodegroups
* WKRMs appear as dynamically-generated classes
* **View models** give native-feeling wrappers for semantic nodes, concepts, etc.
* Does not unnecessarily create Tiles
* PoC-level bulk insertion
* Got spatialite working for Arches tests (not proper use)


```python
import arches_orm.arches_django
from arches_orm.models import Person

st_col = Person()
st_col_name = st_col.names.append()
st_col.full_name = "Saint Columba"
st_col_name = st_col.names.append()
st_col.full_name = "Colum Mór"
st_col.save()
```


```python
columbas: list[Person] = Person.where(full_name="Saint Columba") # PG
greats: list[Person] = Person.search("Mór") # ES
```


```python
address = Address()
address.first_line = "The Abbey"
address.second_line = "Iona"
address.city = 
ash.addresses.append(address)


#### Gory Technical Details


* We abstract out Arches/Django so this library could work client-side
* It can use multiple backends simultaneously, perhaps to write loaders or syncing
* View models for new types can be added easily (e.g. User)



### Future

* Archeslite: tiny, Arches-compatible microsites
* NoArches: rendering static websites (e.g. Github Pages)
* JS &amp; Offline functionality (Owen)
* Cleanly-separated Arches-to-Arches syncing
* Exporters
* Powerful backend business logic in Arches projects



### GraphQL

#### Motivation

* De-couple data from Arches
* Self-documenting
* Standard libraries
* Easy Python client creation


#### What have we done?

* Takes resource models listed in `settings`
* Uses the ORM to turn them into simple(r) Python structures
* Maps them to a GraphQL API with graphene


#### What have we not done?

* Explored chained searches
* Tidied up bulk uploading


#### What does this look like?

```python
from arches_graphql_client import ResourceClient

person_client = ResourceClient(
  "http://example.org",
  "Person",
  "label_name"
)
person_client.connect()
```

```python
person = await person_client.create({
    "name": [{"fullName": "Ash"}]
})
print(person["id"])
```

```python
person = await person_client.get(
  str("0000...0000"),
  ["id", ("name", ["fullName"])]
)
```


### Limitations

* (Currently) unaware of changes since startup
* No special handling of deeply-nested relationships
* Authorization is not yet integrated - admin or nothing
* Entirely ignores cards
* Extremely limited around concepts
* Does do OAuth, but a nicer flow would help


### Opportunities

* Data science
* Ingestion
* Syncing
* Mobile apps
* Mocking Arches instances?
