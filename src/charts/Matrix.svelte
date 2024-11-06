<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import data from '$lib/matrix.json';

	let chartContainerSG1, chartContainerAtlantis, chartContainerUniverse;

	function drawMatrix(container, title, data, allMatrices, matrixId) {
		const margin = { top: 50, right: 50, bottom: 50, left: 70 };
		const width = 300;
		const height = 300;

		const svg = d3.select(container)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);

		const words = Object.keys(data);
		const x = d3.scaleBand().domain(words).range([0, width]).padding(0.05);
		const y = d3.scaleBand().domain(words).range([0, height]).padding(0.05);
		const color = d3.scaleSequential(d3.interpolateReds)
			.domain([0, d3.max(words, w1 => d3.max(words, w2 => data[w1][w2]))]);

		// Title for each matrix
		d3.select(container).insert("div", ":first-child")
			.attr("class", "matrix-title")
			.style("text-align", "center")
			.style("font-size", "16px")
			.style("font-weight", "bold")
			.text(title);

		svg.append('g')
			.attr('transform', `translate(0,${height})`)
			.call(d3.axisBottom(x))
			.selectAll('text')
			.attr('transform', 'rotate(-45)')
			.style('text-anchor', 'end');

		svg.append('g')
			.call(d3.axisLeft(y));

		const cells = svg.selectAll(".cell")
			.data(words.flatMap(word1 => words.map(word2 => ({
				word1,
				word2,
				value: data[word1][word2] || 0
			}))))
			.enter().append("rect")
			.attr("x", d => x(d.word1))
			.attr("y", d => y(d.word2))
			.attr("width", x.bandwidth())
			.attr("height", y.bandwidth())
			.style("fill", d => color(d.value))
			.style("stroke", "#333")
			.style("fill-opacity", 0.8)
			.attr("class", "matrix-cell")
			.on("mouseover", function (event, d) {
				const tooltipText = [];

				// Highlight corresponding cells in each matrix
				allMatrices.forEach(({ matrixData, svgElement, title }) => {
					const correspondingValue = matrixData[d.word1] && matrixData[d.word1][d.word2];
					if (correspondingValue !== undefined) {
						tooltipText.push(`${title}: ${d.word1} - ${d.word2} = ${correspondingValue}`);
						svgElement.selectAll(".matrix-cell")
							.filter(cell => cell.word1 === d.word1 && cell.word2 === d.word2)
							.style("stroke", "green")
							.style("stroke-width", "4px");
					}
				});

				d3.select("#info").text(tooltipText.join(" | "));
			})
			.on("mouseout", function () {
				allMatrices.forEach(({ svgElement }) => {
					svgElement.selectAll(".matrix-cell")
						.style("stroke", "#333")
						.style("stroke-width", "1px");
				});
				d3.select("#info").text("");
			});
	}

	onMount(() => {
		const svgSG1 = d3.select(chartContainerSG1);
		const svgAtlantis = d3.select(chartContainerAtlantis);
		const svgUniverse = d3.select(chartContainerUniverse);

		const allMatrices = [
			{ matrixData: data['Stargate: SG-1'], svgElement: svgSG1, title: 'Stargate: SG-1' },
			{ matrixData: data['Stargate: Atlantis'], svgElement: svgAtlantis, title: 'Stargate: Atlantis' },
			{ matrixData: data['Stargate: Universe'], svgElement: svgUniverse, title: 'Stargate: Universe' }
		];

		drawMatrix(chartContainerSG1, 'Stargate: SG-1', data['Stargate: SG-1'], allMatrices, 'SG1');
		drawMatrix(chartContainerAtlantis, 'Stargate: Atlantis', data['Stargate: Atlantis'], allMatrices, 'Atlantis');
		drawMatrix(chartContainerUniverse, 'Stargate: Universe', data['Stargate: Universe'], allMatrices, 'Universe');
	});
</script>

<style>
    .chart-container {
        width: auto;
        height: auto;
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    #info {
        text-align: center;
				height: 30px;
        font-size: 1rem;
        color: #333;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }
</style>

<section class="flex flex-col items-center justify-center min-h-screen bg-gray-100 text-gray-800 px-6">
	<article class="flex items-start justify-center space-x-10 mb-8">
		<div bind:this={chartContainerSG1} class="chart-container"></div>
		<div bind:this={chartContainerAtlantis} class="chart-container"></div>
		<div bind:this={chartContainerUniverse} class="chart-container"></div>
	</article>

	<div id="info" class="text-lg text-gray-600">
		Hover over cells to see details.
	</div>

	<p class="text-center text-lg text-gray-600 max-w-5xl">
		Hovering over a cell in the above matrices will highlight identical word co-occurrences across responses. From the
		preprocessing done in the Jupyter notebook, we see consistency in which keywords appear the most, as well as which
		keywords they are matched with. Interestingly enough, the pairs and their counts are not consistent across documents.
		This indicates context-specific text based on the document at hand.
	</p>
</section>
