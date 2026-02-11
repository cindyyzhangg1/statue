<script lang="ts">
  export type VideoKind = 'html5' | 'youtube';

  export interface VideoSource {
    id: number;
    label: string;
    src: string;        // original URL
    kind: VideoKind;
    embedUrl?: string;  // for YouTube iframe
    size?: number;      // visual scale (1 = default)
  }

  export let video: VideoSource;
  export let videoRef: HTMLVideoElement | null = null;
  export let iframeRef: HTMLIFrameElement | null = null; // for YouTube API control

  export let onRemove: (id: number) => void;
  export let showSizeControl = false;
  export let size = 1;
  export let onSizeChange: (id: number, newSize: number) => void = () => {};
</script>

<div class="flex h-full flex-col gap-3">
  <!-- Video surface -->
  <div
    class="group relative flex-1 overflow-hidden rounded-3xl border border-white/10 bg-slate-900/80 shadow-[0_18px_60px_rgba(0,0,0,0.65)] backdrop-blur-xl transition-transform duration-200 hover:-translate-y-1"
    style={`--tile-scale: ${size}`}
  >
    <div
      class="pointer-events-none absolute inset-0 bg-[radial-gradient(circle_at_top,_rgba(148,163,184,0.25),_transparent_55%),radial-gradient(circle_at_bottom,_rgba(56,189,248,0.22),_transparent_60%)] opacity-60 mix-blend-screen"
    ></div>

    <!-- Aspect ratio wrapper -->
    <div
      class="relative h-full w-full"
      style="padding-bottom: 56.25%; /* 16:9 */"
    >
      <div
        class="absolute inset-0 flex items-center justify-center"
        style="transform: scale(var(--tile-scale)); transform-origin: center;"
      >
        {#if video.kind === 'youtube' && video.embedUrl}
          <iframe
            bind:this={iframeRef}
            src={video.embedUrl}
            class="h-full w-full rounded-2xl border-0 shadow-inner"
            title={video.label}
            loading="lazy"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            allowfullscreen
          ></iframe>
        {:else}
          <video
            bind:this={videoRef}
            src={video.src}
            class="h-full w-full rounded-2xl bg-black object-contain"
            controls
            playsinline
          ></video>
        {/if}
      </div>
    </div>

    <!-- Overlays -->
    <div class="pointer-events-none absolute inset-0 bg-gradient-to-t from-slate-950/70 via-transparent to-black/55"></div>

    <div class="pointer-events-none absolute inset-x-0 top-0 flex items-start justify-between px-3 pt-3 text-[0.7rem]">
      <div class="flex flex-col gap-1">
        <span class="inline-flex max-w-[72%] items-center gap-1 truncate rounded-full bg-black/70 px-2 py-1 font-medium text-slate-50 ring-1 ring-white/15">
          <span aria-hidden="true">
            {video.kind === 'youtube' ? '▶️' : '🎥'}
          </span>
          <span class="truncate">{video.label}</span>
        </span>

        <span
          class="inline-flex w-fit items-center gap-1 rounded-full bg-slate-900/90 px-2 py-0.5 text-[0.65rem] uppercase tracking-[0.16em] text-slate-300 ring-1 ring-white/10"
        >
          <span
            class="h-1.5 w-1.5 rounded-full {video.kind === 'youtube'
              ? 'bg-red-400'
              : 'bg-emerald-400'}"
          ></span>
          {video.kind === 'youtube' ? 'YouTube embed' : 'Direct video'}
        </span>
      </div>

      <button
        type="button"
        class="pointer-events-auto inline-flex h-7 w-7 items-center justify-center rounded-full bg-black/75 text-slate-200 ring-1 ring-white/20 transition hover:bg-red-500 hover:text-white"
        on:click={() => onRemove(video.id)}
        aria-label="Remove video"
      >
        ✕
      </button>
    </div>

    <div
      class="pointer-events-none absolute bottom-3 left-3 hidden items-center gap-2 rounded-full bg-black/65 px-3 py-1 text-[0.65rem] text-slate-200 ring-1 ring-white/10 group-hover:flex"
    >
      <span class="inline-block h-1.5 w-1.5 rounded-full bg-emerald-400"></span>
      <span>Hover to focus · drag timeline to scrub</span>
    </div>
  </div>

  {#if showSizeControl}
    <div class="space-y-1 text-[0.7rem] text-slate-300">
      <div class="flex items-center justify-between">
        <span class="uppercase tracking-[0.16em] text-slate-400">
          Tile size
        </span>
        <span class="font-mono text-slate-200">{size.toFixed(1)}×</span>
      </div>
      <div class="relative flex items-center gap-3">
        <input
          type="range"
          min="0.5"
          max="1.5"
          step="0.1"
          value={size}
          class="peer w-full accent-emerald-400"
          on:input={(event) => {
            const target = event.currentTarget as HTMLInputElement;
            onSizeChange(video.id, parseFloat(target.value));
          }}
        />
      </div>
    </div>
  {/if}
</div>
