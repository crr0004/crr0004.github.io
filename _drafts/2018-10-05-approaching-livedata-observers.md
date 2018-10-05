---
layout: post
title: Thoughts on Approaching LiveData Observers
category: programming
tag: android
---

## Intro
In my journery of creating my [IntervalMe app]({% post_url 2018-08-07-intervalme %}) I orignally just used [Android's Room](https://developer.android.com/topic/libraries/architecture/room) without using [LiveData](https://developer.android.com/topic/libraries/architecture/livedata) and [Observers](https://developer.android.com/reference/android/arch/lifecycle/Observer) which meant I was struggling with the asynchronous behaviour of database queries. I am going to explain my journery of changing my approach to using LiveData and Observers.

## LiveData
In case you're not aware of the LiveData class is, it's basically a generic class which wraps around any class and employs the observer design pattern to ensure data consistenty across applications. The reason the Android developer team built LiveData was to couple it with their other architecture systems, see [Android and Architecture Developers blog](https://android-developers.googleblog.com/2017/05/android-and-architecture.html).

## Room
In case you're not aware of Room is, it's basically Android's answer to wrapping around sqlite to work with object orientated programming. You can read a full explination on the [Android Developers guide for Room](https://developer.android.com/topic/libraries/architecture/room).

## How I first used Room
If you go back into my commit history for IntervalMe on Github, you can see where I started the switch of when I started using LiveData.