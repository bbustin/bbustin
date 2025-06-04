+++
title = "TitleBridge 1.1 has a major new feature"

[taxonomies]
tags = ["YouTube", "Video", "App", "Swift"]
+++

[TitleBridge](@/apps/titlebridge/index.md) now has a major new feature. You can now select the style of title to generate. This leaves room for a lot more creativity.

<!-- more -->

There did not appear to be a way for title styles to be dragged from Final Cut onto the extension. That would have been my first choice. Instead, I added a dropdown menu to allow the title style selection.

Here is a sampling of some of the title styles:
{{ youtube(id="xbYwk79WYGM") }}

Not all titles that ship with Final Cut work well for captions. I first had to eliminate all title styles which had more than one text field. Then I had to go through all of them to find which were working and which were not. I removed the ones which did not work well. I have a feeling a few non-optimal ones snuck through.

It appears many title styles either shrink text to fit or automatically line wrap. I changed the default selected title style to one which shrinks the text. I did leave title styles which allow text to flow off the screen. Those can be manually edited to add line breaks. I may, in the future, add a capability to automatically create line breaks.
