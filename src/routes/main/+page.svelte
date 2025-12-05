<script>
	// Aucun script nécessaire pour cette page
</script>

<style>
	/* Style global rétro */
	:global(html, body) {
		margin: 0;
		padding: 0;
		height: 100%;
		background: #000;
		font-family: 'Press Start 2P', monospace;
		color: #0ff;
	}

	.container {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		height: 100vh;
		text-align: center;
		padding: 1rem;
	}

	h1 {
		font-size: 2rem;
		margin-bottom: 3rem;
		text-shadow: 0 0 10px #0ff;
	}

	.start-btn {
		font-size: 1.5rem;
		padding: 1.2rem 2.5rem;
		background: #111;
		color: #0ff;
		border: 4px solid #0ff;
		text-decoration: none;
		border-radius: 8px;
		text-transform: uppercase;
		transition: 0.15s ease-in-out;
		cursor: pointer;
	}

	.start-btn:hover {
		transform: scale(1.08);
		box-shadow: 0 0 15px #0ff;
	}

	/* Responsive */
	@media (max-width: 600px) {
		h1 {
			font-size: 1.5rem;
		}

		.start-btn {
			font-size: 1.2rem;
			padding: 1rem 2rem;
		}
	}
</style>

<div class="container">
	<h1>Bienvenue</h1>

	<a href="/form" class="start-btn">
		Start
	</a>
</div>

