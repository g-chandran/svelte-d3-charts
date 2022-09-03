<script>
  import { charts } from "../utils/charts";

  let selected = charts[0];
</script>

<main>
  <select bind:value={selected}>
    {#each charts as chart}
      <option value={chart}>{chart.name}</option>
    {/each}
  </select>
  <h2>{selected.name}</h2>
  <svelte:component this={selected.component} />
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    justify-items: center;
    align-items: center;
  }
  select {
    width: min-content;
    margin: 2rem 0;
  }
</style>
