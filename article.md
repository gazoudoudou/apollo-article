TITLE

Apollo Client is a technology used to perform Graphql queries in the front-end of an app. If using it, a method will consist in storing data in the Apollo cache to avoid making the same Graphql calls to the API repeatedly.

However, a problem may occur: after a mutation which updated data stored in the back-end, Apollo will not update those data in the cache by itself. Therefore, the developers face the problem of forcing this cache update.

This article will cover the 3 main methods used to do this. It will also discuss the different use cases of these methods.

I. The 3 methods

Let’s imagine a developer has to display a list of items, students of a class for instance. He also would like to press a delete button next to a student, so that he is removed from the class, i.e. he does not appear in the displayed list anymore. To do so, he has 2 queries at his disposal :
- getAllStudents
- deleteAStudent(id) 

Here are the 3 methods he could use to perform the Apollo cache update:
- The "refetchQueries" method: first, the "deleteAStudent" mutation is performed. The student does not exists in the back-end anymore, but he still does in the cache. He uses the "getAllStudents" query then, so that it refreshes the students data in the cache.
- The "update" method: here, "deleteAStudent" is the only performed query. If it is successfully completed, the developer uses a script to read and write the cache directly: he removes the student "by hand".
- The "optimisticResponse" method: the "deleteAStudent" is also the only query which is performed. But we are not going here to wait for the end of the request and check if it succeed. On the other hand, we will perform the cache update by reading and writing it directly after the call of the mutation.

The following diagram illustrates these methods and their differences: SCHEMA

II. Implementations

III. Use cases

Every developers updating Apollo cache has to ask himself which method is more appropriate to his use case, as all of them present both advantages and disadvantages. Here, we give tracks for each of these three methods.

For the "refetchQueries": 
The query associated with the refetch part ("getAllStudents" in the previous example) must be quite "light" here. Indeed, if the number of data exchanged during this refetch is important, then the time taken for this update can be extremely long. If a loader is displayed during this request, the quality of the user experience could then be reduced, for what just appears to be in simple list update. 

However, if the number of data remains low, the "refetchQueries" is a good method to consider since it will remain quite fast and it will be very easy to set up technically.

An other case for which the use of this method is advantageous is the one when, during a request, the server performs additional operations that can not be done in the front-end. For instance, if the front sends an address to the API, but it is necessary to display a latitude and a longitude obtained from this address, these data will be computed in the server side and sent back during the refetch, and the cache will be updated with these.

For the "update": 
Just like the "refetchQueries", the "udpate" method allows to update the cache with data that can only come from the server.

However, it is often more complicated to set up: reading and writing the cache by hand may be tricky.

Nevertheless, it has other advantages: only one request is made which makes it very fast compared to the "refetchQueries" where the "refetch" part (the "getAllStudents" query) may be heavy.

For the "optimisticResponse": 
Just like the "update", the "optmisticResponse" is more complicated to set up than the "refetchQueries". It also avoids loading times that are too long since only one request is made.

However, if the data to update in the cache can only come from the server, this method is not suitable since the update of the cache is done before the response of the first request.

The big advantage of this method is that it offers the best user experience: the displayed data are updated almost instantly to the user, giving an animation effect. But this advantage can turn into a disadvantage if the mutation made is "critical": here we mean the fact that the request is associated with a payment validation for example. If the data in the cache tells the user that his payment went well while the request ends up being a failure, then there is a risk of giving false (but essential) information to the user.

TABLEAU