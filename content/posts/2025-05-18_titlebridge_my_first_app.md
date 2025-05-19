+++
title = "TitleBridge is my first app"
aliases = ["/posts/fcp-captions-my-first-app/"]

[taxonomies]
tags = ["YouTube", "Video", "App", "Swift"]
+++

[TitleBridge](@/apps/titlebridge/index.md) converts captions in Final Cut into editable titles. It runs inside Final Cut
and uses easy dragging and dropping. There's no longer any need to export captions, load them up into
another app, have the other app generate FCPXML with the titles, and then import the FCPXML
back into Final Cut.

<!-- more -->

{{ youtube(id="yUEACs1iDj8") }}

# Backstory

In the long, long ago I used to edit videos. It has been years since I last fired up [Final Cut Pro](https://www.apple.com/final-cut-pro/). I've been experimenting with [YouTube videos](/tags/youtube) and wanted to make a video about how at one point [I thought I had invented the modular monolith](@/posts/2025-05-06_how_i_thought_i_had_invented_the_modular_monolith_video.md). I do not really like being on camera much and my setup here is not very good. My microphone; however, which I [built from a kit](https://microphone-parts.com/collections/microphone-kits/products/s25-microphone-kit), sounds really good to my ears. I decided to just record audio.

Since I planned to distribute the video on YouTube, I needed some kind of visual. I'd seen lots of short-form videos with the captions burned in. That seemed like a good place to start. I had Final Cut automatically transcribe my audio to captions. It can export videos with the captions burned in; however, closed captions are very regulated so there is little flexibility on the look and positioning.

I started researching. The best way to handle this, it seemed, was to create precisely timed titles with the words from the captions. I already had the precisely timed captions in Final Cut. It would be a waste of time to manually create a title for each caption and then snap the start and end to the caption. This problem was ripe for automation and others surely had workflows to handle this.

The best I could find were either:

* Websites that convert captions files into titles and then export FCPXML for import into Final Cut
* Applications that mostly require monthly subscriptions

The subscriptions are usually because the apps are using AI to generate the captions. Final Cut now can automatically transcribe your audio to captions as long as you are running on Apple Silicon. I did not need third party transcription and I am pretty sick of how everything is becoming a subscription.

Regardless, all options I could find had a workflow that involves leaving Final Cut. Most need you to:

* Export the captions from Final Cut
* Import the captions
* Export FCPXML
* Import the FCPXML into Final Cut

I thought it would be so much simpler to never leave Final Cut and do everything with drag and drop. It would be extremely cool if you could just drop the titles directly onto the project timeline rather than having them show up somewhere in a new project in the media library. Since no one else did it, I figured I might as well do it.

Right now the application is being reviewed by Apple for inclusion in the App Store. I'll charge a little bit for it, but it will be a one-time fee. If I add new features, I'm planning on just releasing them to those who already purchased it. I wonder if there is a market for this little app...
