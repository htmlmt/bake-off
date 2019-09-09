<script>
import { onMount } from 'svelte';
import Tracks from '../components/Tracks.svelte';
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

<svelte:head>
    <title>The Full MT</title>
    <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
</svelte:head>

<Tracks/>
<Turntable/>