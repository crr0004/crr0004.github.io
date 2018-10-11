---
layout: post
title: Thoughts on Approaching LiveData Observers
category: programming
tag: android
---

## Intro
In my journey of creating my [IntervalMe app]({% post_url 2018-08-07-intervalme %}) I originally just used [Android's Room](https://developer.android.com/topic/libraries/architecture/room) without using [LiveData](https://developer.android.com/topic/libraries/architecture/livedata) and [Observers](https://developer.android.com/reference/android/arch/lifecycle/Observer) which meant I was struggling with the asynchronous behavior of database queries. I am going to explain my journey of changing my approach to using LiveData and Observers.

## LiveData
In case you're not aware of the LiveData class is, it's basically a generic class which wraps around any class and employs the observer design pattern to ensure data consistently across applications. The reason the Android developer team built LiveData was to couple it with their other architecture systems, see [Android and Architecture Developers Blog](https://android-developers.googleblog.com/2017/05/android-and-architecture.html).

## Room
In case you're not aware of Room is, it's basically Android's answer to wrapping around sqlite to work with object orientated programming. You can read a full explanation on the [Android Developers guide for Room](https://developer.android.com/topic/libraries/architecture/room).

## How I first used Room
If you go back into my [commit history for IntervalMe on Github](https://github.com/crr0004/IntervalMe/commit/2ead8d57b458c199bce341b2f2c6e1361f63dc16), you can see where I started the switch of when I started using LiveData. I originally just pulled all the data from the database synchronously in the adapter however as things got a bit more complex, and I started doing larger data tests, the application was **very** slow. You can see an example of this in the [IntervalListAdapter](https://github.com/crr0004/IntervalMe/blob/1ac7ad04e0cf3960b1a493dd1400c21f77c53c26/app/src/main/java/io/github/crr0004/intervalme/IntervalListAdapter.kt#L144). I initially was trying to optimize the app by caching and looking at moving data retrieval to an asynchronous call, and that's when I found LiveData.

## Moving to LiveData
Initially I had a lot of trouble moving my code base to LiveData because it was an new API to me and because the code wasn't designed with it in mind. I ended up re-writing large chunks of how the adapter interacted with data. One of the biggest changes was I needed to stop thinking of the adapter as the bridge between the database and the views, and instead think of the adapter as bridge between whatever data it is given, and the views it will represent. This is in fact how the new [RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview) operates, and ideally I would move the IntervalListAdapter to a recyclerview implementation, but as of writing this, I haven't done it as it's not a priority.

## Moving Design Process
When moving to LiveData, I mentioned I had to re-write large chunks of the code, to do this I had consider how the data moves between the various systems from top-down. The points the data moves through (for the interval list) are as follows:

- The list activity
- The adapter
- The activity to manage each interval
- The view representing each interval
- The controller facade

To re-write all the parts I used a top-down design patten, following how the data would move around.

So to start, I started with the adapter and how the views would get the needed data. I used a hash set, with the key being the group position, and the value being a list of the child intervals. From here I could manually set some sample data and ensure the adapter works with the new data structure.

After getting the adapter working, I continued to work getting the activity to fetch the data and pass it to the adapter. The activity pulls data from a ViewModel, which pulls from a repository, which pulls from the database. The ViewModel gives back a LiveData source, which will, some point in the future, might have data put into it. To work with LiveData, you set the observer and in the observer this is where you put the actions to set the data into the adapter.

`Note: The abstraction layers are so in the future, if you want to move the data to a different source, you can.`

Getting the ViewModel, and repository pulling data is just moving the old way I fetched data from the database into various methods, using the executor patten to keep it off the main thread.

The other part of note, is the controller facade. The controller facade is responsible for controlling how the intervals interact with each other. It uses a singleton pattern to ensure the ease of use for controlling the various behaviors of the controllers. I created the facade because it was becoming very difficult for the various parts to handle how the intervals interact with each other. 

I used a couple interfaces that the various parts of the system set into the facade. This is so it can perform the needed behavior without needing to know all the details of how the other parts of the system operate. For instance, the `IntervalControllerDataSourceI`, is implemented by the adapter to be a data source for the controllers.

`Note: The controllers are responsible for handling the internal clock timing, keep the view updated to the data source, reporting starting and stopping of the intervals, and anything else related to view, single data interactions.`

After re-writing everything to work with LiveData and observers, I found everything worked a lot better and things became a lot easier to maintain and extended.  I now feel a lot more comfortable working with ViewModels, LiveData and observers and can quickly write with them.