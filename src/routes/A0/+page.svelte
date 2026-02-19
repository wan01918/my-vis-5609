<script>
  let maxClicks = 5;
  let remainingClicks = 5;

  function clickOnce() {
    if (remainingClicks > 0) {
      remainingClicks = remainingClicks - 1;
      if (remainingClicks === 0) {
        alert("No more clicks remaining!");
      }
    } else {
      alert("No more clicks remaining!");
    }
  }

  function undoOnce() {
    if (remainingClicks < maxClicks) {
      remainingClicks = remainingClicks + 1;
    }
  }

  function changeMax(event) {
    maxClicks = Number(event.target.value);
    remainingClicks = maxClicks;
  }
</script>

<main>
  <h1>CSCI 5609 - Assignment 0</h1>

<img
  class="logo"
  src="https://blog.nashtechglobal.com/wp-content/uploads/2023/05/svelte-1024x576.jpg"
  alt="Svelte Illustration"
/>

  <p>Remaining clicks: <strong>{remainingClicks}</strong></p>

  <div class="controls">
    <label for="max">Max clicks:</label>
    <select id="max" on:change={changeMax}>
      <option value="5" selected>5</option>
      <option value="10">10</option>
      <option value="15">15</option>
      <option value="20">20</option>
    </select>
  </div>

  <div class="buttons">
    <button class="primary" on:click={clickOnce}>Click</button>
    <button class="secondary" on:click={undoOnce}>Undo</button>
  </div>
</main>

<style>
  main {
    font-family: Arial, sans-serif;
    text-align: center;
    margin-top: 50px;
    padding: 20px;
  }

  .logo {
    width: 100px;
    margin: 20px 0;
  }

  .controls {
    margin: 10px 0 20px 0;
  }

  select {
    margin-left: 8px;
    padding: 6px 8px;
    font-size: 16px;
  }

  .buttons {
    margin-top: 10px;
  }

  .primary {
    background-color: #44aa66;
    font-size: xx-large;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 10px 20px;
    cursor: pointer;
    margin: 6px;
  }

  .secondary {
    font-size: large;
    padding: 10px 20px;
    cursor: pointer;
    margin: 6px;
  }
</style>