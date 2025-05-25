---
title: 'Building a simple code editor in a web browser-take2'
layout: page
---

This page demonstrates a code editor and REPL so you can test it out
yourself here in the page.
The checkbox toggles darkmode on and off for the code editor.
The down spot in the gutter can be clicked to toggle folding
for the prepopulated function.
If you write multiline functions you will see additional
folds any `def` (or `class`) lines.

<style>
    /* Set editor dimensions */
    #editor {
        height: 200px;
        width: 50%;
    }
    /* Stretch editor to fit inside its containing div */
    .cm-editor {
        height: 90%;
        width: 100%;
    }
    .blackout-container {
      display: inline-block; /* Or block, depending on layout */
      position: relative;  /* Needed for absolute positioning of the overlay */
    }
    .blackout-text {
        color: white; /* Start with white text for the "blackout" */
        transition: color 0.3s ease; /* Smooth color change on hover */
    }
    .blackout-container:hover .blackout-text {
        color: black; /* Change to black on hover to reveal */
    }
	.run-button {
	    background-color: #04AA6D;
        border: 2px solid #e7e7e7; /* gray */
	    /* border: 4px solid #555555; */ /* black */
        color: white;
        padding: 15px 32px;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        font-size: 16px;
	    border-radius: 12px;
	    text-shadow: 0 -1px 0 #000;
	    box-shadow: 0 1px 0 #666, 0 5px 0 #444, 0 6px 6px rgba(0,0,0,0.6);
	}
	button:active {
        background: #e5e5e5;
        -webkit-box-shadow: inset 0px 0px 5px #c1c1c1;
        -moz-box-shadow: inset 0px 0px 5px #c1c1c1;
        box-shadow: inset 0px 0px 5px #c1c1c1;
         outline: none;
    }
</style>
<script src="https://cdn.jsdelivr.net/pyodide/v0.27.6/full/pyodide.js"></script>
<div class="blackout-container">
    <span class="blackout-text">Psst, here is a secret, you can import heapq methods if you want to use them.</span>
</div>
<div id="editor"></div>
<input type="checkbox" id="oneDark" name="oneDark" onchange="configChange()">
<label for="oneDark">Toggle <a href="https://github.com/codemirror/theme-one-dark">One Dark</a> Theme</label>
<br>
<script src="../cm6-v2.bundle.min.js"></script>
<script>
    const view = cm6.createEditorView(undefined, document.getElementById("editor"));
    const initialState = cm6.createEditorState("def foo(a:int):\n    return sum(i for i in range(a))\n\nfoo(5)");
    view.setState(initialState);
    function configChange() {
        const oneDarkEl = document.getElementById("oneDark");
	    const e = document.getElementById("indentUnit");
	    const value = e.value;
        const text = e.options[e.selectedIndex].text;
	    const options = {oneDark: oneDarkEl.checked, indentAmount: " ".repeat(Number(text))};
        const newState = cm6.createEditorState(view.state.doc, options);
        view.setState(newState);
	}
</script>
<button class="run-button" onclick="evaluatePython()">Run</button>
<div>Shell Output:</div>
<textarea id="output" style="width: 100%;" rows="16" disabled></textarea>
<script>
  const output = document.getElementById("output");
  const cmEditorElement = document.querySelector(".cm-editor")
  const editorView = cmEditorElement.querySelector(".cm-content").cmView.view
  const code = editorView.viewState.state.doc.toString()
  function addToOutput(s) {
    output.value += ">>>" + s + "\n";
  }
  output.value = "Initializing...\n";
  async function main() {
    const pyodide = await loadPyodide();
    output.value += "Ready!\n";
    return pyodide;
  }
  const pyodideReadyPromise = main();
  async function evaluatePython() {
    const code = editorView.viewState.state.doc.toString()
    const pyodide = await pyodideReadyPromise;
    try {
      const output = pyodide.runPython(code);
      addToOutput(code);
      addToOutput(output);
    } catch (err) {
      addToOutput(err);
    }
  }
  const blackoutContainers = document.querySelectorAll(".blackout-container");
  blackoutContainers.forEach(container => {
    const text = container.querySelector(".blackout-text");
    container.addEventListener("mouseenter", () => {
      text.style.color = "black";
    });
    container.addEventListener("mouseleave", () => {
      text.style.color = "white";
    });
  });
</script>
