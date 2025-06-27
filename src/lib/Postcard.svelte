<script lang="ts">
    import { onMount } from "svelte";

    const maxTilt = 15;
    let flipped = $state(false);
    let el: HTMLElement;
    let cardEl: HTMLElement;
    let rect: DOMRect;

    function updateTilt(e: MouseEvent) {
        const { left, top, width, height } = rect;
        const x = e.clientX - left - width / 2;
        const y = e.clientY - top - height / 2;

        const rotateY = (x / (width / 2)) * maxTilt;
        const rotateX = -(y / (height / 2)) * maxTilt;
        const target = el.querySelector(
            flipped ? ".back" : ".front",
        ) as HTMLElement;

        target.style.transform = `
            rotateX(${rotateX}deg)
            rotateY(${flipped ? 180 + rotateY : rotateY}deg)
            scale(1.05)
        `;
    }

    const handleLeave = () => {
        const target = el.querySelector(
            flipped ? ".back" : ".front",
        ) as HTMLElement;
        console.log(target);
        target.style.transform = `
            rotateX(0deg)
            rotateY(${flipped ? 180 : 0}deg)
            scale(1)
        `;
    };

    const handleClick = () => {
        flipped = !flipped;
        cardEl.classList.toggle("flipped");

        // Disable tilt during flip
        // el.removeEventListener("mousemove", updateTilt, true);
        // setTimeout(() => {
        //     el.addEventListener("mousemove", updateTilt);
        // }, 1000);
    };

    onMount(() => {
        cardEl = el.querySelector(".card")!;
        rect = el.getBoundingClientRect();
        window.addEventListener(
            "resize",
            () => (rect = el.getBoundingClientRect()),
        );
    });
</script>

<div
    bind:this={el}
    class="card-container"
    onclick={handleClick}
    onmousemove={updateTilt}
    onmouseleave={handleLeave}
>
    <div class="card" class:flipped>
        <div class="face front">
            <p>Karstums Parīzē</p>
            <p>1. diena</p>
        </div>
        <div class="face back">
            <img src="/public/image1.jpeg" alt="test" />
        </div>
    </div>
</div>

<style>
    .card-container {
        width: 37.5rem;
        height: 18.75rem;
        perspective: 1000px;
        position: relative;
    }

    .card {
        --rotateX: 0;
        --rotateY: 0;

        display: flex;
        flex-direction: column;
        justify-content: center;
        width: 100%;
        height: 100%;
        padding: 2em;
        border-radius: 0.25rem;
        aspect-ratio: 1 / 1;
        cursor: pointer;
        text-align: center;
        position: relative;
        transform-style: preserve-3d;
        transition: all 500ms ease-out;
        will-change: transform;
        contain: layout;

        &.flipped {
            transform: rotateY(180deg);
        }

        .face {
            display: flex;
            align-items: center;
            justify-content: center;
            position: absolute;
            width: 100%;
            height: 100%;
            left: 0;
            top: 0;
            backface-visibility: hidden;
            border-radius: 0.25em;
            border: 1px solid var(--fg-2);
            box-sizing: border-box;
            transition: transform 0.1s ease-out;
            box-shadow:
                0 20px 20px rgba(0, 0, 0, 0.2),
                0px 0px 50px rgba(0, 0, 0, 0.2);

            img {
                border-radius: 0.25em;
                width: 100%;
                height: 100%;
            }
        }

        .front {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: flex-start;
            padding: 2em;
            z-index: 2;
            font-family: GT-Alpina-Regular;
            font-size: 1.5rem;
            background-color: #fe0100;
            color: #fff;
        }

        .back {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            font-size: 1.125rem;
            font-weight: 700;
            transform: rotateY(180deg);
            background-color: #fff;
            color: #fe0100;
            z-index: 3;

            p {
                align-self: flex-end;
            }
        }

        img {
            transition: all 250ms ease-out;
        }
    }
</style>
