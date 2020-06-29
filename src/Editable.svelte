<script lang="typescript">
  import { onMount } from 'svelte';

  let edit: HTMLElement;
  let html: HTMLElement;

  onMount(() => {
    edit.addEventListener('keypress', keypress);
    edit.addEventListener('input', input);
    edit.focus();
  });

  function handleEnter(e: KeyboardEvent) {
    if (e.key === 'Enter') {
      if (window.getSelection) {
        const selection = window.getSelection();
        const range = selection.getRangeAt(0);
        const p = document.createElement('p');
        const zeroWidthSpace = document.createTextNode('\u200B');
        p.appendChild(zeroWidthSpace);
        const textNodeParent = document.getSelection().anchorNode.parentNode;
        const inPara = textNodeParent.nodeName === 'P';
        const inSpan = textNodeParent.nodeName === 'SPAN';
        const span = document.createElement('span');

        // if the carat is inside a <span>, move it out of the <span> tag
        if (inSpan || inPara) {
          console.log('span1');
          range.setStartAfter(textNodeParent);
          range.setEndAfter(textNodeParent);
        }

        // insert the <p>
        range.deleteContents();
        range.insertNode(p);
        range.setStartAfter(zeroWidthSpace);
        range.collapse(true);

        // create a new span on the next line
        if (inSpan) {
          console.log('span2');
          range.insertNode(span);
          range.setStart(span, 0);
          range.setEnd(span, 0);
        }

        // insert the new range
        selection.removeAllRanges();
        selection.addRange(range);
        e.preventDefault();
      }
    }
  }

  function keypress(e: KeyboardEvent) {
    // If the last character is a zero-width space, remove it
    const contentEditableHTML = edit.innerHTML;
    const lastCharCode = contentEditableHTML.charCodeAt(contentEditableHTML.length - 1);
    if (lastCharCode === 8203) {
      edit.innerHTML = contentEditableHTML.slice(0, -1);
    }
    handleEnter(e);
  }

  function wrapInParagraph() {
    // When first char is inserted, put it in a <p> tag.
    // The browser won’t let me do it earlier.
    const selection = window.getSelection();
    const range = selection.getRangeAt(0);
    const p = document.createElement('p');
    const text = document.createTextNode(edit.innerHTML);
    p.appendChild(text);
    edit.removeChild(edit.firstChild);
    range.insertNode(p);
    range.setStart(p, 1);
    range.collapse(true);
    selection.removeAllRanges();
    selection.addRange(range);
  }

  function input() {
    if (edit.innerHTML.length === 1) {
      wrapInParagraph();
    }
    html.innerText = edit.innerHTML;
  }
</script>

<div id="container">
  <div id="edit" bind:this="{edit}" contenteditable="true"></div>
  <div id="html" bind:this="{html}"></div>
</div>

<style>
  #edit {
    background-color: cornflowerblue;
    grid-area: 'edit';
    padding: 1em;
  }
  #html {
    background-color: lightgray;
    grid-area: 'html';
  }
  #container {
    display: grid;
    grid-template-areas: 'edit html';
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 500px;
  }
</style>
