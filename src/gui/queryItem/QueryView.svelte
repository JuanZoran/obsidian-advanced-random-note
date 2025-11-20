<script lang="ts">
	import { Query } from "src/types";
	import QueryList from "./QueryList.svelte";
	import {
		createQuery,
		getQueryCommandId,
		addOrRemoveQueryCommand,
	} from "src/utilities";
	import { EditQueryModal } from "src/gui/modals/EditQueryModal/editQueryModal";
	import AdvancedRandomNote from "src/main";

	export let plugin: AdvancedRandomNote;
	export let queries: Query[];
	export let saveQueries: (queries: Query[]) => void;
	let queryFormName = "";
	let newQueryInput: HTMLInputElement;

	const deleteQuery = (e: any) => {
		const query: Query = e.detail.query;
		queries = queries.filter((q) => q.id !== query.id);
		plugin.removeQueryCommand(query);
		saveQueries(queries);
	};

	const editQuery = (e: any) => {
		const query: Query = e.detail.query;

		const modal = new EditQueryModal(plugin.app, query, (query) => {
			addOrRemoveQueryCommand(plugin, query);
			queries = queries;
			saveQueries(queries);
		});
		modal.open();

		queries = queries;
		saveQueries(queries);
	};

	const toggleCommandForQuery = (e: any) => {
		const query: Query = e.detail.query;
		query.createCommand = !query.createCommand;
		queries = queries;
		addOrRemoveQueryCommand(plugin, query);
		saveQueries(queries);
	};

	const duplicateQuery = (e: any) => {
		const query: Query = e.detail.query;
		const queryClone = structuredClone(query);
		queryClone.id = getQueryCommandId();
		queryClone.name = queryClone.name + " (Copy)";
		queries = [...queries, queryClone];
		saveQueries(queries);
		if (query.createCommand) {
			// Register command for the new query clone, not the original
			plugin.addQueryCommand(queryClone);
		}
	};

	const handleSubmit = () => {
		if (queryFormName.trim() === "") return;

		queries = [...queries, createQuery(queryFormName, "")];
		queryFormName = "";
		saveQueries(queries);
	};

	const saveChanges = () => {
		queries = queries;
		saveQueries(queries);
	};

	const focusNewQueryInput = () => {
		if (newQueryInput) {
			newQueryInput.focus();
			newQueryInput.select();
			newQueryInput.scrollIntoView({ block: "center", behavior: "smooth" });
		}
	};
</script>

<section class="query-shell">
	<header class="query-shell__header">
		<div>
			<h3 class="query-shell__title">查询命令</h3>
			<p class="query-shell__hint">配置命令并快速进入编辑窗口</p>
		</div>
		<form class="query-form" on:submit|preventDefault={handleSubmit}>
			<input
				type="text"
				placeholder="新的查询名称"
				bind:value={queryFormName}
				bind:this={newQueryInput}
			/>
			<button class="mod-cta" type="submit">添加</button>
		</form>
	</header>

	<div class="query-panel">
		<div class="query-panel__header">
			<span>已有查询</span>
			<span class="query-panel__count">{queries.length}</span>
		</div>

		<QueryList
			bind:queries
			on:deleteQuery={deleteQuery}
			on:editQuery={editQuery}
			on:toggleCommandForQuery={toggleCommandForQuery}
			on:duplicateQuery={duplicateQuery}
			on:saveChanges={saveChanges}
			on:startCreate={focusNewQueryInput}
		/>
	</div>
</section>

<style>
	.query-shell {
		display: grid;
		gap: 0.85rem;
		padding: var(--size-4-3);
		background: var(--background-primary);
		border: 1px solid var(--background-modifier-border);
		border-radius: var(--radius-l);
		box-shadow: 0 6px 24px rgba(0, 0, 0, 0.12);
	}

	.query-shell__header {
		display: grid;
		grid-template-columns: 1fr auto;
		align-items: center;
		gap: 0.5rem;
	}

	.query-shell__title {
		margin: 0;
		font-size: 1.2rem;
	}

	.query-shell__hint {
		margin: 0.15rem 0 0;
		color: var(--text-muted);
		font-size: 0.95rem;
	}

	.query-panel {
		background: var(--background-secondary);
		border: 1px solid var(--background-modifier-border);
		border-radius: var(--radius-m);
		padding: var(--size-4-3);
		display: grid;
		gap: 0.6rem;
	}

	.query-panel__header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		font-weight: 600;
	}

	.query-panel__count {
		background: var(--background-modifier-border);
		color: var(--text-normal);
		border-radius: var(--radius-s);
		padding: 0.1rem 0.5rem;
		font-size: 0.9rem;
	}

	.query-form {
		display: flex;
		align-items: center;
		gap: var(--size-4-2);
		background: var(--background-secondary);
		border: 1px solid var(--background-modifier-border);
		border-radius: var(--radius-m);
		padding: 0.4rem 0.6rem;
	}

	.query-form input {
		width: 14rem;
	}

	@media (max-width: 780px) {
		.query-shell__header {
			grid-template-columns: 1fr;
			align-items: flex-start;
		}

		.query-form {
			width: 100%;
			justify-content: space-between;
		}

		.query-form input {
			flex: 1;
			min-width: 0;
		}
	}
</style>
