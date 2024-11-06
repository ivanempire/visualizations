<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import data from "$lib/sunburst.json";

	onMount(() => {
		const width = 600;
		const radius = width / 2;
		const color = d3.scaleOrdinal(d3.schemePaired);

		const svg = d3.select("#sunburst")
			.attr("width", width)
			.attr("height", width)
			.append("g")
			.attr("transform", `translate(${width / 2},${width / 2})`);

		const partition = d3.partition()
			.size([2 * Math.PI, radius]);

		const root = d3.hierarchy(data)
			.sum(d => d.value)
			.sort((a, b) => b.value - a.value);

		partition(root);

		const arc = d3.arc()
			.startAngle(d => d.x0)
			.endAngle(d => d.x1)
			.innerRadius(d => d.y0)
			.outerRadius(d => d.y1);

		svg.selectAll("path")
			.data(root.descendants())
			.enter().append("path")
			.attr("display", d => d.depth ? null : "none")
			.attr("d", arc)
			.style("fill", d => color((d.children ? d : d.parent).data.name))
			.style("fill-opacity", 0.8)
			.style("stroke", "#fff")
			.style("stroke-width", "1px")
			.on("mouseover", function(event, d) {
				d3.select("#info").text(`${d.ancestors().map(d => d.data.name).reverse().join(" > ")}: ${d.value || ""}`);
				d3.select(this).style("fill-opacity", 1);
			})
			.on("mouseout", function() {
				d3.select("#info").text("");
				d3.select(this).style("fill-opacity", 0.8);
			})
			.on("click", function(event, d) {
				svg.transition()
					.duration(750)
					.tween("scale", function() {
						const xd = d3.interpolate(root.x0, d.x0);
						const yd = d3.interpolate(root.y0, d.y0);
						const yr = d3.interpolate(root.y1, d.y1);
						return t => {
							root.x0 = xd(t);
							root.y0 = yd(t);
							root.y1 = yr(t);
							svg.selectAll("path").attr("d", arc);
						};
					});
			});

		svg.selectAll("text.label")
			.data(root.descendants())
			.enter().append("text")
			.attr("class", "label")
			.attr("transform", d => `translate(${arc.centroid(d)})`)
			.attr("text-anchor", "middle")
			.attr("display", d => (d.data.name === "Nouns" || d.data.name === "Adjectives" || d.data.name === "Verbs") ? null : "none")
			.style("font-size", "16px")
			.style("fill", "#333")
			.text(d => d.data.name);
	});
</script>

<style>
    #info {
        text-align: center;
        margin-top: 10px;
        font-size: 1rem;
        color: #333;
				height:30px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }
</style>

<section class="flex flex-col items-center justify-center min-h-screen bg-blue-50 text-gray-800 px-6">
	<h3 class="text-2xl font-semibold mb-4">Parts of Speech distribution</h3>

	<svg id="sunburst" class="mb-4"></svg>

	<div id="info" class="text-lg text-gray-600">
		Hover over or click on segments to explore.
	</div>

	<p class="text-center text-lg text-gray-600 max-w-5xl">
		The sunburst chart displays the top 10 parts of speech from all 3 documents. What is fascinating to see is the
		overall count - there are many more nouns than other parts of speech combined. This may be attributed to the fact that
		the questions and answers were focused on "factual" information in the context of the shows. The top words do however,
		stick to the general patterns seen in the past charts (which words are popular, and which ones are not).
	</p>
</section>