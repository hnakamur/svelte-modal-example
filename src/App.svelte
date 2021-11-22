<script>
  // This example is copied from
  // https://dev.to/vibhanshu909/how-to-create-a-full-featured-modal-component-in-svelte-and-trap-focus-within-474i
  import Modal from "./lib/Modal.svelte";
</script>

<Modal>
  <div slot="trigger" let:open>
    <button on:click={open}>Open Modal</button>
  </div>
  <div slot="header">
    <h1>First Modal</h1>
  </div>
  <div slot="content">
    <!-- Modal within a Modal -->
    <Modal>
      <div slot="trigger" let:open>
        <button on:click={open}>Open Second Modal</button>
      </div>
      <div slot="header">
        <h1>Second Modal</h1>
      </div>
      <div slot="content">
        <p>
          Lorem ipsum dolor sit amet, consectetur adipisicing elit. Similique,
          magni earum ut ex totam corporis unde incidunt deserunt, dolorem
          voluptatum libero quia. Maiores, provident error vel veritatis itaque
          nemo commodi.
        </p>
      </div>
    </Modal>

    <p>
      Lorem ipsum dolor sit amet, consectetur adipisicing elit. Similique, magni
      earum ut ex totam corporis unde incidunt deserunt, dolorem voluptatum
      libero quia. Maiores, provident error vel veritatis itaque nemo commodi.
    </p>
  </div>

  <div slot="footer" let:store={{ close }}>
    <button on:click={close}>Close First Modal</button>
  </div>
</Modal>
