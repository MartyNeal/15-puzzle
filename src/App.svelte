<script>
    import {Confetti} from "svelte-confetti";
    import shortSuccess from "./assets/short-success.mp3";
    import victory from "./assets/victory.mp3";

    let N = $state(2);
    let centiSeconds = $state(0);
    let squares = $state([]);
    let wasShuffled = $state(false);
    let showNext = $state(false);
    let bestSolved = $state(0);
    let isComplete = $derived(squares.every((x, i) => x === "" || x === i + 1));
    let nextUnsolved = $derived.by(() => {
        let count = 0;
        for (let square of squares) {
            if (square !== ++count) {
                return count;
            }
        }
        return count;
    });

    $effect(() => {
        if (nextUnsolved > bestSolved) {
            bestSolved = nextUnsolved;
        }
        if (bestSolved < N * N) {
            successSound.play();
        }
    })

    const successSound = new Audio(shortSuccess);
    const victorySound = new Audio(victory);
    $effect(() => {
        if (isComplete) {
            stop();
            if (wasShuffled) {
                victorySound.play();
            }
        }
    });

    let timer = null;

    function move(i) {
        let neighbors = [];
        if (i < 0 || i >= N * N) return;
        if (i % N !== 0) neighbors.push(i - 1);
        if (i % N !== N - 1) neighbors.push(i + 1);
        if (i >= N) neighbors.push(i - N);
        if (i < N * N - N) neighbors.push(i + N);
        for (let j of neighbors) {
            if (squares[j] === "") {
                squares[j] = squares[i];
                squares[i] = "";
                break;
            }
        }

        if (!timer) {
            timer = setInterval(() => {
                centiSeconds++;
            }, 10);
        }
    }

    function reset() {
        wasShuffled = false;
        squares = Array(N * N).fill().map((_, i) => {
            return ++i;
        });
        squares[N * N - 1] = "";
        stop();
        centiSeconds = 0;
    }

    function shuffle() {
        // set cursor to busy
        const html = document.getElementsByTagName("html")[0];
        html.style.cursor = "wait";

        wasShuffled = true;
        let blank = squares.indexOf("");
        for (let i = 0; i < N * N * N * N; i++) {
            let neighbors = [];
            if (blank % N !== 0) neighbors.push(blank - 1);
            if (blank % N !== N - 1) neighbors.push(blank + 1);
            if (blank >= N) neighbors.push(blank - N);
            if (blank < N * N - N) neighbors.push(blank + N);
            const neighbor = neighbors[Math.floor(Math.random() * neighbors.length)];
            squares[blank] = squares[neighbor];
            squares[neighbor] = "";
            blank = neighbor;
        }
        stop();
        bestSolved = 0;
        centiSeconds = 0;
        html.style.cursor = "";
    }

    function stop() {
        clearInterval(timer);
        timer = null;
    }
    function moveByArrow(e) {
        if (e.key === "ArrowDown") move(squares.indexOf("") + N);
        if (e.key === "ArrowUp") move(squares.indexOf("") - N);
        if (e.key === "ArrowRight") move(squares.indexOf("") + 1);
        if (e.key === "ArrowLeft") move(squares.indexOf("") - 1);
    }

    $effect(() => {
        document.addEventListener("keydown", moveByArrow);
        return () => document.removeEventListener("keydown", moveByArrow);
    });
</script>

<header>
    <h1>15 Puzzle Website Fun</h1>
    <input type="range" bind:value={N} min="2" max="20" step="1" oninput={reset} disabled={!isComplete}/>
    <h2>
        {#if isComplete && wasShuffled}
            Complete!
            <div class="confetti">
                <Confetti x={[-5, 5]} y={[0, 0.1]} delay={[0, 5000]} duration={5000} amount={2000}
                          fallDistance="150vh"/>
            </div>
        {:else}
            &nbsp;
        {/if}
    </h2>
</header>
<main use:reset>
    <div class="board" style="--N: {N}" class:showNext={showNext}>
        {#each squares as square, i}
            {@const x = square % 2}
            {@const z = (~~((square - 1) / N)) % 2}
            <button class="card"
                    class:blue={N % 2 ? x : x === z}
                    class:lightblue={N % 2 ? !x : x !== z}
                    class:black={square === ""}
                    class:unsolved={square === nextUnsolved}
                    onmousedown={() => move(i)}>{square}</button>
        {/each}
    </div>
    <div>
        <label>
            Show next square
            <input type="checkbox" bind:checked={showNext}/>
        </label>
    </div>
    <button onclick={reset}>Reset</button>
    <button onclick={shuffle}>Shuffle</button>
</main>
<div>
    {~~(centiSeconds / 360000)}h
    {(~~(~~(centiSeconds / 100) / 60) % 60).toFixed(0)}m
    {((centiSeconds / 100) % 60).toFixed(2)}s
</div>

<style>
    header {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    main {
        display: flex;
        flex-direction: column;
        place-items: center;
        gap: 1rem;
    }

    .board {
        display: grid;
        width: fit-content;
        grid-template-columns: repeat(var(--N), 1fr);
        gap: 0;
    }

    .card {
        cursor: pointer;
        border: 1px solid #ccc;

        .showNext &.unsolved {
            border: 4px solid red;
        }

        border-radius: 0;
        user-select: none;
        height: 3.5rem;
        width: 3.5rem;
        font-size: 2rem;
        padding: 1rem;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .blue {
        background: mediumblue;
    }

    .lightblue {
        background: royalblue;
    }

    .black {
        background: blue;
    }

    .confetti {
        position: fixed;
        top: -50px;
        left: 0;
        height: 100vh;
        width: 100vw;
        display: flex;
        justify-content: center;
        overflow: hidden;
        pointer-events: none;
    }

    button, input, label {
        cursor: pointer;
    }
</style>
