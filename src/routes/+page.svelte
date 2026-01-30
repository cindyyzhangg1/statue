<script>
	import { Hero, Stats, Categories, LatestContent, BuiltBy } from 'statue-ssg';
	import MusicPlayer from '$lib/components/MusicPlayer.svelte';

	const { data } = $props();

	// MUST match static/audio/*.mp3
	const demoTracks = [
		{
			title: 'Sentimental Loop',
			artist: 'Cindy',
			src: '/audio/sentimental-loop.mp3',
			type: 'audio/mpeg'
		},
		{
			title: 'Ambient Study',
			artist: 'Cindy',
			src: '/audio/ambient-study.mp3',
			type: 'audio/mpeg'
		}
	];
</script>

<svelte:head>
	<title>Cindy’s Statue Site</title>
	<meta
		name="description"
		content="Cindy’s experimental Statue site for learning Statue, Svelte, and preparing for the practical interview."
	/>
</svelte:head>

<div
	class="min-h-screen bg-linear-to-b from-(--color-hero-from) via-(--color-hero-via) to-(--color-hero-to)"
>
	<!-- Hero section -->
	<Hero />

	<!-- Custom Cindy intro -->
	<section class="mx-auto max-w-3xl px-4 pt-10 text-center text-white">
		<h2 class="text-3xl font-semibold mb-3">
			Hi, I’m Cindy 👋
		</h2>
		<p class="text-lg opacity-90 leading-relaxed">
			This is my personal Statue sandbox where I’m getting familiar with Svelte,
			static content pipelines, and building a polished custom component.
		</p>
	</section>

	<!-- Music Player Card -->
	<section class="mx-auto max-w-4xl px-4 pt-16 pb-10">
		<div
			class="relative rounded-3xl border border-white/10 bg-slate-900/60 px-6 py-8 shadow-[0_24px_80px_rgba(15,23,42,0.9)] backdrop-blur-xl"
		>
			<div class="absolute inset-x-0 -top-8 flex justify-center pointer-events-none">
				<div
					class="inline-flex items-center gap-2 rounded-full bg-slate-900/90 px-4 py-1 text-xs font-medium text-slate-200 border border-white/10 shadow-lg"
				>
					<span>🎧</span>
					<span>Lo-fi Player · Built with Svelte + Statue</span>
				</div>
			</div>

			<div class="mb-6 text-center text-white">
				<h2 class="text-2xl font-semibold mb-1">Cindy’s Focus Playlist</h2>
				<p class="text-sm text-slate-300">
					A reusable music component with playlist support, state persistence, and keyboard controls.
				</p>
			</div>

			<MusicPlayer
				tracks={demoTracks}
				storageKey="cindy-home-player"
				autoResume={true}
			/>
		</div>
	</section>

	<section class="typewriter-showcase">
		<BuiltBy />
	</section>

	<div class="stats-wrapper">
		<Stats />
	</div>

	<div class="container mx-auto px-4 py-10">
		<Categories directories={data.directories} />
		<LatestContent rootContent={data.rootContent} />
	</div>
</div>

<style>
	:global(html) {
		scroll-behavior: smooth;
	}

	.stats-wrapper :global(.container) {
		padding-top: 1rem !important;
	}

	.typewriter-showcase {
		max-width: 1200px;
		margin: 0 auto;
		padding: 1rem 2rem 0;
		text-align: center;
		position: relative;
		z-index: 10;
	}
</style>
