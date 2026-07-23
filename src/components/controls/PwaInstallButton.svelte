<script lang="ts">
  import { onMount } from "svelte";

  let deferredPrompt: any = null;
  let showButton = false;
  let installed = false;
  let dismissTimeout: ReturnType<typeof setTimeout> | null = null;

  // Auto-dismiss after 15s if user doesn't click
  function resetDismissTimer() {
    if (dismissTimeout) clearTimeout(dismissTimeout);
    dismissTimeout = setTimeout(() => {
      showButton = false;
    }, 15_000);
  }

  onMount(() => {
    // Check if already in standalone mode (installed)
    if (window.matchMedia("(display-mode: standalone)").matches) {
      installed = true;
      return;
    }

    // Listen for install prompt
    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      deferredPrompt = e;
      showButton = true;
      resetDismissTimer();
    });

    // Hide button once installed
    window.addEventListener("appinstalled", () => {
      showButton = false;
      installed = true;
      deferredPrompt = null;
    });

    // Hide if display mode changes to standalone
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
  <button
    class="pwa-install-btn"
    on:click={handleInstall}
    aria-label="添加到主屏幕"
    title="添加到主屏幕"
  >
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      <path d="M12 5v14" />
      <path d="M5 12h14" />
      <rect x="3" y="3" width="18" height="18" rx="4" />
    </svg>
    <span class="btn-text">添加到主屏幕</span>
  </button>
{/if}

<style>
  .pwa-install-btn {
    position: fixed;
    bottom: 1rem;
    left: 50%;
    transform: translateX(-50%);
    z-index: 9999;

    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.625rem 1.25rem;

    background: #2563eb;
    color: #fff;
    border: none;
    border-radius: 2rem;
    font-size: 0.875rem;
    font-weight: 600;
    cursor: pointer;
    box-shadow: 0 4px 16px rgba(37, 99, 235, 0.4);
    animation: pwa-slide-up 0.4s ease-out;

    /* Prevent Swup from interfering */
    user-select: none;
    -webkit-tap-highlight-color: transparent;
    touch-action: manipulation;
  }

  .pwa-install-btn:active {
    transform: translateX(-50%) scale(0.97);
    background: #1d4ed8;
  }

  @keyframes pwa-slide-up {
    from {
      opacity: 0;
      transform: translateX(-50%) translateY(1rem);
    }
    to {
      opacity: 1;
      transform: translateX(-50%) translateY(0);
    }
  }

  /* 移动端适配 */
  @media (max-width: 768px) {
    .pwa-install-btn {
      bottom: max(1rem, env(safe-area-inset-bottom, 1rem));
      padding: 0.75rem 1.5rem;
      font-size: 0.9375rem;
    }
    .btn-text {
      display: inline;
    }
  }

  @media (max-width: 480px) {
    .pwa-install-btn {
      left: 1rem;
      right: 1rem;
      transform: none;
      justify-content: center;
      border-radius: 0.75rem;
    }
    .pwa-install-btn:active {
      transform: scale(0.97);
    }
  }
</style>
