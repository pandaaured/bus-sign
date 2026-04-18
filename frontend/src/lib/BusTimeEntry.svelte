<script lang="ts">
    export let route: string;
    export let destination: string;
    export let arrivals: {
        bus_id: string;
        capacity: string;
        seconds: number;
    }[];
    export let paddingX: number; // padding along left-right
    export let paddingY: number; // padding along up-down

    const routeColors: Record<string, string> = {
        "61A": "#e74c3c",
        "61B": "#3498db",
        "61C": "#2ecc71",
        "61D": "#f39c12",
        "67": "#9b59b6",
        "69": "#b359b6",
        "58": "#1abc9c",
    };

    const capacityInfo: Record<string, { label: string; color: string; level: number }> = {
        EMPTY: { label: "Empty", color: "#2ecc71", level: 1 },
        HALF_EMPTY: { label: "Some seats", color: "#f18f0f", level: 2 },
        FULL: { label: "Full", color: "#e74c3c", level: 3 },
    };

    const formatTime = (seconds: number): string => {
        if (seconds < 30) return "Approaching";
        const minutes = Math.ceil(seconds / 60);
        return `${minutes}`;
    };

    $: nextArrival = arrivals[0];
    $: timeDisplay = nextArrival ? formatTime(nextArrival.seconds) : "N/A";
    $: isApproaching = timeDisplay === "Approaching";
    $: upcomingTimes = arrivals
        .slice(1, 3)
        .map((a) => formatTime(a.seconds))
        .join(", ");
    $: badgeColor = routeColors[route] || "#6c757d";
    $: capacity = nextArrival?.capacity
        ? capacityInfo[nextArrival.capacity]
        : null;
</script>

<div class="bus-entry container" style="padding: {paddingY}px {paddingX}px">
    <div class="stack left">
        <div class="route" style="color: {badgeColor}">
            {route}
        </div>
        <div>
            To {destination.toUpperCase()}
        </div>
    </div>
    <div class="right-cluster">
        {#if capacity}
            <div
                    class="capacity-bar-container"
                    style="background-color: {capacity.color}40;"
            >
                <div
                        class="capacity-bar-fill"
                        style="width: {capacity.level === 3 ? '100%' : (capacity.level === 2 ? '45%' : '15%')}; background-color: {capacity.color}"
                ></div>
                <div class="capacity-text" style= "color: black">
                    {capacity.label}
                </div>
            </div>
        {/if}
        <div class="stack right time-section">
            <div class="time" class:approaching={isApproaching}>
                {#if timeDisplay === "Approaching"}
                    {timeDisplay}
                {:else}
                    {timeDisplay} MIN
                {/if}
            </div>
            {#if upcomingTimes.length > 0}
                <div>
                    Next bus in {upcomingTimes} min
                </div>
            {/if}
        </div>
    </div>
</div>

<style>
    .bus-entry {
        background-color: transparent;
        border-bottom: 2px solid #e0e0e0;
        min-width: 100%;
        min-height: 80px;
        box-sizing: border-box;
    }

    .bus-entry:last-child {
        border-bottom: none;
    }

    .route {
        font-size: 40px;
    }

    .time {
        font-size: 32px;
    }

    .approaching {
        color: #2ecc71;
        animation: pulse 1.5s ease-in-out infinite;
    }

    .right-cluster {
        display: flex;
        align-items: center;
        gap: 30px;
    }

    .time-section {
        width: 200px;
        align-items: flex-end;
        text-align: right;
        flex-shrink: 0;
    }

    /*capacity bars*/
    .capacity-bar-container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: flex-start;
        position: relative;
        width: 140px;
        height: 24px;
        background-color: #eee;
        border-radius: 4px;
        overflow: hidden;
    }

    .capacity-bar-fill {
        height: 100%;
        transition: width 0.3s ease;
    }

    .capacity-text {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 16px;
        text-transform: uppercase;
        font-weight: bold;
        z-index: 10;
        color: #333;
        text-shadow: 0 0 4px rgba(255,255,255,0.9);
    }
</style>
