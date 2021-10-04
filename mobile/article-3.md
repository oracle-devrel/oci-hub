
# Build Swift apps with Oracle’s Autonomous Database, episode 3

Build Swift apps with Oracle’s Autonomous Database, episode 3

### We’re ready now to use our client code to change the data stored in the cloud database, so let’s bake this into our framework and app.

Swift(UI), Combine, JSON, and the cloud version of the Oracle Database are the ingredients used to build this second layer of our iOS/macOS native interface for ORDS.

![](https://cdn-images-1.medium.com/max/3736/1*wL0kFWCjZBSod_jeyTkDYQ.png)

## What do we want to do in this episode?

In the [first episode](https://bogdan.farca.ro/build-swift-apps-with-oracles-autonomous-database-and-nosql-f1dee7e7cec3), we’ve created the cloud database instance, added some test data, connected to it from our swift app, and performed a simple query to retrieve all the documents from our Fruit collection.

In the [second episode](https://bogdan.farca.ro/build-swift-apps-with-oracles-autonomous-database-episode-2-20197975e72a), we added the code to retrieve a specific document and present the data as a detail sheet in the app, allowing the user to refresh at will with new data from the server.

**And now, let’s reverse the data flow and send updates made in our client app to the server, and update the corresponding records in the cloud-based Autonomous Database instance.**

## 1. Modify the API framework first

First, let’s focus on our API agent code and add a new request type. We need it because the API request to update a database record doesn’t return any payload, except a success/failure code. Our existing call is assuming something is coming back, so not usable for our new use case.

<iframe src="https://medium.com/media/3a177779ee554f45b71f154410da43f1" frameborder=0></iframe>

Now, equipped with this new tool, let’s add the update method to our SODA framework:

<iframe src="https://medium.com/media/890e62643eaf0b76ab20610ea18405e8" frameborder=0></iframe>

The code is pretty simple, we create the url expected by the SODA API (lines 4–6), we JSON encode the new object in line 12 and add it to the HTTP body (lines 12–16). The required HTTP method in this case is PUT.

Then we just call our agent with the resulting request, and return it as a Combine Publisher.

The method expects any type of Encodable entity, together with the record id of the server record to be updated, in the collection named collection.

## 2. Add some glue code to the view-model

Actually calling the new SODA framework update method is (as expected) pretty straightforward. We do it from the view-model associated with our Fruit detail view.

<iframe src="https://medium.com/media/19789bfce185f6a7c86deb6c6b44ad8b" frameborder=0></iframe>

We simply call our update publisher and (only) when the API reports success we update our model. This will of course trigger an update of all the affected views app-wide.

## 3. View changes

The plan is to add an Edit mode to our Fruit detail view and let the user switch between modes with the usual Edit button.

![](https://cdn-images-1.medium.com/max/2000/1*6EwTwvXTmGtyPWPjuZNcbA.gif)

TheSave button will trigger our updateFruitOnServer method defined above. Let see how SwiftUI code looks like:

<iframe src="https://medium.com/media/95ee9606cf86dd794186ec17ef4bb965" frameborder=0></iframe>

The code that renders our Name, Count and Color rows is now too complex to be kept in the body, so we moved it to their own functions (only one of them is included here, the others are pretty much similar):

<iframe src="https://medium.com/media/95d99bca4afb2cf7521bcaf8c1b24ac0" frameborder=0></iframe>

Based on isEditing state, we render a TextField (for editing) or a Text view (only for display).

Because our model struct Fruit stores the count as an Int, and the TextField expects a String, we need to convert back and forth between the two types in lines 7–8.

## Wrapping up

Adding a new API request type to our framework and app is really straightforward, using robust and well-designed back-end and client technologies pays off :-)

The code is available on Github here:
[**bogdanf/SODA-Swift-Client-episode-3**
*This repository ilustrate my article about how to integrate a modern Swift app with a cloud Oracle Autonomous Database…*github.com](https://bogdanf@github.com/bogdanf/SODA-Swift-Client-episode-3.git)
