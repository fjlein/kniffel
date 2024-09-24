<script lang="ts">
  import card from "$lib/assets/card.jpg";
  import { onMount } from "svelte";

  type Field = { name: string; score: string };

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

  function createFields(): Field[] {
    return Object.keys(scoreOptions).map((key) => ({
      name: key,
      score: scoreOptions[key][0].toString(),
    }));
  }

  function getPlayer() {
    return { name: "", fields: createFields() };
  }

  let players: Player[] = [];

  function calculateScore(player: Player) {
    let upperScore = 0;
    let lowerScore = 0;

    for (let i = 0; i < 6; i++) {
      const score = player.fields[i].score;
      if (score !== "-" && score !== "X") {
        upperScore += parseInt(score);
      }
    }

    const bonus = upperScore >= 63 ? 35 : 0;
    const upperFullScore = upperScore + bonus;

    for (let i = 6; i < player.fields.length; i++) {
      const score = player.fields[i].score;
      if (score !== "-" && score !== "X") {
        lowerScore += parseInt(score);
      }
    }

    const totalScore = upperFullScore + lowerScore;
    return {
      upperScore,
      bonus,
      upperFullScore,
      lowerScore,
      totalScore,
    };
  }

  function resetGame() {
    if (confirm("Every entry will be lost") == true) {
      players = [
        ...Array(6)
          .fill(0)
          .map(() => getPlayer()),
      ];
      localStorage.setItem("kniffel", "");
      saveGame();
    }
  }

  onMount(() => {
    const savedGame = localStorage.getItem("kniffel");
    if (savedGame) {
      players = JSON.parse(savedGame);
    } else {
      resetGame();
    }
  });

  function saveGame() {
    localStorage.setItem("kniffel", JSON.stringify(players));
  }

  let autoCalculate = true;
</script>

{#if players.length > 0}
  <div class="flex ml-5 w-full blur-[1px] gap-5 text-2xl">
    <label class="flex gap-3 items-center">
      <input type="checkbox" bind:checked={autoCalculate} class="scale-150" />
      Auto Calculate
    </label>

    <button
      on:click={resetGame}
      class="disabled:text-neutral-500"
      disabled={players.every((player) => player.name === "")}
      >Reset Game</button
    >
  </div>

  <img
    alt="The project logo"
    src={card}
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
                saveGame();
                player.name.trim();
              }}
              placeholder={`${i + 1}...`}
              autocomplete="off"
              class="w-[87px] h-[47px] text-center"
            />
          </td>
        {/each}
      </tr>
    </thead>
    <tbody class="*:*:border-transparent *:*:border-2">
      {#each players[0].fields.slice(0, 6) as field, i}
        <tr>
          {#each players as player}
            <td class="w-[87px] h-[47px]">
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
            class="w-[87px] h-[47px] {player.name == ''
              ? 'text-transparent'
              : ''}"
            >{autoCalculate ? calculateScore(player).upperScore : ""}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class="w-[87px] h-[47px] {player.name == ''
              ? 'text-transparent'
              : ''}">{autoCalculate ? calculateScore(player).bonus : ""}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class="w-[87px] h-[47px] {player.name == ''
              ? 'text-transparent'
              : ''}"
            >{autoCalculate ? calculateScore(player).upperFullScore : ""}</td
          >
        {/each}
      </tr>
      <tr class="h-[24px]"></tr>
      {#each players[0].fields.slice(6, 13) as field, fieldIndex}
        <tr>
          {#each players as player}
            <td class="w-[87px] h-[47px]">
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
            class="w-[87px] h-[47px] {player.name == ''
              ? 'text-transparent'
              : ''}"
            >{autoCalculate ? calculateScore(player).lowerScore : ""}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class="w-[87px] h-[47px] {player.name == ''
              ? 'text-transparent'
              : ''}"
            >{autoCalculate ? calculateScore(player).upperFullScore : ""}</td
          >
        {/each}
      </tr>
      <tr>
        {#each players as player}
          <td
            class="w-[87px] h-[47px] {player.name == ''
              ? 'text-transparent'
              : ''}"
            >{autoCalculate ? calculateScore(player).totalScore : ""}</td
          >
        {/each}
      </tr>
    </tbody>
  </table>
{/if}
