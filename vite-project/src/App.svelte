<script>
  import Router from "svelte-spa-router";
  import Home from "./Home.svelte";
  import Detail from "./Detail.svelte";

  const routes = {
    "/": Home,
    "/detail/:id": Detail,
  };
</script>

<Router {routes} />
