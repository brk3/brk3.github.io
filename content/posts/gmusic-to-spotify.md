+++
categories = ["howto"]
date = "2017-05-26T10:56:50+01:00"
title = "Importing albums from Google Music to Spotify"

+++

I recently switched from Google Music to Spotify, and wanted to import all the various albums I've
added to my library over the years. As of writing all current methods I found didn't work,
thankfully Spotify have a really nice API which makes this quite straight forward.  You can find the
script I put together at https://github.com/brk3/gmusic-to-spotify, here's a quick overview how to
use it.

You need to acquire api keys (https://developer.spotify.com/my-applications/#!/applications), and
plug these in. Next you need to generate a simple csv key/value list of your albums in the form of
'artist, album'. There is a very useful piece of javascript for this being maintained over at
https://gist.github.com/jmiserez/c9a9a0f41e867e5ebb75.

After that you can let it run, if it finds multiple sources for an album (e.g. explicit vs. edited
versions), it will list links to the albums that you can inspect before choosing which one to add.
For large libraries this happens quite a bit, in my experience the first choice in the list is
usually best.

Hope it helps!
