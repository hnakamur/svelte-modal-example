<script>
  import { booleanStore } from "./booleanStore";

  const store = booleanStore(false);
  const { isOpen, open, close } = store;

  export let focusAfterClosed;
  let dialogNode;
  let lastFocus;
  let ignoresFocusChange;

  function modalAction(node) {
    // mix of https://github.com/KittyGiraudel/focusable-selectors/blob/799829e3b8c329d679b3b414b5dfcfa257a817cf/index.js
    // and https://github.com/focus-trap/tabbable/blob/baa8c3044fe0a8fd8c0826f4a3e284872e1467a5/src/index.js#L1-L13
    const focusableCandidateSelectors =
      'a[href]:not([tabindex^="-"])' +
      ',area[href]:not([tabindex^="-"])' +
      ',input:not([type="hidden"]):not([type="radio"]):not([disabled]):not([tabindex^="-"])' +
      ',input[type="radio"]:not([disabled]):not([tabindex^="-"])' +
      ',select:not([disabled]):not([tabindex^="-"])' +
      ',textarea:not([disabled]):not([tabindex^="-"])' +
      ',button:not([disabled]):not([tabindex^="-"])' +
      ',iframe:not([tabindex^="-"])' +
      ',audio[controls]:not([tabindex^="-"])' +
      ',video[controls]:not([tabindex^="-"])' +
      ',[contenteditable]:not([tabindex^="-"])' +
      ',[tabindex]:not([tabindex^="-"])' +
      ',details>summary:not([tabindex^="-"])' +
      ',details:not([tabindex^="-"])';

    // attemptFocus, focusFirstDescendant, focusLastDesendant, trapFocus logic are
    // ported from https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/dialog.html

    const attemptFocus = (element) => {
      ignoresFocusChange = true;
      try {
        element.focus();
      } catch {}
      ignoresFocusChange = false;
      return element === document.activeElement;
    };

    function focusFirstDescendant(element) {
      if (element) {
        const descendants = element.querySelectorAll(
          focusableCandidateSelectors
        );
        for (const el of descendants) {
          if (attemptFocus(el)) break;
        }
      }
    }

    function focusLastDescendant(element) {
      if (element) {
        const descendants = element.querySelectorAll(
          focusableCandidateSelectors
        );
        for (let i = descendants.length - 1; i >= 0; i--) {
          if (attemptFocus(descendants[i])) break;
        }
      }
    }

    function trapFocus(event) {
      if (!ignoresFocusChange) {
        if (dialogNode.contains(event.target)) {
          lastFocus = event.target;
        } else {
          focusFirstDescendant(dialogNode);
          if (lastFocus === document.activeElement) {
            focusLastDescendant(dialogNode);
          }
          lastFocus = document.activeElement;
        }
      }
    }

    function keydown(e) {
      e.stopPropagation();
      if (e.key === "Escape") {
        close();
      }
    }

    let original;
    // for accessibility
    if (document.body.style.overflow !== "hidden") {
      original = document.body.style.overflow;
      document.body.style.overflow = "hidden";
    }
    document.addEventListener("focus", trapFocus, true);
    node.addEventListener("keydown", keydown);
    focusFirstDescendant(node);
    return {
      destroy() {
        if (original) {
          document.body.style.overflow = original;
        }
        document.removeEventListener("focus", trapFocus, true);
        node.removeEventListener("keydown", keydown);
        focusAfterClosed.focus();
      },
    };
  }
</script>

<slot name="trigger" {open}>
  <!-- fallback trigger to open the modal -->
  <button on:click={open}>Open</button>
</slot>
{#if $isOpen}
  <!-- Bracket the dialog node with two invisible, focusable nodes (<div tablindex="0" />).
  While this dialog is open, we use these to make sure that focus never
  leaves the document even if dialogNode is the first or last node.
  -->
  <div tabindex="0" />

  <div class="modal" bind:this={dialogNode} use:modalAction>
    <div class="backdrop" on:click={close} />

    <div class="content-wrapper">
      <slot name="header" {store}>
        <!-- fallback -->
        <div>
          <h1>Your Modal Heading Goes Here...</h1>
        </div>
      </slot>

      <div class="content">
        <slot name="content" {store} />
      </div>

      <slot name="footer" {store}>
        <!-- fallback -->
        <div>
          <h1>Your Modal Footer Goes Here...</h1>
          <button on:click={close}>Close</button>
        </div>
      </slot>
    </div>
  </div>

  <div tabindex="0" />
{/if}

<style>
  div.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;

    display: flex;
    justify-content: center;
    align-items: center;
    opacity: 1;
  }
  div.backdrop {
    background-color: rgba(0, 0, 0, 0.4);
    position: absolute;
    width: 100%;
    height: 100%;
  }
  div.content-wrapper {
    z-index: 10;
    max-width: 70vw;
    border-radius: 0.3rem;
    background-color: white;
    overflow: hidden;
    padding: 1rem;
  }
  /*   @media (max-width: 767px) {
      div.content-wrapper {
        max-width: 100vw;
      }
    } */
  div.content {
    max-height: 50vh;
    overflow: auto;
  }
  h1 {
    opacity: 0.5;
  }
</style>
