---
layout: post
title: A small little improvement to your testflow
---
Last weekend my company Squeed hosted an internal hackathon were we had the opportunity to meet up, exchange experiences and code together! During the hack we talked a bit about testing in Java and how nice it would be to have instant confirmation about the status of your tests. None of the participants in the discussion had such a setup today, so I went on an adventure all the way to the almighty Google and found this: <https://infinitest.github.io/>



Infinitest allows you to continously run your tests while refactoring/developing in your source code. It has plugins for both eclipse and IntelliJ, though I've only tried it out with IntelliJ. The installation was a breeze since you could simply use the plugin-manager to install it and then activate it by adding it as a facet to your project. You might want to activate "*Make project automatically*" (Settings -&gt; Compiler) to get maximum sweetness out of the plugin



I have not used it much and I tried it out on a small repository but it seems promising. It's definitely worth trying out and at least to keep on your radar.




Happy TDD:ing!
