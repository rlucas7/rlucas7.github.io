---
title: 'Building a simple code editor in a web browser'
layout: page
---

In a previous blog post I did some study of the event processing and other aspects
of codemirror v5. In that post I mentioned there is a new codemirror version,
v6. This post shows the v6 that I build and loads it so you can test it out
yourself here in the page.


<div id="editor"></div>
  <br>
  <input type="checkbox" id="oneDark" name="oneDark" onchange="changeTheme()">
  <label for="oneDark">Toggle <a href="https://github.com/codemirror/theme-one-dark">One Dark</a> Theme</label>
  <br>
  <!-- CodeMirror 6 -->
<script src="../cm6.bundle.min.js"></script>
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
