<script>
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';
	import 'chartjs-adapter-moment';

	import predictions from './predictions.json';
	import sentiment from './nozerosent.json'
	import prices from './prices.json'

	// Set initial time option
	let selectedTime = Object.keys(predictions)[0]

	onMount(() => {
		configureChart()
	});

	function configureChart() {
		let keys = Object.keys(sentiment)
		let index = keys.indexOf(selectedTime)
		let labels = keys.slice(index - 12, index)
		let data = labels.map(label => ({ 
			x: Number(label), 
			y: sentiment[label]['wsentiment']
		}))

		console.log(JSON.stringify(data))

		const ctx = document.getElementById('sentiment-chart').getContext('2d');

		const chart = Chart.getChart('sentiment-chart');
		if(chart)
			chart.destroy()

		new Chart(ctx, {
			type: 'line',
			data: {
				labels,
				datasets: [{
					label: 'BTC Sentiment',
					data: data,
					borderColor: 'rgba(10, 136, 138, 1)',
					backgroundColor: 'rgba(10, 136, 138, 1)',
					borderWidth: 1
				}]
			},
			options: {
				scales: {
					x: {
						type: 'time',
						ticks: {
							beginAtZero: false
						},
						time: {
							unit: 'week',
							parser: function(time){
								return getDateWithOffset(time, 7);
							}
						}
					}
				}
			}
		});
	}

	function getDateWithOffset(time, days=0) {
		let date = new Date(Number(time))
		if (days !== 0)
			date.setDate(date.getDate() + days)
		return date
	}

	function dateString(time, days=0) {
		let date = getDateWithOffset(time, days)
		return date.toLocaleDateString('en-gb')
	}

</script>

<main>
	<h1 id="title">Welcome to the BTC sentiment app!</h1>
	<div class="medium-text">
		This week's sentiment is:
		<span id="sentiment">{Math.round(sentiment[selectedTime]['wsentiment'])}</span>
	</div>
	<div class="medium-text">
		Next weeks predicted price change:
		<span id="price">{Math.round(predictions[selectedTime]['close'])} USD</span>
		
	</div>
	<div class="block">
		<canvas id="sentiment-chart" width="400" height="200"></canvas>
	</div>

	<div id="date-select-container">
		<div>
			You are previewing the page on:
		</div>
		<select id="week" bind:value={selectedTime} on:change={configureChart}>
			{#each Object.keys(predictions) as time}
				<option value={time}>{dateString(time) + ' - ' + dateString(time, 7)}</option>
			{/each}
		</select>
	</div>
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		row-gap: 50px;
	}

	option {
		background: white;
		color: black;
	}

	.medium-text {
		font-size: 2em;
	}

	.block {
		width: 50em;
		height: 25em;
	}

	#title {
		font-size: 4em;
	}

	#sentiment {
		color: green;
		font-weight: bolder;
	}
	
	#price {
		color: grey;
		font-weight: bolder;
	}

	#date-select-container {
		position: fixed;
		display: flex;
		flex-direction: column;
		align-items: center;
		bottom: 40px;
		right: 60px;
		background: black;
		color: white;
		border-radius: 10px;
		padding: 16px 32px;
	}

	#week {
		margin-top: 8px;
		border: none;
		outline: none;
		background: none;
		color: white;
	}

</style>