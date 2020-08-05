<script>
import Game from './Game.svelte'
</script>

<main>
	<h1>LEGO SNAP!</h1>
</main>

<Game />

<style>
	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		                font-size: clamp(16px,7vmin, 4em);

		font-weight: 100;
		text-align: center;
		margin:0;
		border-bottom: 1px solid #666;
	}

	:global(body) {
		display:grid;
		justify-items: center;
		grid-template-rows: min-content min-content 1fr min-content min-content;
		padding:0 8px;
	}

	:global(body > *) {
		width:100%;
		overflow:scroll;
	} 
</style>