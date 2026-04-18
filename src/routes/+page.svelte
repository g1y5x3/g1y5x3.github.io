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
		content="Yixiang Gao — robotics, AI, mining autonomy, differentiable rendering, continual learning, and edge AI."
	/>
</svelte:head>

<div class="page">
	<section class="hero">
		<div class="hero-copy">
			<h1>Yixiang Gao, Ph.D.</h1>
			<p class="tagline">gradient descending through life 📈 📉</p>

			<h2>About Me</h2>
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
			<p>Currently, I am interested in</p>
			<ul>
				<li>Differentiable Rendering and Simulations</li>
				<li>Continual Learning</li>
				<li>Edge AI</li>
			</ul>
			<p>
				that can acceleate the real-world deployment of robotics. My work is driven by
				the conviction that the future of AI should be decentralized, efficient, and
				accessible to everyone, not confined to the cloud.
			</p>
		</div>

		<div class="hero-photo">
			<img src="/assets/images/sandiego_2024.png" alt="Yixiang Gao" />
		</div>
	</section>

	<section class="section">
		<h2>Bios</h2>
		<div class="bios-list">
			{#each bioEntries as entry}
				<div class="bio-card">
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

		<p class="cv-note">
			For a detailed overview of my work and experience, please see my full
			<a href="/assets/YixiangGao_CV.pdf">Curriculum Vitae</a>.
		</p>
	</section>
</div>

<style>
	:global(body) {
		margin: 0;
		font-family:
			Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
		background: #f7f8fb;
		color: #1f2937;
	}

	:global(a) {
		color: #4338ca;
		text-decoration: none;
	}

	:global(a:hover) {
		text-decoration: underline;
	}

	.page {
		max-width: 1100px;
		margin: 0 auto;
		padding: 3rem 1.5rem 4rem;
	}

	.hero {
		display: grid;
		grid-template-columns: minmax(0, 1.7fr) minmax(280px, 0.9fr);
		gap: 2rem;
		align-items: start;
	}

	.hero-copy,
	.section {
		background: white;
		border-radius: 18px;
		padding: 2rem;
		box-shadow: 0 16px 40px rgba(15, 23, 42, 0.08);
	}

	h1 {
		margin: 0;
		font-size: clamp(2.4rem, 5vw, 3.6rem);
		line-height: 1.05;
	}

	h2 {
		margin-top: 2rem;
		margin-bottom: 1rem;
		font-size: 1.45rem;
	}

	p,
	li {
		font-size: 1.02rem;
		line-height: 1.75;
	}

	.tagline {
		margin-top: 0.5rem;
		color: #6b7280;
		font-style: italic;
	}

	.hero-photo {
		background: white;
		border-radius: 18px;
		padding: 1rem;
		box-shadow: 0 16px 40px rgba(15, 23, 42, 0.08);
	}

	.hero-photo img {
		display: block;
		width: 100%;
		height: auto;
		border-radius: 14px;
		object-fit: cover;
	}

	.section {
		margin-top: 2rem;
	}

	.bios-list {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.bio-card {
		display: grid;
		grid-template-columns: minmax(120px, 0.28fr) minmax(0, 1fr) minmax(96px, 0.22fr);
		gap: 1rem;
		align-items: center;
		padding: 1rem 1.2rem;
		border: 1px solid #e5e7eb;
		border-radius: 14px;
		background: #fafafa;
	}

	.bio-years {
		font-weight: 600;
		color: #4b5563;
	}

	.bio-title {
		font-weight: 700;
		margin-bottom: 0.25rem;
	}

	.bio-school {
		color: #4b5563;
	}

	.bio-logo {
		display: flex;
		justify-content: flex-end;
	}

	.bio-logo img {
		max-height: 3rem;
		max-width: 100%;
		object-fit: contain;
	}

	.cv-note {
		margin-top: 1.5rem;
	}

	@media (max-width: 860px) {
		.hero {
			grid-template-columns: 1fr;
		}

		.hero-photo {
			max-width: 24rem;
		}

		.bio-card {
			grid-template-columns: 1fr;
		}

		.bio-logo {
			justify-content: flex-start;
		}
	}
</style>
