---
title: Build Swift apps with Oracle’s Autonomous Database and NoSQL
parent: tutorials
tags: [oci, swift, always-free, back-end]
thumbnail: https://cdn-images-1.medium.com/max/4000/1*L71zFyNH5IoXQqnLxqUszQ.png
toc: true
---

It was simply a no-brainer: *NoSQL*, *JSON*-storing capabilities, Oracle’s cloud-based *Autonomous Database* (I’m no DBA, so having an enterprise-level database at my fingertips, but without having to go through all the admin stuff all by myself, eliminated a big non-starter), plus having all this available under the free tier — I knew I had to use that in a Swift app for Apple’s iOS and MacOS devices.

    There are only 3 easy steps:

    **Step 1**. Create a database instance

    **Step 2**. Add test data

    **Step 3**. The Swift app

So let’s dive right in …

![](https://cdn-images-1.medium.com/max/4000/1*L71zFyNH5IoXQqnLxqUszQ.png)

## Step 1. Create a database instance

First things first, let’s start by creating our database on Oracle Cloud.

### Register for a free trial account

If you don’t have one already, go to [this](https://myservices.us.oraclecloud.com/mycloud/signup) page and register for one.
[**Oracle Cloud Account Signup**
*Use this link to signup as a developer (no marketing follow-ups)*myservices.us.oraclecloud.com](https://myservices.us.oraclecloud.com/mycloud/signup?sourceType=:ex:tb::::RC_WWSA200828P00060:Medium1&SC=:ex:tb::::RC_WWSA200828P00060:Medium1&pcode=WWSA200828P00060)

Please note that even if the free trial has a (temporal and financial) limit, we will use only resources under the free tier — so they’ll stay accessible and free even after the trial is over.

### Create the database instance

In the welcome email you received from OCI you’ll find the Cloud Console* *login link. Use it and login into your main dashboard.

Now let’s go through the instance creation steps:

![](https://cdn-images-1.medium.com/max/4428/1*YVTpt1hAOQsk4FRuv0YgSA.png)

![](https://cdn-images-1.medium.com/max/3200/1*D9Qj_8FvixGLp7q5LD3T-Q.png)

In the database settings pane, we can safely use all the defaults.

Just check that Workload Type* *is set to Transaction Processing* *and* *Access type* *is* *Allow secure access from everywhere*.*

It may be useful to use your own Database name and Display name and of course, you must input your own *password*.

    Click the* Create Autonomous Database* button and wait for the instance to be created.

![](https://cdn-images-1.medium.com/max/4428/1*AarmFsj-imEvmgfnExsY7g.png)

You will get a free tier database instance, with 1 OCPU and 20TB storage. That should be more than enough for our exercise, and maybe it may carry you through all the “side-project” phase of your app :-) There are actually two of these available for free at the time of writing, please [look here](https://www.oracle.com/cloud/free/#always-free) to see all current details.

### Access your instance

Once the database is provisioned you’ll see a screen like this one:

![](https://cdn-images-1.medium.com/max/4428/1*Qb8nRsCtm402LEJL6G08Cw.png)

Let’s have a first glimpse at the JSON/NoSQL capabilities of the database. For a more in-depth view please take a look at [this article by Todd Sharp](https://blogs.oracle.com/developers/launch-persist-json-documents-in-the-cloud-in-10-minutes-or-less-with-autonomous-json-database?SC=:so:tw:or:awr:odv::&pcode=&source=:so:tw:or:awr:odv::).

After opening the Service Console go to the Development section:

![](https://cdn-images-1.medium.com/max/4428/1*DqvfxoR7ILEc1A3ljW-yqQ.png)

    First, scroll down to the section *RESTful Services and SODA* and **make note of the endpoint URL associated with your database**. We’ll use it to access our db via REST later. It’s something like ***https://something-specific-to-your-database.adb.eu-frankfurt-1.oraclecloudapps.com/ords/***

Now, open the SQL Developer Web tool (by clicking the appropriate button in the Service Console).

![](https://cdn-images-1.medium.com/max/4428/1*_gj4rOasLVYSU8VqMPmSPw.png)

## Step 2. Add test data

We have now a fully working relational database, but how do we store our JSON documents and query them afterward?

With …
> Simple Oracle Document Access (SODA) is a set of NoSQL-style APIs that let you create and store collections of documents (in particular JSON) in Oracle Database, retrieve them, and query them, without needing to know Structured Query Language (SQL) or how the documents are stored in the database

Data in SODA is organized in collections and documents, so let’s dive right in by creating a collection named fruit.

![](https://cdn-images-1.medium.com/max/2400/1*rT7hdHlLfniODmkyVN9ggg.png)

![](https://cdn-images-1.medium.com/max/3264/1*JN5aAnyOw41317ZZTowN5g.png)

And now let’s add some JSON-based documents (and yes, these are documents, so there is no fixed schema).

![](https://cdn-images-1.medium.com/max/3264/1*iio2oLWE_s9JtYpjMj1J_w.png)

At this stage, you could (and really should) go ahead and add some fruits of your own.

We will revert for a while now to SQL, just to see how one could query the SODA data (and to see how many fruits we already have in our collection).

![](https://cdn-images-1.medium.com/max/3264/1*nXMBm2SKyoOWw6Nd4nBqRg.png)

Nice. We’re done with setting up the database and test data, so let’s move now to …

## Step 3. The Swift app

We will use the REST API available for our cloud autonomous database, called [*Oracle REST Data Service](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-simple-oracle-document-access-soda)s (*ORDS*).*

For the purposes of this article, I’ll try to keep things as simple as possible. So, for now, our iPhone app will do one thing only: **retrieve the *fruit* collection from our database and display it in a list**.

The ORDS call for obtaining the listing of a collection is (*now it’s time to remember the endpoint URL I’ve asked you to save*):

    $ curl -X GET https://YOUR-ENDPOINT-URL/admin/soda/latest/fruit

If you are inclined to test this from the command line please add the Authorization HTTP header field for ADMIN:[YOUR-PASSWORD]

### Setting up the app

Let’s keep things fun and use the latest *Xcode 12* and *SwiftUI 2.0* (both in beta when I write this, but we like living on the edge, right ?). It goes without saying that all our async code will use *Combine*.

So, fire up Xcode and create a new App project using Swift/SwiftUI.

The HTTP abstraction layer is implemented using ideas and code “borrowed” from [this great article by Vadim Bulavin](https://www.vadimbulavin.com/modern-networking-in-swift-5-with-urlsession-combine-framework-and-codable/). So if you feel you need a more thorough understanding of the concepts you’re encouraged to read it.

### The Agent

The actual REST request is made using Combine’s own dataTaskPublisher*, *encapsulated in Vadim’s Agent* *struct*.*

<iframe src="https://medium.com/media/497e1238f054cdf05c93a94d949fccb4" frameborder=0></iframe>

### The model

We need now to what kind of JSON we get back from the database. if you run the *curl* command listed above you’ll get back this:

<iframe src="https://medium.com/media/2a5c21f7ea99a9aed2676343251acb55" frameborder=0></iframe>

Pretty much all the entities are not changing based on our schema, the only variable part is under the value key.

So let’s model first the invariant entities. The SODA namespace encapsulates everything related to *SODA* (obviously :-) and can be reused between apps, the use of generics ensures that it works with any kind of documents (the T in there is meant to be replaced by our document’s model).

The Item struct should be Identifiable (and we’re lucky we have an id key in the source JSON string) because we’ll use it in our *SwiftUI* views.

<iframe src="https://medium.com/media/1baaa2a226b2fd5f212ca06a7012b1a4" frameborder=0></iframe>

While we’re at it, I added things like our agent and the connection information. You’ll put here you’re own endpoint URL and password.

Now we will model our fruit document, based on the fields we used when we entered our test data.

<iframe src="https://medium.com/media/d9e4da5823c4365cef90f8e64c8a85d9" frameborder=0></iframe>

color is optional because not all documents contain it.

### The API request

Let’s create now a publisher encapsulating our API request (remember, the one returning all the documents in our collection)

<iframe src="https://medium.com/media/841efaf3b07eab5e355789bebc619518" frameborder=0></iframe>

Again, it’s generic, so it’ll accept any kind of document model.

The Agent publisher is sending a Response document, containing both the actual decoded document model object and the HTTP response. On *line 9* we keep only the model object and we pass it downstream.

Our publisher sends a SODA.Collection<T> downstream, where T will be “replaced” by our document model (the Fruit struct in our example).

### The view-model

We need now to actually call the API endpoint, parse the results and store them for later usage in the view.

<iframe src="https://medium.com/media/68ab26a3a308644f6924a89332b5d61a" frameborder=0></iframe>

Now, that’s a pretty standard ObservableObject.

On *line 3* we store the database items containing our Fruits (and using the generics mechanism we ensure that the JSON decoder in the agent uses the Fruit model).

*Line 12*: We receive a SODA.Collection* *value and we keep only the items array.

*Line 13*: Yes, I know, proper error handling should be implemented here, but remember, we want to keep things simple for now.

### The View

This is really the easiest part, we iterate through our items and display them in a readable format.

<iframe src="https://medium.com/media/a25511eb58c1768b1c175e6555f4c6d0" frameborder=0></iframe>

And that’s the cheerful result, exactly what we expected:

![](https://cdn-images-1.medium.com/max/2000/1*gf4diWCZyxAHhJHQ16AQBQ.png)

## Conclusion and next steps

This is only minimal proof of concept, but it goes through all the basics. We’ve created a cloud-hosted Oracle autonomous database, added JSON documents to it, executed some very basic queries, both in SODA and “classic” SQL and then we connected to it in a Swift iOS app via REST/ORDS.

IMHO, until recently, the cost of entry for anyone wanting to use an enterprise-level database as the backend for a mobile app was too high. It required very specialized skills to install, configure, and operate, plus the cost of licenses was above the budget of an independent developer, side-project, or small startup.

The overall solution was too complex, even factoring in the benefits (like raw power, stability, technological maturity, and very high scalability potential). With the advent of the cloud versions of enterprise software, that’s not the case anymore.

So, I plan to use this opportunity to expand our little example a little in future articles, adding all the CRUD operations and, of course, error handling.

The full Swift source is accessible on Github.
[**bogdanf/SODA-Swift-Client**
*Contribute to bogdanf/SODA-Swift-Client development by creating an account on GitHub.*github.com](https://github.com/bogdanf/SODA-Swift-Client)
