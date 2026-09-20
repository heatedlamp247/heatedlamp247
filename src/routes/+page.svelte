<script lang="ts">
  import Option from "$lib/components/Option.svelte";
  import { projects } from "$lib/projects";
  import { fade } from "svelte/transition";
  import type { OptionValue } from "$lib/types";
  import { onMount, tick } from "svelte";
  import { Howl } from "howler";
  import { animate } from "animejs";

  import Control from "$lib/components/Control.svelte";
  import SettingsOption from "$lib/components/SettingsOption.svelte";

  const options: OptionValue[] = [
    { name: "ABOUT ME", description: "Learn a bit", rotation: -25, zIndex: 1, offsetX: -60, offsetY: 55, panel: "about"},
    { name: "CONTACT", description: "Where to reach me", rotation: -15, zIndex: 0, offsetX: 0, offsetY: 30, panel: "contact"},
    { name: "PROJECTS", description: "My projects", rotation: -20, zIndex: 1, offsetX: -50, offsetY: 35, panel: "projects"},
    { name: "COMING SOON", description: "Just wait and see...", rotation: -15, zIndex: 2, offsetX: -80, offsetY: 40 },
    { name: "CREDIT", description: "View the people who made this website", rotation: 0, zIndex: 0, offsetX: 0, offsetY: 15, panel: "credits" },
  ];

  const musicTracks = [
    "Color Your Night.mp3",
  ];
  const panelVideoSrc = "/videos/panel.mp4";
  const mainVideoSrc = "/background.mp4";
  
   let activePanel = $state<string | null>(null);
  let backgroundVideo: HTMLVideoElement;
  let isStarted = $state(false);
  let isMusicEnabled = $state(true);
  let isSFXEnabled = $state(true);
  let selectedIndex = $state(0);
  let currentOptionElement = $state<HTMLButtonElement>();
  let settingsOptionElement = $state<HTMLDivElement>();
  let currentSettingIndex = $state(0);
  let isTransitioning = $state(false);
  let loadingPercent = $state(0);
  let currentVideoSrc = $state("/background.mp4");
  let scale = $state(1);
  let showMobileNotice = $state(false);

 let isMobile = $state(false);

function updateScale() {
  const designWidth = 1920;
  scale = Math.min(window.innerWidth / designWidth, 1);
  isMobile = window.innerWidth < 768;
}

  const navigationSound = new Howl({
    src: ["/sfx/navigation.wav"],
    volume: 0.5,
  
  });

  function setIndex(index: number) {
    if (index === selectedIndex) return;
    selectedIndex = index;
    currentOptionElement = document.getElementById(`option-${index}`) as HTMLButtonElement;
    playSound();
  }

  function setSettingsIndex(index: number) {
    if (index === currentSettingIndex) return;
    currentSettingIndex = index;
    // playSound();
    animate(settingsOptionElement!, {
      translateY: index * 56,
      duration: 100,
      easing: "easeOutQuad"
    });
  }

function runTransition(nextPanel: string | null, nextVideo: string) {
  isTransitioning = true;
  loadingPercent = 0;

  const steps = 20;
  const stepTime = 50 / steps;
  let currentStep = 0;

  const interval = setInterval(async () => {
    currentStep++;
    loadingPercent = Math.round((currentStep / steps) * 100);

    if (currentStep === steps) {
      clearInterval(interval);
      currentVideoSrc = nextVideo;

      await tick();
      backgroundVideo?.load();
      backgroundVideo?.play()?.catch(() => {});

      setTimeout(() => {
        activePanel = nextPanel;
        isTransitioning = false;
      }, 150);
    }
  }, stepTime);
}
function confirmSelection() {
  const option = options[selectedIndex];
  if (option.panel) {
    playSound();
    runTransition(option.panel, panelVideoSrc);
  }
}

function closePanel() {
  playSound();
  runTransition(null, mainVideoSrc);
}
  function start() {
    isStarted = true;
    backgroundVideo?.play();

    if (isMusicEnabled) {
      const randomIndex = Math.floor(Math.random() * musicTracks.length);
      new Howl({
        src: `/music/${musicTracks[randomIndex]}`,
        loop: true,
        autoplay: true,
        volume: 0.5,
      });
    }
  }

  function playSound() {
    if (isSFXEnabled) {
      navigationSound.play();
    }
  }

onMount(() => {
  setIndex(0);
  updateScale();
  if (isMobile) showMobileNotice = true;
  window.addEventListener("resize", updateScale);

  document.addEventListener("keydown", (e) => {
    if (!isStarted) return;

    if (e.key === "ArrowDown" || e.key === "s") {
      setIndex((selectedIndex + 1) % options.length);
    } else if (e.key === "ArrowUp" || e.key === "w") {
      setIndex((selectedIndex - 1 + options.length) % options.length);
    } else if (e.key === "Enter" || e.key === " ") {
      if (activePanel) closePanel();
      else confirmSelection();
    } else if (e.key === "Escape") {
      closePanel();
    }
  });
});
</script>

<main class="h-screen w-screen relative overflow-hidden">
  {#if showMobileNotice}
  <div class="fixed inset-0 z-[60] bg-black/90 flex flex-col items-center justify-center gap-6 px-8 text-center">
    <p class="text-white text-lg max-w-md">
      Currently the website is in a barebones state on mobile. Music, SFX, and live background do not work on mobile, and there may be scaling issues. For the best experience use a laptop/pc. 
    </p>
    <button onclick={() => showMobileNotice = false} class="text-xl px-6 py-2 border border-white text-white rounded-md">
      I understand
    </button>
  </div>
{/if}
  {#if isTransitioning}
    <div class="fixed inset-0 z-50 bg-[#0a1030] flex flex-col items-center justify-center gap-4">
      <div class="text-6xl font-bold text-white">{loadingPercent}%</div>
      <div class="text-xl italic text-white/70">Loading...</div>
    </div>
  {/if}
  <div
    class="absolute top-1/2 left-1/2"
    style="width: 1920px; height: 1080px; transform: translate(-50%, -50%) scale({scale});"
  >
  

  {#if !isStarted}
    <div class="fixed bg-bg/90 size-full flex flex-col gap-32 justify-center items-center z-20" transition:fade>
      <div class="rotate-3 space-y-2">
        <h1 class="bg-fg text-bg px-6 py-4 rounded-md text-6xl tracking-[-0.08em]">
          MY PORTFOLIO
        </h1>
        <h2 class="flex items-center w-full gap-2 italic text-xl font-new-rodin text-shadow-under">
          <span>
            Made with Svelte
          </span>
          <hr class="border border-fg grow shadow-under">
            Repurposed by heatedlamp247 (Jared), Original by delta 
        </h2>
      </div>

      <div class="flex flex-col gap-2 relative">
        <SettingsOption onSelect={() => setSettingsIndex(0)} isSelected={currentSettingIndex === 0} bind:value={isMusicEnabled}>
          Toggle Music
        </SettingsOption>
        <SettingsOption onSelect={() => setSettingsIndex(1)} isSelected={currentSettingIndex === 1} bind:value={isSFXEnabled}>
          Toggle SFX
        </SettingsOption>

        <div
          bind:this={settingsOptionElement}
          class="w-[32rem] h-12 bg-red -z-1 absolute -top-2 -right-1 rounded-md"
        ></div>
      </div>

      <button onclick={start} class="text-6xl flex gap-4">
        <span class="tracking-[-0.05em]">ENTER</span>
        <iconify-icon icon="mdi:arrow-right-bold" class=" text-6xl"></iconify-icon>
      </button>
    </div>
  {/if}

  <!-- background -->
  <!-- svelte-ignore a11y_media_has_caption -->
 {#if !isMobile}
  <video
    bind:this={backgroundVideo}
    loop
    src={currentVideoSrc}
    class="absolute inset-0 w-full h-full object-cover object-left -z-10"
  ></video>
{:else}
  <img
    src="/mobile-background.jpg"
    alt=""
    class="absolute inset-0 w-full h-full object-cover object-left -z-10"
  />
{/if}
   {#if !activePanel}
  <!-- options -->
  <div class="3xl:left-[55rem] left-[42rem] flex flex-col items-start justify-center h-full relative -space-y-32">
    {#each options as option, i}
      <Option
        index={i}
        isSelected={selectedIndex === i}
        onSelect={() => setIndex(i)}
        onConfirm={confirmSelection}
        {option}
      />
    {/each}
  </div>
 {/if} 

  <!-- controls -->
  <div class="absolute bottom-0 right-0 font-new-rodin flex flex-col items-start z-10">
    <p class="italic text-3xl pr-20 text-shadow-under">
      {options[selectedIndex].description}
    </p>

    <div class="flex items-center w-full gap-1">
      <span class="text-shadow-under">Command</span>
      <hr class="border border-fg grow shadow-under">
    </div>

    <!-- controls -->
    <div class="flex gap-4 items-center justify-end w-full pr-20 mt-4 mb-6 text-shadow-under">
      <Control key="B">Confirm</Control>
      <Control key="A">Close</Control>
    </div>
  </div>

  <!-- SECTIONS TEXTS -->
  <div class="absolute rotate-90 tracking-[-0.2em] text-muted italic" style="left: -4.5rem; top: -18rem; font-size: 37vh;">
    <span class="z-1">0{selectedIndex + 1}</span>
  </div>

{#if activePanel === "about"}
   <div class="absolute inset-0 flex flex-col items-center justify-center gap-8 z-20" transition:fade>
      <h1 class="text-6xl text-shadow-under">ABOUT ME</h1>
      <div class="border border-fg rounded-md p-8 max-w-2xl w-full text-xl bg-black/90 shadow-2xl">
        Hello! I'm Jared a student currently studying computer engineering and looking
        to get into Infomation Technology, Please feel free to look at the projects I have currently done and feel free to contact me!
      </div>
      <button onclick={closePanel} class="text-3xl flex items-center gap-2 text-shadow-under">
        <iconify-icon icon="mdi:arrow-left-bold"></iconify-icon>
        Back
      </button>
    </div>
 {:else if activePanel === "contact"}
  <div class="absolute inset-0 flex flex-col items-center justify-center gap-8 z-20" transition:fade>
    <h1 class="text-6xl text-shadow-under">CONTACT</h1>
    <div class="flex gap-6 justify-center">
     <a href="https://mail.google.com/mail/?view=cm&fs=1&to=ree616916@gmail.com" target="_blank" class="w-20 h-20 border border-fg rounded-md overflow-hidden flex items-center justify-center bg-black/90 shadow-2xl hover:bg-black/70 transition">
  <img src="/icons/gmail.png" alt="Gmail" class="w-full h-full object-cover" />
</a>
      <a href="https://github.com/heatedlamp247" target="_blank" class="w-20 h-20 border border-fg rounded-md overflow-hidden flex items-center justify-center bg-black/90 shadow-2xl hover:bg-black/70 transition">
        <img src="/icons/github.png" alt="GitHub" class="w-full h-full object-cover" />
      </a>
      <a href="https://www.instagram.com/jaredo__o/" target="_blank" class="w-20 h-20 border border-fg rounded-md overflow-hidden flex items-center justify-center bg-black/90 shadow-2xl hover:bg-black/70 transition">
        <img src="/icons/instagram.png" alt="Instagram" class="w-full h-full object-cover" />
      </a>
    </div>
    <button onclick={closePanel} class="text-3xl flex items-center gap-2 text-shadow-under">
      <iconify-icon icon="mdi:arrow-left-bold"></iconify-icon>
      Back
    </button>
  </div>
{:else if activePanel === "projects"}
  <div class="absolute inset-0 flex flex-col items-center justify-center gap-8 z-20" transition:fade>
    <h1 class="text-6xl text-shadow-under">PROJECTS</h1>
    <div class="flex gap-6 flex-wrap justify-center max-w-3xl">
      {#each projects as project}
        <a href={project.url} target="_blank" class="w-40 h-40 border border-fg rounded-md flex items-center justify-center bg-black/90 shadow-2xl hover:bg-black/70 transition text-xl text-center p-4">
          {project.name}
        </a>
      {/each}
    </div>
    <button onclick={closePanel} class="text-3xl flex items-center gap-2 text-shadow-under">
      <iconify-icon icon="mdi:arrow-left-bold"></iconify-icon>
      Back
    </button>
  </div>
   {:else if activePanel === "credits"}
  <div class="absolute inset-0 flex flex-col items-center justify-center gap-8 z-20" transition:fade>
    <h1 class="text-6xl text-shadow-under">CREDITS</h1>
    <div class="border border-fg rounded-md p-8 max-w-2xl w-full text-xl bg-black/90 shadow-2xl text-center space-y-3">
      <a href="https://github.com/deltea/p3r-pause-menu" target="_blank" class="underline block">Original Creator</a>
      <p>Made with Svelte</p>
      <p>All Art, Font, Music, and Sound effects are made by Atlus/SEGA</p>
      <p>Based on a design by deltea, reprogrammed by Jared</p>
    </div>
    <button onclick={closePanel} class="text-3xl flex items-center gap-2 text-shadow-under">
      <iconify-icon icon="mdi:arrow-left-bold"></iconify-icon>
      Back
    </button>
  </div>
  {/if}
  </div>
</main>