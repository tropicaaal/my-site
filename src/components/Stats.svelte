<script>
    import Nixies from "./Nixies.svelte";

    const CACHE_TTL_MS = 5 * 60 * 1000; // 5 mins

    function cacheKey(site) {
        return `stats:${site}`;
    }

    function readCache(site) {
        try {
            const raw = localStorage.getItem(cacheKey(site));
            if (!raw) return null;
            const { timestamp, data } = JSON.parse(raw);
            if (Date.now() - timestamp > CACHE_TTL_MS) return null;
            return data;
        } catch {
            return null;
        }
    }

    function writeCache(site, data) {
        try {
            localStorage.setItem(
                cacheKey(site),
                JSON.stringify({ timestamp: Date.now(), data }),
            );
        } catch {}
    }

    let { site, viewsDigits = 8, followersDigits = 8 } = $props();

    let loading = $state(true);
    let failed = $state(false);
    let stats = $state(null);

    $effect(() => {
        const cached = readCache(site);
        if (cached) {
            stats = cached;
            loading = false;
            return;
        }

        fetch(`https://nekoweb.org/api/site/info/${site}`, {
            cache: "force-cache",
        })
            .then((res) => res.json())
            .then((data) => {
                stats = data;
                writeCache(site, data);
            })
            .catch(() => {
                failed = true;
            })
            .finally(() => {
                loading = false;
            });
    });
</script>

<div class="stats">
    {#if failed}
        Failed to fetch! 🐕
    {:else}
        <div class="stat-row">
            <em>Created:</em>
            {#if loading}
                Loading...
            {:else}
                <time>{new Date(stats.created_at).toLocaleDateString()}</time>
            {/if}
        </div>
        <div class="stat-row">
            <em>Updated:</em>
            {#if loading}
                Loading...
            {:else}
                <time>{new Date(stats.updated_at).toLocaleDateString()}</time>
            {/if}
        </div>
        <div class="stat-row">
            <em>Views:</em>
            {#if loading}
                Loading...
            {:else}
                <Nixies value={stats.views} digits={viewsDigits} pad="left" />
            {/if}
        </div>
        <div class="stat-row">
            <em>Followers:</em>
            {#if loading}
                Loading...
            {:else}
                <Nixies
                    value={stats.followers}
                    digits={followersDigits}
                    pad="left"
                />
            {/if}
        </div>
    {/if}
</div>

<style>
    .stats {
        display: table;
    }

    .stat-row {
        display: table-row;
    }

    .stat-row em {
        text-align: left;
        width: 1px;
        padding-right: 8px;
    }

    .stat-row > * {
        display: table-cell;
    }
</style>
