<script lang="ts">
	import { Query } from "src/types";
	import { createEventDispatcher } from "svelte";
	import { Icon } from "svelte-awesome";
	import { faPlusCircle } from "@fortawesome/free-solid-svg-icons";
	import QueryItem from "./QueryItem.svelte";

	export let queries: Query[];

	const dispatch = createEventDispatcher();

	const startCreate = () => {
		dispatch("startCreate");
	};
</script>

<div class="query-list">
	{#each queries as query}
		<QueryItem
			bind:query
			on:editQuery
			on:deleteQuery
			on:duplicateQuery
			on:toggleCommandForQuery
			on:saveChanges
		/>
	{:else}
		<div class="empty">
			<div class="empty__icon">
				<Icon data={faPlusCircle} />
			</div>
			<div class="empty__text">还没有查询命令</div>
			<button type="button" class="empty__cta mod-cta" on:click={startCreate}>
				新建查询
			</button>
		</div>
	{/each}
</div>

<style>
	.query-list {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.empty {
		display: grid;
		place-items: center;
		gap: 0.35rem;
		padding: var(--size-4-4);
		border: 1px dashed var(--background-modifier-border);
		border-radius: var(--radius-m);
		color: var(--text-muted);
		background: var(--background-primary);
		text-align: center;
	}

	.empty__icon {
		font-size: 1.3rem;
		color: var(--text-faint);
	}

	.empty__text {
		font-weight: 600;
		color: var(--text-normal);
	}

	.empty__cta {
		padding: 0.35rem 0.8rem;
	}
</style>
