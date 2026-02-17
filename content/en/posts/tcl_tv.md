+++
title = "My Smart TCL TV is now a dumb TV"
date = "2025-05-22T10:32:06+01:00"
draft = false
tags = ["tech", "tv", "android"]
translationKey = "tcl-tv"
slug = "my-smart-tcl-tv-is-now-a-dumb-tv"
+++

...or at least, it shuts up and leaves me alone.


## TCL and update hell

I recently got a TCL MiniLED 75-inch TV (c89b for France, c855 for the rest of Europe) for about 3 times less than an OLED of the same size — the value for money is truly excellent. But the Chinese aren't quite there yet when it comes to update policies. Since unboxing it in 2024, I've had only 1 single update: v412. If you browse the forums you'll see the mess — some countries got v420, some updates cause bugs on certain models but not others, and there are updates floating around the internet that never make it into the official pipeline.

While firmwares for 2025 models are coming out and are compatible with 2024 TVs, nothing is offered to update my TV. TCL's update policy is an absolute joke at this point. Zero follow-up, zero bug fixes once the TV is officially released. My v412 has a stuttering issue I would have loved to see fixed. So I caved and grabbed the v486 from 2025, upgraded via USB, then did a factory reset.

I'll skip past this story quickly because it's not the heart of the experience, but thankfully the community is there to show how to find the right firmwares for your model, the procedure to install them, and give feedback on bugs — because there's nothing from TCL. For 2024 models all the info is on [XDA](https://xdaforums.com/t/tcl-2024-google-tvs-p655-p755-c655-c655-pro-c765-t8b-c855-115x955.4666345/page-51).

## TV configuration
If Google TV succeeded at one thing, it's making the least user-friendly UI in the world. Recommendations everywhere for services you don't have, recommendations for shows you couldn't care less about, apps are relegated to a tab that requires multiple clicks, etc. Not to mention that Google's OS is rather chatty with the outside world. Without getting into network configurations like Pi-hole, let's see how we can make this TV pleasant to use daily.

First, do NOT enter your Google account during initial setup. This is very important for later. Enter the minimum, complete the basic TCL configuration to arrive at a Google TV screen with a few apps and that's it. Then enable developer mode and activate USB debugging to use adb.

Let's start by installing a new launcher to replace Google's: Pitivy Launcher.

```
adb connect <your ip>
adb install -r pitivylauncher.apk
```

Google doesn't like it when you replace their launcher and by default each press of the home button will send you back to the default launcher. Let's work around that:

```
adb shell pm disable-user -k --user 0 com.google.android.setupwraith
```

Now that we've silenced the Google setup and the launcher is there, you can go to the Google Play Store and add your Google account. Doing it this way, rather than completing the full Google setup, avoids installing all the useless Google apps: movies, games, YouTube Music, nor partner apps like Alexa.

From there you can install your apps via the Play Store, but we'll use adb again to remove some TCL apps:

```
adb shell pm uninstall -k --user 0 com.tcl.
```


## Bonus: nice wallpapers



### Useful links:
- [TCL survival guide](https://docs.google.com/document/d/1sAAPpFFwf1YozScXeZhaVYTczKc9XuOsNNBfcLM3gFg/edit?tab=t.0#heading=h.kgd99a7novk6) (in French)
- [XDA forums](https://xdaforums.com/t/tcl-2024-google-tvs-p655-p755-c655-c655-pro-c765-t8b-c855-115x955.4666345/page-51)
