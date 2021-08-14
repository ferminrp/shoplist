<script>
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
          value: value
        }
        cart = [
          item,
          ...cart
        ]
      }
    })
    .catch((err) => {
      isLoading = false;
      console.log(err);
    });

  // Creacion de un nuevo item
  function addNew() {
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
        };

        cart = [newItem, ...cart];

        newItem = "";
      })
      .catch((error) => console.log("error", error));
  }
</script>

<h1>Domfer's Shopping List</h1>
<input bind:value={newValue} type="text" />
<button on:click={addNew}>Add</button>

{#each cart as item, index}
  <div>
    <input type="checkbox" id="item.id" name="item.id" />
    <label for="item.id">{item.value}</label>
  </div>
{/each}

<style>
  label {
    font: 1rem "Fira Sans", sans-serif;
    display: inline;
  }

  input {
    margin: 0.4rem;
  }
</style>
