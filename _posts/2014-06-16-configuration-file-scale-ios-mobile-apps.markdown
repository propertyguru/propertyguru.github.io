---
layout: post
title:  "Configuration file to scale iOS mobile apps"
date:   2014-06-16 02:08:34
author: de_poon
---

In our team of mobile rockstar engineers, we often ask ourselves this question: **How can we scale our ios mobile apps?**

Lets take a hypothetical situation. Your CEO is very pleased with the first delivery of the company’s mobile app. The other country managers come along and want you to build the same app but adjusted for their individual countries. Assuming they want a separate downloadable app, the general requirements are

1. I want some features taken from the main app
2. I do not want some features from the main app
3. I want some features to be modified from my custom app.

Sounds familiar?

The way to go about is to create a non-binary configuration file that allows us to specifcy what we want in the app.

<img src="http://media.tumblr.com/13eecb429e8915f4b7e0325b5013a565/tumblr_inline_n49k8h9ryQ1sdmkub.png" class="img-responsive" />

The most important principle is that no recompilation is needed when we scale our apps. In our team we’ve chosen to go with PList file as it is easy for developers to create one out of the box in XCode. Here’s the steps

1. Create a PList from XCode  
<img src="http://media.tumblr.com/db0c2e801616572c5e8c74bea96191e8/tumblr_inline_n49kfokByI1sdmkub.png" class="img-responsive" />

2. Set up your features in a .plist  
<img src="http://media.tumblr.com/70a1deba225168814a4e44b861142953/tumblr_inline_n49krhLHTO1sdmkub.png" class="img-responsive" />

3. Add the plist file into your main bundle.

4. Use the following codes to read the contents of the plist as a NSDictionary  
<img src="http://media.tumblr.com/0d3ff46eb8758f3d3107e216462fe91d/tumblr_inline_n49l36LRak1sdmkub.png" class="img-responsive" />

5. In your app features, read the contents off the NSDictionary to determine how your feature should behave. Eg.
- Should this feature be turned on?
- How should I order by menu items on the side menu?
- Should I enable specific UI Animations at a particular screen?


To create additional scalable apps, simply create a new project
- import your lib.a binaries
- import your resource bundle files
- add in your configuration plist
… package and deploy your app!!!

**Considerations**

1. Adding your plist directly into your main app bundle is not a secure way to do (So you think you can start working on it? Gotcha…). It is possible for someone to extract the plist from your app and this exposes all sensitive information such as api end points, security tokens, etc. Dont worry, an upcoming post will address this issue.

2. You might want to create your own configurations file manager to generate your plist on the fly.

3. PList is probably not the best way to encapsulate application configuration. You may want to explore other formats.

Feel free to buzz us your thoughts on this