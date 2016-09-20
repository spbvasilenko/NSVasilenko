---
layout: post
title:  "Swift Array search performance"
date:   2016-09-09 00:00:00
categories: swift ios performance algorithms
---

In this week , I'm interested in how to have fun fast search algorithms in the array in the Swift. And he came to a big surprise!

<b>Story</b>

Right now, for Array implemented ```array.contains(element)``` and ```array.indexOf(element)``` for searching in an array. Both of these methods iterate over all elements in the array, starting at index 0, until they find a match. In Big O notation, the methods’ performance characteristic is O(n). This is usually not a problem for small arrays with only a few dozen elements, but we have array of 100000000 elements? We want to look for a faster search algorithm.

In the Swift, Apple doesn't implemented binary search for arrays yet! It was a big suprise for me!

If the array is sorted by the search key, binary search can give you a huge boost in performance. By comparing the middle element in the array to the search item, the algorithm effectively halves the number of elements it has to search trough with each iteration. Binary search has O(log n) performance. What does this mean in practice? Searching a sorted array of 100,000 elements using binary search would require at most 17 comparisons compared to the 50,000 comparisons a naive linear search would take on average.

<b>Measure performance</b>

Today, I make example project for measure performance beetween function indexOf in the Array with 100000000 elements and with my simple implementation binary search. 

So, finding the last element in sorted array with 100000000 elements takes:

- Standard ```indexOf()``` search - 3 sec average

- Binary search - 1.5 sec average

Not a bad difference, right?!

You can find my performance test in <a href="https://github.com/vasilenkoigor/SwiftArraySearchMeasureTest">my repo<a/>

<b>Swift evolution proposal</b>

Also, I do not understand why Apple has not yet implemented a binary search for the array in the Swift. In this regard, the <a href="https://github.com/apple/swift-evolution/pull/516">proposal<a/> I made in Swift evolution. But PR was closed because this is out of scope for Swift 4 stage 1.

Good link: http://bigocheatsheet.com