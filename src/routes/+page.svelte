<script>
	import { onMount } from 'svelte';

	const themes = {
		light: {
			name: 'Light',
			bg: '#f8f9fa',
			surface: '#ffffff',
			text: '#1a1a2e',
			textSecondary: '#555770',
			accent: '#0d6efd',
			accentHover: '#0b5ed7',
			border: '#dee2e6',
			shadow: 'rgba(0,0,0,0.08)',
			pill: '#e2e4e8',
			pillText: '#343a40'
		},
		dark: {
			name: 'Dark',
			bg: '#000000',
			surface: '#161b22',
			text: '#e6edf3',
			textSecondary: '#a8b5c4',
			accent: '#58a6ff',
			accentHover: '#79c0ff',
			border: '#30363d',
			shadow: 'rgba(0,0,0,0.4)',
			pill: '#21262d',
			pillText: '#c4cdd9'
		},
		ocean: {
			name: 'Ocean',
			bg: '#0b1628',
			surface: '#112240',
			text: '#ccd6f6',
			textSecondary: '#a8b4d0',
			accent: '#64ffda',
			accentHover: '#a8ffe0',
			border: '#233554',
			shadow: 'rgba(0,0,0,0.3)',
			pill: '#1d3461',
			pillText: '#b0bee0'
		},
		forest: {
			name: 'Forest',
			bg: '#1a2e1a',
			surface: '#243b24',
			text: '#d4e8d4',
			textSecondary: '#a4cca4',
			accent: '#66bb6a',
			accentHover: '#81c784',
			border: '#3a5c3a',
			shadow: 'rgba(0,0,0,0.3)',
			pill: '#2e4a2e',
			pillText: '#b0d8b0'
		},
		sunset: {
			name: 'Sunset',
			bg: '#1a1025',
			surface: '#2d1b42',
			text: '#f0e6ff',
			textSecondary: '#c8b0eb',
			accent: '#ff8a65',
			accentHover: '#ffab91',
			border: '#4a2d6e',
			shadow: 'rgba(0,0,0,0.3)',
			pill: '#3a2556',
			pillText: '#d4c0f0'
		}
	};

	let links = $state([]);
	let currentTheme = $state('dark');
	let loaded = $state(false);

	onMount(() => {
		const saved = localStorage.getItem('linkboard-theme');
		if (saved && themes[saved]) {
			currentTheme = saved;
		}

		fetch('/links.json')
			.then(r => r.json())
			.then(data => {
				links = data;
				loaded = true;
			});
	});

	function setTheme(name) {
		currentTheme = name;
		localStorage.setItem('linkboard-theme', name);
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
	<title>LinkBoard</title>
</svelte:head>

<div class="app" style="
	--bg: {themes[currentTheme].bg};
	--surface: {themes[currentTheme].surface};
	--text: {themes[currentTheme].text};
	--text-secondary: {themes[currentTheme].textSecondary};
	--accent: {themes[currentTheme].accent};
	--accent-hover: {themes[currentTheme].accentHover};
	--border: {themes[currentTheme].border};
	--shadow: {themes[currentTheme].shadow};
	--pill: {themes[currentTheme].pill};
	--pill-text: {themes[currentTheme].pillText};
">
	<header>
		<div class="header-content">
			<h1>LinkBoard</h1>
			<p class="subtitle">{links.length} bookmarked links</p>
		</div>
		<div class="theme-switcher">
			{#each Object.entries(themes) as [key, theme]}
				<button
					class="theme-btn"
					class:active={currentTheme === key}
					onclick={() => setTheme(key)}
					title={theme.name}
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

	:global(body) {
		background: var(--bg);
		min-height: 100vh;
		transition: background 0.3s ease;
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
		color: var(--bg);
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
