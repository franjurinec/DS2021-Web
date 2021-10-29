<script>
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';
	import 'chartjs-adapter-moment';

	import predictions from './predictions.json';
	import sentiment from './sentiment.json'
	import prices from './prices.json'

	// Set initial time option
	let selectedTime = Object.keys(predictions)[0]

	onMount(() => {
		configureChart()
	});

	function getSentimentLabelsData() {
		let keys = Object.keys(sentiment)
		let index = keys.indexOf(selectedTime)
		let labels = keys.slice(index - 24, index)
		let data = labels.map(label => ({ 
			x: Number(label), 
			y: sentiment[label]['percentage']
		}))
		return [labels, data]
	}

	function getPriceLabelsData() {
		let keys = Object.keys(prices)
		let index = keys.indexOf(selectedTime)
		let labels = keys.slice(index - 24, index)
		let data = labels.map(label => ({ 
			x: Number(label), 
			y: prices[label]['close']
		}))
		return [labels, data]
	}

	function configureChart() {
		let [sent_labels, sent_data] = getSentimentLabelsData()
		let [_, price_data] = getPriceLabelsData()

		const ctx = document.getElementById('sentiment-chart').getContext('2d');

		const chart = Chart.getChart('sentiment-chart');
		if(chart)
			chart.destroy()

		new Chart(ctx, {
			type: 'line',
			data: {
				sent_labels,
				datasets: [{
					label: 'Sentiment',
					yAxisID: 'A',
					data: sent_data,
					borderColor: 'rgba(10, 136, 138, 1)',
					backgroundColor: 'rgba(10, 136, 138, 1)',
					borderWidth: 1
				},
				{
					label: 'Price',
					yAxisID: 'B',
					data: price_data,
					borderColor: 'rgba(255, 136, 138, 1)',
					backgroundColor: 'rgba(255, 136, 138, 1)',
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
					},
					A: {
						type: 'linear',
						position: 'left',
						title: {
							display: true,
							text: 'Sentiment/%'
						},
						min: -100,
						max: 100,
					},
					B: {
						type: 'linear',
						position: 'right',
						title: {
							display: true,
							text: 'Price/USD'
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

	function currentSent(time) { return Math.round(sentiment[time]['percentage']) }
	function currentPrice(time) { return Math.round(predictions[time]['close']) }

</script>

<main>
	<h1 id="title">Welcome to the BTC sentiment app!</h1>
	<div class="medium-text">
		This week's sentiment is:
		<span class="bold" style={currentSent(selectedTime) >= 0 ? 'color: green' : 'color: red'}>
			{currentSent(selectedTime) >= 0 ? (currentSent(selectedTime) + '% Positive') : (currentSent(selectedTime) + '% Negative')}
		</span>
		<span class="tip">
			🛈
			<span class="tooltip">
				The sentiment percentage represents a relative value of current social sentiment compared to 
				previously recorded values. <br> The source sentiment values are acquired using an advanced sentiment analysis
				algorithm applied on top of a large batch of Tweets from this week!
			</span>
		</span>
	</div>
	<div class="medium-text">
		Next weeks predicted price change:
		<span class="bold" style={currentPrice(selectedTime) >= 0 ? 'color: green' : 'color: red'}>
			{currentPrice(selectedTime) >= 0 ? ('+' + currentPrice(selectedTime)) : currentPrice(selectedTime)} USD
		</span>
		<span class="tip">
			🛈
			<span class="tooltip">
				The predicted price change is caluclated using a pre-trained model with data about price and sentiment from the last 15 weeks. <br> 
				The model was originally trained on a large set of historical Twitter and price data.
			</span>
		</span>
		
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

	.bold {
		font-weight: bolder;
	}

	.block {
		width: 50em;
		height: 25em;
	}

	.tip:hover .tooltip {
		display:block;
	}

	.tooltip {
		display: none;
		color: white;
		font-size: 0.5em;
		background: black;
		border-radius: 10px;
		padding: 16px;
		margin-left: 28px; /* moves the tooltip to the right */
		margin-top: 15px; /* moves it down */
		position: absolute;
		z-index: 1000;
	}

	#title {
		font-size: 4em;
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