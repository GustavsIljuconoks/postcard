<script lang="ts">
    import { onMount } from "svelte";

    const SHEET_CSV =
        "https://docs.google.com/spreadsheets/d/e/2PACX-1vT2z-pph9uPd0s3jI8mcgfSmto87TqICkwhvmMVvW1hNJ9UzrCN_Pk7XSorKC-5D4aqYW7n-SuguXDR/pub?gid=0&single=true&output=csv";

    const maxTilt = 15;
    let flipped = $state(false);
    let el: HTMLElement;
    let cardEl: HTMLElement;
    let rect: DOMRect;
    let day = $state("");
    let imageUrl = $state("");
    let frontText = $state("");

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
        target.style.transform = `
            rotateX(0deg)
            rotateY(${flipped ? 180 : 0}deg)
            scale(1)
        `;
    };

    const handleClick = () => {
        flipped = !flipped;
        cardEl.classList.toggle("flipped");
    };

    function csvRowToObj(csvText) {
        const [headerLine, valueLine] = csvText.trim().split("\n");

        // trim quotes and whitespace on every header / value
        const clean = (s) => s.replace(/^"|"$/g, "").trim();

        const headers = headerLine.split(",").map(clean);
        const values = valueLine.split(",").map(clean);

        return Object.fromEntries(headers.map((h, i) => [h, values[i]]));
    }

    async function loadContent() {
        const res = await fetch(SHEET_CSV);
        const csv = await res.text();
        const data = csvRowToObj(csv);

        frontText = data.frontText;
        day = data.day;
        imageUrl = data.imageUrl;
    }

    onMount(async () => {
        cardEl = el.querySelector(".card")!;
        rect = el.getBoundingClientRect();
        window.addEventListener(
            "resize",
            () => (rect = el.getBoundingClientRect()),
        );

        loadContent(); // first load
        const t = setInterval(loadContent, 60_000);
        return () => clearInterval(t);
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
            <p>{frontText}</p>
            <p>{day}</p>
        </div>
        <div class="face back">
            <img src={imageUrl} alt="image from trip" />
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
