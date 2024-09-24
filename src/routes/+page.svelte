<script lang="ts">
  import sheet from "$lib/assets/sheet.jpg";
  import { onMount } from "svelte";

  type Field = {
    name: keyof typeof scoreOptions;
    score: string;
  };

  type Player = {
    name: string;
    fields: Field[];
  };

  const scoreOptions: Record<string, (string | number)[]> = {
    Ones: ["-", "X", 1, 2, 3, 4, 5],
    Twos: ["-", "X", 2, 4, 6, 8, 10],
    Threes: ["-", "X", 3, 6, 9, 12, 15],
    Fours: ["-", "X", 4, 8, 12, 16, 20],
    Fives: ["-", "X", 5, 10, 15, 20, 25],
    Sixes: ["-", "X", 6, 12, 18, 24, 30],
    ThreeOfAKind: [
      "-",
      "X",
      ...Array(30)
        .fill(0)
        .map((_, i) => i + 1),
    ],
    FourOfAKind: [
      "-",
      "X",
      ...Array(30)
        .fill(0)
        .map((_, i) => i + 1),
    ],
    FullHouse: ["-", "X", 25],
    SmallStraight: ["-", "X", 30],
    LargeStraight: ["-", "X", 40],
    Kniffel: ["-", "X", 50],
    Chance: [
      "-",
      "X",
      ...Array(30)
        .fill(0)
        .map((_, i) => i + 1),
    ],
  };

  let players: Player[] = [];

  function createNewPlayer() {
    return {
      name: "",
      fields: Object.keys(scoreOptions).map((key) => ({
        name: key,
        score: scoreOptions[key][0].toString(),
      })),
    };
  }

  function calculateScore(player: Player) {
    const upperScore = calculateSectionScore(player.fields, 0, 6);
    const lowerScore = calculateSectionScore(player.fields, 6, 13);

    const bonus = upperScore >= 63 ? 35 : 0;
    const upperFullScore = upperScore + bonus;
    const totalScore = upperFullScore + lowerScore;

    return {
      upperScore,
      bonus,
      upperFullScore,
      lowerScore,
      totalScore,
    };
  }

  function calculateSectionScore(
    fields: Field[],
    start: number,
    end: number
  ): number {
    return fields.slice(start, end).reduce((total, field) => {
      const score = parseInt(field.score);
      return total + (!isNaN(score) ? score : 0);
    }, 0);
  }

  function initGame() {
    players = [
      ...Array(6)
        .fill(0)
        .map(() => createNewPlayer()),
    ];
    saveGame();
  }

  function resetGame() {
    if (confirm("Every entry will be lost") == true) {
      initGame();
    }
  }

  onMount(() => {
    const savedGame = localStorage.getItem("kniffel");
    if (savedGame) {
      players = JSON.parse(savedGame);
    } else {
      initGame();
    }
  });

  function saveGame() {
    localStorage.setItem("kniffel", JSON.stringify(players));
  }

  let autoCalculate = true;
</script>

{#if players.length > 0}
  <div class="flex ml-5 mb-5 w-full blur-[1px] gap-10 *:whitespace-nowrap">
    <label class="flex gap-3 items-center">
      <input type="checkbox" bind:checked={autoCalculate} class="scale-150" />
      Auto Calculate?
    </label>

    <button
      on:click={resetGame}
      class="disabled:text-neutral-400"
      disabled={players.every(({ name }) => name === "")}>Reset Game</button
    >

    <a href="https://github.com/fjlein/kniffel">See Source ↗</a>
  </div>

  <img
    alt="the kniffel sheet"
    src={sheet}
    class="absolute !w-[800px] !min-w-[800px]"
  />

  <table class="absolute ml-[255px] mt-[5px] text-center blur-[1px]">
    <thead>
      <tr>
        {#each players as player, i}
          <td>
            <input
              type="text"
              bind:value={player.name}
              on:change={() => {
                player.name = player.name.trim();
                saveGame();
              }}
              placeholder="name..."
              autocomplete="off"
              class="w-[87px] h-[47px] text-center placeholder:text-neutral-400"
            />
          </td>
        {/each}
      </tr>
    </thead>
    <tbody
      class="*:*:border-transparent *:*:border-2 *:*:w-[87px] *:*:h-[47px]"
    >
      {#each players[0].fields.slice(0, 6) as field, i}
        <tr>
          {#each players as player}
            <td>
              <select
                bind:value={player.fields[i].score}
                on:change={saveGame}
                class="w-full h-full bg-transparent appearance-none [text-align-last:center] {player.name ==
                ''
                  ? 'hidden'
                  : ''}"
              >
                {#each scoreOptions[field.name] as option}
                  <option value={option}>{option}</option>
                {/each}
              </select>
            </td>
          {/each}
        </tr>
      {/each}
      <tr class="h-[3px]"></tr>
      <tr>
        {#each players as player}
          <td
            class={player.name == "" || !autoCalculate
              ? "text-transparent"
              : ""}
          >
            {calculateScore(player).upperScore}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class={player.name == "" || !autoCalculate
              ? "text-transparent"
              : ""}
          >
            {calculateScore(player).bonus}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class={player.name == "" || !autoCalculate
              ? "text-transparent"
              : ""}
          >
            {calculateScore(player).upperFullScore}</td
          >
        {/each}
      </tr>
      <tr class="h-[24px]"></tr>
      {#each players[0].fields.slice(6, 13) as field, fieldIndex}
        <tr>
          {#each players as player}
            <td>
              <select
                bind:value={player.fields[fieldIndex + 6].score}
                on:change={saveGame}
                class="w-full h-full bg-transparent appearance-none [text-align-last:center] {player.name ==
                ''
                  ? 'hidden'
                  : ''}"
              >
                {#each scoreOptions[field.name] as option}
                  <option value={option}>{option}</option>
                {/each}
              </select>
            </td>
          {/each}
        </tr>
      {/each}
      <tr class="h-[5px]"></tr>
      <tr>
        {#each players as player}
          <td
            class={player.name == "" || !autoCalculate
              ? "text-transparent"
              : ""}
          >
            {calculateScore(player).lowerScore}
          </td>
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class={player.name == "" || !autoCalculate
              ? "text-transparent"
              : ""}
          >
            {calculateScore(player).upperFullScore}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class={player.name == "" || !autoCalculate
              ? "text-transparent"
              : ""}
          >
            {calculateScore(player).totalScore}</td
          >
        {/each}
      </tr>
    </tbody>
  </table>
{/if}
