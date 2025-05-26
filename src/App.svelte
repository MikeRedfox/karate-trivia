<script lang="ts">
  import Footer from "./lib/Footer.svelte";
  const imageModules = import.meta.glob("./assets/*.png", { eager: true });

  function isTechniqueCorrect(technique: string, correctTechnique: string) {
    if (technique == correctTechnique) {
      result = "Bravo! Risposta esatta";
      score++;
      if (score > Number(maxScore)) {
        maxScore = JSON.stringify(score);
      }
    } else {
      result = `No, era ${correctTechnique}`;
    }
  }
  function shuffle<T>(array: T[]): T[] {
    let currentIndex = array.length,
      randomIndex;

    // While there remain elements to shuffle.
    while (currentIndex != 0) {
      // Pick a remaining element.
      randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex--;

      // And swap it with the current element.
      [array[currentIndex], array[randomIndex]] = [
        array[randomIndex],
        array[currentIndex],
      ];
    }

    return array;
  }

  const techniques = Object.keys(imageModules).map((path) => {
    const parts = path.split("/");
    const fileName = parts[parts.length - 1];
    const techniqueName = fileName.replace(".png", "");
    return techniqueName;
  });

  let chosenWaza: string = $state("");
  let wrongTech1: string;
  let wrongTech2: string;
  let techniquesNames: string[] = $state([]);
  let result = $state("");
  let score = $state(0);
  let maxScore = $state(localStorage.getItem("karateTriviaMaxScore"));

  $effect(() => localStorage.setItem("karateTriviaMaxScore", maxScore!));

  function setup() {
    chosenWaza = techniques[Math.floor(Math.random() * techniques.length)];
    wrongTech1 = techniques[Math.floor(Math.random() * techniques.length)];
    wrongTech2 = techniques[Math.floor(Math.random() * techniques.length)];

    while (wrongTech1 == chosenWaza) {
      wrongTech1 = techniques[Math.floor(Math.random() * techniques.length)];
    }
    while (wrongTech2 == chosenWaza || wrongTech1 == wrongTech2) {
      wrongTech2 = techniques[Math.floor(Math.random() * techniques.length)];
    }
    techniquesNames = [wrongTech1, wrongTech2, chosenWaza];
    techniquesNames = shuffle(techniquesNames);
  }
  let openModalId = $state(); // To track which modal is open
  setup();
  function handle(index: number, t: string) {
    openModalId = index;
    isTechniqueCorrect(t, chosenWaza);
    setTimeout(() => {
      openModalId = null;
      setup(); // Move setup here so next question loads after modal closes
    }, 750);
  }
</script>

<div class="grid grid-cols-1 place-items-center gap-4 p-5">
  <h1 class="text-3xl">Indovina la tecnica di karate!</h1>
  <div class="card bg-base-100 w-96 shadow-sm">
    <figure>
      <img src={`./dist/assets/${chosenWaza}.png`} alt="Technique" />
    </figure>
    <div class="card-body">
      <h2 class="card-title">Che tecnica è?</h2>
    </div>
  </div>

  {#each techniquesNames as t, index}
    <div>
      <button class="btn btn-primary" onclick={() => handle(index, t)}
        >{t}</button
      >

      {#if openModalId === index}
        <dialog open class="modal">
          <div class="modal-box">
            <h3 class="text-lg font-bold">
              {result}
            </h3>
            <form method="dialog" class="modal-backdrop">
              <button type="button" onclick={() => (openModalId = null)}
                >Close</button
              >
            </form>
          </div>
        </dialog>
      {/if}
    </div>
  {/each}

  <h2>Punteggio: {score}</h2>
  <h2>High score : {maxScore}</h2>

  <Footer />
</div>
