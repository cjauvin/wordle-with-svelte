<script lang="ts">
  import { onMount } from "svelte";

  let words = [];
  let word = null;
  let rows = [...Array(6)].map((e) => Array(5).fill(null));
  let i = 0;
  let j = 0;
  let done = false;

  onMount(async () => {
    const res = await fetch(
      "https://raw.githubusercontent.com/tabatkins/wordle-list/main/words"
    );
    const text = await res.text();
    words = text.split(/\r?\n/);
    word = words[Math.floor(Math.random() * words.length)];
    console.log(word);
    window.addEventListener("keydown", handleKeyDown);
  });

  function testGuess(guess, word) {
    let statuses = [];
    for (let i = 0; i < 5; i++) {
      if (guess[i].letter === word[i]) {
        statuses.push("yes");
      } else if (word.includes(guess[i].letter)) {
        statuses.push("present");
      } else {
        statuses.push("no");
      }
    }
    return statuses;
  }

  function handleKeyDown(e) {
    if (done) {
      return;
    }

    // Submit guess
    if (e.key === "Enter") {
      if (j < 5) {
        return;
      }
      let statuses = testGuess(rows[i], word.toUpperCase());

      rows[i] = rows[i].map((cell, j) => ({
        letter: cell.letter,
        status: statuses[j],
      }));

      if (statuses.every((v) => v === "yes")) {
        done = true;
        console.log("you won!");
      } else if (i < 5) {
        i = i + 1;
        j = 0;
      } else {
        console.log("you lost..");
      }

      // Backspace
    } else if (e.key === "Backspace") {
      if (j === 0) {
        return;
      }

      rows[i][j - 1] = null;

      j -= 1;

      // Enter letter
    } else {
      if (j > 4 || e.key < "a" || e.key > "z") {
        return;
      }

      rows[i][j] = {
        letter: e.key.toUpperCase(),
        status: "",
      };

      j += 1;
    }
  }

  function handleRestartButton(e) {
    e.target.blur();
    rows = [...Array(6)].map((e) => Array(5).fill(null));
    i = 0;
    j = 0;
    done = false;
    word = words[Math.floor(Math.random() * words.length)];
    console.log(word);
  }
</script>

<div class="frame">
  <div class="board">
    {#each rows as row}
      <div class="row">
        {#each row as cell}
          <div class="cell {cell ? cell.status : ''}">
            {cell ? cell.letter : ""}
          </div>
        {/each}
      </div>
    {/each}
  </div>

  <button on:click={handleRestartButton}>Restart</button>
</div>
