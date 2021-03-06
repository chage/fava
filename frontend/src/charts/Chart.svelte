<script>
  import { setContext } from "svelte";
  import { writable } from "svelte/store";

  import { _ } from "../helpers";
  import { keyboardShortcut } from "../keyboard-shortcuts";
  import { chartCurrency, chartMode, showCharts } from "../stores/chart";

  import ChartLegend from "./ChartLegend.svelte";
  import BarChart from "./BarChart.svelte";
  import HierarchyContainer from "./HierarchyContainer.svelte";
  import LineChart from "./LineChart.svelte";
  import ScatterPlot from "./ScatterPlot.svelte";

  // The chart to render.
  export let chart;

  // Width of the chart.
  let width;

  const currencies = writable([]);
  const legend = writable([]);
  setContext("chart-legend", legend);
  setContext("chart-currencies", currencies);

  $: if (chart) {
    // Reset the chart legend on chart change.
    legend.set([]);
  }

  const components = {
    barchart: BarChart,
    hierarchy: HierarchyContainer,
    linechart: LineChart,
    scatterplot: ScatterPlot,
  };
</script>

<form class="flex-row">
  {#if $showCharts}
    <div>
      <ChartLegend legend={$legend} />
    </div>
    <span class="spacer" />
    {#if chart.type === 'hierarchy'}
      <select bind:value={$chartCurrency} hidden={$chartMode !== 'treemap'}>
        {#each $currencies as currency}
          <option value={currency}>{currency}</option>
        {/each}
      </select>
      <span class="chart-mode">
        <label>
          <input type="radio" bind:group={$chartMode} value="treemap" />
          <span class="button">{_('Treemap')}</span>
        </label>
        <label>
          <input type="radio" bind:group={$chartMode} value="sunburst" />
          <span class="button">{_('Sunburst')}</span>
        </label>
      </span>
    {/if}
  {:else}
    <span class="spacer" />
  {/if}
  <slot />
  <button
    type="button"
    on:click={() => {
      showCharts.update((v) => !v);
    }}
    use:keyboardShortcut={'Control+c'}
    class:closed={!$showCharts}
    class="toggle-chart" />
</form>
<div hidden={!$showCharts} bind:clientWidth={width}>
  {#if width}
    {#if components[chart.type]}
      <svelte:component
        this={components[chart.type]}
        data={chart.data}
        tooltipText={chart.tooltipText}
        {width} />
    {:else}Invalid chart: {chart.type}{/if}
  {/if}
</div>
