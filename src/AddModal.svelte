<script>
    import { modalState } from "./store.js";

    const closeModal = () => {
        modalState.set(false);
    }
</script>

<div class="modal-overlay">
    <div class="add-modal">
        <button on:click={closeModal}>close</button>
    </div>
</div>

<style>
    .modal-overlay {
      height: 100vh;
      width: 100vw;
      position: absolute;
      top: 0;
      left: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      background-color: rgba(0, 0, 0, 0.8);
    }
  
    .add-modal {
      height: 22rem;
      width: 32rem;
      background-color: #4a4a4a;
    }
  </style>