<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data;
    onMount(
        async() => {
            data = await d3.csv('https://raw.githubusercontent.com/Jystine/Internet-Usage/main/data/internet_usage.csv')
        }
    )
    $: console.log(data)
</script>