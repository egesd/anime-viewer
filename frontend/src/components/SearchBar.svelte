<script>
  import { FontAwesomeIcon } from '@fortawesome/svelte-fontawesome';
  import { faSearch } from '@fortawesome/free-solid-svg-icons';

  export let searchQuery;
</script>

<div class="mb-4 flex items-center justify-center gap-4">
  <label for="search" class="text-accent2">
    <FontAwesomeIcon icon={faSearch} class="mr-1" /> Search:
  </label>
  <input
    id="search"
    type="text"
    placeholder="Search anime by title..."
    bind:value={searchQuery}
    class="p-2 rounded bg-secondary text-white placeholder-white w-64"
  />
</div>
