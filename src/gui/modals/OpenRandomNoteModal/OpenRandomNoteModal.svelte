<script lang="ts">
	import { Query } from "src/types";

	export let queries: Query[];
	export let handleSubmit: (query: Query) => void;
	let selectedQuery: Query | undefined;

	// Default to first query if available to avoid undefined submission
	$: if (!selectedQuery && queries.length > 0) {
		selectedQuery = queries[0];
	}

	const handleFormSubmit = () => {
		if (!selectedQuery) return;
		handleSubmit(selectedQuery);
	};
</script>

<h3>选择随机笔记的查询</h3>

{#if queries.length > 0}
	<form on:submit|preventDefault={handleFormSubmit}>
		<select class="dropdown" bind:value={selectedQuery}>
			{#each queries as query}
				<option value={query}>{query.name}</option>
			{/each}
		</select>
		<button class="mod-cta" type="submit">随机打开笔记</button>
	</form>
{:else}
	<p class="setting-item-description">
		前往设置页创建查询。
	</p>
{/if}

<style>
	form {
		display: flex;
		flex-direction: column;
		gap: 1em;
	}

	button {
		max-width: max-content;
	}
</style>
