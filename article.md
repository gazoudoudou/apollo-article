# TITLE

Apollo Client is a technology used to perform Graphql queries in the front-end of an app. To avoid calling the same Graphql queries repeatedly to the server side, a method will consist in storing data locally, in the Apollo cache.

However, a problem may occur: in some specific cases, after a mutation which updated data stored in the back-end, Apollo will not always update its cache by itself in the front side. Therefore, the developers face the problem of forcing this cache update so that the modified data appear as expected in the app.

This article will briefly explain the structure of the Apollo cache and how the data is stored in it. From there, it will explain why the cache is not always updated by itself. At last, it will cover the 3 main methods used to force this update and discuss the different use cases of these methods.

### I. The Apollo cache structure

Let's imagine a developer has to display a list of items, all the pokemons for instance (THE pokedex). Each pokemon has an id, a name and a level. To do so, he has one query at his disposal: getAllPokemons.

After calling this query when a user first arrives on the page where the pokemons should be displayed, the server will return a list of objects { id, name, level }. From there, two things will happen:

- These objects will be placed in a global store with all entities, as a dictionnary.
- In another store, the "getAllPokemons" query will only be associated to an array of ids: the ids of all the pokemons in this list.

Therefore, when the user will come back later in this same page, the query will be called another time. However, no server side call will be made: Apollo will try to get the data from the local store. Two things will happen there too:

- Apollo will look at the list of ids associated to "getAllPokemons"
- From these ids, it will get the corresponding objects from the store

### II. Understanding the udpate

Now, the developer also would like that the user could create a new pokemon, and increase the level of a specific pokemon. To do so, he has 2 mutations at his disposal:

- updatePokemon({id, level})
- createPokemon(name)

In the app, by clicking on a pokemon from the previous list, the user calls the "updatePokemon" mutation to increase the level by 1. After the mutation, the object stored in the Apollo cache and which id corresponds to this pokemon will by updated.

Therefore, when the "getAllPokemons" will be called again, Apollo will well return the updated object since its id is already associated to the query: there is no problem here.

Now imagine that the user decides to create a new pokemon by entering its name (level = 0 by default). Then, a new object will be insert in the store. However, it is really important to insist on the fact that the id of this new object won't be associated to the "getAllPokemons" method, i.e. inserted in the ids list.

Therefore, when the user will come back on the pokemons list, this new pokemon won't appear: this is a typic situation where the developer has to use a method to force the Apollo cache update.

### III. The 3 cache update methods

Here are the 3 methods a developer could use to perform the Apollo cache update:

- The "**refetchQueries**" method: after the "createPokemon" mutation, the "getAllPokemons" query is called again (real call on the server), so that it refreshes the list of ids associated to it: the id of the new pokemon now appears in this list.
- The "**update**" method: here, "createPokemon" is the only performed query. If it is successfully completed, the developer uses a script to read and write the cache directly: he associates the new pokemon "by hand" to the "getAllPokemons" query.
- The "**optimisticResponse**" method: the "createPokemon" is also the only query which is performed. But we are not going here to wait for the end of the request and check if it succeed. On the other hand, we will perform the cache update by reading and writing it directly after the call of the mutation.

The following diagram illustrates these methods and their differences:

![The 3 methods](/images/update_methods.png)

### II. Implementations

### III. Use cases

Every developers updating Apollo cache has to ask himself which method is more appropriate to his use case, as all of them present both advantages and disadvantages. Here, we give tracks for each of these three methods.

**For the "refetchQueries":**

The query associated with the refetch part ("getAllStudents" in the previous example) must be quite "light" here. Indeed, if the number of data exchanged during this refetch is important, then the time taken for this update can be extremely long. If a loader is displayed during this request, the quality of the user experience could then be reduced, for what just appears to be in simple list update.

However, if the number of data remains low, the "refetchQueries" is a good method to consider since it will remain quite fast and it will be very easy to set up technically.

An other case for which the use of this method is advantageous is the one when, during a request, the server performs additional operations that can not be done in the front-end. For instance, if the front sends an address to the API, but it is necessary to display a latitude and a longitude obtained from this address, these data will be computed in the server side and sent back during the refetch, and the cache will be updated with these.

**For the "update":**

Just like the "refetchQueries", the "udpate" method allows to update the cache with data that can only come from the server.

However, it is often more complicated to set up: reading and writing the cache by hand may be tricky.

Nevertheless, it has other advantages: only one request is made which makes it very fast compared to the "refetchQueries" where the "refetch" part (the "getAllStudents" query) may be heavy.

**For the "optimisticResponse":**

Just like the "update", the "optmisticResponse" is more complicated to set up than the "refetchQueries". It also avoids loading times that are too long since only one request is made.

However, if the data to update in the cache can only come from the server, this method is not suitable since the update of the cache is done before the response of the first request.

The big advantage of this method is that it offers the best user experience: the displayed data are updated almost instantly to the user, giving an animation effect. But this advantage can turn into a disadvantage if the mutation made is "critical": here we mean the fact that the request is associated with a payment validation for example. If the data in the cache tells the user that his payment went well while the request ends up being a failure, then there is a risk of giving false (but essential) information to the user.

TABLEAU
