<script context="module">
    export async function preload({ params: { slug } }) {
        const response = await this.fetch(`process.env.FEATHER_API/docs/page/${encodeURI(slug)}`)
        let markdown_html = await response.text();
        return { html: markdown_html };
    }
</script>

<script>
    export let html
</script>

{@html html}

<style>

</style>