<script lang="ts">
  import { stores } from "@sapper/app";
  import { derived } from "svelte/store";
  import type { Readable } from "svelte/store";
  import { onDestroy } from "svelte";

  const { preloading } = stores();
  let ready = false;
  let readyTimeout: number;
  let preloadingDelayTimeout: number;

  // store that lags behind "preloading" store a few ms until the progress bar
  // should be shown. we don't want to show it for short loading times
  const preloadingDelayed = derived(preloading, (value, set) => {
    clearTimeout(preloadingDelayTimeout);

    if (value) {
      preloadingDelayTimeout = setTimeout(() => set(value), 200);
    } else {
      set(value);
    }
  }) as Readable<boolean>;

  const unsub = preloadingDelayed.subscribe((value) => {
    // set initial "ready" value so element is added to DOM
    ready = ready || value;

    clearTimeout(readyTimeout);

    // clear "ready" value after animations are settled, this will remove the
    // element from DOM
    if (value === false) {
      readyTimeout = setTimeout(() => (ready = false), 600 + 200);
    }
  });
  onDestroy(unsub);
</script>

{#if ready}
  <div
    class="loading-progress"
    class:loading={$preloadingDelayed}
    class:finished={!$preloadingDelayed}
    aria-hidden="true"
  />
{/if}

<style>
  .loading-progress {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    height: 2px;
    overflow: hidden;
  }

  .loading-progress::after {
    content: "";
    display: block;
    width: 100%;
    height: 100%;
    background: var(--color-primary-400);
  }

  @keyframes loading {
    from {
      margin-left: -100%;
    }
    to {
      margin-left: -30%;
    }
  }

  @keyframes fade-out {
    from {
      opacity: 1;
    }
    to {
      opacity: 0;
    }
  }

  @keyframes finish {
    from {
      margin-left: -30%;
    }
    to {
      margin-left: -0%;
    }
  }

  .loading-progress.loading::after {
    animation: 10s loading cubic-bezier(0.05, 1.02, 0.25, 1) forwards;
  }

  .loading-progress.finished {
    animation: 600ms fade-out var(--standard-curve) forwards 200ms;
  }

  .loading-progress.finished::after {
    animation: 400ms finish var(--deceleration-curve) forwards;
  }
</style>
