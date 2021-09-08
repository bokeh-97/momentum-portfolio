<script>
	import * as myjson from "./portfolio.json";
	import { Button, MaterialApp } from 'svelte-materialify';

	console.log(myjson.default);
	const obj = myjson.default;
	console.log(typeof obj);
	console.log(Object.values(obj));
	var capital_input = 15;
	var chart_data = Object.values(obj);
	var first_round_allocated_capital = 0;
	var total_capital_allocated = 0;
	var total_returns_next_month = 0;

	function AllocateCapital(available_capital, Dataset) {
		total_capital_allocated = 0;
		total_returns_next_month = 0;
		first_round_allocated_capital = 0;
		Dataset.forEach((element) => {
			console.log(element.Stock);
			element["Weight"] = 0.04;
			element["Number of shares"] = Math.floor(
				(available_capital * element["Weight"]) / element["Price"]
			);
			first_round_allocated_capital +=
				element["Number of shares"] * element["Price"];
			return true;
		});
		console.log(
			"first_round_allocated_capital:" + first_round_allocated_capital
		);

		var balance = available_capital - first_round_allocated_capital;
		console.log("balance" + balance);
		console.log(Dataset);
		// //capital balance allocation
		for (var i = 0; i < 3; i++) {
			Dataset.forEach((element) => {
				if (balance >= 100) {
					if (element["Price"] <= balance) {
						balance -= element["Price"];
						element["Number of shares"] += 1;
					}
				} else {
					return false;
				}
			});
			console.log(i + 1 + "th round balance" + balance);
		}
		console.log("final balance" + balance);
		console.log(Dataset);
		//Calculating capital allocated for each stock, Next month returns for each stock, Summing up total capital and total returns
		Dataset.forEach((element) => {
			element["Capital Allocated"] =
				element["Price"] * element["Number of shares"];
			total_returns_next_month +=
				element["Capital Allocated"] * element["Next_1M_return"];
			total_capital_allocated += element["Capital Allocated"];
		});

		return Dataset;
	}
	async function plotChart() {
		chart_data = AllocateCapital(capital_input * 1000, chart_data);
		am4core.ready(function () {
			// Themes begin
			am4core.useTheme(am4themes_animated);
			// Themes end

			var chart = am4core.create("chartdiv", am4charts.PieChart);
			chart.hiddenState.properties.opacity = 0; // this creates initial fade-in
			chart.data = chart_data;
			chart.innerRadius = am4core.percent(30);
			chart.legend = new am4charts.Legend();

			var series = chart.series.push(new am4charts.PieSeries());
			series.dataFields.value = "Number of shares";

			series.dataFields.category = "Stock";
			series.slices.template.cornerRadius = 0;
			series.colors.step = 5;
			series.labels.template.text = "{Stock}";
			chart.numberFormatter.numberFormat = "#.##";

			series.slices.template.tooltipText =
				" Momentum Rank : #{Momentum Rank} \n {Stock Name}\n Shares : {Number of shares} \n Capital Allocated : {Capital Allocated}\n Stock Price : ${Price}";
		}); // end am4core.ready()
	}
</script>

<body class="body">
	Input Capital $<input
		type="number"
		bind:value={capital_input}
		min="15"
		max="100"
	/>
	x1000
	<p>Available capital = $  {capital_input * 1000}</p>

	<Button class="primary-color" on:click={plotChart}>ALLOCATE</Button>
	<div>
		<p>Total Capital Allocated = ${total_capital_allocated.toFixed(2)}</p>
		<p>
			Returns earned in August 2021 = ${total_returns_next_month.toFixed(
				2
			)}
		</p>
		<p>
			Percentage of returns in August 2021 = {(
				(total_returns_next_month / total_capital_allocated) *
				100
			).toFixed(2)}%
		</p>
	</div>
	<br />
	<div id="chartdiv" />
	<br />

</body>

<style>
	#chartdiv {
		width: 100%;
		height: 90%;
		background-color: rgb(255, 255, 255);
	}
	.body {
		background-color: rgb(255, 255, 255);
	}
	.btn{
		background-color: #4CAF50; /* Green */

	}
</style>
