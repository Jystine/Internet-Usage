<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import Internet from "./internet.svelte";

    let data = [];
    onMount(
        async() => {
            data = await d3.csv('https://raw.githubusercontent.com/Jystine/Internet-Usage/main/data/internet_usage.csv').then(function (d) {
                data = d;
            });
        }
    )

</script>

<main>
  <h1>Internet Usage</h1>
  <Internet {data} />
</main>

<style>
    main {
        text-align: center;
    }
</style>