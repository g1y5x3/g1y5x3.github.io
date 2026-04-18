<script lang="ts">
	interface BioEntry {
		years: string;
		title: string;
		institution: string;
		institutionUrl: string;
		logoSrc: string;
		logoAlt: string;
	}

	const bioEntries: BioEntry[] = [
		{
			years: '2024 - Current',
			title: 'Post-Doctoral Fellow, Mining and Explosive Engineering',
			institution: 'Missouri University of Science and Technology',
			institutionUrl: 'https://www.mst.edu/',
			logoSrc: '/assets/images/MissouriS&T_Logo.png',
			logoAlt: 'Missouri S&T logo'
		},
		{
			years: '2017 - 2024',
			title: 'Ph.D. Electrical and Computer Engineering',
			institution: 'University of Missouri - Columbia',
			institutionUrl: 'https://vigir.missouri.edu/',
			logoSrc: '/assets/images/vigir.jpeg',
			logoAlt: 'ViGIR logo'
		},
		{
			years: '2014 - 2017',
			title: 'B.S. Computer Engineering & Electrical Engineering',
			institution: 'University of Missouri - Columbia',
			institutionUrl: 'https://missouri.edu/',
			logoSrc: '/assets/images/MU_logo.svg',
			logoAlt: 'Mizzou logo'
		}
	];
</script>

<svelte:head>
	<title>Yixiang Gao</title>
	<meta
		name="description"
		content="Yixiang Gao — robotics, mining autonomy, and edge AI."
	/>
</svelte:head>

<div class="page">
	<header class="site-header">
		<div>
			<h1>Yixiang Gao</h1>
			<p class="tagline">gradient descending through life 📈 📉</p>
		</div>
		<nav class="links" aria-label="Primary links">
			<a href="/assets/YixiangGao_CV.pdf">CV</a>
			<a href="https://scholar.google.com/citations?user=7104qXwAAAAJ&hl=en">Google Scholar</a>
		</nav>
	</header>

	<section class="intro">
		<div class="intro-copy">
			<h2>About</h2>
			<p>
				I am a <strong>Post-Doctoral Fellow</strong> at <em>Missouri S&amp;T</em> specializing in
				robotics and AI for the mining industry. Under the guidance of
				<a href="https://sites.mst.edu/kwame/">Dr. Kwame Awuah-Offei</a>, my research is centered
				on developing autonomous systems for miner search and rescue missions. This work
				integrates several key technologies such as Autonomous Navigation, Digital Twin,
				and Computer Vision.
			</p>
			<p>
				I earned my <strong>Ph.D.</strong> from <em>University of Missouri - Columbia</em>
				under the supervision of
				<a href="https://engineering.missouri.edu/faculty/guilherme-desouza/"
					>Dr. Gui DeSouza</a
				>
				at the
				<a href="http://vigir.missouri.edu/index.html"
					>Vision Guided Intellegent Robotics Laboratory</a
				>. My doctoral research, supported by the NIH, pioneered machine learning
				applications for voice pathology, culminating in publications that bridge the
				fields of engineering and clinical science.
			</p>
			<p>
				Currently, I am interested in Differentiable Rendering and Simulations, Continual
				Learning, and Edge AI that can acceleate the real-world deployment of robotics. My
				work is driven by the conviction that the future of AI should be decentralized,
				efficient, and accessible to everyone, not confined to the cloud.
			</p>
		</div>

		<div class="intro-photo">
			<img src="/assets/images/sandiego_2024.png" alt="Yixiang Gao" />
		</div>
	</section>

	<section class="section">
		<h2>Experience</h2>
		<div class="bio-list">
			{#each bioEntries as entry}
				<div class="bio-item">
					<div class="bio-years">{entry.years}</div>
					<div class="bio-main">
						<div class="bio-title">{entry.title}</div>
						<div class="bio-school"><em>{entry.institution}</em></div>
					</div>
					<div class="bio-logo">
						<a href={entry.institutionUrl} aria-label={entry.institution}>
							<img src={entry.logoSrc} alt={entry.logoAlt} />
						</a>
					</div>
				</div>
			{/each}
		</div>
	</section>
</div>

<style>
	:global(body) {
		margin: 0;
		font-family:
			Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
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
		grid-template-columns: minmax(0, 1.6fr) minmax(220px, 0.8fr);
		gap: 2rem;
		align-items: start;
		margin-top: 2rem;
	}

	.intro-copy h2,
	.section h2 {
		margin: 0 0 1rem;
		font-size: 1.35rem;
	}

	.intro-copy p,
	.bio-item {
		font-size: 1rem;
		line-height: 1.8;
	}

	.intro-copy p {
		margin: 0 0 1rem;
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
		grid-template-columns: minmax(120px, 0.26fr) minmax(0, 1fr) auto;
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

	.bio-logo img {
		max-height: 2.8rem;
		max-width: 6rem;
		object-fit: contain;
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
			max-width: 18rem;
		}

		.bio-item {
			grid-template-columns: 1fr;
		}

		.bio-logo {
			justify-content: flex-start;
		}
	}
</style>
