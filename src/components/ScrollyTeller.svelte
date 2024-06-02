<script>
  import { onMount } from "svelte";
  import Police from "./Police.svelte";
  import Police2 from "./Police2.svelte";

  // import { currentPage } from "./store.js"; // import the store

  let sections, currentPage;
  // let data = [];
  // onMount(async () => {
  //   data = await d3.csv('temp_all_pop.csv', d3.autoType);
  // });

  onMount(() => {
    sections = document.querySelectorAll("section");
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const index = Array.from(sections).indexOf(entry.target);
            currentPage = index; // update the store
            console.log(`Current visible section: ${index}`);
          }
        });
      },
      {
        root: null,
        rootMargin: "0px",
        threshold: 0.5,
      }
    );

    sections.forEach((section) => observer.observe(section));

    return () => {
      sections.forEach((section) => observer.unobserve(section));
    };
  });
</script>

<div class="foreground">
  {#each Array(6) as _, index}
    <section>This is section {index + 1}</section>

    <!-- add visulation page to each section below -->
    {#if index == 0}
      <Police />
      <!-- <Police2 {data} /> -->
    {/if}

    
  {/each}
</div>

<style>
  section {
    height: 100vh;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .foreground{
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
      'Helvetica Neue', Arial, sans-serif;
    max-width: 1000px;
    margin: 1em auto;
    padding: 1em;
  }
</style>
