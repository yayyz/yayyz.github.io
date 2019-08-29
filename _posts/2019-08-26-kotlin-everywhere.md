---
title: Kotlin's Experimental features (from Kotlin Everywhere Seoul 2019 event)
layout: article
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 139, 87 , .4), rgba(139, 34, 139, .4))'
    src: /assets/images/IMG_8930.jpg
sharing: true
license: true
aside:
  toc: true
show_edit_on_github: true
show_subscribe: true
pageview: true
tags: conference kotlin
---
My notes from Kotlin Everywhere Seoul 2019 event
<!--more-->

## Experimental Features

### inline class

- avoids extra object allocation
- wrapper for only one value, no mutable property

### Contract

- allows to share extra information about the function behaviors with the compiler.
- Why can't compiler infer such information?
    - the inferred information...
        - can be implicitly change
        - can break the code
- provides explicit behavior for the compiler
- DSL will change (syntax) in the future

### Immutable Collections

- (before) despite it is read-only, the content can be changed by others who has the access to the reference.
- (after the change) there will be a new interface added "PersistentList" and it supports modification.
- ImmutableList is going to be "truly immutable"
- List that is created by PersistentList, shares the same data structure with the List.
- (side note) Similar languages like Scala, shares the same concepts about the immutable collections.

### Flows

- what is it? → suspend-based reactive stream
- reactive stream vs coroutine
    - if you have streams of events, data ⇒ reactive stream (Flow)
    - asynchronous programming ⇒ coroutine
- Integration with RxJava
    - Publisher < - > Flow
    - interchangeable
- Back pressure: automatically works
- Flows brings reactive streams to coroutine library.
- it is going to be stable REAL SOON

### Multi-platform projects

- what is it? → shares the common code between different platforms / shares business logic
- it is NOT the concept that has only one set of code for every platform.
- common code
    - can define `expect` for the common code
    - define `actual` for platform-specific codes to implement using their own platform languages
    - can use other multi-platform libraries

### Side-notes

- 12/4-6 KotlinConf in Copenhagen
    - expect some cool presentations about the multi-platform projects

## Presentation slide

[https://speakerdeck.com/svtk/whats-new-in-kotlin](https://speakerdeck.com/svtk/whats-new-in-kotlin)

