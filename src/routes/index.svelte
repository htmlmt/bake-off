<script>
import { onMount } from 'svelte';
import Turntable from '../components/Turntable.svelte';

onMount(() => {
    if (window.netlifyIdentity) {
        window.netlifyIdentity.on("init", user => {
            if (!user) {
                window.netlifyIdentity.on("login", () => {
                    document.location.href = "/admin/";
                });
            }
        });
    }
});
</script>

<style>

</style>

<svelte:head>
    <title>The Full MT</title>
    <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
</svelte:head>

<Turntable/>