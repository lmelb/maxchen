<script lang="ts">
	export let show: boolean;
</script>

{#if show}
	<div id="shot" />
{/if}

<style>
	#shot {
		position: fixed;
		background-image: url('/berlin.png');
		background-position: center;
		background-size: contain;
		background-repeat: no-repeat;
		top: 0;
		bottom: 0;
		width: 100%;
		height: 100%;
		animation: spin 1s ease-in-out;
		opacity: 0;
	}

	@keyframes spin {
		0% {
			opacity: 1;
			scale: 0.3;
			transform: rotate(0deg);
		}
		70% {
			opacity: 1;
		}
		100% {
			transform: rotate(360deg);
			opacity: 0;
			scale: 2;
		}
	}
</style>
