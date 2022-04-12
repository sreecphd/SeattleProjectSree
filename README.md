This repo represents my attempt to create a proper Android app. By proper I mean an Android app built following Google's recommended approach, which means:

1. Follow clean architecture principles
2. Use the MVVM design pattern
3. Use Google's Jetpack components Navigation and Fragments for UI
4. Use Room for local storage
5. Use Dagger2 for dependency injection.
6. Use Google Firebase/Firestore as my online storage.

The app I'm creating is called Seattle Explorer. It will list interesting locations around the city of Seattle.

I'll document each step here in the README.md to help me learn and remember the steps involved and the problems I've run into along the way. I've found my brain remembers coding concepts better when I force myself to add comments to my code as I'm coding. I'll track my progress using branches, so anyone can follow along with my progress. There's plenty of MVVM sample code projects out there, but I've often found them difficult to follow. The samples are often working apps in their final working state. But how did they get from start to finish? With this project I'll attempt to document the process from start to finish.

Finally, if anyone using this repo/code find bugs please submit an issue using Githubs issue tracking.

**MY STARTING POINT**
I consider myself a beginner Android developer. I've written one large-ish Android application. My application was fairly complex; I wrote custom views, used Google location services and utilized Firebase as my backend. But I don't have any experience with Fragments, Databinding or Dagger and like most beginner developers most of my code ended up in the Activity. Now it's time I grow as a developer and learn the proper way to architect an Android app.

**STEP #1. Creating a new Android project.** I'm using Android Studio 3.2. I created a new Android project, gave it a name (Seattle Explorer), chose a minimum SDK (21 Lollipop) and when asked to chose an activity type, I chose Fragment+Viewmodel. I chose Fragment+ViewModel because it's a new option added to help facilitate the use of Fragments and ViewModels, two things I'll want to add to my project. I'm hoping the boilerplate code will be helpful. WOOHOO I have a working Android app with fragments already. I added this README.md file and this will be my first commit.

**STEP #2. Basic project clean up.** I've created enough Android apps to know that it's nice to have some organization in my build.gradle files. I'll start by consolidating all my versions numbers in the project build.gradle file. This it pretty standard, just create an ext section within the buildscript section. I also lowered the compileSdkVersion to 27 so my app will work with more phones, but still work with the Jetpack Navigation library (which requires AppCompat 27+). It's best to do this at the start of a project before I add any code.

**STEP #3: Configuring Jetpack Navigation.** We'll we have a working Android app, with a Fragment and viewModel but I need to convert this boilerplate code to instead be using Google's new Navigation library. To configure Navigation you need to do the following:
* Make sure you're using the latest version of Android 3.2.
* In Android Studio visit FILE -> SETTINGS and go to the Experimental tab. Make sure "Enable Navigation Editor" is enabled. 
* Add the two new dependencies for Navigation they are: android.arch.navigation:navigation-fragment, and android.arch.navigation:navigation-ui
* Add the navigation-safe-args-gradle-plugin Gradle plugin in the Project build.gradle.

**STEP #4: Setting up Navigation.** Now that Navigation is configured, I'll go ahead and create a basic setup with two fragments and viewModels. Here are the substeps needed:
* Start by adding a navigation graph. You do this by right-clicking the res folder and selecting NEW -> ANDROID RESOURCE FILE, naming it nav_graph and for resource type selecting "Navigation". That will create a navigation resource folder and create this new XML file.
* I created 2 new fragments using the "Create Blank Fragment" tool within the navigation editor. The only thing I added was the code to instantiate a viewModel within each fragment.
* Back in the editor I added both fragments to the graph, made StartFragment the Starting fragment, and then connected the two fragments with an action. For more information see (https://codelabs.developers.google.com/codelabs/android-navigation/#0)
* Next you need to add a placeholder view within the layout of your Activity. This fragment view Id is nav_host_fragment.
Finally to test the functionality of my viewModels I added a simple link in the PoiDetailFragment from it's viewModel

**STEP #5. Creating the Domain layer.** Now that I have a working Android app with a simple navigation graph. I'm going to start adding a bit more complexity to my existing app. Here is the architecture I'm building: ![Imgur](https://i.imgur.com/e7xHP9Q.png)

The domain layer will host use case classes. These use case classes will represent information needed by the presentation layer, AKA viewModels. Right now my apps PoiDetailViewModel is creating detailMessage string resource itself but in it's final form the presentation layer would get this information from the domain layer. I'll go ahead and alter the app so the domain layer is passing this information into presentation layer (via the PoiDetailViewModel). I just created the Domain package and added a GetPois class. POI is short for "Point of Interest" one of the main data types I'll be using in the application. At the moment my class has just one function, getPoiDetailMessage as I'm trying to keep it as simple as possible. In the viewModel I now instantiate a GetPois class and ask it for the detailMessage string. WooHoo! I now have a working Android app with a presentation layer consisting of viewModels and fragments (managed by a NavController) and a domain layer that is passing information into the viewModel. However now I'm breaking one of the main tenants of clean architecture. The Presentation layer code has a dependency on another external layer, the domain layer. To fix this problem I'll need to re-organize my Viewmodel classes and move the dependencies into the constructors and a that means dependency injection. Dagger time.

**STEP #6. Setting up basic Dagger Dependency Injection.** Here is an overview of all the steps needed to configure Dagger DI and getting it working in my project:
* Add the Dagger dependencies (see the app build.gradle file) to the project. There are a total of 5 dependencies required. 
* Create an Application class. As someone unfamiliar with Dagger here was the first place I initially got confused about the Dagger setup. I wasn't sure if I needed to implement ActivityInjection if I wasn't planning to inject anything into an Activity. Could I stick to FragmentInjection only? After some experimentation it seems ActivityInjection was required. Okay then, my application class would implement both HasActivityInjector and HasFragmentInjector. Now I ran into my second problem. The AndroidInjection.inject(this) method was complaining that my Fragment was not a true Fragment, at lease not according to the AndroidInjection class. It seems Dagger has 2 versions of the Fragment injector. This wasn't documented very well in the Dagger documentation so I'm guessing a bit here, but apparently if you're using the Support library you need to use the Support injector instead. So instead of implementing HasFragmentInjector I need to instead implement HasSupportFragmentInjector and use AndroidSupportInjection.inject. After that change everything was working correctly.
* Create the AppComponent interface. This is the interface Dagger uses to configure your Dagger injection classes.
* Create various DI Modules. These classes are used by Dagger to create the Dagger injection classes. The ActivityModule and FragmentModule are used to tell Dagger where injection begins. 
* Create the ViewModel module and factory. Daggers default viewModel factory apparently only works with empty constructors, but since I plan to pass in Domain parameters to my viewModels we'll need to update the viewModel factory. This improved ViewModel factory design pattern has been used often in other Android apps.
* Inject the GetPois class. The whole point of this step was to be able to inject my GetPois class into the PoiDetailViewModel. So I altered the PoiDetailViewModel class, adding the @Inject constructor and moving GetPois to the constructor. Finally I also altered the GetPois class to add an empty @Inject constructor. This tells Dagger that it can instantiate the GetPois class

After all of this work what is Dagger able to inject into my app? Anything with the @Inject annotation. That means:
* ViewModelFactory (from PoiDetailFragment)
* GetPois (from PoiDetailsViewModel)

At this point I have a working Android app with a presentation layer consisting of fragments and viewModels, app navigation controlled via the Android JetPack Navigation library, a super simple Domain layer and dependency injection via Dagger. Time for a commit.

**STEP #7. Flesh out the Domain layer.** The domain layer will house my business logic. This business logic will be broken down into usecase classes following the clean architecture pattern. Also my plan is to use the Observable pattern to move data between my repository layers and up to the Domain layer. The Domain layer will send the Observables data along to the ViewModel, where the viewModel will convert it to LiveData, so that it's easier to present in the views. With that in mind here are the steps required to flesh out the Domain layer:
* Installing the ReactiveX dependencies. This should be a pure kotlin layer, so all I'll need is io.reactivex.rxjava2:rxkotlin
* POI_domain model. Poi's are one of the main entities I'll be using in the app. Since I want to follow the clean architecture principles the layers should not share any knowledge about each other. That means a POI in the domain layer could be different than a POI in the presentation layer or Repo layer. That means that I need to have separate POI models for each layer. I tried to determine if there is any sort of standard naming convention for this, since I really don't want to use the same name for the models as that might get confusing. So I'm going to suffix each model with the name of the layer. So my POI model in the domain layer will be called poi_domain. I'll need to create mapper classes in each layer to convert these similar model classes.
* Threading responsibilities. Using RxJava means I'll need to manage 2 threads, where I subscribe on and where I observe on. But what layer should handle this logic? According to [this article](https://medium.com/@yoelglus/threading-strategies-on-android-with-clean-architecture-57f6d3714d96) the Viewmodel and repository should. I'm not so sure and instead chose the Domain class to handle this, but it's definitely a point of contention in the industry. With that decision made I'll create an interface called ObservingThread so I can get a link to the main thread from the Presentation layer in the domain layer
* ObservableUseCaseBase. To eliminate some of the boilerplate code, I'll create a base class for my UseCase classes. Credit to [Joe Birch](https://twitter.com/hitherejoe) for showing me this pattern. The base class is specifically for Observable return types. There is a lot of stuff happening there so feel free to check out the file.
* The Repository Layer. To really flesh out the Domain layer I need to do some refactoring starting with the creation of the Repository layer. I created an interface for repos called ExplorerRepository and created a simple stand-in repository so that I can test my domain code. I also created some helper utility classes to help me fake a repo. Next I refactored my getPois usecase class to inherit from ObservableUseCaseBase. This is getting to be too much work for one step (and one commit). So I'll go ahead and stop and commit these code changes before I continue.

**STEP #8. Refactoring the Presentation layer.** So far my test data was a simple string passed along to the Presentation layer. Now I've moved on to an Observable data type and need to alter the Presentation layer to be able to accept it. Here are my tasks in this step:
* I'll start with a minor change, renaming the ui package presentation.
* Then I'll create a Poi_Presentation model (as describe earlier)
* Subscriber class. Accepting an RxJava observable means the viewModel needs to act as the observer and subscribe to the Domain layers getPois Observable stream. I'll create a inner class in the viewModel to handle this.
* Mapper class. Note in the PoiSubscriber classes onNext method I'm using a mapper class to convert Poi-Domain's to Poi_presentations. This required a new PoiMapper class
* I need the Viewmodel to execute the use case. This action is the viewModel asking for this observable data from the domain layer. I accomplished this by adding an init block.
* LiveData. Since I want to take advantage of Google's LiveData component, here in the ViewModel I convert the incoming RxJava Observable data into LiveData. So I create a private variable to store a reference to this MutableLiveData and in the PoiSubscriber classes OnNext method I'm posting values to the LiveData stream.
* Disposables. Since the Viewmodel is acting as a subscriber to the Domains RxJava stream, I need to make sure to dispose of the subscription properly. So I override the onCleared method and added a dispose call there.
* Observing LiveData. Finally in the fragment itself I created an observer to the LiveData, and grabbed the name of the first item in my POI list.

Great work so far. I now have a working Android app using the MVVM design pattern, RxJava, a domain and simple repository layer...navigation, fragments and viewModels. 

**STEP #9. Viewmodel Unit Tests.** Now that I have a properly designed Android app, with dependency injection I can write my first unit test. I'll start with the PoiDetailsViewModel. Actually I'll begin by refactoring the feature name and package to better align with where I'm going with this feature. Instead of a Detail view it's actually a list of POI's, so I'll change the name of the Fragment to poiListFragment and change any references to it or its viewModel. With that done I can install a few new test dependencies; com.nhaarman.mockitokotlin2:mockito-kotlin a popular 3rd party library that adds some Kotlin goodness to Mockito, and android.arch.core:core-testing which is needed to test Jetpack components. After a day of struggles I was able to add some unit tests to the project. I ran into problems trying to stub the mapper class. I could not find a ArgumentMatcher for any NonNull object. Anyway time to commit and move on to something else so I can be productive again.

**STEP #10. Creating the Repository module.** The Repository module is the module that will handle managing the dataStores. At this point that means a local cache, which will be handled by Room and a Remote data store, which will be handled by Firebase. The main class here is DataRepository. This class determines what DataStore to use to complete our data requests and I added extensive notes in the source code. The other work I did as part of this step:
 * I created yet another version of the POI model so that this module has no knowledge or dependency on outside classes. 
 * I added interfaces for Cache and Remote modules I'll be creating in the future. 
 * I created Remote and Cache broker classes. Hopefully broker is the correct name to use here. These classes are used to communicate with the Cache and Remote modules. I'll admit I have some concern that I may be over designing this app, but what the heck. I'll deal with that problem when it comes.
 * I wrote unit tests for the DataRepository and Poi_RepoMapper classes.

**STEP #11. Building the Firebase Remote Module.** I created a Remote package and a FirestoreManager class. This class implements the ExplorerRemote interface, which at this point is only one method "getPois". This step obviously required that I have a Firebase project and data stored in Firestore that I can pull down. You can read their documentation to figure out how to add Firebase to your project and what dependencies were needed. It's pretty straight forward. I originally tried to use Francisco Sierra's RxFirebase library, but did not have luck getting it to work, so instead I went with a more standard approach. And like other layers, I've created yet another POI model, this one called FireStorePoiResponse, but in this case I've added the mapper class functionality right into the model class. I'm not sure which mapper approach I like more.

**STEP #12. Continuous Integration with CircleCI.** Now that I've started to write some unit tests I decided to be a bit more professional with my build process. I created a [Kanban board](https://github.com/szaske/SeattleExplorer/projects/1), which comes free with Github and I setup the project in CircleCI so that my project is continuously being integrated into my master branch. That includes decoding my google_services.json file (API keys for Firebase) as an environment variable within CircleCI. Thanks to Alan Tai for showing me [this approach](https://medium.com/@ayltai/all-you-need-to-know-about-circleci-2-0-with-firebase-test-lab-2a66785ff3c2). Now with that setup complete, I have CircleCI lint and unit testing each branch at check-in. The final step was to enable a master branch protection within Github so that I'm not allowed to complete the pull request until the branch passes my tests.

**STEP #13. FirestoreManager Unit Tests.** Before I wrote some complex unit tests I decided to performed some major refactoring of the FirestoreManager class itself. The biggest change was to convert the results from Firestore into a Single instead of an Observable. As I was writing FirestoreManager I noticed that I was essentially using the Observable stream as a Single anyway. Users will usually be retrieving this data from the local cache and when I need to update the local cache I should only need to grab the data once a day, so I single seems to be the best approach. That meant I needed to go back down the chain and make changes to various interfaces in the other layers. I also needed to create a new SingleUseCaseBase class in the domain layer. Some of the notable things about these unit tests:
* mockDatabase needed to support RETURNS_DEEP_STUBS so that I wouldn't need to mock the whole method chain.
* I used an argumentCaptor to mock the return callback from my OnCompleteListener. It took me a long time to figure out how to accomplish this, and finally stumbled upon [this article](https://fernandocejas.com/2014/04/08/unit-testing-asynchronous-methods-with-mockito/) by Fernando Cejas.

**STEP #14. RecyclerView and Adapter.** This was a fairly simple task, one most Android developers would have done many times before since RecyclerViews are probably the most used Android view. So I wont add much commentary here.

**STEP #15. Resource wrapper class in Presentation layer.** Now that I'm pulling down a large list of data from Firestore and loading it into a RecyclerView there's a noticeable lag in the UI while this is happening. The RecyclerView sits empty for ~2 seconds while I pull down the data in the background. The fix for this is to have the viewModel/fragment keep track of the status of the poiList Single call. If the UI can determine the status of the data download it can show a progress bar and let users know that it's waiting for data. To accomplish this I'll create a Resource wrapper class that will include my original data (A list of POI) as well as a status (a ResourceState enum) and a message, in the case that an error message needs to be sent. This means I needed to change the poiData LiveData from a List of POI to a Resource that wraps that list. The Resource states are Loading, Success and Error. I've written code in the fragment to show the progress bar when the livedata Resource is in the LOADING state. I also needed to refactor the unit tests to make them aware of this new LiveData class.

**STEP #16. Adding a local Cache Data Store.** Now it's time to a local cache data store option. This will help the app function more smoothly and save me some bandwidth expenses. Since I'm attempting to follow Google's Android recommendations, I'm going to use Room as my database ORM. You can learn more about it [here](https://medium.com/androiddevelopers/room-rxjava-acb0cd4f3757). The basic logic is this; The Data Repository class will first check to see if I have the data cached locally and what the age of that cache is. That means I need 2 tables of data. One table to store my POIs data and another simpler one to store the date of my last save to the POIs table. Here is what the poi table looks like:
![poi_table](https://i.imgur.com/MIT4Pg0.png)

And here is the table where I store the lastCacheTime:
![lastCacheTime](https://i.imgur.com/caR5Izm.png)

Here are the steps involved to get my local cache up and running:
* I created 2 new models one for each table I needed; the POIs (PoiCache) and the CacheStatus. These models need to have the @Entity annotation to work with Room.
* I created 2 Constant classes to store my SQL query strings; one for each table.
* I created 2 Dao classes for each table.
* I created the Implementation class, which implemented the ExplorerCache interface.
* I created the Database class ExplorerDatabase
* I wrote mapper classes to map POIs between the Cache and the Repository
* I had to refactor my existing DataRepository class to choose between the two data sources
* I wrote unit tests to test the logic of these classes.

I ran into a few problems trying to find a method to check for the existence of data in the POIs table. I originally was going to call the getPois function and then call the IsEmpty method on those results. Unfortunately this wouldn't work. I was returning a Single with the getPois call and Single do not allow for returning a null/empty response. I ended up creating a CASE SQL query that solved my problem. I also ran into errors with ROOM's "Cannot access the database on the main thread" error. This required that I inspect each Observable I was creating and making sure to use RxJava's SubscribeOn and ObserveOn functions.

**STEP #17. Add Collections list functionality.** The POIs (Points of Interest) in the app are grouped in what I call collections. Examples include "Henry", which are locations that showcase art by the popular Seattle Muralist Ryan "Henry" Ward, and Oddities which is a collection of Odd Seattle POIs of which there is a good amount. I have collections data in Firestore that needs to be downloaded to the app, so that is the feature I'll add next. The POIs list fragment and UI was just a functionality test and will not actually appear in the final app. Instead when users browse the full list of POI's they'll start by first being shown a list of collections and once they click on a collection they'll be shown a list of the POIs that are included in that collection. Now that I have experience downloading POIs adding this functionality was trivial. I just needed to replicate the POIs features and rename any new classes or functions using the term 'Collection' instead of 'Pois'. 

Once I started this process I found I needed to refactor a bunch of my previous classes that I had built considering only POIs and had included the name Pois in the class name. In some cases these classes needed to now be shared amongst all of my various data types. For example I had a class called PoiMapper that was used to map POIs from domain models to repository models. It made sense to name it PoiMapper when that was the only model being mapped, but now that I also needed to map Collection models it no longer made sense. I wanted to have only one mapper class per layer so I renamed this class to RepositoryMapper, since it was in the Repository layer and now it acts as a universal mapper for all model types in the Repository layer. Here are some of the other notable work items in this step:
* Added a Collections entity class in the Cache layer and added it to the ExplorerDatabase class. This told Room that I wanted a new table in the database.
* Created a CollectionsDao class to help me get data from the database
* Refactored the CacheStatus class. This is the class that tracks the last time I updated the local cache. I needed to alter it to be able to track multiple tables.
* Created a new Collection List fragment and viewModel.
* Updated my dependency inject classes to be able to inject this new fragment.
* Updated the DataRepository class to have the ability to getCollections
* Updated ExplorerCacheImpl to support Collection functions
* Updated ExplorerRemoteImpl to support Collection functions

**STEP #18. Collection list unit tests & Timber.** This step is just some basic project clean up. I added a lot of code in the last step and now I needed to catch up on my unit tests. It's a good thing I did too. I found a bug in my downloading collections Firestore code. Firestore's QuerySnapshot class includes a nice toObject method that can convert your documents to any class you'd like. Unfortunately because of the way Firestore stores it's documents that method does include the document id strings when you query the documents. That meant I needed to add some additional code to ensure that Id's get included. Creating the unit tests helped me find this bug. Also, since my code is starting to get larger I decided to also include [Timber](https://github.com/JakeWharton/timber) in my project.

**STEP #19. Communication between Fragments and the Activity**. Now that my app is starting to get more complex I need to consider how I'll handle communications between the Activity and the various fragments I'll need to design. Here are some helpful links:
* https://proandroiddev.com/what-problems-exist-with-multi-activities-4ea1a335d85a
* https://medium.com/mindorks/how-to-communicate-between-fragments-and-activity-using-viewModel-ca733233a51c
My main concern is user authentication. Authentication will be handled by Firebase, but how will I keep track of the user and whether they have been authenticated or not? Since I'm using RxJava it seems obvious that i'd want to use an observable to track the status of the user. After some investigation I've decided to move to a shared viewModel design pattern. Up until now each fragment in the app had it's own Viewmodel and this is nice for separating the code into logical components. But it also had some drawbacks, at this point mainly related to ny user authentication problem. I want multiple fragments to know who the user is and having viewModels per fragment means I'd need to call the same UserAuth use case class each time the user switched fragments. With the shared viewModel approach all fragments share the same viewModel, so I need to only call this once when my one activity loads. Oh and the activity shares the viewModel as well. At least that's my thinking so far. So the work in the step involved creating some new fragments and a SharedViewModel that is shared by all the fragments. I also spent a lot of time tracking down bugs and strangeness with the Jetpack Navigation library. You can read more about it in this [SO post](https://stackoverflow.com/questions/51173002/how-to-change-start-destination-of-a-navigation-graph-programmatically-jetpack/53017625#53017625). I'm starting to have concerns about the libraries youthful age. Perhaps it's not yet ready for prime-time. I guess I'll figure that out as I continue to build out the application. 

**STEP #20. Adding an authentication service and login abilities.** This was more difficult then I was expecting. I needed to add the ability for a user to login to the app.  Since I'm using Firebase as my authentication provider I started by investigating FirebaseAuth-UI. FirebaseAuth-UI is a pretty slick library provided by Google.  It creates all of the login UI for you and supports all of the various authentication providers supported by Firebase; Google, Facebook, Twitter, etc. It makes adding authentication really simple.  Unfortunately it also had a funky non-standard user flow that I was not liking. Most apps have a standard login workflow; 2 screens one for registering and one for logging in.  FirebaseAuth-UI combined the two screens, asking the user to type in an email first and then either registering or logging in depending on if it recognizes the email. I wanted separate login and register screens and since I couldn't get FirebaseAuth-UI to customize to this sort of UI I gave up on using it.
That meant I needed to create my own Login UI. That was easy enough since I already had my fragment system working and a shared viewmodel. Here are some of the more interesting activities I need to complete to add this functionality:
* I needed to create an AuthService. This meant creating an interface and implementation, and hooking it up through Dagger so that I could inject the service.
* Determine where to inject the AuthService into? I could inject the service into my activity, the loginFragment or the SharedViewModel and I spent a lot of time trying to figure out the best approach. My original thought was, since I've been falling in love with rxJava was that I should have a user observable in the viewmodel that keeps track of authentication status of the user. I started down this path and quickly found this was the wrong approach. The FirebaseAuth service itself wants to be the class that manages the status of the user. Keeping a copy of the user in your viewmodel just breaks the whole thing. All of the sample code provided by Google has authentication being provided by the activity but it didn't seem right to inject the service there as I wanted user data to be available to my various fragments as well. I ended up with a sort of hybrid approach, injecting the service into the SharedViewModel and accessing it from the Activity
* Determine how to interact with the Activity. Once I had the basic login process working I needed to do some interaction with the activity. This is one of the reasons I ended up having the activity handle the process itself. When the event is kicked off I wanted to grey out/disable the screen and show a progress bar. I also needed to alter the options menu to make a logout option appear. Doing this work in the activity made this a simple tasks. Although I also resorted to using the OnFragmentInteraction interface to make these abilities accessible from my fragments as well.
* Fix problems with the actionbar up arrow. The Jetpack navigation sample code suggests that you `setupActionBarWithNavController`, however this setup creates a situation where you all of your fragments *except* your start destination will have an up arrow on the actionbar that takes you to the fragments parent. The problem with this is that it breaks the LoginFragment by giving the user a way to circumvent the login process (they just need to press the up arrow). To solve this problem I used the pattern suggested in this [excellent SO answer](https://stackoverflow.com/questions/50301820/remove-up-button-from-action-bar-when-navigating-using-bottomnavigationview-with)
* Update CircleCI configuration.  Firebase Auth requires a SHA-1 fingerprint in your application.  That means adding a keystore to your Android project.  Make sure to add *.keystore files to your .gitignore file so your keystore does not get added to your repository and shared with the world.  Also, once you add a keystore to your project Android Studio unfortunately adds your keystore password in plain sight within your apps build.gradle file.  You'll want to follow [these instructions](https://stackoverflow.com/questions/20562189/sign-apk-without-putting-keystore-info-in-build-gradle) to remove these secrets from your repo and protect your keystore password.  Since I'm using CircleCI as my continuous integration service, these changes meant I needed to update my Circle config to get my builds completing successfully.  You can follow [these steps](https://stackoverflow.com/questions/44911547/where-to-store-android-keystore-file-for-cirlceci-build) to learn how to import secret files using CircleCI environment variables feature.  My CircleCI config.yml file should also be helpful in showing you how to accomplish this.  Finally, adding a SHA-1 also required that I update my google_services.json file, another secret file that does not get stored in the repo.  My config.cml successfully imports all 3 secret files; google_services.json, keystore.properties and debug.keystore into CircleCI.

With this work complete, I have a working login process, if a bit basic. I have not added any validation to the login UI or truly disabled to screen when waiting for the login event to complete. but more importantly I haven't added the ability to register a new user. But for now I'll check in this code and get to that next.

**STEP #21. Add user registration functionality.** This was a fairly easy task, so not much to report here. I added a new registerFragment and linked it into the NavGraph. I also ran into another problem similar to my issue in the previous step with the up arrow. I was not testing the back button (know programatically as the back stack).  With my new Registerfragment and the LoginFragment a user was able to press the back button and get back into the app without logging in. To solve this problem I needed to use some functionality offered by the navigation library with the method:
```
NavOptions.Builder()
    .setPopUpTo(registerFragment,
        true)
```
I hate the method name as it makes no sense to me, but what it does is it clears the back stack so you are not able to return to the named fragment.  Here's a list of the other work I accomplished in this step:
* Validation. I added validation to both the login and registration fragments. I wasn't sure where to put the code and settled for putting it in the viewModel. Now I see why some Android developers worry about the viewModel housing too much code.
* Split ViewModel. Once I added validation it became clear to me that my approach of using a shared ViewModel was not a good one.  it was just getting too messy. So I ended up splitting up the viewmodel again.
* Began Instrumented Testing. Now that I've added UI that user actually interact with I needed to start thinking about UI testing. I began to implement automated tests using the [robot design pattern](https://academy.realm.io/posts/kau-jake-wharton-testing-robots/) created by Jake 'I create all the good stuff' Wharton but quickly realized that this would be too much for one step. So I'll stop here, check in my changes and leave instrumented tests for my next step.

**Step #22. Instrumentation Tests.** As mentioned previously in this step I added instrumentation tests for the Login and Registration fragments. To accomplish this I relied on several key testing libraries and tools:

* Spoon. An excellent automation tool developed by Square. Spoon is a distributed instrumentation tool that runs your tests on multiple devices simultaneously, takes screenshots and records the logcat for each device.
* Falcon. An improved screenshot tool that is able to capture dialog boxes and snackbars.
* Android Test Orchestrator. When I started using Espresso it quickly became apparent why so many devs aren't fans of it. Writing tests is pretty straight forward, but you can quickly run into aggravating road blocks. One of the first I ran into was "how would I ensure my tests all started with the same state?" Login is a good example here. If I wrote a test that logged a user in, at the end of that instrumented test the user would stay logged in. The second time you run this test it will fail because the user would already be logged in. This is the opposite that happens with unit tests and can be frustrating. Android Test Orchestrator solves this problem. It ensures that after each test the app is refreshed to a clean state. Once the dependency was install I just needed to add `execution 'ANDROID_TEST_ORCHESTRATOR'` to the testOptions in my app build.gradle file.

The robot pattern makes tests very readable and I'm really liking the pattern. You start by creating a BaseTestRobot class that includes some basic functions like clickButton and selectMenuItem. Then for each UI screen you create a specific "robot" class. This class is specifically for testing that one screen and contains specific functions for exercising each view/control in the UI.

Perhaps the most difficult aspect of Espresso testing is finding a way to handle asynchronous activities. Espresso has no way of knowing when the app is waiting for a network call or in my case when the app is waiting for a Firebase callback. This is especially important when testing the Login and Registration UI as Espresso will not wait and crash when it tries to interact with UI that has not yet been created. Google created a solution for this called IdlingResource, but the documentation for this class is slight to say the least. StackOverflow was also unfortunately not very helpful giving multiple complex and out-dated responses on how to implement a IdlingResource class. My solution required that I add functions to the Activity and SharedViewModel, functions only needed for testing purposes. That rubs me the wrong way, but in the end I do believe it's the best approach and the code was minor. I created a new state variable in the SharedViewModel called `progress`. This boolean keeps track of when the user is waiting for an asynchronous network call. Since I already display a progressbar it was fairly easy to map this boolean to the state of the progressbar. Then I just needed to create my own IdlingResource class that mapped this variable to the isIdleNow function. Here is that code:

```
import android.support.test.espresso.IdlingResource
import com.loc8r.seattleexplorer.MainActivity

class RegisterIdlingResource constructor(
        private val mainActivity: MainActivity
) : IdlingResource{

    private var resourceCallback: IdlingResource.ResourceCallback? = null

    override fun getName(): String {
        return RegisterIdlingResource::class.java.name
    }

    override fun isIdleNow(): Boolean {
        return !mainActivity.isInProgress()
    }

    override fun registerIdleTransitionCallback(callback: IdlingResource.ResourceCallback?) {
        this.resourceCallback = callback
    }
}
```

Finally I needed to register this class before each instrumented test, which I accomplished by adding

```
IdlingRegistry.getInstance().register(RegisterIdlingResource(activity))
```

to my @Before function. With this in place Espresso becomes smarter and waits for asynchronous activities to complete before continuing the test steps.

Base version
Added second changed line to this file
Added third line to this file
Added forth line