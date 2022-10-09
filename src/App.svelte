
<script>
  import { onMount } from "svelte";
  import { modalState } from "./store.js";
  import AddModal from "./AddModal.svelte";

  let song = "";
  let artist = "";
  let instrument = "";
  let tuning = "";
  let link = "";

  let modal = false;

  let addTab = false;

  modalState.subscribe(value => {
    modal = value;
  })

  let sortedOn = "artist";
  let order = 1; // 1 = ascending; -1 = descending
  
  let tabs = []
  onMount(async () => {
    const res = await fetch("https://mytabs-api.herokuapp.com/");
    tabs = await res.json();
  })

  const sortClick = async (e) => {
    // get the column name to sort
    let header = e.target.innerText.toLowerCase()

    // if sorting new column change sortedOn
    if (sortedOn !== header) {
      sortedOn = header
      order = 1;
    } else {
      if (order === 1) { 
        order = -1;
      } else { 
        order = 1;
      }
    }

    const res = await fetch(`http://mytabs-api.herokuapp.com/sort?sortHeader=${header}&dir=${order}`)
    tabs = await res.json();
  }

  const editTab = (e) => {
    // // find tab object from tab row id
    const selectedTab = tabs.find(i => i._id === e);
    song = selectedTab.song;
    artist = selectedTab.artist;
    instrument = selectedTab.instrument;
    tuning = selectedTab.tuning;
    link = selectedTab.link;
    selectedTab.editMode = !selectedTab.editMode;

    tabs = [...tabs];
  }

  const updateTab = async (e) => {
    const res = await fetch("http://mytabs-api.herokuapp.com/submit", {
      method: "PUT",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        id: e,
        song: song,
        artist: artist,
        instrument: instrument,
        tuning: tuning,
        link: link
      })
    })

    if(res.status === 200) {
      editTab(e)
      const res = await fetch("http://mytabs-api.herokuapp.com");
      tabs = await res.json();
    }
    
  }

  const deleteTab = async (e) => {
    const res = await fetch(`http://mytabs-api.herokuapp.com/submit?id=${e}`, {
      method: "DELETE"
    })

    if(res.status === 200) {
      const res = await fetch("http://mytabs-api.herokuapp.com");
      tabs = await res.json();
    }
  }

  const showModal = () => {
    tabs.unshift({ _id: 0 })
    tabs = [...tabs]
    editTab(0);
    addTab = true;
  }

  const cancelModal = () => {
    tabs.shift();
    tabs = [...tabs]
    addTab = false;
  }


</script>

{#if modal}
<AddModal />
{/if}
<main>
  <div class="add-container">
    <button class="add-button">
      <img src="icons8-plus-math-24.png" alt="add tab" on:click={showModal} />
    </button>
  </div>
  <ul class="list-headers">
    <li class="song" on:click={sortClick}>
      <div class="sort-column">
        {#if sortedOn === "song" && order === 1}
          <img src="icons8-sort-down-24.png" alt="add tab" />
        {:else}
          {#if sortedOn === "song" && order === -1}
            <img src="icons8-sort-up-24.png" alt="add tab" />
          {/if}
        {/if}
        Song
      </div>
    </li>
    <li class="artist" on:click={sortClick}>
      <div class="sort-column">
        {#if sortedOn === "artist" && order === 1}
          <img src="icons8-sort-down-24.png" alt="add tab" />
        {:else}
          {#if sortedOn === "artist" && order === -1}
            <img src="icons8-sort-up-24.png" alt="add tab" />
          {/if}
        {/if}
        Artist
      </div>
    </li>
    <li class="instrument" on:click={sortClick}>
      <div class="sort-column">
        {#if sortedOn === "instrument" && order === 1}
          <img src="icons8-sort-down-24.png" alt="add tab" />
        {:else}
          {#if sortedOn === "instrument" && order === -1}
            <img src="icons8-sort-up-24.png" alt="add tab" />
          {/if}
        {/if}
        Instru
      </div>
    </li>
    <li class="tuning" on:click={sortClick}>
      <div class="sort-column">
        {#if sortedOn === "tuning" && order === 1}
          <img src="icons8-sort-down-24.png" alt="add tab" />
        {:else}
          {#if sortedOn === "tuning" && order === -1}
            <img src="icons8-sort-up-24.png" alt="add tab" />
          {/if}
        {/if}
        Tuning
      </div>
    </li>
    <li class="link" on:click={sortClick}>
      <div class="sort-column">
        {#if sortedOn === "link" && order === 1}
          <img src="icons8-sort-down-24.png" alt="add tab" />
        {:else}
          {#if sortedOn === "link" && order === -1}
            <img src="icons8-sort-up-24.png" alt="add tab" />
          {/if}
        {/if}
        Link
      </div>
    </li>
  </ul>
  <ul>
    {#each tabs as tab}
      {#if tab.editMode}
        <form on:submit|preventDefault={t => updateTab(tab._id)}>
          <li class="song">
            <input type="text" bind:value={song} autofocus />
          </li>
          <li class="artist">
            <input type="text" bind:value={artist} />
          </li>
          <li class="instrument">
            <input type="text" bind:value={instrument} />
          </li>
          <li class="tuning">
            <input type="text" bind:value={tuning} />
          </li>
          <li class="link">
            <input type="text" bind:value={link} />
          </li>
          <li class="actions">
            <button type="submit" class="confirm-button"><img src="icons8-done.svg" alt="edit tab" /></button>
            {#if addTab}
              <img src="icons8-close.svg" alt="delete tab" style="margin-left: 1.5rem" on:click={cancelModal} />
            {:else}
              <img src="icons8-close.svg" alt="delete tab" style="margin-left: 1.5rem" on:click={e => editTab(tab._id)} />
            {/if}
          </li>
        </form>
      {:else}
        <li class="song">{tab.song}</li>
        <li class="artist">{tab.artist}</li>
        <li class="instrument">{tab.instrument === undefined ? "" : tab.instrument}</li>
        <li class="tuning">{tab.tuning === undefined ? "" : tab.tuning}</li>
        {#if tab.link === undefined}
          <li class="link"></li>
        {:else}
          <li class="link"><a href={tab.link} target="_blank">{tab.link}</a></li>
        {/if}
        <li class="actions">
          <img src="icons8-edit.svg" style="margin-right: 1.5rem" alt="edit tab" on:click={e => editTab(tab._id)} />
          <img src="icons8-trash.svg" alt="delete tab" on:click={e => deleteTab(tab._id)} />
        </li>
      {/if}
    {/each}
  </ul>
</main>
