---
title: How I Built an Escape Room Magic Mirror
layout: post
date: '2020-07-15 22:30:00'
categories: null
tags: escape-room
---

![Title Image](/assets/images/building-an-escape-room-magic-mirror/header.jpg)

This post is part of a series on an escape room I designed for my wife.  In fact, the whole idea for the escape room started while I was making a Smart Mirror for her.  It was just meant to be a simple project; a small mirror that shows a new poem every day.  Yet like many ideas in my life the spark lit a fire and I found myself flung into three weeks of feverishly building an escape room...

## The Idea

As part of the escape room the Smart Mirror became a Magic Mirror and served as the main character's, Mabel's, way of speaking to the player.  Early on in the game when a mysterious sheet of paper is held up it triggers a video of Mabel talking to the player through the mirror.  Later, once the player has beaten the escape room, the mirror is triggered again to show Mabel congratulating them.  Overall I was satisfied with how the mirror worked in the game, and I think it was delightfully surprising to the player when the mirror suddenly sprung to life and played a message.  So here's how everything was built.

## Hardware Build

The hardware build wasn't terribly unique in the Smart Mirror world.  Like all Smart Mirrors it involves a digital display behind a one-way mirror.  They look and work like regular mirrors, but can also superimpose text and simple graphics on the reflected image.  But besides text and simple graphics, they can also show full images and video.  People don't tend to use them that way since it's harder to see your reflection when a full-screen image is showing, and the quality of the image through the mirror is not very good.  For a talking magical mirror, however, the washed out haziness of full-screen images works to its advantage.

![Tablet](/assets/images/building-an-escape-room-magic-mirror/tablet-installation.jpg)

Most Smart Mirrors involve a Raspberry Pi and either a repurposed monitor or TV for the screen.  However I was only building a small mirror (8"x10"), so a monitor or TV was out.  RasPi is nice, but would be a "wart" on the back of picture frame.  Finally, I wanted to keep cost and complexity down, since I would be spending money and time elsewhere on the escape room.  To that end I decided to go with a cheap tablet; namely a FireHD 8.  I was able to get an older model off Amazon Warehouse for $50, far cheaper than the all-in cost of a RasPi+accessories+screen.  Plus it gives me a camera, microphone, and a thin form factor that could fit into a picture frame.  Most Warehouse deals are brand-new items that have just been opened and returned.  Amazon cannot sell them as new, so they flip them on Warehouse.  More importantly though, it let me get an older model FireHD which could be easily rooted.

I rooted the tablet in the usual way and put LineageOS on it.  LineageOS makes it more responsive, uses less power, and doesn't come with any of the default cruft which would detract from a seamless Magic Mirror experience.  But it's not without its faults.  Only older releases of LineageOS were well supported, so I was stuck on LOS 14.1 (Android Nougat).  And being non-standard, some things were a bit buggy (this will come back to haunt me later in the story...).

For the frame, I used an [8x10 picture frame](https://amzn.to/3ewrUOg).  8x10 is larger than the tablet, but I couldn't find a better fitting smaller frame given the tablet's awkward dimensions.  A hand-built frame would be perfect, but was decided against since I was going to spend time building and coding a _lot_ of other things for the escape room.  Simple, ready-made was king.  In the end, for the mirror's main purpose of displaying poems in the corner, the mismatched size is fine.

The glass of the picture frame was replaced with a sheet of [one-way acrylic mirror](https://amzn.to/3946Y03).  Compared to glass, acrylic mirrors are cheaper, easier to cut to size, and don't run the risk of shattering into flesh seeking knives.  They just suffer from being very easy to scratch and slightly lower optical quality.  Cost being king, I went with acrylic.  The sheet I got was 12x12.  To cut it to size, most people use those special acrylic tools to score the sheet several times and then "snap" the sheet.  Luckily, I have a table saw!  I used a medium-toothed blade (fine would be better; I didn't have one on hand), and fed it through on my sled at a slow-medium feed rate.  Worked great.

![Frame's extended back](/assets/images/building-an-escape-room-magic-mirror/back-of-frame.jpg)

All-in, everything only cost $100.  To assemble it, I couldn't just throw the tablet behind the mirror and call it a day; tablets tend to be a bit thicker than a photos...  So I whipped up some parts in Fusion 360 and 3D printed them to extend the back of the picture frame enough for the tablet to fit.  I also added some 3D printed filler to take up the extra space and secure the tablet in place.  It includes a channel to route the USB power through.  Again, a hand-built picture frame would have been nicer and neater, but this was good enough.

## Poetry App

![Poetry in Mirror](/assets/images/building-an-escape-room-magic-mirror/poem-mirror.jpg)

With the magic mirror built, it was time to build the software for it.  The daily poem app is straight-forward; just a simple Android app with a text element and some timer logic in the background for updating it at night.  I used the lovely [PoetryDB](https://poetrydb.org) as the source of the poems.  I only had to do a little filtering to skip poems that would be too long for the screen.  And of course the usual Android stuff for keeping the screen always on.

## Escape Room App

For the escape room, I built a second app that would run instead.  It had to handle two things: detect when the player holds a specific note up to the mirror and play the first video; and waiting for a trigger signal from the Magic Pedestal to play the final video.  The Magic Pedestal will be covered in another post, along with the design for the overall escape room.  For now it's enough to say that when the player has collected three magic books and placed them on the Magic Pedestal, the pedestal sends a signal to the Magic Mirror.

<div class="post-gif"><video preload="auto" autoplay="autoplay" muted="muted" loop="loop" webkit-playsinline=""><source src="/assets/images/building-an-escape-room-magic-mirror/demo.mp4" type="video/mp4"></video></div>

The note that "unlocks" the first video is a QR code with the words "Take a moment to reflect" written backwards (so it'd look correct when read in a mirror).  Originally I had planned for the QR code to be hidden within a complex design so it wasn't obvious and would seem more magical.  But after testing I noticed that the escape room's lighting (candle light) made it difficult enough as it was to recognize the QR code.

I used Google's ML Kit to handle QR decoding, as it seemed like the easiest to get working.  As usual, Google's documentation is atrocious and outdated, but with a bit of effort and piecing together four or five different example codebases that all had some kind of deprecated APIs in them, I was able to get it to work.  Once the correct QR code is detected and decoded, it plays the relevant video in a VideoView.

The other function of the app, triggering a video when the Magic Pedestal says so, was supposed to be the easiest part.  I figured I could just listen on a TCP port in the app and wait for the Magic Pedestal to connect and send a specific packet.  Alas, this is where LineageOS came back to haunt me.  No matter how much I tried it simply _refused_ to accept connections.  There's no firewall that I could find, no privacy setting, no set of permissions.  Nothing.  Not even root and SSHd could accept connections.  So I decided on a more hacky approach that actually killed two birds with one stone.  Plugging the tablet's USB into the Magic Pedestal's Raspberry Pi.  That handles both powering the tablet during the escape room, and gives the pedestal access to adb.  With adb I could have the pedestal simulate a keystroke on the tablet, which the escape room app would recognize as the signal to trigger the final video.  Is that convoluted and stupid?  You bet!  But it works.

That's about it.  In Mabel's final, congratulatory video message, she tells the player that they can keep the magic mirror as a gift, and that she would cast a spell on it so it would show a poem for them every day.  That was my way of revealing my wife's gift.  But it also served a useful purpose, because after the final video played the escape room app "magically" launched the poetry app.  That way, after the escape room was done, the tablet was already setup and ready to be used for its original purpose.

## Conclusion

Overall I'm pleased with how the Magic Mirror turned out.  Using a tablet made a lot of things easier, and opened up the possibility of using its camera as part of the escape room puzzles.  It definitely wasn't without its caveats though.  RasPis are far easier to setup and less buggy compared to a rooted, bottom barrel tablet.  Hard to beat that price though.

I hope this post has been fun, and maybe even inspires other escape room artists to build something neat.