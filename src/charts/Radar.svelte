<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import data from '$lib/radar.json';

	onMount(() => {
		updateCharts();

		function drawPolarChart(series, data, containerId, title) {
			const width = 500;
			const height = 500;
			const radius = 400 / 2;
			const maxDataValue = 50;
			const color = d3.scaleOrdinal(d3.schemeCategory10);

			d3.select(containerId).html("");

			const svg = d3.select(containerId)
				.append("svg")
				.attr("width", width)
				.attr("height", height + 40)
				.append("g")
				.attr("transform", `translate(${width / 2},${height / 2 + 40})`);

			svg.append("text")
				.attr("x", 0)
				.attr("y", -height / 2 + - 10)
				.attr("text-anchor", "middle")
				.style("font-size", "16px")
				.style("font-weight", "bold")
				.text(title);

			const categories = ["offensive", "defensive", "transportation", "communication"];
			const angleSlice = (2 * Math.PI) / categories.length;
			const gridLevels = 5;

			for (let level = 1; level <= gridLevels; level++) {
				const levelRadius = (radius / gridLevels) * level;
				svg.append("circle")
					.attr("cx", 0)
					.attr("cy", 0)
					.attr("r", levelRadius)
					.attr("fill", "none")
					.attr("stroke", "#CDCDCD")
					.attr("stroke-width", 0.75);

				svg.append("text")
					.attr("x", 4)
					.attr("y", -levelRadius)
					.attr("dy", "0.4em")
					.style("font-size", "10px")
					.style("fill", "#737373")
					.text(`${(level / gridLevels) * maxDataValue}%`);
			}

			categories.forEach((cat, i) => {
				const angle = angleSlice * i - Math.PI / 2;
				svg.append("line")
					.attr("x1", 0)
					.attr("y1", 0)
					.attr("x2", radius * Math.cos(angle))
					.attr("y2", radius * Math.sin(angle))
					.attr("stroke", "#CDCDCD")
					.attr("stroke-width", 1);

				svg.append("text")
					.attr("x", (radius - 10) * Math.cos(angle))
					.attr("y", (radius + 15) * Math.sin(angle))
					.style("font-size", "12px")
					.style("text-anchor", "middle")
					.style("fill", "#333")
					.text(cat.charAt(0).toUpperCase() + cat.slice(1));
			});

			data[series].forEach((techData, idx) => {
				const points = categories.map((cat, i) => {
					const angle = angleSlice * i - Math.PI / 2;
					return [
						(radius * (techData.usage[cat] / maxDataValue)) * Math.cos(angle),
						(radius * (techData.usage[cat] / maxDataValue)) * Math.sin(angle)
					];
				});

				const line = svg.append("path")
					.datum(points)
					.attr("d", d3.line().curve(d3.curveLinearClosed))
					.attr("fill", color(idx))
					.attr("fill-opacity", 0.1)
					.attr("stroke", color(idx))
					.attr("stroke-width", 2)
					.attr("class", `tech-line-${idx}`);

				const label = svg.append("text")
					.attr("x", 0)
					.attr("y", -radius - 15 - idx * 15)
					.style("font-size", "12px")
					.style("text-anchor", "middle")
					.style("fill", color(idx))
					.attr("class", `tech-label-${idx}`);

				line.on("mouseover", () => {
					svg.selectAll("path").style("opacity", 0.1);
					svg.selectAll("text").style("opacity", 0.1);

					line.style("opacity", 1);
					label.style("opacity", 1);

					svg.append("text")
						.attr("x", 0)
						.attr("y", -radius - 10)
						.attr("class", `hover-label-${idx}`)
						.style("font-size", "12px")
						.style("fill", "#333")
						.style("text-anchor", "middle")
						.text(`${techData.tech}`);

					categories.forEach((cat, i) => {
						svg.append("text")
							.attr("x", points[i][0])
							.attr("y", points[i][1] - 5)
							.attr("class", `hover-text-${idx}`)
							.style("font-size", "10px")
							.style("fill", "#333")
							.style("text-anchor", "middle")
							.text(`${techData.usage[cat].toFixed(2)}%`);
					});
				}).on("mouseout", () => {
					svg.selectAll("path").style("opacity", 1);
					svg.selectAll("text").style("opacity", 1);
					svg.selectAll(`.hover-text-${idx}`).remove();
					svg.selectAll(`.hover-label-${idx}`).remove();
				});
			});
		}

		function updateCharts() {
			drawPolarChart("SG-1", data, "#chart-sg1", "Stargate: SG-1");
			drawPolarChart("Atlantis", data, "#chart-atlantis", "Stargate: Atlantis");
			drawPolarChart("Universe", data, "#chart-universe", "Stargate: Universe");
		}
	});
</script>

<style>
    .chart-container {
				width: auto;
				height: auto;
    }
</style>

<section class="flex flex-col items-center justify-center min-h-screen bg-blue-50 text-gray-800 px-6">
	<article class="flex items-center justify-center space-x-10 mb-8">
		<div id="chart-sg1" class="chart-container"></div>
		<div id="chart-atlantis" class="chart-container"></div>
		<div id="chart-universe" class="chart-container"></div>
	</article>

	<p class="text-center text-lg text-gray-600 max-w-5xl">
		The radar charts above list the key power sources present in each response. Using HuggingFace APIs, the key entities
		were classified based on application - defensive, offensive, transportation, or other. Of note is the fact that
		the first response contained many more entities than the others - suggesting a stronger noise/randomness factor from
		ChatGPT (the system sometimes is very verbose). From a classification standpoint, it is interesting to see that
		most key terms are not classified "too much" in any one direction - suggesting a sort of "neutrality" (semantically)
		when it comes to text generation. This is further backed up by the discarded sentiment analysis - the text looks to
		be fairly neutral across the board.
	</p>
</section>