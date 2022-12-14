<script lang="ts">
  import { onMount } from "svelte";
  onMount(() => {
    createArray();
  });

  let array: number[] = [];
  let subArray: number[] = [];
  let searchTarget: number;
  let guessIndex: number;
  let guessCount = 0;
  let hint = "";
  let searching = false;

  let numberRefs: HTMLDivElement[] = [];
  let markerRef: HTMLDivElement;

  $: targetDigits = searchTarget?.toString().length;
  $: guessString = `${guessCount}${
    guessCount < 4
      ? guessCount < 3
        ? guessCount < 2
          ? "st"
          : "nd"
        : "rd"
      : "th"
  }`;

  // Generates an array and resets values from previouse runs
  function createArray() {
    // Resets
    array = [];
    searchTarget = undefined;
    guessIndex = undefined;
    guessCount = 0;
    markerRef.style.display = "none";
    markerRef.style.transition = "unset";

    // Generate array and searchtarget
    const count = getRandomInt(12, 18);
    while (array.length < count) {
      const number = getRandomInt(0, 99);
      if (!array.includes(number)) {
        array.push(number);
      }
    }
    // Array must be sorted
    array.sort((a, b) => a - b);
    // Separate subArray to keep starting values in 'array'
    subArray = array;
  }

  // Recursive binary search algorithm
  async function search(arr: number[] = array) {
    searching = true;

    // Guessing in the middle of the array
    guessIndex = Math.floor((arr.length - 1) / 2);
    guessCount++;

    // Move guess marker
    const guessRef = numberRefs.find(
      (_item, i) => array[i] === subArray[guessIndex]
    );
    markerRef.style.display = "flex";
    markerRef.style.left = `${guessRef.offsetLeft}px`;
    markerRef.style.top = `${guessRef.offsetTop}px`;
    markerRef.style.width = `${guessRef.clientWidth}px`;
    markerRef.style.height = `${guessRef.clientHeight}px`;
    markerRef.style.transition = "all 1s cubic-bezier(0.19, 1, 0.22, 1)";

    // found
    if (searchTarget === arr[guessIndex]) {
      hint = "";
      searching = false;
      return guessIndex;
    }

    // smaller
    if (searchTarget < arr[guessIndex]) {
      hint = "<";
      // Delay before next guess
      await new Promise((r) => setTimeout(r, 1000));
      subArray = [...arr].splice(0, guessIndex);
      return await search(subArray);
    }

    // bigger
    if (searchTarget > arr[guessIndex]) {
      hint = ">";
      // Delay before next guess
      await new Promise((r) => setTimeout(r, 1000));
      subArray = [...arr].splice(guessIndex + 1, arr.length);
      const rec = await search(subArray);
      return rec === -1 ? -1 : guessIndex + rec;
    }

    return -1;
  }

  function onItemClicked(number: number) {
    searchTarget = number;
    search();
  }

  function getRandomInt(min: number, max: number): number {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  // Snippet for using variables in styles
  function cssVariables(node, variables) {
    setCssVariables(node, variables);
    return {
      update(variables) {
        setCssVariables(node, variables);
      },
    };
  }
  function setCssVariables(node, variables) {
    for (const name in variables) {
      node.style.setProperty(`--${name}`, variables[name]);
    }
  }
</script>

<main>
  <h1>Binary search</h1>
  <p>
    A binary search algorithm is a fast way to find an item in a sorted array.
    It works by selecting the item in the middle of the array. If the selected
    item is smaller or bigger than the searched item, the lower or upper half of
    the array is used to repeat the process until the searched item is found.
    <br />
    <br />
    This animated visualization was created as an exercise for trying out Svelte.
    You can view the source code on
    <a href="https://github.com/mzbrandl/binary-search">GitHub</a>.
  </p>
  <div class="card">
    <div class="numbers">
      {#each array as number, i}
        <div
          class={`number${number === searchTarget ? " search-target" : ""}${
            !subArray.includes(number) ? " excluded" : ""
          }`}
          bind:this={numberRefs[i]}
          on:click={() => !searching && !searchTarget && onItemClicked(number)}
          disabled={searching || !!searchTarget}
        >
          <span class="number-text">{number}</span>
          {#if number === subArray[guessIndex] && number === searchTarget}
            <span class="search-target-text" use:cssVariables={{ targetDigits }}
              >Target found!</span
            >
          {:else if number === searchTarget}
            <span class="search-target-text" use:cssVariables={{ targetDigits }}
              >Target</span
            >
          {/if}
        </div>
      {/each}
      <div class="marker" bind:this={markerRef}>
        <span class="hint">{hint}</span>
        <span class="guess-text">{guessString} guess</span>
      </div>
    </div>
    <p class="propmt"><b>Click on an item to search it!</b></p>
    <button on:click={createArray} disabled={searching}
      >Generate sorted array</button
    >
  </div>
</main>

<style>
  .numbers {
    display: flex;
    margin: 10vh 0 8vh 0;
    justify-content: center;
  }

  .number {
    padding: 0.5em 0.75em;
    margin: 0.2em;
    background-color: #333;
    border-radius: 0.5em;
    cursor: pointer;
  }

  .number[disabled="true"] {
    cursor: default !important;
  }
  .number[disabled="false"]:hover {
    outline: solid 2px #646cff;
  }

  .number-text {
    position: relative;
    z-index: 2;
  }

  .search-target {
    outline: solid 2px #646cff;
    border-radius: 0.5em;
  }

  .search-target-text {
    position: absolute;
    margin-top: -3.2em;
    text-align: center;
    transform: translate(calc(-1ch * (var(--targetDigits) * 0.5) - 50%));
  }

  .marker {
    display: none;
    background-color: #646cff;
    border-radius: 0.5em;
    position: absolute;
    transition: all 1s cubic-bezier(0.19, 1, 0.22, 1);
    justify-content: center;
    align-items: center;
  }

  .hint {
    position: absolute;
    margin-top: -6.5em;
    text-align: center;
    width: inherit;
  }

  .guess-text {
    margin-top: 6.5em;
    white-space: nowrap;
  }

  .excluded {
    background-color: #242424;
  }

  .propmt {
    margin-bottom: 5vh;
  }

  @media only screen and (max-width: 600px) {
    h1 {
      margin: 0 !important;
    }
    .numbers {
      flex-direction: column;
      margin-bottom: 1em;
      align-items: center;
    }

    .number {
      width: min-content;
    }

    .search-target-text {
      position: absolute;
      margin-top: unset;
      margin-left: -6em;
    }

    .hint {
      position: absolute;
      margin-top: unset;
      margin-left: -9em;
    }

    .guess-text {
      margin-top: unset;
      margin-right: -9.6em;
      white-space: nowrap;
    }

    button {
      margin-top: 0.5em;
    }
  }
</style>
