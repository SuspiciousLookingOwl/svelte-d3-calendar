<script lang="ts">
	import CalendarHeatmap, { Data, ColorSchema } from "$lib/CalendarHeatmap.svelte";

	interface CustomData {
		date: Date;
		value: number;
		reversed: number;
	}

	// generate data
	let data: Data<CustomData>[] = [...Array(365)]
		.map((_, i) => {
			const date = new Date();
			date.setDate(date.getDate() - i);
			const value = Math.floor(Math.random() * 100);
			return {
				date,
				value,
				data: { date, value, reversed: 100 - value }
			};
		})
		.reverse();

	// create custom color schema
	const colors: ColorSchema = [
		[1, "#EBEDF0"],
		[25, "#9BE9A8"],
		[89, "#30A14E"],
		[90, "#ffc21c"],
		[100, "#ff8833"]
	];
</script>

<CalendarHeatmap {data} {colors}>
	<span slot="tooltip" let:data={d} class="tooltip">
		{d.date.toDateString()} - <b>{d.value}</b> (-{d.reversed})
	</span>
</CalendarHeatmap>

<style>
	.tooltip {
		font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
		background-color: #ededed;
		padding: 4px;
	}
</style>
