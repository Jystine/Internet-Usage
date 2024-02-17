<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import Internet from "./internet.svelte";

    let data = [];

 onMount(async () => {
            const res = await fetch('cleaned_internet.csv'); 
            const csv = await res.text();
            data = d3.csvParse(csv, d3.autoType)
            console.log(data);
        });

</script>

<main>
  <h1>Internet Usage Overtime</h1>
  <Internet {data} />
</main>

<style>
    main {
        text-align: center;
    }
</style>