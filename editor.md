---
title: 'Building a simple code editor in a web browser'
layout: page
---

This page demonstrates a code editor so you can test it out
yourself here in the page.

The checkbox toggles darkmode on and off for the code editor.

The down spot in the gutter can be clicked to toggle folding
for the prepopulated function.

If you write more multiline functions you will see additional
folds any `def` (or `class`) lines.


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
  const initialState = cm6.createEditorState("def foo(a:int):\n\tprint(\"multiplying by e\")\n\ta *= 2.718281828\n\treturn a\n\n#try out the folds on the first line in the gutter");
  view.setState(initialState);
  function changeTheme() {
      let options = {oneDark: oneDarkEl.checked};
      let newState = cm6.createEditorState(view.state.doc, options);
      view.setState(newState);
  }
</script>
