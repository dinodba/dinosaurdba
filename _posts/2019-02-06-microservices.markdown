---
layout: post
title:  "Microservices"
date:   2019-02-06 19:38:03 +0000
permalink: /microservices
---

So in the last post I rambled about REST APIs and gave the example of Spotify having lots of separate services that talk to each other using those REST APIs. The idea of modern service creation is to reduce the size of individual components to as small as possible - turning them into microservices. Each of these services able to be updated or re-written in a completely isolated fashion allowing change to happen much more frequently and, perhaps more importantly, testing to be completed on a much smaller part of a larger service - this reduces the testing requirements and the possible failure vectors.

For us dinosaurs, especially those of us still in large enterprises, we'll be quite used to multi-week release cycles with lots of testing and a rather larger and complex outage to our monolithic service introducing lots of change with lots of risk all at once. Not only does this create a lengthy service outage and bring with it risk that if realised would result in a difficult and lengthy backout it also slows the rate of change for your business. Microservices reduce the risk, simplify the testing and backout and allow continuous business improvement with really quick feedback if required.

For fellow DBAs [this](https://medium.com/oracledevs/data-consistency-among-microservices-is-it-possible-fe48938235d1) is a great write up that discusses microservices and specifically how to manage data consistency across microservices with their own separate data persistence layers.

And for fellow **dinosaur** DBAs, [this talk](https://youtu.be/pMiJSvz2q2c?t=5399) by Lucas Jellema is worth watching to learn that it's OK to use other database technologies.
