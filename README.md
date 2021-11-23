# Svelte modal dialog example with focus trapping using CSS :focus-within

This example is slightly modified from [How to create a Full-Featured Modal Component in Svelte, and trap focus-within - DEV Community](https://dev.to/vibhanshu909/how-to-create-a-full-featured-modal-component-in-svelte-and-trap-focus-within-474i).

* Added a invisible input after the topmost Modal.
    * This is needed for a transitioned event to be fired when the tab key is pressed
      after the last child in the topmost Modal is focused.
    * This hack is used in the last example of [A CSS Approach to Trap Focus Inside of an Element | CSS-Tricks](https://css-tricks.com/a-css-approach-to-trap-focus-inside-of-an-element/)
* Focus the first focusable descendant element when a Modal is opened.
    * `focusFirstFocusableDescendant` function in this example uses the same way as in
      [dialog.js](https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/js/dialog.js) in
      [Modal Dialog Example | WAI-ARIA Authoring Practices 1.1](https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/dialog.html).
      It actually tries to focus an element in `try catch` then checks whether `document.activeElement` becomes the element.
