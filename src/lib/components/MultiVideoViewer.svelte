<script lang="ts">
  import { onMount } from 'svelte';
  import VideoTile, { type VideoSource } from './VideoTile.svelte';

  // Initial demo videos
  let videos: VideoSource[] = [
    {
      id: 1,
      label: 'Product demo – main view',
      src: 'https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4',
      kind: 'html5',
      size: 1
    }
  ];

  let nextId = 3;

  // Form state
  let newLabel = '';
  let newSrc = '';

  // HTML5 video refs
  let videoRefs: (HTMLVideoElement | null)[] = [];

  // YouTube iframe refs + API players
  let youtubeIframeRefs: (HTMLIFrameElement | null)[] = [];
  let youtubePlayers: (any | null)[] = [];
  let youtubeApiReady = false;

  // Layout
  let rows = 1;
  let cols = 1;

  $: tileCount = videos.length;

  $: {
    if (tileCount <= 1) {
      rows = 1;
      cols = 1;
    } else if (tileCount === 2) {
      rows = 1;
      cols = 2;
    } else if (tileCount <= 4) {
      rows = 2;
      cols = 2;
    } else {
      cols = Math.ceil(Math.sqrt(tileCount));
      rows = Math.ceil(tileCount / cols);
    }
  }

  function extractYouTubeId(url: string): string | null {
    try {
      const u = new URL(url);
      if (u.hostname.includes('youtube.com')) {
        const id = u.searchParams.get('v');
        return id || null;
      }
      if (u.hostname === 'youtu.be') {
        const parts = u.pathname.split('/');
        return parts[1] || null;
      }
    } catch {
      // not a valid URL
    }
    return null;
  }

  function buildYouTubeEmbedUrl(id: string): string {
    // enablejsapi=1 so the JS API can control it
    return `https://www.youtube.com/embed/${id}?enablejsapi=1&rel=0&modestbranding=1`;
  }

  function buildVideoSource(label: string, rawUrl: string, id: number): VideoSource {
    const youtubeId = extractYouTubeId(rawUrl);

    if (youtubeId) {
      const embedUrl = buildYouTubeEmbedUrl(youtubeId);
      return {
        id,
        label,
        src: rawUrl,
        kind: 'youtube',
        embedUrl,
        size: 1
      };
    }

    return {
      id,
      label,
      src: rawUrl,
      kind: 'html5',
      size: 1
    };
  }

  function addVideo() {
    const trimmedSrc = newSrc.trim();
    if (!trimmedSrc) return;

    const label = newLabel.trim() || `Video ${nextId}`;
    const video = buildVideoSource(label, trimmedSrc, nextId++);

    videos = [...videos, video];
    newLabel = '';
    newSrc = '';
  }

  function removeVideo(id: number) {
    const idx = videos.findIndex((v) => v.id === id);
    videos = videos.filter((v) => v.id !== id);

    if (idx !== -1) {
      // clean up matching youtube player
      if (youtubePlayers[idx]) {
        try {
          youtubePlayers[idx]?.destroy?.();
        } catch {
          // ignore
        }
      }
      youtubePlayers.splice(idx, 1);
      youtubeIframeRefs.splice(idx, 1);
      videoRefs.splice(idx, 1);
    }
  }

  function updateSize(id: number, newSize: number) {
    videos = videos.map((v) => (v.id === id ? { ...v, size: newSize } : v));
  }

  let allMuted = false;

  // ---- YouTube API wiring ----

  async function loadYouTubeAPI() {
    if (typeof window === 'undefined') return null;

    if ((window as any).YT && (window as any).YT.Player) {
      return (window as any).YT;
    }

    return new Promise((resolve) => {
      const existing = document.querySelector(
        'script[src="https://www.youtube.com/iframe_api"]'
      );
      if (existing) {
        (window as any).onYouTubeIframeAPIReady = () => {
          resolve((window as any).YT);
        };
        return;
      }

      const tag = document.createElement('script');
      tag.src = 'https://www.youtube.com/iframe_api';
      (window as any).onYouTubeIframeAPIReady = () => {
        resolve((window as any).YT);
      };
      document.head.appendChild(tag);
    });
  }

  function initYouTubePlayers() {
    if (!youtubeApiReady || typeof window === 'undefined') return;
    const YT = (window as any).YT;
    if (!YT || !YT.Player) return;

    youtubePlayers = youtubeIframeRefs.map((iframe, index) => {
      if (!iframe) return youtubePlayers[index] ?? null;
      if (youtubePlayers[index]) return youtubePlayers[index];

      try {
        const player = new YT.Player(iframe, {
          events: {
            onReady: () => {
              if (allMuted) {
                player.mute();
              }
            }
          }
        });
        return player;
      } catch {
        return youtubePlayers[index] ?? null;
      }
    });
  }

  onMount(async () => {
    const YT = await loadYouTubeAPI();
    if (!YT) return;
    youtubeApiReady = true;
    initYouTubePlayers();
  });

  // 👇 KEY FIX: make this reactive to BOTH youtubeApiReady AND youtubeIframeRefs
  $: if (youtubeApiReady && youtubeIframeRefs) {
    initYouTubePlayers();
  }

  // ---- Global controls ----

  function playAll() {
    for (const el of videoRefs) {
      if (el) {
        void el.play();
      }
    }

    if (youtubeApiReady) {
      for (const player of youtubePlayers) {
        try {
          player?.playVideo?.();
        } catch {
          // ignore
        }
      }
    }
  }

  function pauseAll() {
    for (const el of videoRefs) {
      if (el) {
        el.pause();
      }
    }

    if (youtubeApiReady) {
      for (const player of youtubePlayers) {
        try {
          player?.pauseVideo?.();
        } catch {
          // ignore
        }
      }
    }
  }

  function toggleMuteAll() {
    allMuted = !allMuted;

    for (const el of videoRefs) {
      if (el) {
        el.muted = allMuted;
      }
    }

    if (youtubeApiReady) {
      for (const player of youtubePlayers) {
        try {
          if (allMuted) player?.mute?.();
          else player?.unMute?.();
        } catch {
          // ignore
        }
      }
    }
  }

  function stopAllAndClear() {
    for (const el of videoRefs) {
      if (el) {
        el.pause();
        el.currentTime = 0;
      }
    }

    if (youtubeApiReady) {
      for (const player of youtubePlayers) {
        try {
          player?.pauseVideo?.();
          player?.seekTo?.(0, true);
        } catch {
          // ignore
        }
      }
    }

    videos = [];
    youtubePlayers = [];
    youtubeIframeRefs = [];
    videoRefs = [];
  }
</script>

<section class="mx-auto max-w-6xl px-4 py-16 text-white">
  <div class="rounded-[32px] border border-white/10 bg-gradient-to-b from-slate-950/90 via-slate-900/90 to-slate-950/95 p-[1px] shadow-[0_32px_120px_rgba(15,23,42,0.9)]">
    <div class="rounded-[30px] bg-[radial-gradient(circle_at_top,_rgba(148,163,184,0.25),_transparent_55%),radial-gradient(circle_at_bottom,_rgba(45,212,191,0.18),_transparent_55%)] px-5 py-6 md:px-7 md:py-7">
      <!-- Header -->
      <div class="mb-6 flex flex-col gap-4 md:flex-row md:items-end md:justify-between">
        <div class="space-y-3">
          <p class="inline-flex items-center gap-2 rounded-full bg-slate-900/70 px-3 py-1 text-[0.7rem] font-semibold uppercase tracking-[0.16em] text-emerald-300 ring-1 ring-emerald-400/40">
            <span class="inline-block h-1.5 w-1.5 rounded-full bg-emerald-400"></span>
            Multi-video workspace
          </p>
          <div class="space-y-1">
            <h2 class="text-2xl font-semibold md:text-3xl">
              Watch, compare, and review videos in one view.
            </h2>
            <p class="max-w-xl text-sm text-slate-200/90">
              Paste local video paths or full YouTube links. The viewer arranges them into a responsive grid so you can compare different takes, angles, or demos side by side.
            </p>
          </div>

          <div class="flex flex-wrap items-center gap-3 text-[0.7rem] text-slate-300">
            <span class="inline-flex items-center gap-1 rounded-full bg-slate-950/70 px-3 py-1 ring-1 ring-white/10">
              <span class="h-1.5 w-1.5 rounded-full bg-emerald-400"></span>
              {tileCount} {tileCount === 1 ? 'video active' : 'videos active'}
            </span>
            {#if tileCount > 0}
              <span class="inline-flex items-center gap-1 rounded-full bg-slate-950/70 px-3 py-1 ring-1 ring-white/10">
                Grid:
                <span class="font-mono">{rows}×{cols}</span>
              </span>
            {/if}
            <span class="text-slate-400">
              Global controls reach both HTML5 videos and YouTube players.
            </span>
          </div>
        </div>

        <!-- Add video card -->
        <div
          class="w-full max-w-md rounded-2xl border border-white/15 bg-slate-950/80 p-4 text-xs text-slate-200 shadow-xl backdrop-blur-xl"
        >
          <p class="mb-2 text-[0.75rem] font-semibold text-slate-100 uppercase tracking-[0.16em]">
            Add a video source
          </p>
          <form
            class="space-y-3"
            on:submit|preventDefault={addVideo}
          >
            <div class="space-y-1">
              <label class="block text-[0.7rem] uppercase tracking-[0.16em] text-slate-400">
                Label
              </label>
              <input
                type="text"
                placeholder="e.g. Live demo – mobile view"
                bind:value={newLabel}
                class="w-full rounded-xl border border-slate-700 bg-slate-900/80 px-3 py-2 text-xs text-white placeholder:text-slate-500 focus:border-emerald-400 focus:outline-none focus:ring-1 focus:ring-emerald-400"
              />
            </div>
            <div class="space-y-1">
              <label class="block text-[0.7rem] uppercase tracking-[0.16em] text-slate-400">
                Video URL or static path
              </label>
              <input
                type="text"
                placeholder="https://… · youtu.be/… · /video/demo.mp4"
                bind:value={newSrc}
                class="w-full rounded-xl border border-slate-700 bg-slate-900/80 px-3 py-2 text-xs text-white placeholder:text-slate-500 focus:border-emerald-400 focus:outline-none focus:ring-1 focus:ring-emerald-400"
              />
            </div>
            <p class="text-[0.65rem] text-slate-400">
              Works with direct video files and YouTube links. For best performance on this Statue site, use
              <code class="font-mono text-emerald-300">/video/*.mp4</code> from your static assets.
            </p>
            <button
              type="submit"
              class="mt-1 inline-flex w-full items-center justify-center gap-1 rounded-xl bg-emerald-500 px-3 py-2 text-xs font-semibold text-slate-950 shadow-md shadow-emerald-500/30 transition hover:bg-emerald-400 active:bg-emerald-600 disabled:opacity-40"
              disabled={!newSrc.trim()}
            >
              + Add video
            </button>
          </form>
        </div>
      </div>

      <!-- Global controls -->
      <div class="mb-5 flex flex-wrap items-center gap-2 text-xs">
        <div class="inline-flex rounded-full bg-slate-950/80 p-1 ring-1 ring-white/10">
          <button
            type="button"
            class="rounded-full px-4 py-1.5 font-medium text-slate-100 transition hover:bg-slate-800/90 disabled:opacity-40"
            on:click={playAll}
            disabled={tileCount === 0}
          >
            ▶ Play all
          </button>
          <button
            type="button"
            class="rounded-full px-4 py-1.5 font-medium text-slate-100 transition hover:bg-slate-800/90 disabled:opacity-40"
            on:click={pauseAll}
            disabled={tileCount === 0}
          >
            ⏸ Pause all
          </button>
          <button
            type="button"
            class="rounded-full px-4 py-1.5 font-medium text-slate-100 transition hover:bg-slate-800/90 disabled:opacity-40"
            on:click={toggleMuteAll}
            disabled={tileCount === 0}
          >
            {#if allMuted}
              🔊 Unmute all
            {:else}
              🔇 Mute all
            {/if}
          </button>
        </div>

        <button
          type="button"
          class="ml-auto inline-flex items-center rounded-full bg-red-500/90 px-4 py-1.5 text-xs font-semibold text-white shadow-md shadow-red-500/40 transition hover:bg-red-400 disabled:opacity-40"
          on:click={stopAllAndClear}
          disabled={tileCount === 0}
        >
          ✕ Clear all
        </button>
      </div>

      <!-- Grid / empty state -->
      {#if tileCount === 0}
        <div class="rounded-3xl border border-dashed border-slate-700 bg-slate-950/70 p-10 text-center">
          <p class="text-sm text-slate-200">
            No videos yet. Add a URL or static file path above to start a multi-view session.
          </p>
          <p class="mt-2 text-[0.7rem] text-slate-400">
            Tip: mix a local <code class="font-mono text-emerald-300">/video/*.mp4</code> with a YouTube link to compare different versions.
          </p>
        </div>
      {:else}
        <div
          class="grid gap-4 auto-rows-[minmax(0,1fr)]"
          style={`grid-template-columns: repeat(${cols}, minmax(0, 1fr)); grid-template-rows: repeat(${rows}, minmax(0, 1fr));`}
        >
          {#each videos as video, index (video.id)}
            <VideoTile
              {video}
              size={video.size ?? 1}
              showSizeControl={true}
              onRemove={removeVideo}
              onSizeChange={updateSize}
              bind:videoRef={videoRefs[index]}
              bind:iframeRef={youtubeIframeRefs[index]}
            />
          {/each}
        </div>
      {/if}
    </div>
  </div>
</section>
