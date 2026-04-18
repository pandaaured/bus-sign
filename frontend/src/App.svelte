<script lang="ts">
    import {onMount} from "svelte";
    import BusTimeEntry from "./lib/BusTimeEntry.svelte";

    type RouteInformation = {
        route: string;
        destination: string;
        arrivals: {
            bus_id: string;
            capacity: string;
            seconds: number;
        }[];
    };

    type APIResponse = {
        [stopId: string]: RouteInformation[];
    };

    let entriesUC: RouteInformation[] = [];
    let entriesTep: RouteInformation[] = [];
    let lastUpdated: Date | null = null;

    let paddingX: number = 4;
    let paddingY: number = 3;

    const API_BASE = import.meta.env.VITE_API_BASE || "";

    const fetchPredictions = async (): Promise<APIResponse> => {
        const response = await fetch(`${API_BASE}/predictions`, {
            cache: "no-store",
        });

        if (!response.ok) {
            throw new Error(`Failed to fetch predictions: ${response.status}`);
        }
        return (await response.json()) as APIResponse;
    };

    const refresh = async () => {
        try {
            const data = await fetchPredictions();
            entriesUC = (data["7117"] || []).sort(
                (a, b) =>
                    (a.arrivals[0]?.seconds || Infinity) -
                    (b.arrivals[0]?.seconds || Infinity),
            );
            entriesTep = (data["4407"] || []).sort(
                (a, b) =>
                    (a.arrivals[0]?.seconds || Infinity) -
                    (b.arrivals[0]?.seconds || Infinity),
            );
            lastUpdated = new Date();
            paddingX = Math.max(entriesUC.length, entriesTep.length) <= 5 ? 16 : 4;
            paddingY = Math.max(entriesUC.length, entriesTep.length) <= 5 ? 12 : 3;
        } catch (error) {
            console.error(error);
        }
    };

    $: formattedTime = lastUpdated
        ? lastUpdated.toLocaleTimeString("en-US", {
              hour: "numeric",
              minute: "2-digit",
              second: "2-digit",
          })
        : "";

    onMount(() => {
        void refresh();
        const interval = setInterval(refresh, 3_000);
        return () => clearInterval(interval);
    });
</script>

<main>
    <div class="header">
        <div class="header-left">
            <img src="../public/scotty.svg" alt="Scotty Logo" class="header-logo" />
            <span class="cmu-text">CARNEGIE MELLON<br/>UNIVERSITY</span>
        </div>
        Forbes and Morewood Bus Stops
    </div>
    <div class="main-body">
        <div class="tartan-column"></div>
        <div
            class="container"
            style="justify-content: start; align-items: flex-start"
        >
            <div class="stack left bus-list-column">
                <div class="stop-header" style="padding: 0 {paddingX}px;">
                    UC Side <span class="arrow">&rarr;</span> <span class="stop-id">(Stop 7117)</span>
                    <span class="walk-time">(1-2) minutes walk</span>
                </div>
                <div class="bus-list">
                    {#each entriesUC as entry (entry.route + entry.destination)}
                        <BusTimeEntry {...entry} {paddingX} {paddingY} />
                    {:else}
                        <BusTimeEntry
                            route={"No Buses Running"}
                            destination={""}
                            arrivals={[]}
                            paddingX={16}
                            paddingY={12}
                        />
                    {/each}
                </div>
            </div>
            <div class="divider-vertical"></div>
            <div class="stack left bus-list-column">
                <div class="stop-header" style="padding: 0 {paddingX}px;">
                    Tepper Side <span class="arrow">&larr;</span> <span class="stop-id">(Stop 4407)</span>
                    <span class="walk-time">(3-5) minutes walk</span>
                </div>
                <div class="bus-list">
                    {#each entriesTep as entry (entry.route + entry.destination)}
                        <BusTimeEntry {...entry} {paddingX} {paddingY} />
                    {:else}
                        <BusTimeEntry
                            route={"No Buses Running"}
                            destination={""}
                            arrivals={[]}
                            paddingX={16}
                            paddingY={12}
                        />
                    {/each}
                </div>
            </div>
        </div>
    </div>
    <footer class="footer">
        <div class="stack footer-text">
            <p class="attribution">
                Project by Undergraduate Student Senate via collaboration with ScottyLabs. Funded in part by your Student Activities Fee.
            </p>
            <p class="disclaimer">
                Data provided under license from PRT; this application is not
                officially endorsed by Pittsburgh Regional Transit.
            </p>
        </div>
        {#if formattedTime}
            <p class="last-updated">Last updated: {formattedTime}</p>
        {/if}
    </footer>
</main>

<style>
    .main-body {
        display: flex;
        flex: 1 1 auto;
        width: 100%;
    }

    .container {
        display: flex;
        width: 100%;
        padding-bottom: 80px;
    }

    .bus-list-column {
        padding: 0 1rem;
    }

    .bus-list {
        display: flex;
        flex-direction: column;
        width: 100%;
        margin-top: 10px;
        gap: 0;
    }

    .divider-vertical {
        width: 2px;
        background-color: #e0e0e0;
        align-self: stretch;
        margin: 0 1rem;
    }

    .tartan-column {
        width: 55px;
        flex-shrink: 0;
        background-image: url('/cmu-tartan-wave-full-color-crop-01.svg');
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
    }

    .stack {
        flex: 1 1 0;
        gap: 0.75rem;
    }

    .stop-header {
        font-size: 40px;
        font-weight: bold;
        display: flex;
        align-items: center;
        gap: 12px;
        width: 100%;
        box-sizing: border-box;
    }

    .walk-time {
        margin-left: auto;
        font-size: 24px;
        font-weight: normal;
        color: #666;
    }

    .stop-id {
        font-size: 30px;
        font-weight: normal;
        color: #666;
    }

    .arrow {
        font-size: 60px;
        font-weight: 900;
        color: rgba(41, 38, 38, 0.8);
        line-height: 1;
        transform: translateY(-4px);
    }

    .attribution {
        font-weight: bold;
        font-size: 20px;
        color: black;
        padding-left: 20px;
        margin: 0;
    }

    .disclaimer {
        font-weight: normal;
        color: gray;
        padding-left: 20px;
        margin: 0;
    }

    .footer-text {
        gap: 0;
        padding: 0.25rem 0;
    }

    .header {
        position: relative;
        display: flex;
        background-color: #ad2a3c;
        color: #ffffff;
        justify-content: center;
        align-items: center;
        font-size: 60px;
        padding: 20px;
    }

    .header-left {
        position: absolute;
        left: 30px;
        display: flex;
        align-items: center;
        gap: 20px;
    }

    .cmu-text {
        font-size: 26px;
        font-weight: bold;
        text-align: left;
        line-height: 1.1;
    }

    .header-logo {
        width: auto;
        height: 67px;
        flex-shrink: 0;
    }

    .footer {
        position: fixed;
        bottom: 0;
        left: 0;
        width: 100%;
        background: white;
        border-top: 1px solid #ccc;
        display: flex;
        justify-content: space-between;
        align-items: flex-end;
        z-index: 1000;
    }

    .last-updated {
        font-weight: normal;
        color: gray;
        font-size: 14px;
        padding-right: 20px;
    }
</style>
