<script>
    import { onMount } from "svelte";

    function r(scale) {
        return Math.round(Math.random() * scale);
    }

    function spawn(href) {
        const image = { href, score: 0 };
        return respawn(image);
    }

    function respawn(image) {
        image.x = r(wx - size);
        image.y = r(wy - size);
        image.dx = r(4) + 1;
        image.dy = r(4) + 1;
        image.killed = false;
        return image;
    }

    const shapes = ["critic_1.jpeg", "critic_2.jpeg", "critic_3.jpeg"];
    let images = [];

    const size = 150;

    function kill(image) {
        image.killed = true;
        image.score += 1;
    }

    function move() {
        for (let image of images) {
            if (image.x + size > wx || image.x < 0) image.dx = -image.dx;
            if (image.y + size > wy || image.y < 0) image.dy = -image.dy;
            image.x += image.dx;
            image.y += image.dy;
            const gone = image.ref.getBoundingClientRect().width < 10;
            if (gone) {
                respawn(image);
            }
        }
        images = [...images];
        requestAnimationFrame(move);
    }
    requestAnimationFrame(move);

    function resize() {
        for (let image of images) {
            image.x = Math.min(image.x, wx - size);
            image.y = Math.min(image.y, wy - size);
        }
        height = window.innerHeight;
    }

    onMount(() => {
        resize();
        images = shapes.map(spawn);
    });

    let wx = 0;
    let wy = 0;

    $: height = window.innerHeight;
</script>

<svelte:window on:resize={resize} />

<div class="info">
    <p>{wx}x{wy}</p>
    {#each images as image}
        <p>{image.killed}: {image.x}x{image.y}</p>
    {/each}
</div>

<div class="scores">
    {#each images as image}
        <span class="scores-circle">
            <!-- svelte-ignore a11y-missing-attribute -->
            <img src={image.href} class="rounded" />
        </span>
        <span class="scores-count">{image.score}</span>
    {/each}
</div>

<div style:height="{height}px">
    <div
        style:width="100%"
        style:height="100%"
        style:overflow="clip"
        bind:clientWidth={wx}
        bind:clientHeight={wy}
    >
        {#each images as image}
            <!-- svelte-ignore a11y-click-events-have-key-events -->
            <div
                class="circle"
                class:killed={image.killed}
                on:mousedown={() => kill(image)}
                style:translate="{image.x}px {image.y}px"
                bind:this={image.ref}
            >
                <img src={image.href} alt="" draggable="false" />
            </div>
        {/each}
    </div>
</div>

<style>
    .circle {
        width: 150px;
        height: 150px;
        position: absolute;
        overflow: clip;
        border-radius: 50%;
        border: 1px solid #000;
        animation: spin 8s linear infinite;
    }
    .killed {
        animation: killed 1s linear;
    }
    img {
        display: inline;
        margin: 0 auto;
        height: 100%;
        width: auto;
    }
    @keyframes spin {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }
    @keyframes killed {
        0% {
            transform: scale(1);
        }
        100% {
            transform: scale(0);
        }
    }
    .info {
        display: none;
        position: fixed;
        top: 0;
        left: 0;
    }
    .scores {
        position: fixed;
        bottom: 0;
        left: 50%;
        width: 100dvw;
        text-align: center;
        transform: translateX(-50%);
        vertical-align: middle;
    }
    .scores-circle {
        height: 50px;
        width: auto;
        display: inline-block;
        overflow: hidden;
        border-radius: 50%;
        border: 1px solid #000;
    }
    .scores-count {
        display: inline-block;
        padding: 0;
        margin: 0;
        font-size: 50px;
        line-height: 50px;
        vertical-align: top;
    }
</style>
