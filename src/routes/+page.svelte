<script>
	import { Hero, Stats, Categories, LatestContent, BuiltBy } from 'statue-ssg';
	import MusicPlayer from '$lib/components/MusicPlayer.svelte';
	import MultiVideoViewer from '$lib/components/MultiVideoViewer.svelte';

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
	class="min-h-screen bg-linear-to-b from-(--color-hero-from) via-(--color-hero-via) to-(--color-hero-to) text-white"
>
	<!-- Hero section -->
	<Hero />

	<!-- Main content stack -->
	<main class="mx-auto flex max-w-6xl flex-col gap-14 px-4 pt-10 pb-16">
		<!-- Custom Cindy intro -->
		<section class="text-center md:text-left">
			<div class="mb-4 flex flex-wrap items-center justify-center gap-2 md:justify-start">
				<span
					class="inline-flex items-center gap-2 rounded-full bg-slate-900/80 px-3 py-1 text-[0.7rem] font-semibold uppercase tracking-[0.16em] text-emerald-300 ring-1 ring-emerald-400/40"
				>
					<span class="inline-block h-1.5 w-1.5 rounded-full bg-emerald-400" />
					Cindy’s Statue sandbox
				</span>
				<span class="text-xs text-slate-300">
					Svelte · Static content · Custom media components
				</span>
			</div>

			<h2 class="mb-3 text-3xl font-semibold md:text-4xl">
				Hi, I’m Cindy 👋
			</h2>
			<p class="mx-auto max-w-3xl text-base leading-relaxed text-slate-200 md:text-lg">
				This is my personal Statue sandbox where I’m getting familiar with Svelte,
				static content pipelines, and building polished, reusable components that
				feel like real product features.
			</p>
		</section>

		<!-- Music Player Card -->
		<section class="mx-auto w-full max-w-4xl">
			<div
				class="relative rounded-3xl border border-white/10 bg-slate-900/70 px-6 py-8 shadow-[0_24px_80px_rgba(15,23,42,0.9)] backdrop-blur-xl"
			>
				<div class="absolute inset-x-0 -top-8 flex justify-center pointer-events-none">
					<div
						class="inline-flex items-center gap-2 rounded-full bg-slate-900/90 px-4 py-1 text-[0.7rem] font-semibold uppercase tracking-[0.16em] text-slate-200 border border-white/10 shadow-lg"
					>
						<span class="inline-block h-1.5 w-1.5 rounded-full bg-violet-400" />
						Featured component · Lo-fi player
					</div>
				</div>

				<div class="mb-6 text-center md:text-left">
					<h3 class="mb-1 text-2xl font-semibold">Cindy’s Focus Playlist</h3>
					<p class="text-sm text-slate-300">
						A reusable audio component with playlist support, state persistence, and keyboard
						controls, wired directly into Statue’s static asset pipeline.
					</p>
				</div>

				<MusicPlayer
					tracks={demoTracks}
					storageKey="cindy-home-player"
					autoResume={true}
				/>
			</div>
		</section>

		<!-- Multi-video viewer section (stacked below music) -->
		<section class="w-full">
			<header class="mx-auto mb-6 max-w-3xl text-center">
				<p class="mb-2 text-[0.7rem] font-semibold uppercase tracking-[0.16em] text-emerald-300">
					Multi-video workspace
				</p>
				<h3 class="mb-2 text-2xl font-semibold md:text-3xl">
					Multi-Video Review Studio
				</h3>
				<p class="text-sm text-slate-300">
					Add local <code class="font-mono text-emerald-300">/video/*.mp4</code> files or paste
					YouTube links. The viewer automatically arranges tiles into a responsive grid as you
					add or remove videos, so you can compare demos, recordings, or alternate takes in one place.
				</p>
			</header>

			<!-- Full-width viewer, but constrained to the content width -->
			<div class="mx-auto max-w-6xl">
				<MultiVideoViewer />
			</div>
		</section>

		<!-- Statue attribution + stats -->
		<section class="space-y-6">
			<div class="typewriter-showcase">
				<BuiltBy />
			</div>
			<div class="stats-wrapper">
				<Stats />
			</div>
		</section>

		<!-- Content section -->
		<section class="space-y-6">
			<header class="text-center md:text-left">
				<h3 class="text-lg font-semibold uppercase tracking-[0.16em] text-slate-300">
					Content
				</h3>
				<p class="mt-1 max-w-xl text-sm text-slate-300">
					All of the components above sit on top of Statue’s content system. Browse categories
					and the latest posts generated from Markdown.
				</p>
			</header>

			<div class="container mx-auto px-0 py-4">
				<Categories directories={data.directories} />
				<LatestContent rootContent={data.rootContent} />
			</div>
		</section>
	</main>
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
		padding: 1rem 0 0;
		text-align: center;
		position: relative;
		z-index: 10;
	}
</style>
