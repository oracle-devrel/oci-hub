
# Build Swift apps with Oracle’s Autonomous Database — episode 2

Build Swift apps with Oracle’s Autonomous Database, episode 2

### In this episode, we test our initial design choices by adding a new use case to the framework: retrieve a specific document and present the data as a detail sheet in the app, allowing the user to refresh at will with new data from the server.

Swift(UI), Combine, JSON, and the cloud version of the Oracle Database are the ingredients used to build this second layer of our iOS/macOS native interface for ORDS.

## Before we begin

This is the second article in the series, so it builds on what was done in the [first post](https://medium.com/so-much-code/build-swift-apps-with-oracles-autonomous-database-and-nosql-f1dee7e7cec3). Please [read it](https://medium.com/so-much-code/build-swift-apps-with-oracles-autonomous-database-and-nosql-f1dee7e7cec3) before going forward otherwise this one wouldn’t make any sense.
[**Build Swift apps with Oracle’s Autonomous Database and NoSQL**
*It was simply a no-brainer: NoSQL, JSON-storing capabilities, Oracle’s cloud-based Autonomous Database (I’m no DBA, so…*medium.com](https://medium.com/so-much-code/build-swift-apps-with-oracles-autonomous-database-and-nosql-f1dee7e7cec3)

## What do we want to do in this episode

In the first episode, we’ve created the cloud database instance, added some test data, connected to it from our swift app, and performed a simple query to retrieve all the documents from our Fruit collection.

**Now, let's test our initial design choices by adding a new use case to the framework: retrieve a specific document and present the data as a detail sheet in the app, allowing the user to refresh at will with new data from the server.**

![](https://cdn-images-1.medium.com/max/4000/1*OItJYLcmMZFvE-WshnAJxw.png)

## 1. Modify the API handling code

Let’s start by adding a method to retrieve a single document from the database, given it’s id. The relevant REST request is something like this:

    [https://[endpoint]/[collection](https://[endpoint]/[collection) name]/[document id]

And the response is a JSON containing the requested document:

    {
     “name”: “banana”,
     “count”: 590,
     “color”: “dark yellow”
    }

The new method is pretty straightforward, we append the *collection name *and the *document id* to the request URL, we feed the API Agent with it and then return the resultingPublisher with the model object as payload (because async).

<iframe src="https://medium.com/media/ad16f40305aaa83230f2f691907fd77f" frameborder=0></iframe>

## 2. Connect the new API to our own model

The method above belongs to our SODA API layer therefore it’s generic enough to be used with any kind of model object and any collection. But our app’s model layer deals specifically with Fruit objects, so we use the DataStore we’ve built before to specifically request a Fruit document from the server.

<iframe src="https://medium.com/media/bf4a1e1d5806740f164ca2023fd01081" frameborder=0></iframe>

The retrieveFruit(with:) method ensures that specifically the fruit collection is used and, by returning a Publisher with a Fruit payload, feeds our specific model struct to the generic methods upstream.

The second method update(fruit:with:) finds a specific Fruit object in the model array and, if found, replaces it with the new value received (auto-magically triggering a refresh on all the connected views in the process)

Both methods will be used in the *view-model* layer when we trigger the operations from the *view*.

## 3. Change the view

For starters, we want to be able to show a detail screen about each Fruit object on our list.

![](https://cdn-images-1.medium.com/max/2000/1*Xe6IEcbKtshilTlxmJmXmA.gif)

So let’s change our main view to this one:

<iframe src="https://medium.com/media/47e21b28c7a54ca8ea4eb66bd4187885" frameborder=0></iframe>

Nothing too complicated here, we added a NavigationLink to the detail view FruitView and moved the cells’ rendering to their own method, for clarity.

The interesting stuff is happening in FruitViewand in the associated *view-model*, so let’s take a look:

<iframe src="https://medium.com/media/df4f5bf717df273cb3877580983cd1bb" frameborder=0></iframe>

First, to keep this *view* as business logic-free as possible, we use a separate *view-model* object (aptly named model in line 3) and we initialize it with the Fruit object passed from the main view in line 6.

The second point of interest is the toolbar button defined starting with line 30. Its task is to trigger a refresh of the represented Fruit document from the server. In the *view* here it just calls the refreshFruit() method from our *view-model*.
The button is disabled when the *view-model* is in process of retrieving data from the server database (line 33).

The rest of the view is merely showing a form with rows corresponding to Fruit properties.

Now the *view-model* class (it needs to conform to ObservableObject):

<iframe src="https://medium.com/media/580939def565ed215f0506a35cb361ee" frameborder=0></iframe>

We use two @Published variables — they will trigger a *view* refresh each time they change:

1. the modeled object, a SODA.Item with a Fruit payload — we use the Item here to have access also to the associated metadata, specifically to the id field needed to retrieve the latest document from the cloud database.

1. A boolean (isLoading) signaling when an API operation is in progress.

The private tokens variable is needed to retain Combine cancellables. Failing to do that will result in our Combine async pipelines not being executed (because they’re deallocated before completing).

The refreshFruit() methods (called by our Refresh button in the *view*) finally calls the newly defined retrieveFruit(with:) in the DataStore and triggers an API call to the cloud database.

When concluding the pipeline (line 16) we update the model data (remember, stored as an array of [SODA.Item<Fruit>] in our DataStore) with the newly retrieved Fruit object $0 .

And yes, still no proper error handling, the app will just quit if an error is propagated down the pipeline (line 15). I will dedicate one of the future articles in the series only to error handling.

## Conclusions

And that’s all it takes to refresh our detail view with fresh data from our Autonomous backend in the cloud.

To test our shiny new button, login into the* OCI console* (as explained in the [previous article](https://medium.com/so-much-code/build-swift-apps-with-oracles-autonomous-database-and-nosql-f1dee7e7cec3)), go to the SQL Developer Web tool and change one of the documents stored in the fruit collection.
Then, in the app, open the detail view corresponding to that document and hit the *Refresh* button:

![](https://cdn-images-1.medium.com/max/2000/1*CdB2Wbn0QammbSDhjjoICQ.gif)

As always, the full Swift code is available on Github:
[**bogdanf/SODA-Swift-Client-episode-2**
*This repository ilustrate my article about how to integrate a modern Swift app with a cloud Oracle Autonomous Database…*github.com](https://github.com/bogdanf/SODA-Swift-Client-episode-2)
