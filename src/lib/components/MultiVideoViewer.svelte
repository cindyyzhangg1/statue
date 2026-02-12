<script lang="ts">
  import { onMount } from 'svelte';
  import VideoTile, { type VideoSource } from './VideoTile.svelte';

  // ---------------------------------------------------------
  // INITIAL DEMO VIDEOS
  // ---------------------------------------------------------
  let videos: VideoSource[] = [
    {
      id: 1,
      label: 'Product demo – main view',
      src: 'https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4',
      kind: 'html5',
      size: 1
    }
  ];

  let nextId = 2;

  // ---------------------------------------------------------
  // FORM STATE
  // ---------------------------------------------------------
  let newLabel = '';
  let newSrc = '';
  let newFile: File | null = null; // 👈 NEW — supports uploads

  // ---------------------------------------------------------
  // REFS
  // ---------------------------------------------------------
  let videoRefs: (HTMLVideoElement | null)[] = [];
  let youtubeIframeRefs: (HTMLIFrameElement | null)[] = [];
  let youtubePlayers: (any | null)[] = [];
  let youtubeApiReady = false;

  // ---------------------------------------------------------
  // LAYOUT LOGIC
  // ---------------------------------------------------------
  let rows = 1;
  let cols = 1;

  $: tileCount = videos.length;

  $: {
    if (tileCount <= 1) {
      rows = 1; cols = 1;
    } else if (tileCount === 2) {
      rows = 1; cols = 2;
    } else if (tileCount <= 4) {
      rows = 2; cols = 2;
    } else {
      cols = Math.ceil(Math.sqrt(tileCount));
      rows = Math.ceil(tileCount / cols);
    }
  }

  // ---------------------------------------------------------
  // YOUTUBE HELPERS
  // ---------------------------------------------------------
  function extractYouTubeId(url: string): string | null {
    try {
      const u = new URL(url);

      if (u.hostname.includes('youtube.com')) {
        return u.searchParams.get('v');
      }
      if (u.hostname === 'youtu.be') {
        const parts = u.pathname.split('/');
        return parts[1];
      }
    } catch {}
    return null;
  }

  function buildYouTubeEmbedUrl(id: string) {
    return `https://www.youtube.com/embed/${id}?enablejsapi=1&rel=0&modestbranding=1`;
  }

  // ---------------------------------------------------------
  // BUILD SOURCE OBJECT
  // ---------------------------------------------------------
  function buildVideoSource(label: string, rawUrl: string, id: number): VideoSource {
    const youtubeId = extractYouTubeId(rawUrl);

    if (youtubeId) {
      const embedUrl = buildYouTubeEmbedUrl(youtubeId);
      return {
        id,
        label,
        src: rawUrl,
        embedUrl,
        kind: 'youtube',
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

  // ---------------------------------------------------------
  // FILE UPLOAD HANDLER
  // ---------------------------------------------------------
  function handleFileChange(event: Event) {
    const input = event.currentTarget as HTMLInputElement;
    const file = input.files?.[0] ?? null;
    newFile = file;

    if (file && !newLabel) {
      newLabel = file.name.replace(/\.[^/.]+$/, '');
    }
  }

  // ---------------------------------------------------------
  // ADD VIDEO (URL or UPLOAD)
  // ---------------------------------------------------------
  function addVideo() {
    const trimmedSrc = newSrc.trim();

    // If no URL *and* no file, skip
    if (!trimmedSrc && !newFile) return;

    // Compute label
    let label = newLabel.trim();
    if (!label) {
      if (newFile) label = newFile.name.replace(/\.[^/.]+$/, '');
      else label = `Video ${nextId}`;
    }

    let video: VideoSource;

    if (newFile) {
      const objectUrl = URL.createObjectURL(newFile);
      video = {
        id: nextId++,
        label,
        src: objectUrl,
        kind: 'html5',
        size: 1,
        isObjectUrl: true
      };
    } else {
      video = buildVideoSource(label, trimmedSrc, nextId++);
    }

    videos = [...videos, video];

    // reset form
    newLabel = '';
    newSrc = '';
    newFile = null;
  }

  // ---------------------------------------------------------
  // REMOVE VIDEO (cleanup blob + player)
  // ---------------------------------------------------------
  function removeVideo(id: number) {
    const idx = videos.findIndex((v) => v.id === id);
    const video = videos[idx];

    if (video?.isObjectUrl) {
      try { URL.revokeObjectURL(video.src); } catch {}
    }

    videos = videos.filter((v) => v.id !== id);

    if (youtubePlayers[idx]) {
      try { youtubePlayers[idx].destroy(); } catch {}
    }

    youtubePlayers.splice(idx, 1);
    youtubeIframeRefs.splice(idx, 1);
    videoRefs.splice(idx, 1);
  }

  // ---------------------------------------------------------
  // UPDATE SIZE
  // ---------------------------------------------------------
  function updateSize(id: number, newSize: number) {
    videos = videos.map((v) => (v.id === id ? { ...v, size: newSize } : v));
  }

  // ---------------------------------------------------------
  // YOUTUBE API LOADING
  // ---------------------------------------------------------
  async function loadYouTubeAPI() {
    if (typeof window === 'undefined') return null;

    if (window.YT && window.YT.Player) return window.YT;

    return new Promise((resolve) => {
      window.onYouTubeIframeAPIReady = () => resolve(window.YT);

      const script = document.createElement('script');
      script.src = 'https://www.youtube.com/iframe_api';
      document.head.appendChild(script);
    });
  }

  function initYouTubePlayers() {
    if (!youtubeApiReady) return;

    const YT = window.YT;
    youtubePlayers = youtubeIframeRefs.map((ref, index) => {
      if (!ref) return youtubePlayers[index] || null;

      try {
        return new YT.Player(ref, {
          events: {
            onReady: () => {
              if (allMuted) youtubePlayers[index]?.mute?.();
            }
          }
        });
      } catch {
        return youtubePlayers[index] || null;
      }
    });
  }

  onMount(async () => {
    const YT = await loadYouTubeAPI();
    if (!YT) return;

    youtubeApiReady = true;
    initYouTubePlayers();
  });

  $: if (youtubeApiReady && youtubeIframeRefs) {
    initYouTubePlayers();
  }

  // ---------------------------------------------------------
  // GLOBAL CONTROLS
  // ---------------------------------------------------------
  let allMuted = false;

  function playAll() {
    videoRefs.forEach((el) => el?.play());
    youtubePlayers.forEach((p) => p?.playVideo?.());
  }

  function pauseAll() {
    videoRefs.forEach((el) => el?.pause());
    youtubePlayers.forEach((p) => p?.pauseVideo?.());
  }

  function toggleMuteAll() {
    allMuted = !allMuted;

    videoRefs.forEach((el) => {
      if (el) el.muted = allMuted;
    });

    youtubePlayers.forEach((p) => {
      try {
        allMuted ? p?.mute?.() : p?.unMute?.();
      } catch {}
    });
  }

  function stopAllAndClear() {
    videoRefs.forEach((el) => {
      if (el) {
        el.pause();
        el.currentTime = 0;
      }
    });

    youtubePlayers.forEach((p) => {
      try {
        p?.pauseVideo?.();
        p?.seekTo?.(0, true);
      } catch {}
    });

    videos.forEach((v) => {
      if (v.isObjectUrl) URL.revokeObjectURL(v.src);
    });

    videos = [];
    youtubeIframeRefs = [];
    youtubePlayers = [];
    videoRefs = [];
  }
</script>

<!--
===========================================================
 UI TEMPLATE
===========================================================
-->
<section class="mx-auto max-w-6xl px-4 py-16 text-white">
  <div class="rounded-[32px] border border-white/10 bg-gradient-to-b from-slate-950/90 via-slate-900/90 to-slate-950/95 p-[1px] shadow-[0_32px_120px_rgba(15,23,42,0.9)]">
    <div class="rounded-[30px] bg-[radial-gradient(circle_at_top,_rgba(148,163,184,0.25),_transparent_55%),radial-gradient(circle_at_bottom,_rgba(45,212,191,0.18),_transparent_55%)] px-5 py-6 md:px-7 md:py-7">
      
      <!-- HEADER -->
      <div class="mb-6 flex flex-col gap-4 md:flex-row md:justify-between md:items-end">
        <div class="space-y-3">
          <p class="inline-flex items-center gap-2 rounded-full bg-slate-950/70 px-3 py-1 text-[0.7rem] font-semibold uppercase tracking-[0.16em] text-emerald-300 ring-1 ring-emerald-400/40">
            <span class="h-1.5 w-1.5 rounded-full bg-emerald-400"></span>
            Multi-video workspace
          </p>

          <h2 class="text-2xl md:text-3xl font-semibold">Watch, compare, and review videos in one view.</h2>
          <p class="max-w-xl text-sm text-slate-200/90">
            Paste a link, upload a video file, or add a YouTube link. Videos arrange into a responsive grid so you can review angles or demos side by side.
          </p>

          <div class="flex gap-3 text-[0.7rem] items-center flex-wrap text-slate-300">
            <span class="inline-flex rounded-full items-center gap-1 bg-slate-950/70 px-3 py-1 ring-1 ring-white/10">
              <span class="h-1.5 w-1.5 rounded-full bg-emerald-400"></span>
              {tileCount} {tileCount === 1 ? 'video active' : 'videos active'}
            </span>

            {#if tileCount > 0}
            <span class="inline-flex rounded-full items-center gap-1 bg-slate-950/70 px-3 py-1 ring-1 ring-white/10">
              Grid:
              <span class="font-mono">{rows}×{cols}</span>
            </span>
            {/if}
          </div>
        </div>

        <!-- ADD VIDEO CARD -->
        <div class="w-full max-w-md rounded-2xl border border-white/15 bg-slate-950/80 p-4 text-xs shadow-xl backdrop-blur-xl text-slate-200">
          <p class="mb-2 text-[0.75rem] font-semibold uppercase tracking-[0.16em] text-slate-100">Add a video source</p>

          <form class="space-y-3" on:submit|preventDefault={addVideo}>
            
            <!-- LABEL -->
            <div class="space-y-1">
              <label class="block text-[0.7rem] uppercase tracking-[0.16em] text-slate-400">Label</label>
              <input
                type="text"
                placeholder="e.g. Newsletter demo"
                bind:value={newLabel}
                class="w-full rounded-xl border border-slate-700 bg-slate-900/80 px-3 py-2 text-xs text-white placeholder:text-slate-500 focus:ring-emerald-400 focus:border-emerald-400"
              />
            </div>

            <!-- URL INPUT -->
            <div class="space-y-1">
              <label class="block text-[0.7rem] uppercase tracking-[0.16em] text-slate-400">Video URL</label>
              <input
                type="text"
                placeholder="https://… · youtu.be/… · /video/demo.mp4"
                bind:value={newSrc}
                class="w-full rounded-xl border border-slate-700 bg-slate-900/80 px-3 py-2 text-xs text-white placeholder:text-slate-500 focus:ring-emerald-400 focus:border-emerald-400"
              />
            </div>

            <!-- OR UPLOAD -->
            <div class="space-y-1">
              <div class="flex gap-2 text-[0.7rem] text-slate-500 items-center">
                <span class="uppercase tracking-[0.16em]">or</span>
                <span>Upload a video</span>
              </div>

              <label class="inline-flex items-center gap-2 cursor-pointer rounded-full border border-slate-700 bg-slate-900/80 px-3 py-1.5 text-[0.7rem] hover:border-emerald-400">
                <input
                  type="file"
                  accept="video/*"
                  class="hidden"
                  on:change={handleFileChange}
                />
                <span>Select file</span>
                {#if newFile}
                <span class="truncate max-w-[120px] text-emerald-300">{newFile.name}</span>
                {/if}
              </label>
            </div>

            <button
              type="submit"
              class="mt-1 inline-flex w-full items-center justify-center gap-1 rounded-xl bg-emerald-500 px-3 py-2 text-xs font-semibold text-slate-950 shadow-md hover:bg-emerald-400 disabled:opacity-40"
              disabled={!newSrc.trim() && !newFile}
            >
              + Add video
            </button>
          </form>
        </div>
      </div>

      <!-- GLOBAL CONTROLS -->
      <div class="mb-5 flex flex-wrap items-center gap-2 text-xs">
        <div class="inline-flex p-1 rounded-full bg-slate-950/80 ring-1 ring-white/10">
          <button on:click={playAll} disabled={tileCount===0} class="px-4 py-1.5 rounded-full hover:bg-slate-800 text-slate-100">▶ Play all</button>
          <button on:click={pauseAll} disabled={tileCount===0} class="px-4 py-1.5 rounded-full hover:bg-slate-800 text-slate-100">⏸ Pause all</button>
          <button on:click={toggleMuteAll} disabled={tileCount===0} class="px-4 py-1.5 rounded-full hover:bg-slate-800 text-slate-100">
            {#if allMuted} 🔊 Unmute all {:else} 🔇 Mute all {/if}
          </button>
        </div>

        <button on:click={stopAllAndClear} disabled={tileCount===0} class="ml-auto bg-red-500/90 text-white px-4 py-1.5 rounded-full shadow-md">✕ Clear all</button>
      </div>

      <!-- GRID -->
      {#if tileCount === 0}
      <div class="rounded-3xl border border-dashed border-slate-700 p-10 bg-slate-950/70 text-center">
        <p class="text-sm text-slate-200">No videos yet. Add a URL or upload a file.</p>
      </div>

      {:else}
      <div
        class="grid gap-4 auto-rows-[minmax(0,1fr)]"
        style={`grid-template-columns: repeat(${cols}, minmax(0, 1fr)); grid-template-rows: repeat(${rows}, minmax(0, 1fr));`}
      >
        {#each videos as video, index (video.id)}
          <VideoTile
            {video}
            size={video.size}
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
