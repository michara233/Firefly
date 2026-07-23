<script lang="ts">
  import { onMount } from "svelte";
  import Icon from "@/components/common/Icon.svelte";

  let deferredPrompt: any = null;
  let showButton = false;
  let installed = false;
  let dismissTimeout: ReturnType<typeof setTimeout> | null = null;

  function resetDismissTimer() {
    if (dismissTimeout) clearTimeout(dismissTimeout);
    dismissTimeout = setTimeout(() => {
      showButton = false;
    }, 15_000);
  }

  onMount(() => {
    if (window.matchMedia("(display-mode: standalone)").matches) {
      installed = true;
      return;
    }

    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      deferredPrompt = e;
      showButton = true;
      resetDismissTimer();
    });

    window.addEventListener("appinstalled", () => {
      showButton = false;
      installed = true;
      deferredPrompt = null;
    });

    window.matchMedia("(display-mode: standalone)").addEventListener("change", (e) => {
      if (e.matches) {
        showButton = false;
        installed = true;
      }
    });
  });

  async function handleInstall() {
    if (!deferredPrompt) return;
    deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    console.log(`PWA install: ${outcome}`);
    deferredPrompt = null;
    showButton = false;
    if (dismissTimeout) clearTimeout(dismissTimeout);
  }
</script>

{#if showButton && !installed}
  <div
    class="pwa-install-btn card-base hide"
    class:show={showButton}
    class:hide={!showButton}
  >
    <button
      aria-label="添加到主屏幕"
      title="添加到主屏幕"
      on:click={handleInstall}
    >
      <Icon icon="material-symbols:install-mobile-rounded" size="md" />
    </button>
  </div>
{/if}

<style>
  .pwa-install-btn {
    position: fixed;
    z-index: 999;
    right: 1rem;
    bottom: 8rem;

    width: 3rem;
    height: 3rem;
    border-radius: 1rem;
    overflow: hidden;

    color: var(--primary);
    backdrop-filter: blur(12px);
    background-color: var(--card-bg);
    border: 1px solid rgba(0, 0, 0, 0.1);

    opacity: 1;
    cursor: pointer;
    pointer-events: auto;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

    display: flex;
    align-items: center;
    justify-content: center;
  }

  .pwa-install-btn button {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: none;
    border: none;
    cursor: pointer;
    color: inherit;
    font-size: 1.25rem;
  }

  .pwa-install-btn:hover {
    background-color: var(--btn-card-bg-hover);
    color: var(--primary);
    box-shadow: var(--shadow-button);
  }

  .pwa-install-btn:active {
    transform: scale(0.9);
  }

  .pwa-install-btn.hide {
    transform: scale(0.5);
    opacity: 0;
    pointer-events: none;
    height: 0 !important;
    width: 0 !important;
    margin: 0 !important;
    border-width: 0 !important;
    padding: 0 !important;
  }

  /* 暗色主题 */
  :global(.dark) .pwa-install-btn {
    background-color: var(--card-bg);
    border: 1px solid rgba(255, 255, 255, 0.15);
    color: var(--primary);
  }

  :global(.dark) .pwa-install-btn:hover {
    background-color: var(--btn-card-bg-hover);
    color: var(--primary);
    box-shadow: var(--shadow-button-dark);
  }

  /* 响应式 */
  @media (max-width: 768px) {
    .pwa-install-btn {
      right: 1rem;
      bottom: 8rem;
      width: 3rem;
      height: 3rem;
      border-radius: 0.75rem;
    }
  }

  @media (max-width: 480px) {
    .pwa-install-btn {
      right: 0.5rem;
      bottom: 8rem;
      width: 2.5rem;
      height: 2.5rem;
      border-radius: 0.5rem;
    }
  }

  @media (max-width: 360px) {
    .pwa-install-btn {
      width: 2rem;
      height: 2rem;
      border-radius: 0.375rem;
    }
  }
</style>
