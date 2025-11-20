<script lang="ts">
  import { QUERY_OPEN_TYPES, QUERY_TYPES, type Query } from "src/types";
  import { onDestroy } from "svelte";

  export let query: Query;
  export let handleChange: (query: Query) => void;

  let advancedEdit = query.type !== "Default";
  let previewQuery = "";
  let queryDescription = "";
  let queryPlaceholder = "";

  // 默认模式表单字段
  let defaultPath = "";
  let defaultFile = "";
  let includeTags = "";
  let excludeTags = "";

  // Regex 校验
  let regexError = "";

  const parseDefaultQuery = (value) => {
    const pathMatch = /path:(.*?)(tag:|file:|$)/.exec(value);
    const fileMatch = /file:(.*?)(tag:|path:|$)/.exec(value);
    const tagMatch = /tag:(.*?)(file:|path:|$)/.exec(value);
    defaultPath = pathMatch ? pathMatch[1].trim() : "";
    defaultFile = fileMatch ? fileMatch[1].trim() : "";
    const tags = tagMatch ? tagMatch[1].trim().split(" ").filter(Boolean) : [];
    const included = [];
    const excluded = [];
    tags.forEach((tag) => {
      if (tag.startsWith("!")) {
        excluded.push(tag.replace("!", "").replace(/^#/, ""));
      } else {
        included.push(tag.replace(/^#/, ""));
      }
    });
    includeTags = included.join(" ");
    excludeTags = excluded.join(" ");
  };

  const buildDefaultQuery = (pathValue, fileValue, includeValue, excludeValue) => {
    const chunks = [];
    if (pathValue.trim()) chunks.push(`path: ${pathValue.trim()}`);
    if (fileValue.trim()) chunks.push(`file: ${fileValue.trim()}`);
    const include = includeValue
      .split(" ")
      .map((t) => t.trim())
      .filter(Boolean)
      .map((t) => `#${t}`);
    const exclude = excludeValue
      .split(" ")
      .map((t) => t.trim())
      .filter(Boolean)
      .map((t) => `!#${t}`);
    if (include.length || exclude.length) {
      chunks.push(`tag: ${[...include, ...exclude].join(" ")}`);
    }
    return chunks.join(" ").trim();
  };

  if (query.type === "Default" && !advancedEdit) {
    parseDefaultQuery(query.query || "");
  }

  $: if (query.type === "Default" && !advancedEdit) {
    const built = buildDefaultQuery(defaultPath, defaultFile, includeTags, excludeTags);
    if (query.query !== built) {
      query.query = built;
      handleChange(query);
    }
    previewQuery = built;
  } else {
    previewQuery = query.query;
  }

  $: if (query.type === "Regex") {
    regexError = "";
    try {
      // eslint-disable-next-line no-new
      new RegExp(query.query);
    } catch (error) {
      regexError = error instanceof Error ? error.message : "正则格式错误";
    }
  }

  $: {
    if (query.type === "Dataview") {
      queryDescription = "只支持 Dataview 的 LIST 查询。";
      queryPlaceholder = "LIST\n...";
    } else if (query.type === "Regex") {
      queryDescription = "使用正则表达式匹配路径。";
      queryPlaceholder = "正则表达式...";
    } else {
      queryDescription = "支持 path / file / tag 等条件。";
      queryPlaceholder = "path: ... file: ... tag: ...";
    }
  }

  const emitHandleChange = () => {
    handleChange(query);
  };

  onDestroy(() => {
    emitHandleChange();
  });

  const copyText = async (value) => {
    if (!value) return;
    try {
      await navigator.clipboard.writeText(value);
      new Notice("已复制到剪贴板");
    } catch (error) {
      new Notice("复制失败，请手动复制");
    }
  };

  const copyExample = async (value) => {
    await copyText(value);
  };

  const handleRegexInput = (value) => {
    query.query = value;
    regexError = "";
    try {
      // eslint-disable-next-line no-new
      new RegExp(value);
    } catch (error) {
      regexError = error instanceof Error ? error.message : "正则格式错误";
    }
    emitHandleChange();
  };

  const handleTypeChange = (value) => {
    query.type = value;
    if (value === "Default") {
      advancedEdit = false;
      parseDefaultQuery(query.query || "");
    } else {
      advancedEdit = true;
    }
    regexError = "";
    handleChange(query);
  };

  const toggleAdvancedMode = () => {
    advancedEdit = !advancedEdit;
    if (!advancedEdit && query.type === "Default") {
      parseDefaultQuery(query.query || "");
    }
  };
</script>

<h2 class="title">
	<input
		class="title__input"
		type="text"
		placeholder="查询名称"
		bind:value={query.name}
		on:input={emitHandleChange}
	/>
</h2>

<div class="edit-query">
	<section class="section-card">
		<div class="section-card__header">
			<div class="section-card__title">基础设置</div>
			<div class="section-card__note">命令与打开行为</div>
		</div>

		<div class="form-grid">
			<!-- Create command -->
			<div class="setting-item mod-toggle">
				<div class="setting-item-info">
					<div class="setting-item-name">创建命令</div>
					<div class="setting-item-description">
						将查询添加为可执行命令，并在命令面板中可用。
					</div>
				</div>
				<div class="setting-item-control">
					<div
						class="checkbox-container"
						class:is-enabled={query.createCommand}
						role="switch"
						tabindex="0"
						aria-checked={query.createCommand}
						on:click={() => {
							query.createCommand = !query.createCommand;
							emitHandleChange();
						}}
						on:keydown={(e) => {
							if (e.key === "Enter" || e.key === " ") {
								query.createCommand = !query.createCommand;
								emitHandleChange();
								e.preventDefault();
							}
						}}
					>
						<input
							type="checkbox"
							tabindex="0"
							bind:checked={query.createCommand}
							on:change={emitHandleChange}
						/>
					</div>
				</div>
			</div>

			<!-- Use excluded folders -->
			<div class="setting-item mod-toggle">
				<div class="setting-item-info">
					<div class="setting-item-name">使用禁用的文件夹</div>
					<div class="setting-item-description">
						沿用设置里的禁用文件夹列表。
					</div>
				</div>
				<div class="setting-item-control">
					<div
						class="checkbox-container"
						class:is-enabled={query.useDisabledFolders}
					>
						<input
							type="checkbox"
							tabindex="0"
							bind:checked={query.useDisabledFolders}
							on:change={emitHandleChange}
						/>
					</div>
				</div>
			</div>

			<!-- Open position -->
			<div class="setting-item">
				<div class="setting-item-info">
					<div class="setting-item-name">打开位置</div>
					<div class="setting-item-description">选择文件打开的位置。</div>
				</div>
				<div class="setting-item-control">
					<select
						class="dropdown"
						bind:value={query.openType}
						on:change={emitHandleChange}
					>
						{#each QUERY_OPEN_TYPES as type}
							<option value={type}>{type}</option>
						{/each}
					</select>
				</div>
			</div>

			<!-- Query type -->
			<div class="setting-item">
				<div class="setting-item-info">
					<div class="setting-item-name">查询类型</div>
					<div class="setting-item-description">
						选择 Regex、Dataview 或默认语法。
					</div>
				</div>
				<div class="setting-item-control">
					<select
						class="dropdown"
						bind:value={query.type}
						on:change={(e) => handleTypeChange(e.currentTarget.value)}
					>
						{#each QUERY_TYPES as type}
							<option value={type}>{type}</option>
						{/each}
					</select>
				</div>
			</div>
		</div>
	</section>

	<section class="section-card">
		<div class="section-card__header section-card__header--tight">
			<div>
				<div class="section-card__title">查询内容</div>
				<div class="section-card__note">{queryDescription}</div>
			</div>
			<div class="query-actions">
				{#if query.type === "Default"}
					<button
						type="button"
						class="toggle-mode"
						on:click={toggleAdvancedMode}
					>
						{advancedEdit ? "返回表单模式" : "高级编辑"}
					</button>
				{/if}
			</div>
		</div>

		<div class="query-body">
			{#if query.type === "Default" && !advancedEdit}
				<div class="builder">
					<label>
						<span>路径包含</span>
						<input
							type="text"
							bind:value={defaultPath}
							placeholder="如 templates/ 或 /"
						/>
					</label>
					<label>
						<span>文件名包含</span>
						<input
							type="text"
							bind:value={defaultFile}
							placeholder="如 Note"
						/>
					</label>
					<label>
						<span>包含标签（空格分隔）</span>
						<input
							type="text"
							bind:value={includeTags}
							placeholder="idea project"
						/>
					</label>
					<label>
						<span>排除标签（空格分隔）</span>
						<input
							type="text"
							bind:value={excludeTags}
							placeholder="complete draft"
						/>
					</label>
				</div>
			{:else}
				{#if query.type === "Regex"}
					<textarea
						bind:value={query.query}
						rows={8}
						on:input={(e) => handleRegexInput(e.currentTarget.value)}
						placeholder={queryPlaceholder}
					/>
					{#if regexError}
						<div class="error">{regexError}</div>
					{/if}
				{:else}
					<textarea
						bind:value={query.query}
						rows={12}
						on:input={emitHandleChange}
						placeholder={queryPlaceholder}
					/>
				{/if}
			{/if}

			<div class="preview">
				<div class="preview__label">生成的查询</div>
				<code class="preview__code">
					{previewQuery || "填写路径/文件名/标签后自动生成查询"}
				</code>
			</div>
		</div>
	</section>
</div>

<style>
	.title {
		text-align: center;
	}

	.title__input {
		text-align: center;
		background: transparent;
		padding: 0;
		border: none;
		font-size: var(--h2-size);
		width: 100%;
	}

	.edit-query {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		margin-top: 0.5rem;
	}

	.section-card {
		border: 1px solid var(--background-modifier-border);
		border-radius: var(--radius-l);
		padding: var(--size-4-3);
		background: var(--background-secondary);
		display: grid;
		gap: 0.6rem;
	}

	.section-card__header {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		gap: 0.75rem;
	}

	.section-card__header--tight {
		align-items: center;
	}

	.section-card__title {
		font-weight: 700;
	}

	.section-card__note {
		color: var(--text-muted);
		font-size: 0.95rem;
		margin-top: 0.15rem;
	}

	.form-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
		gap: 0.75rem;
	}

	.query-actions {
		display: flex;
		justify-content: flex-end;
		align-items: center;
		gap: 0.5rem;
	}

	.toggle-mode {
		border: 1px solid var(--background-modifier-border);
		border-radius: var(--radius-s);
		padding: 0.35rem 0.65rem;
		background: var(--background-primary);
		color: var(--text-muted);
	}

	.toggle-mode:hover {
		color: var(--text-normal);
	}

	.query-body {
		display: grid;
		gap: 0.9rem;
	}

	.builder {
		display: grid;
		gap: 0.75rem;
	}

	.builder label {
		display: grid;
		gap: 0.35rem;
	}

	.preview {
		display: grid;
		gap: 0.25rem;
		align-items: center;
	}

	.preview__label {
		color: var(--text-muted);
		font-size: 0.9em;
	}

	.preview__code {
		display: block;
		background: var(--background-secondary);
		padding: var(--size-2-3);
		border-radius: var(--radius-s);
		white-space: pre-wrap;
	}

	.error {
		color: var(--text-error);
		font-size: 0.9em;
	}

	@media (max-width: 720px) {
		.section-card__header {
			flex-direction: column;
		}
	}

	:global(.arn-edit-modal) {
		position: relative;
	}

	:global(.arn-edit-modal .modal-close-button) {
		position: sticky;
		top: var(--size-2-2, 8px);
		right: var(--size-2-2, 8px);
		align-self: flex-end;
		margin-left: auto;
		z-index: 2;
	}
</style>
