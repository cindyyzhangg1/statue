<script lang="ts">
  export type VideoSource = {
    id: number;
    label: string;
    src: string;
    kind: 'html5' | 'youtube';
    embedUrl?: string;
    size: number;
    isObjectUrl?: boolean; // 👈 NEW: marks uploaded blob URLs
  };

  export let video: VideoSource;
  export let size: number = 1;
  export let showSizeControl: boolean = true;

  export let onRemove: (id: number) => void;
  export let onSizeChange: (id: number, size: number) => void;

  // refs
  export let videoRef: HTMLVideoElement | null = null;
  export let iframeRef: HTMLIFrameElement | null = null;

  function handleSizeChange(event: Event) {
    const input = event.currentTarget as HTMLInputElement;
    const value = parseFloat(input.value);
    onSizeChange(video.id, value);
  }
</script>

<div
  class="relative rounded-3xl border border-white/10 bg-slate-950/60 shadow-lg backdrop-blur-xl overflow-hidden"
  style={`transform: scale(${video.size}); transform-origin: top left;`}
>
  <button
    class="absolute top-3 right-3 z-20 h-7 w-7 rounded-full bg-black/40 text-white text-xs flex items-center justify-center hover:bg-black/70 transition"
    on:click={() => onRemove(video.id)}
  >
    ✕
  </button>

  <div class="p-3 pb-6 text-slate-200 text-xs flex items-center gap-2">
    <span class="inline-flex items-center gap-1 rounded-full bg-slate-900/80 px-2 py-1 ring-1 ring-white/10">
      <span class="h-1.5 w-1.5 rounded-full bg-blue-400"></span>
      {video.kind === 'youtube' ? 'YOUTUBE EMBED' : 'DIRECT VIDEO'}
    </span>
    <span class="font-semibold truncate">{video.label}</span>
  </div>

  <!-- VIDEO CONTENT -->
  {#if video.kind === 'youtube'}
    <iframe
      bind:this={iframeRef}
      class="w-full h-[260px] rounded-b-3xl"
      src={video.embedUrl}
      frameborder="0"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
      allowfullscreen
    />
  {:else}
    <video
      bind:this={videoRef}
      class="w-full rounded-b-3xl"
      src={video.src}
      controls
      playsinline
    ></video>
  {/if}

  <!-- SIZE CONTROL -->
  {#if showSizeControl}
    <div class="px-4 py-3 text-slate-300 text-[0.7rem]">
      <label class="block mb-1 uppercase tracking-widest text-slate-400">Tile Size</label>
      <input
        type="range"
        min="0.6"
        max="1.4"
        step="0.05"
        value={video.size}
        on:input={handleSizeChange}
        class="w-full accent-emerald-400"
      />
    </div>
  {/if}
</div>

<style>
  video::-webkit-media-controls-panel {
    background: rgba(0,0,0,0.25);
  }
</style>
