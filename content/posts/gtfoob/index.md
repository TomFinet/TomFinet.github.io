---
title: "January: Get the F#!k out of bed!"
date: 2023-01-06T18:09:23+01:00
draft: true
---

## Introduction

I want trade value for money to gain financial freedom. Since I enjoy working with computers, I am making apps. My first app tracks the number of hours wated lying in bed. Not sure how to monetise yet or if people actually care about this.

## Day 1: Planning (1 hr)

I will use the following respective technical modules.

0. Use [react-native](https://reactnative.dev).
1. User must press a button signaling that they are awake, and the device will need to register geographical movement using the [geolocation](https://github.com/michalchudziak/react-native-geolocation) module.
2. Use the asynchronous, persistent, key-value-pair [AsyncStorage](https://github.com/react-native-async-storage/async-storage) module.
3. Use some data visualisation module.

Here are my initial UI designs.

![Initial app screen UIs](posts/gtfoob/gtfoob.jpg)

## Day 2: Basic layout

I spent an hour or so installing android studio, react-native, enabling my android phone for USB debugging and running a test app on it. All pretty boring. Next I wrote the UI components. 

I started with the bottom tab navbar, which uses the [react-navigation](https://reactnavigation.org/docs/tab-based-navigation) module. Next I tackled the wakeup time screen. All I need is a list of weekdays each paired with a time picking component to allow the user to enter the desired wakeup time. I found a [datetime-picker](https://github.com/mmazzarolo/react-native-modal-datetime-picker) module that does exactly this.

At this stage, my app looks horrible, but a user can pick the desired wakeup time for each day of the week. I am prioritising functionality over beauty ... for now.

In the evening, I worked on the StatScreen a bit, and found a nice [bar chart](https://gifted-charts.web.app/barchart) module.

## Day 3: Storing data + bar chart

I only need to store the time wasted for each day. If we number each day, we can use it as the key, and the time wasted in seconds as the value. I'll keep track of the

0. current day number.
1. total time wasted,
2. total time wasted this year,
3. total time wasted this month,
4. total time wasted this week,
5. time wasted each of the last 12 months,
6. time wasted each of the last 4 weeks,
7. time wasted each of the last 365 days.

This amounts to 386 key-value-pairs of storage. Using 32-bit integers, that's roughly 3 Kilo-bytes. I know everything could be recovered from item 7, but I'm happy to trade space for time in this case.

