<script>
    import { onMount } from "svelte";
    import { goto } from "@sapper/app";

    onMount(async () => {
        await goto("/docs/introduction");
    })
</script>