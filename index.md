---
layout: default
thumb: "/assets/images/jel_banner_official.png"
---
# (M)jir ECS Library

An Entity Component System Library written in C

Probably not the best out there, this is more like an experiment and practice in making a large scale project. If you actually want to use a really good C ECS use flecs.

This is also my first project in C

This documentation may not be the best out there, but it will be continually and gradually improved upon.

# Quickstart

Here is a fast guide on getting started, I would also recommend checking out the
examples in the GitHub organization.

## Init and Quit

```c
#include <JEL/jel.h>

int main(void)
{
  JEL_init();
  JEL_quit();
  return 0;
}
```
Init JEL to create the context which holds some global information, quit before your
program exits to free up the memory

You must init JEL before calling any other JEL function

## Entities

Entities are just an unsigned int defined by JEL_Entity, by default it is a uint32_t
```c
JEL_Entity e = JEL_entity_create();
JEL_entity_destory(e);
```
Entities will be covered more in depth in a different section, for quick start, entities
are just ids

## Components

Components are what create types and what hold data. You must create a component
and register it before using any function which would need a component (so just
register it before calling any other JEL function)

This is how to create a component
```c
struct Position {
  int x;
  int y;
};
JEL_COMPONENT(Position, x, y);
```
This is how to register a component (make sure to define JEL_REGISTER_COMPONENTS before including jel.h)
```c
#define JEL_REGISTER_COMPONENTS
#include <JEL/jel.h>

// Create component here or include it from a file

int main()
{
  JEL_init();
  JEL_REGISTER(Position);
}
```
After registering a component, you can "attach" it to entities. There are a couple
of macros to do these gets and sets. Just calling the set macro (but not set prop)
will attach that component to the entity if it doesn't already have it.
```c
JEL_SET(entity, Position, 4, 5);

struct Position p = {2, 2};
JEL_SET_STRUCT(entity, Position, p);

JEL_SET_PROP(entity, Position, x, 10);
JEL_SET_PROP(entity, Position, y, p.y);

JEL_GET(entity, Position, &p);
```
## Queries
You can query for tables which hold all the component data and
iterate through them. It makes more sense with an example. When you create a component,
you get a struct called componentIt, which is an iterator thing, so you do not have to
make the It structs you see in this example.
```c
struct JEL_Query q;

for (unsigned int t = 0; t < q.count; ++t) {
  struct PositionIt pos; JEL_IT(pos, q.tables[t], Position);
  struct PhysicsIt phys; JEL_IT(phys, q.tables[t], Physics);

  for (unsigned int i = 0; i < q.tables[t]->count; ++i) {
    pos.x[i] += phys.x_vel[i];
    pos.y[i] += phys.y_vel[i];
  }
}

JEL_query_destroy(&q);
```
The iterator has the same members as the component, except they are arrays. It will make
more sense after the section about tables. Tables store all the data and they store big
buffers of data: arrays of the individual members of every component in that table.

# Breakdown

This section is going to attempt to explain how everything in jEL works.

## Logger

The way to output information is probably the most important part about jEL. There
is a ```JEL_log``` function, which is enabled by the ```JEL_LOG``` macro during
the compilation of jEL.

## Types

jEL is an archetype ECS. A type is a combination of components, all components
are associated with a type (a type index specifically). Every combination of
components is a different type, and there is a table for each type. Types are
represented by a bit array.

## Entities
Entities are just numbers. Abstractly, entities are just ids. The bits of an entity
have two groups, the index and the generation. The index and generation make more
sense after knowing about the entity manager. An entity with a value of 0 is invalid.

### Entity Manager
The entity manager manages all of the entities. There is a member called ```generations```
in the entity manager, and this stores the generation value for each entity. Since
entities are weak references, how would you know if an entity is alive or not?

You create an entity variable player at index 1. Then you use it for a bit, and
something else refers to the player. Eventually you destroy the player entity and
make an enemy entity. Instead of using a new index, you can reuse the player's index
since its been destroyed. The problem is, any references to the player will now be
actually referencing the enemy, and that will cause problems. The generation is increased
whenever a slot is reused, so the player was at index 1 with generation 0, and the
enemy is also index 1 with generation 1. This is how you can tell if an entity is
alive or not, the index and generation must match. (There is a problem where if an
index is used too much the generation will wrap back to 0, but hopefully all your
references to that entity would be gone by that point.)

The specifics of the entity manager are not that important, but it's useful to know
how the entity index and generation work.

### Creating entities
There is some bootstrapping jEL does every time you create an entity. In addition to
providing a valid ID, an entity is also given a ```JEL_EntityC``` component and is added
to the first table in the table stack which has a type of 2 (which is just JEL_EntityC).

### Entity Magic
There is an entity magic headers with useful macros for entities. These macros will
actually be covered in components, because they are all about editing data.

## Components
Components are data. It is a little bit ambiguous, because tables are what store the
data. They both fall in the C of ECS. A component is a struct with different properties
(properties are the same as struct members). Components are associated with a type index,
they are the building blocks of types.

### Creating Components
The quickstart demonstrates how to create a component. Create a struct, then use the
macro to 'create' the component by passing in the struct name and the name of all the
members. What this macro does is create the iterator struct (for queries). It also
sets up some boilerplate for registering the macro, you need to register the component
before using it.

### Registering components
Define ```JEL_REGISTER_COMPONENTS``` before including jEL. Then, after including jEL
and init'ing, use the JEL_REGISTER macro to register the component. This adds an entry
to a component map, which stores component data as the name of the component, the
amount of members, the sizes of each member, and the offsets of each member.

### Component Map
The component map is really an implementation detail to support tables that are all the
same ```struct JEL_Table```, but are composed of different component types. It will make
more sense once tables are explained.

## Tables

## Queries
