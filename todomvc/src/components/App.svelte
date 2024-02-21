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
  <h1>Internet Usage Increases Dramatically Since the Start of the Century!</h1>
  <p>The Internet was first invented back in 1983 and has since then become a staple in today's world with a lot of people using
    it on a daily basis. <br> <a href = "https://data.un.org/">UNdata </a>has recorded how much of a region's population uses the Internet during certain years. Let's see how Internet usage has changed
    <br>since the start of the 21st century.</p>
  <Internet {data} />
</main>

<style>
    main {
        text-align: center;
        font-family: 'Open Sans', sans-serif;
    }
</style>