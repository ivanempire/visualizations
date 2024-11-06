<script>
	import { onMount } from "svelte";
	import * as d3 from "d3";
	import data from "$lib/stacked.json";

	let selectedDocument = "Stargate: SG-1";

	onMount(() => {
		const svg = d3.select("#chart")
			.attr("width", 1100)
			.attr("height", 600);

		const margin = { top: 60, right: 30, bottom: 70, left: 70 };
		const width = 1100 - margin.left - margin.right;
		const height = 600 - margin.top - margin.bottom;

		const chartGroup = svg.append("g")
			.attr("transform", `translate(${margin.left},${margin.top})`);

		const color = d3.scaleOrdinal()
			.range(["#1f77b4", "#ff7f0e", "#2ca02c", "#d62728", "#9467bd", "#8c564b", "#e377c2", "#7f7f7f", "#bcbd22", "#17becf"]);

		const x = d3.scaleBand().padding(0.2).range([0, width]);
		const y = d3.scaleLinear().range([height, 0]);

		const xAxisGroup = chartGroup.append("g")
			.attr("class", "x-axis")
			.attr("transform", `translate(0,${height})`);

		const yAxisGroup = chartGroup.append("g")
			.attr("class", "y-axis");

		const tooltip = d3.select("body").append("div")
			.attr("class", "tooltip")
			.style("position", "absolute")
			.style("visibility", "hidden")
			.style("padding", "6px")
			.style("background", "rgba(0, 0, 0, 0.7)")
			.style("border-radius", "4px")
			.style("color", "#fff")
			.style("font-size", "12px");

		function updateChart(doc) {
			const documentData = data[doc];

			const keys = Object.keys(documentData[0]).filter(
				key => key !== "lineBin" && documentData.some(d => d[key] > 0)
			);

			const stackedData = d3.stack().keys(keys)(documentData);

			x.domain(documentData.map(d => d.lineBin));
			y.domain([0, d3.max(stackedData, d => d3.max(d, d => d[1]))]).nice();

			xAxisGroup.call(d3.axisBottom(x).tickSize(0).tickPadding(10))
				.selectAll("text").style("font-size", "12px");

			svg.select(".x-axis-label").remove();
			svg.append("text")
				.attr("class", "x-axis-label")
				.attr("x", width / 2 + margin.left)
				.attr("y", height + margin.top + 50)
				.style("text-anchor", "middle")
				.style("font-size", "16px")
				.text("Line Bins");

			yAxisGroup.call(d3.axisLeft(y).ticks(5))
				.selectAll("text").style("font-size", "12px");

			svg.select(".y-axis-label").remove();
			svg.append("text")
				.attr("class", "y-axis-label")
				.attr("transform", "rotate(-90)")
				.attr("y", margin.left / 2 - 20)
				.attr("x", -height / 2 - 60)
				.style("text-anchor", "middle")
				.style("font-size", "16px")
				.text("Word Count");

			svg.select(".chart-title").remove();
			svg.append("text")
				.attr("class", "chart-title")
				.attr("x", width / 2 + margin.left)
				.attr("y", margin.top / 2 - 10)
				.style("text-anchor", "middle")
				.style("font-size", "20px")
				.style("font-weight", "bold")
				.text(`Word Distribution in ${doc}`);

			const groups = chartGroup.selectAll("g.layer")
				.data(stackedData, d => d.key);

			groups.enter().append("g")
				.attr("class", "layer")
				.attr("fill", d => color(d.key))
				.merge(groups)
				.selectAll("rect")
				.data(d => d, d => d.data.lineBin)
				.join("rect")
				.attr("x", d => x(d.data.lineBin))
				.attr("y", d => y(d[1]))
				.attr("height", d => y(d[0]) - y(d[1]))
				.attr("width", x.bandwidth());

			groups.exit().remove();

			chartGroup.selectAll("rect")
				.on("mouseover", function(event, d) {
					const values = keys.map(key => ({ key, value: d.data[key] || 0 }))
						.filter(v => v.value > 0)
						.sort((a, b) => b.value - a.value)
						.slice(0, 3);

					const tooltipText = values.map(v => `${v.key}: ${v.value}`).join(", ");
					tooltip.style("visibility", "visible")
						.text(`Line Bin: ${d.data.lineBin} | Top Words: ${tooltipText}`);
				})
				.on("mousemove", function(event) {
					tooltip.style("top", (event.pageY - 10) + "px")
						.style("left", (event.pageX + 10) + "px");
				})
				.on("mouseout", function() {
					tooltip.style("visibility", "hidden");
				});
		}

		updateChart(selectedDocument);

		document.getElementById("documentSelector").addEventListener("change", function() {
			selectedDocument = this.value;
			updateChart(selectedDocument);
		});
	});
</script>

<style>
    .select-container {
        margin-bottom: 20px;
    }

    .tooltip {
        position: absolute;
        text-align: center;
        padding: 6px;
        font-size: 12px;
        background: rgba(0, 0, 0, 0.7);
        color: #fff;
        border-radius: 4px;
        pointer-events: none;
        visibility: hidden;
    }
</style>

<section class="flex flex-col items-center justify-center min-h-screen bg-gray-100 text-gray-800 px-6">
	<div class="select-container">
		<label for="documentSelector" class="text-lg font-semibold">Select response:</label>
		<select id="documentSelector" class="ml-2 p-1 rounded border border-gray-300">
			<option value="Stargate: SG-1">Stargate: SG-1</option>
			<option value="Stargate: Atlantis">Stargate: Atlantis</option>
			<option value="Stargate: Universe">Stargate: Universe</option>
		</select>
	</div>

	<svg id="chart" class="mb-8"></svg>

	<p class="text-center text-lg text-gray-600 max-w-5xl">
		The above bar chart displays the trends of top keywords across the various documents. It is interesting to see how
		certain key terms like <b>power</b> and <b>energy</b> show up and drop off in various line counts. This indicates
		a certain flux in the text - where the focus may be elaboration, as opposed explicit descriptions of the main
		topics. Some plot-related keywords show up earlier in the text as well, and then drop off as the answer progresses.
		Hovering shows a move specific breakdown.
	</p>
</section>
