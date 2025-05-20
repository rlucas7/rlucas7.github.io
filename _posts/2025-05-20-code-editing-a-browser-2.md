---
title: 'Building a simple code editor in a web browser'
date: 2025-05-20
permalink: /posts/2025/05/code-editor-2/
tags:
  - coding
  - editor
  - javascript
  - code
  - codemirror-v6
---


Building a code editor for a browser or web client.
======

In a previous post I did some study of the event processing and other aspects
of codemirror v5. In that post I mentioned there is a new codemirror version,
v6. This post shows the v6 that I build and loads it so you can test it out
yourself here in the page.

# Code Edits in a web browser

I encourage you to try it out.

  <div id="editor"></div>
    <br>
    <input type="checkbox" id="oneDark" name="oneDark" onchange="changeTheme()">
    <label for="oneDark">Toggle <a href="https://github.com/codemirror/theme-one-dark">One Dark</a> Theme</label>
    <br>
    <!-- CodeMirror 6 -->
    <script src="./cm6.bundle.min.js"></script>
    <script>
        const oneDarkEl = document.getElementById("oneDark");
        const view = cm6.createEditorView(undefined, document.getElementById("editor"));
        const initialState = cm6.createEditorState("def foo():\n    print(\"hello world\")\n");
        view.setState(initialState);
        function changeTheme() {
            let options = {oneDark: oneDarkEl.checked};
            let newState = cm6.createEditorState(view.state.doc, options);
            view.setState(newState);
        }
    </script>

### Cite this post

```
@misc{LucasRoberts2025aicodecritique,
  author = {Lucas Roberts},
  title = {Code editing in a browser 2},
  year = 2025,
  howpublished = {\url{https://rlucas7.github.io/posts/2025/05/code-editor}},
  note = {Written: 2025-05-14}
}
```

