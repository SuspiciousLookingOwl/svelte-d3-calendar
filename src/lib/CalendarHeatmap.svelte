<script lang="ts" context="module">
	export interface Data<T> {
		date: Date;
		value: number;
		data?: T;
	}

	export type ColorSchema = [number, string][];
</script>

<script lang="ts">
	import * as d3 from "d3";
	import { onMount } from "svelte";

	type DataType = $$Generic;

	interface Data {
		date: Date;
		value: number;
		data?: DataType;
	}

	interface SanitizedData {
		week: string;
		day: string;
		date: Date;
		data?: DataType;
		value: number;
	}

	interface Tooltip {
		left: number;
		top: number;
		data: DataType | null;
		visible: boolean;
	}

	export let data: Data[] = [];
	export let margin = { top: 50, right: 50, bottom: 0, left: 30 };
	export let colors: ColorSchema = [
		[1, "#EBEDF0"],
		[25, "#9BE9A8"],
		[75, "#30A14E"],
		[100, "#216E39"]
	];
	export let width = 748 - margin.left - margin.right;
	export let height = 142 - margin.top - margin.bottom;

	let chart: HTMLDivElement;
	let tooltip: Tooltip = {
		left: 0,
		top: 0,
		data: null,
		visible: false
	};

	// Convert the date and value to the format that D3 understands
	const sanitize = ({ date, data, value }: Data): SanitizedData => {
		const year = date.getFullYear();
		const onejan = new Date(year, 0, 1);
		let week = Math.ceil(
			((date.getTime() - onejan.getTime()) / 86400000 + onejan.getDay() + 1) / 7
		);

		if (date.getDay() === 6) week--;

		return {
			date,
			data,
			day: days[date.getDay()],
			week: week === 53 && new Date().getFullYear() != year ? `1_${year + 1}` : `${week}_${year}`,
			value
		};
	};

	const days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"] as const;
	const months = [
		"Jan",
		"Feb",
		"Mar",
		"Apr",
		"May",
		"Jun",
		"Jul",
		"Aug",
		"Sep",
		"Oct",
		"Nov",
		"Dec"
	] as const;

	onMount(() => {
		const serialized = data.map(sanitize);
		const weeks = [...new Set(serialized.map((s) => s.week))];

		// Base
		const svg = d3
			.select(chart)
			.append("svg")
			.attr("width", width + margin.left + margin.right)
			.attr("height", height + margin.top + margin.bottom)
			.attr("font-family", "sans-serif")
			.attr("font-size", 10)
			.append("g")
			.attr("transform", `translate(${margin.left},${margin.top})`);

		// Build X & Y scales and axis:
		const x = d3.scaleBand().range([0, width]).domain(weeks).padding(0.18);
		const y = d3.scaleBand().range([height, 0]).domain(days.slice().reverse()).padding(0.2);

		// Left label
		svg.append("g").call(d3.axisLeft(y).tickSize(0).tickPadding(6));

		// Top label
		const monthsLabel = weeks.map((w) => {
			const data = serialized.find((s) => s.week === w && s.date.getDate() === 1);
			if (data) return months[data.date.getMonth()];
			else return (Math.random() + 1).toString(36).substring(7); // will be hidden
		});
		const labels = d3
			.axisTop(d3.scaleBand().range([0, width]).domain(monthsLabel))
			.tickValues(monthsLabel.filter((d) => d.length === 3))
			.tickSize(0)
			.tickPadding(6);
		svg.append("g").attr("transform", `translate(0, 0)`).call(labels);

		// Color scale
		const domain = colors.map((c) => c[0]);
		const range = colors.map((c) => c[1]);

		const colorScale = d3
			.scaleLinear()
			.range(range as any)
			.domain(domain);

		// Render
		svg
			.selectAll()
			.data(serialized, (d) => d.week + ":" + d.day)
			.join("rect")
			.attr("rx", 2)
			.attr("ry", 2)
			.attr("x", (d) => x(d.week))
			.attr("y", (d) => y(d.day))
			.attr("width", x.bandwidth())
			.attr("height", y.bandwidth())
			.style("fill", (d) => colorScale(d.value))
			.on("mouseover", () => (tooltip.visible = true))
			.on("mousemove", (e: MouseEvent, { data }: SanitizedData) => {
				tooltip = {
					left: e.pageX + 5,
					top: e.pageY - 25,
					data,
					visible: true
				};
			})
			.on("mouseleave", () => (tooltip.visible = false));
	});
</script>

<div
	class="fixed"
	class:visible={tooltip.visible}
	class:invisible={!tooltip.visible}
	style={`left: ${tooltip.left}px; top: ${tooltip.top}px`}
>
	{#if tooltip.data}
		<slot name="tooltip" data={tooltip.data} />
	{/if}
</div>

<div bind:this={chart} />

<style>
	.fixed {
		position: fixed;
		z-index: 1000;
	}

	.visible {
		visibility: visible;
	}

	.invisible {
		visibility: hidden;
	}
</style>
