<script>
  import { onMount } from "svelte";

  let data = [];
  let error = null;

  onMount(async () => {
    try {
      const response = await fetch("/data/stardata.json");

      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }

      const contentType = response.headers.get("content-type");
      if (!contentType || !contentType.includes("application/json")) {
        throw new TypeError("Response is not JSON");
      }

      data = await response.json();
      if (data.length > 0) {
        console.log("ID of the first object:", data[0].id);
      } else {
        console.log("No data available");
      }
    } catch (err) {
      console.error("Error fetching data:", err);
      error = err.message;
    }
  });
</script>

{#if error}
  <p>Error: {error}</p>
{:else if data.length > 0}
  <div>
    <h1>Star Data</h1>
    <p>Check the console for the ID of the first object.</p>
  </div>
{:else}
  <p>Loading...</p>
{/if}
