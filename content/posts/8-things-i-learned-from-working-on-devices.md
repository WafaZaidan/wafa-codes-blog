---
title: "8 Things I Learned From Working on Devices"
date: 2021-10-18T13:19:36+01:00
draft: false
toc: false
images:
tags:
  - posts
---

Over the past 3 years, I have been working on Peacock, the video streaming app by Comcast. My team builds and maintains a library in Typescript that powers video playback on tens of devices. And here are the lessons I learned

### 1. No such thing as a one-size-fits-all 👗

In the streaming world, there are so many cool features to build and while I’m still fascinated by the fact that you can build an MSE (Media Source Extension) video player in a few lines of code, there are many cool things like ad insertion, linear scrubbing, prefetching and more.

Building these features however is not an easy job, not just because they can be complicated and require a lot of domain knowledge but there’s also the big challenge of making them work on tens of devices which have different APIs and a varied performance level.

The truth is when you build on devices, it’s never a one-size-fits-all situation. For instance if you want to make ads work on an old LG TV, the code is likely to look different to the one built for the Comcast-owned Xfinity devices. It might be even different for the same device brand depending on the model. And we have tens of devices from different brands and models to support so you might soon end up with spaghetti code where if statements are everywhere!

{{< image src="/images/spaghetti-code.png" position="center" style="border-radius: 8px; width: 300px" >}}

That’s why it's important to have a well architected code base. One that is clean, applies the SOLID principles and is flexible enough to accommodate the addition of new features that have to potentially be built differently for each device.

### 2. Get your hands dirty and build custom tools ⚒️

Sadly, in the video playback world, we mostly work on old/low end devices which are usually very slow. People usually buy TVs and consoles for years. Its not something you change every year which means most of our customers have old devices and so we need to support and work on those most of the times.

In our daily work, we rely on the debugging tools provided by each device. They allow us to continuously see logs from our app and show us the network calls which help us get constant feedback from the app and build off that. However, most low end devices don’t work well with the debugger on for a long period of time. For example, the Chromecast ultra (which is considered a high end device) would typically crash after 4 minutes of having the debugger on which slows down our work.

That’s where you might want to get your hands dirty and build your own custom tools.

One of the cool tools we built in my previous team was a stream proxy that directs all the video-related network calls (manifest and fragments) from the device to your machine/laptop which would be running Charles so that you can track and debug the network calls without having to use the device’s debugger.

I also built a tool called the Log Server, which allows the device to periodically send logs from the device to a local server running on your laptop with all the logs from your app that you would normally need the device’s debugger to see. This gave us an eye into what's going on on the device side without having to use the device’s debugger and causing the device to crash or freeze.

{{< image src="/images/devices.jpg" position="center" style="border-radius: 8px; width: 300px" >}}

### 3. Say goodbye to Stack overflow 🤯

Building a video library is a very niche job especially when it's combined with working on devices. Challenges are unique and require creativity and most of the time you can’t find the answer you want on Google or Stack overflow. This is changing by the day and more resources can be found online now but it's still poor compared to other areas like building UI or kubernetes for example.

And although this might sound like a nightmare, it’s actually a fun challenge as you get to be creative and come up with unique solutions yourself, or with the help of your colleagues but with only little help from the internet. How awesome is that!

### 4. Clean as you go, document as you go 🧽📋

This point applies when building any application but since working on devices makes development much slower and harder, the need for clean code and good documentation of the device setup and code is very important as it saves time and helps developers produce features and fix bugs faster.

### 5. Build on high end, test on low end 📺

Every time I'd build a new feature, I'd choose a high end device. Once the feature is in a working condition, I would turn on my low end devices and test it there.

This was a rule of thumb for me. You see, working on devices is slow as mentioned before. You need to start the device, run your app locally, start the device’s development platform, launch your app onto the device and then start coding while waiting for feedback from the device. So, to get the ball rolling I'd choose a fast and high end device to get something working first and then I'd test it on low end devices.

For bugs too, I would try to reproduce them on a high end device at first, maybe even on a browser. If it's reproducible there, I can fix it faster. If not, I have to try and reproduce it on the low end device that it was reported on.
It's all about saving time and trying to see if we can build/fix things on a faster device and avoiding working on a painfully slow low end device.

### 6. Check it out locally when needed 🕵️

As the team grows, the number of pull requests increases which means there is a higher chance for bugs to slip in especially when working on devices. It takes one device to be neglected for a new serious bug to be introduced in production.

I have found it useful when reviewing a PR that I suspect might introduce a bug to check out the branch locally and test the code on one or more devices.

This could take between 15-30 minutes but it could save us a bug in production, hours of investigation later on and many QA hours needed to notice that bug. Remember, in the streaming world it's hard to catch bugs automatically, and you need actual human eyes in many cases to catch them.

Of course I wouldn’t check out every PR locally, but only the ones that I sense could break something.

{{< image src="/images/code-bug.jpeg" position="center" style="border-radius: 8px; width: 300px" >}}

### 7. You’re giving up a freedom, negotiate a higher rate 💸

I’ve always imagined that it would be nice to book a little cute cottage in the Cotswolds for a week and work from there because as software developers, all we need is a laptop and a good internet connection to get the job done. Well, that is not an option when you’re working on devices as you will always be tied to where your devices are. You can’t even work from the café down the road!

In the age where working from home has become more the norm, working on devices takes away the precious freedom to work from anywhere you like so make sure to negotiate a higher salary for giving that up.

Many people avoid working on devices because it's slow and tedious but above all it's the freedom to work from anywhere that is the biggest sacrifice, so make sure to get fairly compensated for that.

{{< image src="/images/wfh.png" position="center" style="border-radius: 8px; width: 300px" >}}

### 8. Do it if you love it, be patient and have fun with your team 🐹

I found working on devices fun and rewarding despite all the challenges it entails. I learned a lot and that came with a lot of patience. But what made the experience more enjoyable is working with a fun and kind team that shares the passion for building cool things on devices and that in my opinion was the key to the success of our work.

{{< image src="/images/hamster_dance.gif" position="center" style="border-radius: 8px; width: 70px" >}}

I definitely see myself going back to working on devices again one day. It can be addictive, I suppose.
