<script>
  // import loading svelte component
  import Loading from "./UI/Loading.svelte";

  let newValue = "";
  let cart = [];
  let isLoading = true;

  // Primer fetch de items
  let getItems = fetch(
    "https://shopping-list-70f25-default-rtdb.firebaseio.com/.json"
  )
    .then((res) => {
      if (!res.ok) {
        throw new Error("Failed!");
      }
      return res.json();
    })
    .then((data) => {
      console.log(data);
      // Armo un listado de todos los keys
      let keys = Object.keys(data);
      // for every key in keys in data
      for (let i = 0; i < keys.length; i++) {
        let key = keys[i];
        let value = data[key];
        let item = {
          id: key,
          value: value,
          shopped: false,
        };
        cart = [item, ...cart];
        isLoading = false;
      }
    })
    .catch((err) => {
      isLoading = false;
      console.log(err);
    });

  // Creacion de un nuevo item
  function addNew() {
    if (newValue == "") {
      return;
    }

    isLoading = true;
    console.log("adding " + newValue);
    let firebaseId = "";

    // Posteo en firebase el nuevo item
    fetch("https://shopping-list-70f25-default-rtdb.firebaseio.com/.json", {
      method: "POST",
      body: JSON.stringify(newValue),
    })
      .then((response) => response.text())
      .then((responseText) => (firebaseId = JSON.parse(responseText).name))
      .then(() => {
        // Agrego el nuevo item a la lista de items
        let newItem = {
          id: firebaseId,
          value: newValue,
          shopped: false,
        };

        cart = [newItem, ...cart];

        newValue = "";
        isLoading = false;
      })
      .catch((error) => {
        console.log("error", error);
        isLoading = false;
      });
  }

  function clearAll() {
    isLoading = true;
    fetch("https://shopping-list-70f25-default-rtdb.firebaseio.com/.json", {
      method: "DELETE",
    })
      .then((res) => {
        if (!res.ok) {
          throw new Error("Failed!");
        }
        cart = [];
        isLoading = false;
      })
      .catch((error) => {
        console.log("error", error);
        isLoading = false;
      });
  }

  function doneTask(e) {
    let id = e;
    let item = cart.find((i) => i.id === id);
    cart = [
      ...cart.filter((i) => i.id !== id),
      {
        id: id,
        value: item.value,
        shopped: true,
      },
    ];
    fetch(
      "https://shopping-list-70f25-default-rtdb.firebaseio.com/" + id + ".json",
      {
        method: "DELETE",
      }
    )
      .then((res) => {
        if (!res.ok) {
          throw new Error("Failed!");
        }
        isLoading = false;
      })
      .catch((error) => {
        console.log("error", error);
        isLoading = false;
      });
  }

  function submit(e) {
    if (e.key === "Enter" && newValue !== "") {
      addNew();
    }
  }

  function undo(e) {
    let id = e;
    let item = cart.find((i) => i.id === id);
    let firebaseId
    fetch("https://shopping-list-70f25-default-rtdb.firebaseio.com/.json", {
      method: "POST",
      body: JSON.stringify(item.value),
    })
      .then((response) => response.text())
      .then((responseText) => (firebaseId = JSON.parse(responseText).name))
      .then(() => {
        // Agrego el nuevo item a la lista de items
        let newItem = {
          id: firebaseId,
          value: item.value,
          shopped: false,
        };

        cart = [
          newItem,
          ...cart.filter((i) => i.id !== id)
        ];

        newValue = "";
        isLoading = false;
      })
      .catch((error) => {
        console.log("error", error);
        isLoading = false;
      });
    /*

    */
  }
</script>

{#if isLoading}
  <Loading />
{/if}

<h1>Domfer's Shopping List</h1>

{#each cart as item, index (item.id)}
  {#if !item.shopped}
    <div id={item.id}>
      <input
        on:change={(event) => doneTask(item.id)}
        type="checkbox"
        id="item.id"
        name="item.id"
      />
      <label for="item.id">{item.value}</label>
    </div>
  {/if}
{/each}

{#if cart.some((i) => i.shopped)}
  <h2>Completed</h2>
{/if}

{#each cart as item, index (item.id)}
  {#if item.shopped}
    <div id={item.id}>
      <input
        checked
        on:change={(event) => undo(item.id)}
        type="checkbox"
        id="item.id"
        name="item.id"
      />
      <label class="strikethrough" for="item.id">{item.value}</label>
    </div>
  {/if}
{/each}

<footer>
  <div>
    <input on:keyup={(e) => submit(e)} bind:value={newValue} type="text" />
    <button on:click={addNew}>Add</button>
  </div>
  <button on:click={clearAll}>Clear all</button>
</footer>

<style>
  label {
    font: 1rem "Fira Sans", sans-serif;
    display: inline;
  }

  input {
    margin: 0.4rem;
  }

  footer {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100vw;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }

  footer > div {
    width: 90vw;
    display: flex;
    margin: auto;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  footer > div > input {
    width: 60vw;
    margin: 0;
    box-sizing: border-box;
    height: 2rem;
    padding: 0, 1rem;
  }
  footer > div > button {
    width: 20vw;
    height: 2rem;
    box-sizing: border-box;
    padding: 0, 1rem;
    margin: 0;
  }

  footer > button {
    width: 90vw;
    margin-left: 0;
    margin-right: 0;
  }

  .strikethrough {
    text-decoration: line-through;
    color: gray;
  }
</style>
