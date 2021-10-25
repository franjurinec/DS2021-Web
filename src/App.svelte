<script>
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';
	onMount(() => {
		const ctx = document.getElementById('sentiment-chart').getContext('2d');
		const sentimentChart = new Chart(ctx, {
			type: 'bar',
			data: {
				labels: ['Red', 'Blue', 'Yellow', 'Green', 'Purple', 'Orange'],
				datasets: [{
					label: '# of Votes',
					data: [12, 19, 3, 5, 2, 3],
					backgroundColor: [
						'rgba(255, 99, 132, 0.2)',
						'rgba(54, 162, 235, 0.2)',
						'rgba(255, 206, 86, 0.2)',
						'rgba(75, 192, 192, 0.2)',
						'rgba(153, 102, 255, 0.2)',
						'rgba(255, 159, 64, 0.2)'
					],
					borderColor: [
						'rgba(255, 99, 132, 1)',
						'rgba(54, 162, 235, 1)',
						'rgba(255, 206, 86, 1)',
						'rgba(75, 192, 192, 1)',
						'rgba(153, 102, 255, 1)',
						'rgba(255, 159, 64, 1)'
					],
					borderWidth: 1
				}]
			},
			options: {
				scales: {
					y: {
						beginAtZero: true
					}
				}
			}
		});
	});


</script>

<main>
	<h1 id="title">Welcome to the BTC sentiment app!</h1>
	<div class="medium-text">
		This week's sentiment is:
		<span id="sentiment">30% Positive</span>
	</div>
	<div class="block">
		<canvas id="sentiment-chart" width="400" height="200"></canvas>
	</div>
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		row-gap: 50px;
	}

	.medium-text {
		font-size: 2em;
	}

	#title {
		font-size: 4em;
	}

	#sentiment {
		color: green;
		font-weight: bolder;
	}

	.block {
		width: 50em;
		height: 25em;
	}
</style>