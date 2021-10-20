# svelte-d3-calendar

**THIS IS NOT A COMPONENT LIBRARY**, this is just an example of how to make a GitHub-like calendar heatmap component on Svelte using [D3](https://d3js.org/)

**Check the component file [here](https://github.com/SuspiciousLookingOwl/svelte-d3-calendar/tree/main/src/lib/CalendarHeatmap.svelte)**

[**Svelte REPL**](https://svelte.dev/repl/aa417cca2aff48ba804f1e7c32086271?version=3.44.0) (non-TS)

---

Example:

```svelte
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
```

Output: 

<img src="https://i.ibb.co/zmH6zhB/image.png">

---

**Feel free to use this as you wish, or make a component library out of this**
