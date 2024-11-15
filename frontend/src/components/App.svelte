<script>
  import { animeData, loading, error, fetchAnimeData } from './animeStore.js';
  import { onMount } from 'svelte';
  import Header from './Header.svelte';
  import MediaFilter from './MediaFilter.svelte';
  import SearchBar from './SearchBar.svelte';
  import SeasonYearSelector from './SeasonYearSelector.svelte';
  import AnimeCard from './AnimeCard.svelte';
  import InfiniteScrollIndicator from './InfiniteScrollIndicator.svelte';
  import SlideIn from './SlideIn.svelte';

  function getCurrentSeason() {
    const month = new Date().getMonth();
    if (month >= 0 && month <= 2) return 'winter';
    if (month >= 3 && month <= 5) return 'spring';
    if (month >= 6 && month <= 8) return 'summer';
    return 'fall';
  }

  let season = getCurrentSeason();
  let year = new Date().getFullYear();
  let mediaFilter = 'tv';
  let searchQuery = '';
  let hoverAnime = null;

  function debounce(func, wait) {
    let timeout;
    return function (...args) {
      clearTimeout(timeout);
      timeout = setTimeout(() => func(...args), wait);
    };
  }

  // Load initial anime data
  fetchAnimeData(season, year, true);

  // Filter anime based on type and search query
  $: filteredAnime = $animeData
    .filter((anime) => anime.node.media_type === mediaFilter)
    .filter((anime) =>
      anime.node.title.toLowerCase().includes(searchQuery.toLowerCase())
    );

  function handleMouseEnter(anime) {
    hoverAnime = anime;
  }

  function handleMouseLeave() {
    hoverAnime = null;
  }

  // Infinite scroll handler with loading check
  function handleScroll() {
    if ($loading) return; // Stop if data is already loading
    if (
      window.innerHeight + window.scrollY >=
      document.body.offsetHeight - 500
    ) {
      fetchAnimeData(season, year);
    }
  }

  const debouncedHandleScroll = debounce(handleScroll, 200);

  onMount(() => {
    window.addEventListener('scroll', debouncedHandleScroll);
    return () => window.removeEventListener('scroll', debouncedHandleScroll);
  });
</script>

<Header />

<main class="p-4 bg-primary min-h-screen text-secondary flex justify-center">
  <div class="w-full max-w-screen-xl">
    <!-- Media Filter -->
    <MediaFilter
      {mediaFilter}
      on:filterChange={(e) => (mediaFilter = e.detail)}
    />

    <!-- Search Bar -->
    <SearchBar bind:searchQuery />

    <!-- Season and Year Selector -->
    <SeasonYearSelector
      {season}
      {year}
      on:seasonChange={(e) => {
        season = e.detail;
        fetchAnimeData(season, year, true);
      }}
      on:yearChange={(e) => {
        year = parseInt(e.detail, 10);
        fetchAnimeData(season, year, true);
      }}
    />

    <!-- Display Spinner when loading and no data is yet available -->
    {#if $loading && !$animeData.length}
      <div class="flex justify-center items-center">
        <InfiniteScrollIndicator />
      </div>
    {/if}

    <!-- Error Message -->
    {#if $error}
      <p class="text-center text-accent1">{$error}</p>
    {/if}

    <!-- Display Anime Cards -->
    {#if filteredAnime && filteredAnime.length}
      <div
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4"
      >
        {#each filteredAnime as anime}
          <SlideIn distance={30} duration={500}>
            <AnimeCard
              {anime}
              on:hoverEnter={handleMouseEnter}
              on:hoverLeave={handleMouseLeave}
            />
          </SlideIn>
        {/each}
      </div>
    {:else if !$loading && !filteredAnime.length}
      <h2 class="text-center text-accent2">No anime found.</h2>
    {/if}

    <!-- Infinite Scroll Loading Indicator -->
    {#if $loading && $animeData.length}
      <div class="flex justify-center mt-4">
        <InfiniteScrollIndicator />
      </div>
    {/if}
  </div>
</main>
