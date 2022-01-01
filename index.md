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
JEL_entity_destory();
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
JEL_ENTITY_SET(entity, Position, 4, 5);

struct Position p = {2, 2};
JEL_ENTITY_SET_STRUCT(entity, Position, p);

JEL_ENTITY_SET_PROP(entity, Position, x, 10);
JEL_ENTITY_SET_PROP(entity, Position, y, p.y);

JEL_ENTITY_GET(entity, Position, &p);
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
