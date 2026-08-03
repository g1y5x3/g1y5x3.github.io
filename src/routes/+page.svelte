<script lang="ts">
	import { asset } from '$app/paths';
	import { onMount } from 'svelte';

	interface BioEntry {
		years: string;
		title: string;
		institution: string;
		institutionUrl: string;
		logoSrc: string;
		darkLogoSrc?: string;
		logoAlt: string;
	}

	let useDarkLogos = $state(false);

	onMount(() => {
		let updateFrame = 0;

		const updateLogoTheme = () => {
			cancelAnimationFrame(updateFrame);
			updateFrame = requestAnimationFrame(() => {
				const channels = getComputedStyle(document.body).backgroundColor.match(/[\d.]+/g);
				if (!channels || channels.length < 3) return;

				const [red, green, blue] = channels.slice(0, 3).map(Number);
				const luminance = (0.2126 * red + 0.7152 * green + 0.0722 * blue) / 255;
				useDarkLogos = luminance < 0.5;
			});
		};

		const themeObserver = new MutationObserver(updateLogoTheme);
		themeObserver.observe(document.documentElement, { attributes: true });
		themeObserver.observe(document.body, { attributes: true });
		themeObserver.observe(document.head, {
			attributes: true,
			childList: true,
			characterData: true,
			subtree: true
		});

		const colorScheme = window.matchMedia('(prefers-color-scheme: dark)');
		colorScheme.addEventListener('change', updateLogoTheme);
		updateLogoTheme();

		return () => {
			cancelAnimationFrame(updateFrame);
			themeObserver.disconnect();
			colorScheme.removeEventListener('change', updateLogoTheme);
		};
	});

	const bioEntries: BioEntry[] = [
		{
			years: 'Aug 2026 - Current',
			title: 'Chief Scientist in Robotics & AI',
			institution: 'TouchTronix Robotics',
			institutionUrl: 'https://www.touchtronix.io/',
			logoSrc: asset('/assets/images/TouchTronix_logo.png'),
			darkLogoSrc: asset('/assets/images/TouchTronix_logo_dark.png'),
			logoAlt: 'TouchTronix Robotics logo'
		},
		{
			years: '2025 - 2026',
			title: 'Part-Time Robotic Consultant, Innovation Implementation and Strategy',
			institution: 'Ameren',
			institutionUrl: 'https://www.ameren.com/',
			logoSrc: asset('/assets/images/Ameren_logo.png'),
			darkLogoSrc: asset('/assets/images/Ameren_logo_dark.png'),
			logoAlt: 'Ameren logo'
		},
		{
			years: '2024 - 2026',
			title: 'Post-Doctoral Fellow, Mining and Explosive Engineering',
			institution: 'Missouri University of Science and Technology',
			institutionUrl: 'https://www.mst.edu/',
			logoSrc: asset('/assets/images/MissouriS&T_Logo.png'),
			darkLogoSrc: asset('/assets/images/MissouriS&T_Logo_dark.png'),
			logoAlt: 'Missouri S&T logo'
		},
		{
			years: '2017 - 2024',
			title: 'Ph.D. Electrical and Computer Engineering',
			institution: 'University of Missouri - Columbia',
			institutionUrl: 'http://vigir.missouri.edu/index.html',
			logoSrc: asset('/assets/images/vigir.png'),
			darkLogoSrc: asset('/assets/images/vigir_dark.png'),
			logoAlt: 'ViGIR logo'
		},
		{
			years: '2014 - 2017',
			title: 'B.S. Computer Engineering & Electrical Engineering',
			institution: 'University of Missouri - Columbia',
			institutionUrl: 'https://missouri.edu/',
			logoSrc: asset('/assets/images/MU_logo.png'),
			logoAlt: 'Mizzou logo'
		}
	];
</script>

<svelte:head>
	<title>Yixiang Gao</title>
	<meta
		name="description"
		content="Yixiang Gao — robotics, egocentric vision, tactile sensing, and edge AI."
	/>
</svelte:head>

<div class="page" id="about">
	<header class="site-header">
		<div>
			<h1>Yixiang Gao</h1>
			<p class="tagline">gradient descending through life 📈 📉</p>
		</div>
		<nav class="links" aria-label="Primary links">
			<a href="#about">About</a>
			<a href="#projects">Projects</a>
			<a href="https://github.com/justyx404">GitHub</a>
			<a href="https://scholar.google.com/citations?user=7104qXwAAAAJ&hl=en">Google Scholar</a>
			<a href={asset('/assets/YixiangGao_CV.pdf')}>CV</a>
		</nav>
	</header>

	<section class="intro">
		<div class="intro-copy">
			<h2>About</h2>
			<p>
				I am the <strong>Chief Scientist in Robotics &amp; AI</strong> at
				<a href="https://www.touchtronix.io/"><em>TouchTronix Robotics</em></a>, where I lead
				efforts in vision+tactile data collection and robot policy learning.
			</p>
			<p>
				Previously, I was a <strong>Post-Doctoral Fellow</strong> at
				<em>Missouri S&amp;T</em>, where I used the Spot quadruped platform to develop autonomous
				systems for miner search and rescue missions. Under the guidance of
				<a href="https://sites.mst.edu/kwame/">Dr. Kwame Awuah-Offei</a>, this research integrated
				autonomous navigation, digital twins, and computer vision. During this period, I also served
				as a <strong>Part-Time Robotic Consultant</strong> with <em>Ameren</em>'s Innovation
				Implementation and Strategy team, led by
				<a href="https://www.linkedin.com/in/alex-rojas-220bb17">Alex Rojas</a>. There, I worked on
				deploying Spot for autonomous substation inspections, including step-voltage measurements,
				to assist field workers. The shared quadruped platform closely connected my research and
				industry work.
			</p>
			<p>
				I earned my <strong>Ph.D.</strong> from <em>University of Missouri - Columbia</em>
				under the supervision of
				<a href="https://engineering.missouri.edu/faculty/guilherme-desouza/">Dr. Gui DeSouza</a>
				at the
				<a href="http://vigir.missouri.edu/index.html"
					>Vision Guided Intelligent Robotics Laboratory</a
				>. My doctoral research, supported by the NIH, pioneered machine learning applications for
				voice pathology, culminating in publications that bridge the fields of engineering and
				clinical science.
			</p>
		</div>

		<div class="intro-photo">
			<img
				src={asset('/assets/images/sandiego_2024.webp')}
				alt="Yixiang Gao"
				width="1200"
				height="900"
			/>
		</div>
	</section>

	<section class="section">
		<h2>Experience</h2>
		<div class="bio-list">
			{#each bioEntries as entry (entry.title)}
				<div class="bio-item">
					<div class="bio-years">{entry.years}</div>
					<div class="bio-main">
						<div class="bio-title">{entry.title}</div>
						<div class="bio-school"><em>{entry.institution}</em></div>
					</div>
					<div class="bio-logo">
						<a href={entry.institutionUrl} rel="external" aria-label={entry.institution}>
							<img
								src={useDarkLogos && entry.darkLogoSrc ? entry.darkLogoSrc : entry.logoSrc}
								alt={entry.logoAlt}
							/>
						</a>
					</div>
				</div>
			{/each}
		</div>
	</section>

	<section class="section" id="projects">
		<h2>Projects</h2>
		<article class="project-item">
			<a
				class="project-media"
				href="https://justyx404.github.io/spot-edge-nav/"
				rel="external"
				aria-label="Open Spot edge navigation project"
			>
				<video
					src={asset('/assets/videos/spot-edge-nav.mp4')}
					poster={asset('/assets/images/spot-edge-nav-poster.webp')}
					aria-label="Mission 4 autonomous underground navigation demo with Spot"
					width="480"
					height="260"
					autoplay
					muted
					loop
					playsinline
					preload="metadata"
				></video>
			</a>
			<div class="project-copy">
				<h3>
					<a href="https://justyx404.github.io/spot-edge-nav/" rel="external">
						Efficient Autonomous Navigation of a Quadruped Robot in Underground Mines on Edge
						Hardware
					</a>
				</h3>
				<p class="project-authors">Y. Gao · K. Awuah-Offei</p>
				<p class="project-venue">Pre-print · 2026</p>
				<p class="project-description">
					Runs entirely on a 40 W Intel NUC without a GPU or network connectivity, achieving 100%
					success across 20 underground field trials.
				</p>
				<nav class="project-links" aria-label="Project resources">
					<a href="https://justyx404.github.io/spot-edge-nav/" rel="external">Project</a>
					<span aria-hidden="true">·</span>
					<a href="https://arxiv.org/pdf/2603.04470" rel="external">Paper</a>
					<span aria-hidden="true">·</span>
					<a href="https://github.com/g1y5x3/spot-edge-nav" rel="external">Code</a>
				</nav>
			</div>
		</article>

		<article class="project-item">
			<a
				class="project-media"
				href="https://ieeexplore.ieee.org/abstract/document/10253983"
				rel="external"
			>
				<img
					src={asset('/assets/images/ga-svm-confounding.webp')}
					alt="GA-SVM confounding mitigation workflow"
					width="720"
					height="722"
					loading="lazy"
				/>
			</a>
			<div class="project-copy">
				<h3>
					<a href="https://ieeexplore.ieee.org/abstract/document/10253983" rel="external">
						Removal of Confounding Factors using GA-SVM Feature Adaptation: Application on Detection
						of Vocal Fatigue thru sEMG Classification
					</a>
				</h3>
				<p class="project-authors">Y. Gao · G. N. DeSouza · M. L. Berardi · M. Dietrich</p>
				<p class="project-venue">IEEE Congress on Evolutionary Computation (CEC) · 2023</p>
				<p class="project-description">
					Uses a genetic algorithm to adapt feature vectors, improving SVM generalization while
					reducing correlation with confounding factors in sEMG vocal-fatigue classification.
				</p>
				<nav class="project-links" aria-label="GA-SVM project resources">
					<a href="https://ieeexplore.ieee.org/abstract/document/10253983" rel="external">Paper</a>
					<span aria-hidden="true">·</span>
					<a href="https://github.com/justyx404/Mit_GA" rel="external">Code</a>
				</nav>
			</div>
		</article>

		<article class="project-item">
			<a
				class="project-media"
				href="http://vigir.missouri.edu/Research/sEMG_dataset.html"
				rel="external"
			>
				<img
					src={asset('/assets/images/semg-dataset.webp')}
					alt="Surface electromyography data-collection hardware"
					width="720"
					height="326"
					loading="lazy"
				/>
			</a>
			<div class="project-copy">
				<h3>
					<a href="https://doi.org/10.3390/app11104335" rel="external">
						Classification of Vocal Fatigue Using sEMG: Data Imbalance, Normalization, and the Role
						of Vocal Fatigue Index Scores
					</a>
				</h3>
				<p class="project-authors">Y. Gao · M. Dietrich · G. N. DeSouza</p>
				<p class="project-venue">Applied Sciences · 2021</p>
				<p class="project-description">
					Examines data imbalance, signal normalization, and VFI-based labeling for machine-learning
					detection of vocal fatigue using sEMG data from 88 participants.
				</p>
				<nav class="project-links" aria-label="sEMG project resources">
					<a href="http://vigir.missouri.edu/Research/sEMG_dataset.html" rel="external">Project</a>
					<span aria-hidden="true">·</span>
					<a href="https://doi.org/10.3390/app11104335" rel="external">Paper</a>
				</nav>
			</div>
		</article>
	</section>

	<footer class="site-footer">© 2026 Yixiang Gao</footer>
</div>

<style>
	:global(body) {
		margin: 0;
		font-family:
			Inter,
			ui-sans-serif,
			system-ui,
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			sans-serif;
		background: #ffffff;
		color: #1f2937;
	}

	:global(a) {
		color: #1d4ed8;
		text-decoration: none;
	}

	:global(a:hover) {
		text-decoration: underline;
	}

	.page {
		max-width: 960px;
		margin: 0 auto;
		padding: 2.5rem 1.5rem 4rem;
	}

	#about,
	#projects {
		scroll-margin-top: 1rem;
	}

	.site-header {
		display: flex;
		justify-content: space-between;
		align-items: baseline;
		gap: 1rem;
		padding-bottom: 1.5rem;
		border-bottom: 1px solid #e5e7eb;
	}

	h1 {
		margin: 0;
		font-size: 2.5rem;
		font-weight: 700;
		letter-spacing: -0.03em;
	}

	.tagline {
		margin: 0.4rem 0 0;
		color: #6b7280;
		font-style: italic;
	}

	.links {
		display: flex;
		gap: 1rem;
		flex-wrap: wrap;
		font-size: 0.95rem;
	}

	.intro {
		display: grid;
		grid-template-columns: minmax(0, 1.35fr) minmax(300px, 1fr);
		gap: 2.5rem;
		align-items: start;
		margin-top: 2rem;
	}

	.intro-copy h2,
	.section h2 {
		margin: 0 0 1rem;
		font-size: 1.35rem;
	}

	.intro-copy h2 {
		line-height: 1.2;
	}

	.intro-copy p,
	.bio-item {
		font-size: 1rem;
		line-height: 1.8;
	}

	.intro-copy p {
		margin: 0 0 1rem;
	}

	.intro-photo {
		margin-top: calc(1.35rem * 1.2 + 1rem);
	}

	.intro-photo img {
		display: block;
		width: 100%;
		height: auto;
		border-radius: 10px;
	}

	.section {
		margin-top: 2.5rem;
	}

	.bio-list {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.bio-item {
		display: grid;
		grid-template-columns: 9rem minmax(0, 1fr) 8rem;
		gap: 1rem;
		align-items: center;
		padding: 1rem 0;
		border-top: 1px solid #e5e7eb;
	}

	.bio-item:last-child {
		border-bottom: 1px solid #e5e7eb;
	}

	.bio-years {
		font-size: 0.92rem;
		font-weight: 600;
		color: #6b7280;
	}

	.bio-title {
		font-weight: 600;
		line-height: 1.5;
	}

	.bio-school {
		color: #4b5563;
	}

	.bio-logo {
		display: flex;
		justify-content: flex-end;
	}

	.bio-logo a {
		display: flex;
		width: 8rem;
		height: 3.6rem;
		align-items: center;
		justify-content: center;
	}

	.bio-logo img {
		display: block;
		max-width: 100%;
		max-height: 100%;
		object-fit: contain;
	}

	.project-item {
		display: grid;
		grid-template-columns: minmax(240px, 0.85fr) minmax(0, 1.15fr);
		gap: 1.5rem;
		align-items: center;
		padding: 1rem 0;
		border-top: 1px solid #e5e7eb;
		border-bottom: 1px solid #e5e7eb;
	}

	.project-item + .project-item {
		border-top: 0;
	}

	.project-media {
		display: flex;
		aspect-ratio: 16 / 9;
		align-items: center;
		justify-content: center;
	}

	.project-media img,
	.project-media video {
		display: block;
		width: 100%;
		height: 100%;
		object-fit: contain;
		border-radius: 10px;
	}

	.project-copy h3 {
		margin: 0;
		font-size: 1.2rem;
		line-height: 1.4;
	}

	.project-copy .project-authors {
		margin: 0.5rem 0 0.15rem;
		line-height: 1.5;
	}

	.project-copy .project-venue {
		margin: 0;
		color: #4b5563;
		font-style: italic;
	}

	.project-copy .project-description {
		margin: 0.65rem 0 0;
		line-height: 1.55;
	}

	.project-links {
		display: flex;
		gap: 0.45rem;
		margin-top: 0.65rem;
		font-weight: 600;
	}

	.site-footer {
		margin-top: 2rem;
		color: #6b7280;
		font-size: 0.9rem;
		text-align: right;
	}

	@media (max-width: 760px) {
		.site-header,
		.intro {
			grid-template-columns: 1fr;
			flex-direction: column;
			align-items: start;
		}

		.intro {
			display: flex;
			flex-direction: column-reverse;
		}

		.intro-photo {
			max-width: 24rem;
			margin-top: 0;
		}

		.bio-item,
		.project-item {
			grid-template-columns: 1fr;
		}

		.bio-logo {
			justify-content: flex-start;
		}

		.project-media {
			max-width: 24rem;
		}
	}
</style>
