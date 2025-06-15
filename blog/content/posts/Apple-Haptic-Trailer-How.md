---
title: "Feel What You Hear: The Magic Behind Apple's Haptic Trailer & Recreating It"
date: 2025-06-07T16:26:13+05:30
summary: Building a DIY smart home in India is not as easy as getting a bunch of smart plugs & bulbs. This is my journey of converting my "dumb" home to a "smart" home in India using DIY methods using custom built hardware, software & opensource stack.
tags: ['video engineering', 'innovation', 'apple']
categories:
- software
- video engineering
- architecture
- innovation
---

Apple recently released a new promotional trailer of their upcoming F1 movie, they're calling it the "Haptic Trailer". As the name suggest, the trailer incorporates Haptic feedback based on the scene and sounds. It is a novel experiment on user engagement when it comes to video consumption. Haptic feedback isn't a new concept in the world of entertainment, game console contollers like Sony DualShock and XBox Controllers have been rocking these since the last few decades. 

![Cover Image](/assets/images/haptic-video/haptic-video-cover.png)

There have been such experiements related to video in the past but Apple's newest implementation is surely going to be a taken a storm, even if the number of devices where this is supported, Apple simply outshines the competition with their "ecosystem" and if anyone can make it popular, its going to be Apple!

## What Is It?

If you have an iOS device, you can watch it [here](https://tv.apple.com/us/clip/haptic-trailer/umc.cmc.5834l80x7lpxvb1wqiz7uvgj7?targetId=umc.cmc.3t6dvnnr87zwd4wmvpdx5came&targetType=Movie). This particular trailer uses Apple's Taptic Engine(their fancy name for the vibration motor, but its not a motor with counter weights!) to add a new dimension to content consumption, it definitely feels super cool and as an F1 fan, I would say it adds on to the overall experience.

## How Does It Work? Apple Magic!

Like every engineer, I immediately started poking around the network calls by loading the trailer in a Safari window and since I work with video manifest files for a living, the m3u8 master manifest really looked interesting!

In addion to the usual ABR ladder details, audio tracks and subtitle info, there is an additional`EXT-X-SESSION-DATA`with`DATA-ID`set to`com.apple.hls.haptics.url` and`VALUE` set to an "ahap" file url:

![Apple Manifest With AHAP File](/assets/images/haptic-video/manifest-extract.png)

The tag `EXT-X-SESSION-DATA` allows arbitrary session data to be carried in a Master Playlist, think of it like additional data readily there in the manifest. You can read the RFC doc [here](https://datatracker.ietf.org/doc/html/draft-pantos-hls-rfc8216bis-01#section-4.4.4.4)

### Apple Magic!

Since iOS 13.0+ and iPadOS 13.0+, Apple has included *Core Haptics* capability which lets the developer add custom haptic feedbacks to their apps to have a bit more physical engagement with on screen interactions. You might be already familiar with some of the input controls having haptic feedback when you interact with them - the alarm time picker for example. You can read Apple's documentation on Core Haptics [here](https://developer.apple.com/documentation/corehaptics).

The Taptic Engine built into iPhones and iPads (and other wearables) are in a way high definition, you can differentiate between a mild tap, a sharp tap and even a continous rumble, I am yet to find a mobile device with this level of resolution and definition when it comes to haptic feedback.

![Apple Taptic Engine](/assets/images/haptic-video/taptic-engine.png)

What makes Core Haptics interesting is the ability to play a continous sequence of patterns - once again thanks to Apple's Hardware + Software integration harmony! 

From an implementation perspective, every haptic feedback event is an object of`CHHapticEvent`from the Core Haptics library, which essentially is a simple way to represent one haptic event, there are audio and haptic feedbacks lets ignore audio for now.

When it comes to haptics, there are 2 main event types based on the duration of the effect - **transient** & **continous**.

Transient events are impulses while continous ones represent vibrations or rumble.

The image below shows how these events are represented against time:

![CHHapticEvent Types](/assets/images/haptic-video/haptic-event-types.png)


On a high level, each haptic event can be represented with the following attributes:

*relativeTime* - The start time of the event, relative to other events in the same pattern.  
*duration* - The duration of the haptic event.  
*hapticIntensity* - The strength of a haptic event.  
*HapticSharpness* - The feel of a haptic event.  

![CHHapticEvent Intensity](/assets/images/haptic-video/haptic-intensity.png)

Both intensity and sharpness are values between 0.0 and 1.0

![CHHapticEvent Sharpness](/assets/images/haptic-video/haptic-sharpness.png)

Also there are a few other parameters associated with Haptics, however for the sake of brevity we can ignore them for now. In case you're interested, you can read more [here](https://developer.apple.com/documentation/corehaptics/chhapticevent/parameterid#topics)

Now the engineering masterpiece - Core Haptics allows the developer to create a list of haptic events and play them back like an audio/video file!

Apple even has a specification for these files - Apple Haptic Audio Pattern (AHAP) files, its a simple text file with an array of `CHHapticEvent` objects in a JSON format(Apple says JSON like). You can read Apple's specification [here](https://developer.apple.com/documentation/corehaptics/representing-haptic-patterns-in-ahap-files)


![AHAP File Example](/assets/images/haptic-video/example-ahap-file.png)

## Generating Haptics (AHAP File Generation)

An audio clip is essentailly a continous analog signal, the differences in frequency & amplitude of the signal is what we perceive as sound, speakers are essentailly motors that convert electrical energy to sound waves through electromagnetic effect. The vibrations of a speaker cone is what we hear as sound.

If you connect a powerful enough amplifier to an electric motor, you can actually hear the music play because a motor is essentially a coil sitting in a magnetic field just like a speaker.

Keeping this in mind, on a high level - just feeding the audio signal to some kind of function that generates the corresponding haptic sequence is the key to haptic videos, this can can be done with the following steps:

1. Extract out the audio from the video
2. Analyse the audio and extract audio features
3. Convert these audio features and corresponding timestamps to AHAP format

![Haptic File Generation](/assets/images/haptic-video/haptic-generation-workflow.png)

Extracting audio from video is as trival as running an ffmpeg command, but analysing the audio & feature extraction the challenging part.

The following video helps visualise how an audio clip can be represented visually, if we just convert the spikes to transient haptic events and the constant audio part to a continous haptic event, it would still become a crude haptic sequence that will match the audio clip.


{{< video src="/assets/videos/haptic-video/audio-spectrum.mp4" type="video/mp4" loop="false">}}

### Audio Feature Extraction 
We cannot convert all audio features to haptics, that would mean every cough, every scream and every cry would be converted to a haptic sequence, which would definitely won't feel natural nor pleasant. With the (limited)research along with some trial & error I've done so far, the audio feature extraction should look for the following for a natural & smooth experience:

1. Tempo
2. Beats
3. Onsets
4. Mood - Bright/dark (or technically Spectral Centroid)

Once this is extraction part is done, separate out the components of the audio clip:

1. Harmonics (For Sustained noises)
2. Percussive Sounds (For Drums & transients)
3. Bass
4. Treble

Once we have these split out and available, its just a matter of converting these to corresponding continous haptic events:

Generate `HapticTransient` events from onsets & beats  
Generate `HapticContinuous` events from the sustained energy + harmonics

And then adjust the haptic feel(ParameterCurve) for intensity and sharpness based on the amplitude and other signal parameters.

{{< video-js src="https://demo.unified-streaming.com/k8s/features/stable/video/tears-of-steel/tears-of-steel.ism/.m3u8" type="application/x-mpegURL" >}}

