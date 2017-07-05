title: "The Data Migration Problem"
date: 2016-08-18 17:37:27
tags: [Algorithm]
categories: [Engineering, Algorithm]
---

![Migrate the Hotel!](http://7ktqal.com1.z0.glb.clouddn.com/img/blog/cool-jenkins2x3.png)
Today I am trying to solve a very interesting problem, I would like to call it the "data migration problem". 

<!-- more -->

## Background
Let me illustrate to problem in short. 
* We are working on data migration of a PMS system to a newer version. 
* This system contains 2 major entities: **USER** & **HOTEL**
* Each **USER** could operate 1 or more **HOTELS**
* Each **HOTELS** could be operated by 1 or more **USERS**
* Migrations are operated by batch of hotels. Each batch we could migrate 1 or more hotels.

## Requirement
* After each batch of migration, all hotels operated by the same user should at the same status (migrated / non-migrated).
* In the migration process, all hotels in the same batch could not be operate (downtime happens). So we are trying to minimize size of each batch with above requirement.

## Examples
Each input line implies a relation of operation / management
```
input:
    user_1   hotel_1
    user_2   hotel_2
    user_3   hotel_3

output:
    [hotel_1]
    [hotel_2]
    [hotel_3]
```
Each user operate individual hotels, so each hotel could be migrated in separate batch.

```
input:
    user_1   hotel_1
    user_2   hotel_1
    user_2   hotel_2
    user_3   hotel_2
    user_3   hotel_3

output:
    [hotel_1, hotel_2, hotel_3]
```

All hotels should be migrate in the same batch. Once hotel_1 is migrated, all other hotels user_2 operates (here, hotel_2) should be migrated as well because user_2 also operate hotel_1. Similarly, once hotel_2 migrated, hotel_3 should also be migrated (in the same batch) because user_3 operate these two hotels.

## Solutions

At the first glance, The data structure are very similar to [BIPARTITE GRAPH](https://en.wikipedia.org/wiki/Bipartite_graph). The nodes consist of two part are the hotels and the users, edges are relations between hotels and users. There no edges between hotels, neither do users.

![Bipartite graph](http://7ktqal.com1.z0.glb.clouddn.com/img/blog/Simple-bipartite-graph.png)

However, we are not trying to apply capable users to manage hotel as much as possible. So it's not a classical bipartite graph [MATCH PROBLEM](https://en.wikipedia.org/wiki/Matching_%28graph_theory%29). Instead, we are trying to judge whether nodes are connected via edges, which could be convert to another problem -- "Finding all [CONNECTED COMPONENT](https://en.wikipedia.org/wiki/Connected_component_%28graph_theory%29)". 

![Connected Components](http://7ktqal.com1.z0.glb.clouddn.com/img/blog/Pseudoforest.svg.png)

Of course we have BFS/DFS to answer such questions. However, some information is not important to our current problem -- the structure. We don't need to know which manager make 'Ritz Carlton' and 'Hilton' connected, we just need to make sure both hotel are in the same batch. So we are able to make our algorithm more effective using [DISJOINT SET](https://en.wikipedia.org/wiki/Disjoint-set_data_structure).

## Explanations
Let's see the `Elem` class. Actually it's an implementation of the Disjoint Set.

* When initialize, Each `Elem` could be seen as an single element set.
* `parent` is a pointer to its parent `Elem`.
* To judge whether two element is in the same set, check whether their `root` (utmost parent / ancestor) are the same element
* To `union` two set, we find the `root` of each set, set parent of one root to the other root

``` python
    def root(self):
        if self.parent == self:
            return self
        else:
            # recursively find the ancestor
            return self.parent.root() 

    def union(self, y):
        my_root = self.root()
        my_root.parent = y.root()
```

And the iteration algorithm is:
1. Each user / hotels are initialized as a individual set.
2. Iterate all hotels, for each hotel:
    * iterate each user it belongs to:
        * If the user has not been union into any hotel, then union with current hotel
        * Else, the user has been union into an existing hotel set. In this situation, current hotel should also be merged into the same hotel.

``` python
    for h in hotels:
        for u in hotels[h]:
            if up[u].parent == up[u]:
                up[u].parent = hp[h]
            else:
                up[u].union(hp[h])
```

The [Loop Invariant](https://en.wikipedia.org/wiki/Loop_invariant) of this algorithm guarantee that:
1. After each hotel looping, the root of any hotel Elem is always another hotel or itself. This means it's correctly unitied with all those hotels known and supposed to be migrated together, if any, in the same batch.
2. After each user looping for any hotel, the root of the user Elem is always a hotel. 

That's it. 

## Code 

Here I feed this python script with a comma separated csv file, marking users and hotels.

``` python
import sys

class Elem():
    parent = None
    value = None

    def __init__(self, value):
        self.value = value
        self.parent = self

    def root(self):
        if self.parent == self:
            return self
        else:
            return self.parent.root()

    def union(self, y):
        my_root = self.root()
        my_root.parent = y.root()

    def __str__(self):
        return "[value: %s, parent: %s]" % (self.value, self.root().value)

if __name__ == "__main__":
    users = set()
    hotels = {}
    hp = {}
    up = {}
    res = {}

    for line in sys.stdin:
        (u, h) = line.strip().split(",")

        if u not in users:
            users.add(u)

        if h in hotels:
            hotels[h].add(u)
        else:
            hotels[h] = set([u])

    # init hotel_sets 
    for h in hotels:
        hp[h] = Elem(h) 

    for u in users:
        up[u] = Elem(u)

    # start partitions
    for h in hotels:
        for u in hotels[h]:
            if up[u].parent == up[u]:
                up[u].parent = hp[h]
            else:
                up[u].union(hp[h])

    for h in hp:
        root = hp[h].root().value 
        if root in res:
            res[root].append(h)
        else:
            res[root] = [h]

    for r in res:
        print res[r]
```
