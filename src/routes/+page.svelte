<script>
	import { onMount } from 'svelte';

	const THEMES = [
		{ key: 'light', name: 'Light' },
		{ key: 'dark', name: 'Dark' },
		{ key: 'ocean', name: 'Ocean' },
		{ key: 'forest', name: 'Forest' },
		{ key: 'sunset', name: 'Sunset' }
	];

	let links = $state([]);
	// Read from the DOM, not storage — app.html already resolved it, and starting
	// anywhere else reintroduces the flash.
	let currentTheme = $state(document.documentElement.dataset.theme ?? 'dark');
	let loaded = $state(false);

	$effect(() => {
		document.documentElement.dataset.theme = currentTheme;
	});

	onMount(async () => {
		try {
			links = await (await fetch('/links.json')).json();
		} catch {
			// leave links empty — the empty state explains what to do
		}
		loaded = true;
	});

	function setTheme(name) {
		currentTheme = name;
		// Throws when site data is blocked; the theme still applies, just isn't kept.
		try {
			localStorage.setItem('linkboard-theme', name);
		} catch {}
	}

	function getDomain(url) {
		try {
			return new URL(url).hostname.replace('www.', '');
		} catch {
			return url;
		}
	}

	function getFavicon(url) {
		try {
			const domain = new URL(url).hostname;
			return `https://www.google.com/s2/favicons?domain=${domain}&sz=32`;
		} catch {
			return '';
		}
	}
</script>

<svelte:head>
	<title>Ganesh's LinkBoard</title>
</svelte:head>

<div class="app">
	<header>
		<div class="header-content">
			<h1>Ganesh's Linkboard</h1>
			<p class="subtitle">{links.length} bookmarked links</p>
		</div>
		<div class="theme-switcher" role="group" aria-label="Colour theme">
			{#each THEMES as theme}
				<button
					class="theme-btn"
					class:active={currentTheme === theme.key}
					aria-pressed={currentTheme === theme.key}
					onclick={() => setTheme(theme.key)}
				>
					{theme.name}
				</button>
			{/each}
		</div>
	</header>

	<main>
		{#if loaded}
			{#if links.length === 0}
				<div class="empty">
					<p>No links found. Add some to <code>static/links.json</code></p>
				</div>
			{:else}
				<div class="links-grid">
					{#each links as item}
						<a href={item.link} target="_blank" rel="noopener noreferrer" class="link-card">
							<div class="link-icon">
								<img
									src={getFavicon(item.link)}
									alt=""
									width="24"
									height="24"
									onerror={(e) => e.target.style.display = 'none'}
								/>
							</div>
							<div class="link-info">
								<h2>{item.title}</h2>
								<span class="link-domain">{getDomain(item.link)}</span>
							</div>
							<div class="link-arrow">&rarr;</div>
						</a>
					{/each}
				</div>
			{/if}
		{:else}
			<div class="loading">
				<div class="spinner"></div>
			</div>
		{/if}
	</main>

	<footer>
		<p>Edit <code>static/links.json</code> to add your own links</p>
	</footer>
</div>

<style>
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	/* Must be :root — custom properties inherit downward only, so html/body
	   can't see them anywhere else. Dark lives here, so it needs no block. */
	:global(:root) {
		color-scheme: dark;
		--bg: #0d1117;
		--surface: #161b22;
		--text: #e6edf3;
		--text-secondary: #a8b5c4;
		--accent: #58a6ff;
		--accent-text: #0d1117;
		--border: #30363d;
		--shadow: rgba(0, 0, 0, 0.4);
		--pill: #21262d;
		--pill-text: #c4cdd9;
	}

	:global(:root[data-theme='light']) {
		color-scheme: light;
		--bg: #f1f3f5;
		--surface: #ffffff;
		--text: #1a1a2e;
		--text-secondary: #4d5064;
		--accent: #0b5ed7;
		--accent-text: #ffffff;
		--border: #c8ced4;
		--shadow: rgba(13, 17, 23, 0.1);
		--pill: #e2e4e8;
		--pill-text: #343a40;
	}

	:global(:root[data-theme='ocean']) {
		--bg: #0b1628;
		--surface: #112240;
		--text: #ccd6f6;
		--text-secondary: #a8b4d0;
		--accent: #64ffda;
		--accent-text: #06111f;
		--border: #233554;
		--shadow: rgba(0, 0, 0, 0.3);
		--pill: #1d3461;
		--pill-text: #b0bee0;
	}

	:global(:root[data-theme='forest']) {
		--bg: #1a2e1a;
		--surface: #243b24;
		--text: #d4e8d4;
		--text-secondary: #a4cca4;
		--accent: #66bb6a;
		--accent-text: #10200f;
		--border: #3a5c3a;
		--shadow: rgba(0, 0, 0, 0.3);
		--pill: #2e4a2e;
		--pill-text: #b0d8b0;
	}

	:global(:root[data-theme='sunset']) {
		--bg: #1a1025;
		--surface: #2d1b42;
		--text: #f0e6ff;
		--text-secondary: #c8b0eb;
		--accent: #ff8a65;
		--accent-text: #1a1025;
		--border: #4a2d6e;
		--shadow: rgba(0, 0, 0, 0.3);
		--pill: #3a2556;
		--pill-text: #d4c0f0;
	}

	:global(html),
	:global(body) {
		background-color: var(--bg);
	}

	:global(body) {
		min-height: 100vh;
		transition: background-color 0.3s ease;
	}

	@media (prefers-reduced-motion: reduce) {
		:global(*) {
			transition-duration: 0.01ms !important;
			animation-duration: 0.01ms !important;
			animation-iteration-count: 1 !important;
		}
	}

	.app {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
		color: var(--text);
		transition: color 0.3s ease;
	}

	header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 2rem 2rem 1.5rem;
		max-width: 900px;
		margin: 0 auto;
		width: 100%;
	}

	h1 {
		font-size: 1.75rem;
		font-weight: 700;
		letter-spacing: -0.02em;
		color: var(--text);
		transition: color 0.3s ease;
	}

	.subtitle {
		color: var(--text-secondary);
		font-size: 0.85rem;
		margin-top: 0.15rem;
		transition: color 0.3s ease;
	}

	.theme-switcher {
		display: flex;
		gap: 0.35rem;
		background: var(--pill);
		padding: 0.25rem;
		border-radius: 0.5rem;
		transition: background 0.3s ease;
	}

	.theme-btn {
		padding: 0.4rem 0.75rem;
		border: none;
		border-radius: 0.35rem;
		background: transparent;
		color: var(--pill-text);
		font-size: 0.75rem;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s ease;
		white-space: nowrap;
	}

	.theme-btn:hover {
		color: var(--text);
	}

	.theme-btn.active {
		background: var(--accent);
		/* Not var(--bg) — that failed contrast in Light. */
		color: var(--accent-text);
	}

	main {
		flex: 1;
		max-width: 900px;
		margin: 0 auto;
		padding: 0 2rem 2rem;
		width: 100%;
	}

	.links-grid {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.link-card {
		display: flex;
		align-items: center;
		gap: 1rem;
		padding: 1rem 1.25rem;
		background: var(--surface);
		border: 1px solid var(--border);
		border-radius: 0.75rem;
		text-decoration: none;
		color: var(--text);
		transition: all 0.2s ease;
	}

	.link-card:hover {
		border-color: var(--accent);
		box-shadow: 0 2px 12px var(--shadow);
		transform: translateY(-1px);
	}

	.link-icon {
		width: 2rem;
		height: 2rem;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
		background: var(--pill);
		border-radius: 0.4rem;
		overflow: hidden;
	}

	.link-icon img {
		width: 20px;
		height: 20px;
	}

	.link-info {
		flex: 1;
		min-width: 0;
	}

	.link-info h2 {
		font-size: 0.95rem;
		font-weight: 600;
		margin-bottom: 0.15rem;
	}

	.link-domain {
		font-size: 0.8rem;
		color: var(--text-secondary);
		transition: color 0.3s ease;
	}

	.link-arrow {
		color: var(--text-secondary);
		font-size: 1.1rem;
		transition: all 0.2s ease;
		flex-shrink: 0;
	}

	.link-card:hover .link-arrow {
		color: var(--accent);
		transform: translateX(3px);
	}

	.loading {
		display: flex;
		justify-content: center;
		padding: 4rem;
	}

	.spinner {
		width: 24px;
		height: 24px;
		border: 2.5px solid var(--border);
		border-top-color: var(--accent);
		border-radius: 50%;
		animation: spin 0.6s linear infinite;
	}

	@keyframes spin {
		to { transform: rotate(360deg); }
	}

	.empty {
		text-align: center;
		padding: 4rem 2rem;
		color: var(--text-secondary);
	}

	.empty code {
		background: var(--pill);
		color: var(--text);
		padding: 0.15rem 0.4rem;
		border-radius: 0.25rem;
		font-size: 0.85rem;
	}

	footer {
		text-align: center;
		padding: 1.5rem 2rem;
		color: var(--text-secondary);
		font-size: 0.8rem;
		transition: color 0.3s ease;
	}

	footer code {
		background: var(--pill);
		color: var(--text);
		padding: 0.15rem 0.4rem;
		border-radius: 0.25rem;
		font-size: 0.75rem;
	}

	@media (max-width: 600px) {
		header {
			flex-direction: column;
			align-items: flex-start;
			gap: 1rem;
			padding: 1.5rem 1rem 1rem;
		}

		main {
			padding: 0 1rem 1.5rem;
		}

		.theme-switcher {
			flex-wrap: wrap;
		}
	}
</style>
