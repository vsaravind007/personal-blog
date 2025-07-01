---
title: "Feel What You Hear: The Magic Behind Apple's Haptic Trailer & Recreating It"
date: 2025-06-16T00:26:13+05:30
summary: Exploration of the technology and engineering behind Apple's new Haptic Trailer
tags: ['video engineering', 'innovation', 'apple', 'swift']
categories:
- software
- video engineering
- architecture
- innovation
- swift
---

Apple recently released a new promotional trailer of their upcoming F1 movie, they're calling it the "Haptic Trailer". As the name suggests, the trailer incorporates Haptic feedback based on the scene and sounds. It is a novel experiment on user engagement when it comes to video consumption. Haptic feedback isn't a new concept in the world of entertainment, gaming console controllers like Sony DualShock and XBox Controllers have been rocking these since the last few decades. 

![Cover Image](/assets/images/haptic-video/haptic-video-cover.jpg)

There have been such experiments related to video in the past but Apple's newest implementation is surely going to be a taken a storm, Apple simply outshines the competition with their "ecosystem" and if anyone can make this concept popular, its going to be Apple!

## What Is It?

If you have an iOS device, you can watch it [here](https://tv.apple.com/us/clip/haptic-trailer/umc.cmc.5834l80x7lpxvb1wqiz7uvgj7?targetId=umc.cmc.3t6dvnnr87zwd4wmvpdx5came&targetType=Movie). This particular trailer uses Apple's Taptic Engine(their fancy name for the vibration motor, but its not a motor with counter weights!) to add a new dimension to content consumption, it definitely feels super cool and as an F1 fan, I would say it adds on to the overall experience.

## How Does It Work?

Like every sensible engineer, I wanted to see how this was pulled off, I immediately started poking around the network calls by loading the trailer in a Safari window. Since I work with video manifest files for a living, I could easily recognize differences without much effort. The m3u8 master manifest for this trailer looked really interesting!

In addion to the usual ABR ladder details, audio tracks and subtitle info, there is an additional`EXT-X-SESSION-DATA`with`DATA-ID`set to`com.apple.hls.haptics.url` and`VALUE` set to an "ahap" file url, this is the what makes it tick!

![Apple Manifest With AHAP File](/assets/images/haptic-video/manifest-extract.png)

The tag `EXT-X-SESSION-DATA` allows arbitrary session data to be carried in a Master Playlist, think of it like additional data readily there in the manifest. You can read the RFC doc [here](https://datatracker.ietf.org/doc/html/draft-pantos-hls-rfc8216bis-01#section-4.4.4.4) if you fancy!

### Apple Magic!

Since iOS 13.0+ and iPadOS 13.0+, Apple has included *Core Haptics* capability which lets the developer add custom haptic feedback to their apps to have a bit more physical engagement with on screen interactions. You might be already familiar with some of the input controls having haptic feedback when you interact with them - the alarm time picker for example. You can read Apple's documentation on Core Haptics [here](https://developer.apple.com/documentation/corehaptics).

The Taptic Engine built into iPhones and iPads (and other wearables) are in a way high definition, you can differentiate between a mild tap, a sharp tap and even a continuous rumble, I am yet to find a mobile device with this level of resolution and definition when it comes to haptic feedback.

![Apple Taptic Engine](/assets/images/haptic-video/taptic-engine.png)

What makes Core Haptics interesting is the ability to play a continuous sequence of patterns - once again thanks to Apple's Hardware + Software integration harmony! 

From an implementation perspective, every haptic feedback event is an object of`CHHapticEvent`from the Core Haptics library, which essentially is a simple way to represent one haptic event, there are audio and haptic feedback, lets ignore audio for now.

When it comes to haptics, there are 2 main event types based on the duration of the effect - **transient** & **continuous**.

Transient events are impulses while continuous ones represent vibrations or rumble.

The image below shows how these events are represented against time:

![CHHapticEvent Types](/assets/images/haptic-video/haptic-event-types.png)


On a high level, each haptic event can be represented with the following attributes:

*relativeTime* - The start time of the event, relative to other events in the same pattern.  
*duration* - The duration of the haptic event.  
*hapticIntensity* - The strength of the haptic event.  
*HapticSharpness* - The feel of the haptic event.  

![CHHapticEvent Intensity](/assets/images/haptic-video/haptic-intensity.png)

Both intensity and sharpness are values between 0.0 and 1.0

![CHHapticEvent Sharpness](/assets/images/haptic-video/haptic-sharpness.png)

(All 3 images were shamelessly stolen from Apple's documentation!😶‍🌫️)

Also there are a few other parameters associated with Haptics, however for the sake of brevity we can ignore them for now. In case you're interested, you can read more [here](https://developer.apple.com/documentation/corehaptics/chhapticevent/parameterid#topics)

Now the engineering masterpiece - Core Haptics allows the developer to create a list of haptic events and play them back like an audio/video file!

Apple has a specification for these files - Apple Haptic Audio Pattern (AHAP) files, its a simple text file with an array of `CHHapticEvent` objects in a JSON format(Apple says JSON like). You can read Apple's specification [here](https://developer.apple.com/documentation/corehaptics/representing-haptic-patterns-in-ahap-files). Below is an example AHAP file with 2 haptic events:

![AHAP File Example](/assets/images/haptic-video/example-ahap-file.png)

## Generating Haptics (AHAP File Generation)

An audio clip is essentially a continuous analog signal, the differences in frequency & amplitude of the signal is what we perceive as sound, speakers are essentially motors that convert electrical energy to sound waves through electromagnetic effect. The vibrations from a speaker cone is what we hear as sound.

If you connect a powerful enough amplifier to an electric motor, you can actually hear the music play because a motor is essentially a coil sitting in a magnetic field just like a speaker.

Keeping this in mind, on a high level - just feeding the audio signal to some kind of function that generates the corresponding haptic sequence is the key to haptic videos, this can can be done with the following steps:

1. Extract out the audio from the video
2. Analyse the audio and extract audio features
3. Convert these audio features and corresponding timestamps to AHAP format

![Haptic File Generation](/assets/images/haptic-video/haptic-generation-workflow.png)

Extracting audio from video is as trivial as running a simple one liner ffmpeg command, but analysing the audio & feature extraction the challenging part. Creating the haptic sequence comes next but its relatively easy because you can play with numbers.

The following video helps visualise how an audio clip can be represented visually, if we just convert the spikes to transient haptic events and the constant audio part to a continuous haptic event, it would still become a crude haptic sequence that will match the audio clip.


{{< youtube lfpqh5Q-I8g >}}

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

Once we have these split out and available, its just a matter of converting these to corresponding continuous haptic events:

Generate `HapticTransient` events from onsets & beats  
Generate `HapticContinuous` events from the sustained energy + harmonics

And then adjust the haptic feel(ParameterCurve) for intensity and sharpness based on the amplitude and other signal parameters.

## Putting It All Together

I have used the following video to do my analysis and Haptics generation, this video is also publicly available as an HLS stream, which allows for a quick POC using AVPlayer in a test app.

You can download the full AHAP file from [here](/assets/others/haptic-video/tears-of-steel.ahap)

{{< youtube R6MlUcmOul8 >}}

### Making It Play!


There are a few ways to do this, what Apple has done with their original implementation is to include the AHAP file in the mainfest file itself as a `EXT-X-SESSION-DATA` tag. However, what I have done is to load the AHAP file simliar to a sidecar subtitle and timesync the video & haptic playback timelines.

**Relevant Code Blocks**

The following code blocks will give an idea about how this can be implemented, skipping boilerplate, player controls and other essentials for brevity.

Setup of video player and the haptic player:

```
    // Setup
    func setup(videoURL: URL) throws {
        // Setup video player
        videoPlayer = AVPlayer(url: videoURL)
        
        // Setup haptic engine
        hapticEngine = try CHHapticEngine()
        try hapticEngine.start()
        
        // Load AHAP from URL
        let ahapURL = URL(string: "https://blog.aravindvs.com/assets/others/haptic-video/tears-of-steel.ahap")!
        let hapticPattern = try CHHapticPattern(contentsOf: ahapURL)
        hapticPlayer = try hapticEngine.makeAdvancedPlayer(with: hapticPattern)
    }
```

Start playback

```
    // Synchronized Playback, we want our haptics & video to be in sync
    func play() throws {
        // Setup synchronization
        let interval = CMTime(seconds: 0.1, preferredTimescale: 1000)
        //Setup a timeObserver so that we can keep track of the time to keep both players in sync
        timeObserver = videoPlayer.addPeriodicTimeObserver(forInterval: interval, queue: .main) { [weak self] time in
            self?.synchronize(videoTime: time)
        }
        
        // Start both players
        videoPlayer.seek(to: .zero)
        hapticPlayer.seek(toOffset: 0.0)
        
        videoPlayer.play()
        try hapticPlayer.start(atTime: CHHapticTimeImmediate)
    }
```

Sync

```
    // Do Synchronization
    func synchronize(videoTime: CMTime) {
        let videoSeconds = CMTimeGetSeconds(videoTime)
        let hapticTime = hapticPlayer.currentTime
        let timeDiff = abs(videoSeconds - hapticTime)
        
        // Re-sync if drift > 100ms
        if timeDiff > 0.1 {
            try? hapticPlayer.seek(toOffset: videoSeconds)
        }
        
        // Pause haptics if video isn't playing
        if videoPlayer.rate == 0 {
            try? hapticPlayer.pause(atTime: CHHapticTimeImmediate)
        } else {
            try? hapticPlayer.resume(atTime: CHHapticTimeImmediate)
        }
    }
```

## Closing Thoughts

Even though the technology is cool, it has a few issues related to user experience. Main one being battery drain, my phone battery went down by a percent or two while I was watching the F1 trailer, this is expected however.

Its Apple devices only (for now)!

Its only suitable for short videos like trailers and teasers, it won't make much sense for longer videos. 

Generation of AHAP files is another challenge, I'm currently working on a tool that will generate AHAP files from audio files, I will be updating this blog with the tool once ready and available for public use.
