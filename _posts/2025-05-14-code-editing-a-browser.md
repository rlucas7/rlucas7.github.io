---
title: 'Building a simple code editor in a web browser'
date: 2025-05-14
permalink: /posts/2025/05/code-editor/
tags:
  - coding
  - editor
  - javascript
  - code
---


Building a code editor for a browser or web client.
======

If you've ever wondered how code VCS services like github or competitive coding sites like leetcode facilitate code editing directly in the browser this post is for you.

# Code Edits in a web browser

Basically, we need an editor that accepts text inputs and then renders those onto a screen.

Because we want to do this in a browser we're necessarily limited to javascript (js).
Or we could build something in typescript and transpile it to js but that is a bit more work for this proof of concept.

The proof of concept repo is here if you want to clone it locally and use it directly, nothing else is needed besides
these commands

```bash
git clone git@github.com:rlucas7/client-editor.git
cd client-editor
open index.html
```
That will open the demo in your default web browser on mac, if you're reading this and on a different OS
you can hopefully fill in the details for your own setup.

While I definitely did not setup all of the features that are possible I did a couple to figure out how things work.
This uses the codemirror v5 js package to power the browser editor.

The features of codemirror I looked at were:

1. Configuring the editor dynamically from the browser.
2. Template Shortcuts and intellisense style autocompletes.
3. Loading text to edit from a file on the local filesystem.
4. Downloading the editor text to a file and you decide the filename.

# Configuring from the browser dynamically

This is probably the most straightforward part of the setup.
You can see [here](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L15) how accomplish that via html input and a click event.
The [switchLineNumbers](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L93) function is really nothing more than a ternary operator but it does the job.

Select vs unselect will toggle the line numbers in the editor.
For me, I usually configure my text editors to have line numbers on by default but knowing how to easily turn them off is handy,
especially when you want to copy+paste something and you don't want the line numbers to be selected in the copy paste.

Many more options are configurable and they would involve more than a simple ternary function like done here but in principle
this is how you would accomplish the configuration, a custom function that toggles the editor config and an event to trigger the change.

You can see the checkbox and toggle the line numbers on and off here in the screenshot

<img src='{{site.baseurl}}/images/basic-functionality.png'/>

I encourage you to try it out.

# Template shortcuts and intellisense autocomplete

In codemirror these are called `hints` likely for proprietary reasons-maybe someone has trademarked or copywritten 'intellisense'?

Ok, so these get into the purview of addons, the ones that ship with codemirror directly are in the [addons folder](https://github.com/rlucas7/client-editor/tree/main/codemirror-5.65.19/addon).

Of course if those don't suit your fancy and you want more or unique customization you can roll your own or find a third party addon that suits your needs. Google is your friend here, eg. they wrote one for diff views that I peeked at a bit.
I didn't try to implement though because this was timeboxed to a single day project.

## Intellisense completions (hints)

I got these working first. There are a couple things you need to add to the page.
You need [a key or keys](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L62) to trigger the hints and you need to [link the addons](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L7), note that there are actually a couple files here in the header of the page.

<img src='{{site.baseurl}}/images/ctrl-q-intellisense.png'/>

I was focusing on js so I only included the hints for that language but there are hints for the other supported languages too.

## Template shortcuts

Ok so that's great, but what if there is some bit of code I want to auto-populate because I find I write it a lot?
Or maybe I notice this from analyzing my history files, I'd like to be able to do that here too.

The working solution I came up with is, ... an array, nothing complicated, and most importantly-it works.

The [snippets array](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L27)
contains the blocks of code that we want to be able to inject on demand to the editor.

<img src='{{site.baseurl}}/images/ctrl-e-custom-autocompletes.png'/>

What you'll notice when you look at the screenshot is that the code itself isn't shown in the completion window, the `displayText` field of the object is, this is good. You can have some somewhat long bit of templated code and have a readable shorthand to look up the template.

I experimented a bit with adding a second editor to the browser and allowing the user to define things there.
This was a bit more complicated to get the workflow how I wanted it, formatting and syntax highlighting didn't match
because I needed to use raw string literals and then there is the issue of converting from the string to an actually js array and passing it to the other editor. All somewhat complicated and what we have here works so it's ok for now.

One nice useability feature here would be a move to front rule or a splay tree style rotation that swaps the value with the one closer to the top. This makes sense if you had lots of templates for autocomplete and didn't want to scroll through all of them to find one especially if your autocomplete workflows are 'bursty'-using the same autocompletion several times in a row or nearly so.

I didn't implement the move to front heuristic though because there are only a couple templates so it wouldn't make much difference for the existing proof of concept setup.

# Load text from a file

Suppose this is the front end of a pastebin or github's gist?

You've got some file you're working with and don't want to retype things from scratch.
Instead you want to import the file and then edit is a bit to make it simple, simple to read, simple for the other person to understand, and simply illustrating whatever point your trying to communicate about achieving the thing.

This means we want to read in a file and inject the contents into the editor.

Another input tag and some file system codes will achieve this for us, the function is named [fileInput](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L99) and the input tag in the html is [here](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L19).

Ok so we've achieved file input. Check it out for yourself, it's handy.

# Downloading the editor contents into a file

This is probably the hackiest part of the demo code.
I give the user the option of passing a filename as input and provide a button to trigger the download of the file into the downloads folder. It takes the text from the editor verbatim.

The html button is [here](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L22) and the code to effect the [download is here](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L119).

If you were using this in a browser app you could imagine how making it easy for humans to download the contents of the shared file is good for making the workflow easier. The whole point is sharing some bits of code to help unblock your colleague.

# Special Bonus for the Blog-Track changes events

This isn't in the readme file on the repo but if you looked at the webpage javascript closely you might be wondering what
the [track changes code](https://github.com/rlucas7/client-editor/blob/bd3c674435704b2d0b32c9a0d630f6be56220d35/index.html#L70) is actually doing?

It doesn't track changes but only because we're not storing the changes anywhere.
After all, this is just the client side, although I guess you could format the changes into something serializeable and then write it to a file. Again, this is a timeboxed proof of concept, nothing super polished.

If you open the webpage in your browser and inspect the contents with the developer tools for the page you'll see the
edit events firing.

One thing you'll notice is the `origin` part of the js objects being fired, these are the additions `origin: '+input'`, deletions `origin: '+delete'` and paste events `origin: 'paste'`. I'm sure all the others have reasonable names too but you get the idea, and if not you can check out what I mean here in the screenshot.

<img src='{{site.baseurl}}/images/editor-edit-events.png'/>

If you've ever done an interview via coderpad and seen the recording that comes out of the application after this is one way you could implement something like that, store all the edits and replay them on a timeline, the timestamps are stored inside the events that are fired I'm just not making much light out of it here.

Coderpad does a bit more than this though, they provide simulataneous user edits and replay.
The scenario would be if you and your colleague are both editing the browser editor surface at the same time, perhaps in different timezones and then the 45 minute session is stored and replayable afterward.

I definitely didn't do that here but I'd guess with some work you could get something like that working using one of the
simulataneous edit approaches that exist-operational transforms seems to be one of the popular ones as are CRDTs so if you're interested those are things you can find papers to read up on.

# Closing thoughts and additional items

Codemirror is a super handy library and feature rich. There is an existing ecosystem of codes and addons.
I chose codemirror v5 rather than codemirror v6 because there were plentiful bits of code and discussion forum posts
that illustrated codemirror v5 in action.

However, the site indicates the v6 is the latest and greatest so it may be worthwhile to invest getting an example working
with v6 if you intend to build a project with codemirror.

Some example site functionalities:

1. pastebin or gist.github.com
2. leetcode or coderpad style web app where you write and submit text directly from the app
3. text editor or a fully fledged IDE, IIUC this is basically what vscode is doing via electron.js

Probably many other cool things too.

## Other things that would be fun to get working

If I had more time to devote to this proof of concept I'd likely implement:

1. Code folding-it's part of the addons that ship with codemirror
2. Multiple languages: select the syntax highlight from a dropdown of languages (needed for a gist.github.com clone)
3. Posting to a pastebin style site (or gist) directly from the page.
4. Using this with an LLM to populate text into the browser based editor
5. There are many more...

### Cite this post

```
@misc{LucasRoberts2025aicodecritique,
  author = {Lucas Roberts},
  title = {Code editing in a browser},
  year = 2025,
  howpublished = {\url{https://rlucas7.github.io/posts/2025/05/code-editor}},
  note = {Written: 2025-05-14}
}
```

<script src="https://giscus.app/client.js"
        data-repo="rlucas7/rlucas7.github.io"
        data-repo-id="MDEwOlJlcG9zaXRvcnkzODMyNTM2MzA="
        data-category="General"
        data-category-id="DIC_kwDOFtf8fs4Cqeqt"
        data-mapping="url"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
