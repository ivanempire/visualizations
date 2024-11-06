<script>
	import { onMount } from "svelte";
	import * as d3 from "d3";
	import data from "$lib/dumbbell.json";

	let selectedDocument = "Stargate: SG-1";

	onMount(() => {
		const svg = d3.select("#dumbbell-chart")
			.attr("width", 1100)
			.attr("height", 800);

		const margin = { top: 60, right: 30, bottom: 70, left: 100 };
		const width = 1100 - margin.left - margin.right;
		const height = 800 - margin.top - margin.bottom;

		const chartGroup = svg.append("g")
			.attr("transform", `translate(${margin.left},${margin.top})`);

		const x = d3.scaleLinear().range([0, width]);
		const y = d3.scaleBand().range([0, height]).padding(0.5);

		const xAxisGroup = chartGroup.append("g")
			.attr("transform", `translate(0,${height})`);
		const yAxisGroup = chartGroup.append("g");

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
			const documentData = data[doc].filter(d => d.minWordLength !== null && d.maxWordLength !== null);

			x.domain([0, d3.max(documentData, d => d.maxWordLength)]).nice();
			y.domain(documentData.map(d => d.lineBin));

			xAxisGroup.call(d3.axisBottom(x).ticks(5).tickFormat(d => d + " chars"))
				.selectAll("text")
				.style("font-size", "14px");

			svg.select(".x-axis-label").remove();
			svg.append("text")
				.attr("class", "x-axis-label")
				.attr("x", width / 2 + margin.left)
				.attr("y", height + margin.top + 40)
				.style("text-anchor", "middle")
				.style("font-size", "16px")
				.text("Word Length (characters)");

			yAxisGroup.call(d3.axisLeft(y).tickFormat(d => "Lines " + d))
				.selectAll("text")
				.style("font-size", "14px");

			svg.select(".y-axis-label").remove();
			svg.append("text")
				.attr("class", "y-axis-label")
				.attr("transform", "rotate(-90)")
				.attr("y", margin.left / 2 - 40)
				.attr("x", -height / 2)
				.attr("text-anchor", "middle")
				.style("font-size", "16px")
				.text("Line Bins");

			// Title
			svg.select(".chart-title").remove();
			svg.append("text")
				.attr("class", "chart-title")
				.attr("x", width / 2 + margin.left)
				.attr("y", margin.top / 2)
				.attr("text-anchor", "middle")
				.style("font-size", "20px")
				.style("font-weight", "bold")
				.text("Minimum and Maximum Word Lengths Across Line Bins");

			const lines = chartGroup.selectAll(".line").data(documentData, d => d.lineBin);

			lines.enter().append("line")
				.attr("class", "line")
				.attr("x1", d => x(d.minWordLength))
				.attr("x2", d => x(d.maxWordLength))
				.attr("y1", d => y(d.lineBin) + y.bandwidth() / 2)
				.attr("y2", d => y(d.lineBin) + y.bandwidth() / 2)
				.attr("stroke", "#999")
				.style("opacity", 0)
				.transition().duration(500)
				.style("opacity", 1);

			lines.exit().remove();

			const minDots = chartGroup.selectAll(".min-dot").data(documentData, d => d.lineBin);

			minDots.enter().append("circle")
				.attr("class", "min-dot")
				.attr("r", 6)
				.attr("cx", d => x(d.minWordLength))
				.attr("cy", d => y(d.lineBin) + y.bandwidth() / 2)
				.attr("fill", "#1f77b4")
				.style("opacity", 0)
				.transition().duration(500)
				.style("opacity", 1);

			minDots.exit().remove();

			const maxDots = chartGroup.selectAll(".max-dot").data(documentData, d => d.lineBin);

			maxDots.enter().append("circle")
				.attr("class", "max-dot")
				.attr("r", 6)
				.attr("cx", d => x(d.maxWordLength))
				.attr("cy", d => y(d.lineBin) + y.bandwidth() / 2)
				.attr("fill", "#ff7f0e")
				.style("opacity", 0)
				.transition().duration(500)
				.style("opacity", 1);

			maxDots.exit().remove();

			chartGroup.selectAll("circle")
				.on("mouseover", function(event, d) {
					tooltip.style("visibility", "visible")
						.text(`Line Bin: ${d.lineBin} | Min Length: ${d.minWordLength} | Max Length: ${d.maxWordLength}`);
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

		document.getElementById("dumbbellSelector").addEventListener("change", function() {
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
		<select id="dumbbellSelector" class="ml-2 p-1 rounded border border-gray-300">
			<option value="Stargate: SG-1">Stargate: SG-1</option>
			<option value="Stargate: Atlantis">Stargate: Atlantis</option>
			<option value="Stargate: Universe">Stargate: Universe</option>
		</select>
	</div>

	<svg id="dumbbell-chart" class="mb-8"></svg>

	<p class="text-center text-lg text-gray-600 max-w-5xl">
		Finally, we take a look at the minimum and maximum word lengths in each response, based on binned line counts.
		Noteworthy is the fact that even after removing stop words during preprocessing, the range of word
		lengths is very consistent across responses. Hovering on the caps will show the plotted numerical data.
	</p>
</section>
