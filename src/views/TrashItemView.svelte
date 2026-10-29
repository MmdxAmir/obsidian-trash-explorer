<script lang="ts">
	import { formatItemStats } from "../formatting";
	import type { TrashItem } from "../models";
	import type { TrashExplorerViewNode } from "../view";
	import TrashItemView from "./TrashItemView.svelte";

	interface Props {
		viewNode: TrashExplorerViewNode;
		restoreItem: (item: TrashItem) => void;
		deleteItem: (item: TrashItem) => void;
	}

	let { viewNode, restoreItem, deleteItem }: Props = $props();

	const item = $derived(viewNode.item);
	const itemStats = $derived(formatItemStats(item));
	const hasChildren = $derived(viewNode.nodes.length > 0);
	let expanded = $state(true);
</script>

<div class="tree-node">
	<div class="trash-item" class:has-children={hasChildren}>
		{#if hasChildren}
			<button
				class="disclosure"
				aria-label={expanded ? `Collapse ${item.basename}` : `Expand ${item.basename}`}
				aria-expanded={expanded}
				title={expanded ? "Collapse folder" : "Expand folder"}
				onclick={() => (expanded = !expanded)}
			>
				<svg viewBox="0 0 16 16" aria-hidden="true">
					<path d={expanded ? "m4 6 4 4 4-4" : "m6 4 4 4-4 4"} />
				</svg>
			</button>
		{:else}
			<span class="disclosure-placeholder" aria-hidden="true"></span>
		{/if}

		<div class="textcontainer">
			<div class="name" aria-label={item.basename}>{item.basename}</div>
			<div class="info">{itemStats}</div>
		</div>

		<div class="buttons">
			<button class="action-button" aria-label="Restore" title="Restore" onclick={() => restoreItem(item)}>
				<svg viewBox="0 0 24 24" aria-hidden="true">
					<path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
					<path d="M3 3v5h5" />
				</svg>
			</button>
			<button class="action-button delete-button" aria-label="Delete permanently" title="Delete permanently" onclick={() => deleteItem(item)}>
				<svg viewBox="0 0 24 24" aria-hidden="true">
					<path d="M3 6h18" />
					<path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6" />
					<path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2" />
					<line x1="10" x2="10" y1="11" y2="17" />
					<line x1="14" x2="14" y1="11" y2="17" />
				</svg>
			</button>
		</div>
	</div>

	{#if hasChildren && expanded}
		<div class="children">
			{#each viewNode.nodes as childNode (childNode.item.path)}
				<TrashItemView viewNode={childNode} {restoreItem} {deleteItem} />
			{/each}
		</div>
	{/if}
</div>

<style>
	.tree-node {
		position: relative;
	}

	.trash-item {
		position: relative;
		min-height: 34px;
		box-sizing: border-box;
		border-radius: var(--radius-s);
		color: var(--nav-item-color);
		font-size: var(--nav-item-size);
		font-weight: var(--nav-item-weight);
		line-height: var(--line-height-tight);
		padding: 4px 4px 4px 0;
		display: flex;
		align-items: center;
		gap: 4px;
	}

	.trash-item:hover,
	.trash-item:focus-within {
		background-color: var(--nav-item-background-hover);
	}

	.textcontainer {
		min-width: 0;
		flex: 1;
		display: flex;
		flex-direction: column;
		overflow: hidden;
	}

	.name {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.info {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		color: var(--text-faint);
		font-size: var(--font-ui-small);
	}

	.disclosure,
	.disclosure-placeholder {
		flex: 0 0 20px;
		width: 20px;
		height: 24px;
		display: inline-flex;
		align-items: center;
		justify-content: center;
	}

	.disclosure {
		padding: 0;
		border: 0;
		background: transparent;
		box-shadow: none;
		color: var(--text-muted);
		cursor: pointer;
	}

	.disclosure:hover {
		color: var(--text-normal);
		background: transparent;
	}

	.disclosure svg {
		width: 14px;
		height: 14px;
		fill: none;
		stroke: currentColor;
		stroke-width: 1.8;
		stroke-linecap: round;
		stroke-linejoin: round;
	}

	.buttons {
		flex: 0 0 auto;
		display: flex;
		align-items: center;
		gap: 2px;
		margin-left: auto;
		opacity: 0;
		pointer-events: none;
		transition: opacity 100ms ease;
	}

	.trash-item:hover .buttons,
	.trash-item:focus-within .buttons {
		opacity: 1;
		pointer-events: auto;
	}

	.action-button {
		width: 24px;
		height: 24px;
		padding: 3px;
		border: 0;
		border-radius: 0;
		background: transparent;
		box-shadow: none;
		color: var(--text-muted);
		display: inline-flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
	}

	.action-button:hover {
		background: transparent;
		color: var(--text-normal);
	}

	.delete-button:hover {
		color: var(--text-error);
	}

	.action-button svg {
		width: 16px;
		height: 16px;
		fill: none;
		stroke: currentColor;
		stroke-width: 1.8;
		stroke-linecap: round;
		stroke-linejoin: round;
	}

	.children > :global(.tree-node)::before {
		content: "";
		position: absolute;
		top: 17px;
		left: -11px;
		width: 10px;
		border-top: 1px solid var(--tree-connector-color, var(--background-modifier-border));
	}

	.children {
		position: relative;
		margin-left: 10px;
		padding-left: 10px;
		border-left: 1px solid var(--tree-connector-color, var(--background-modifier-border));
	}

	/* Keep action buttons available on touch devices that have no hover state. */
	@media (hover: none) {
		.buttons {
			opacity: 1;
			pointer-events: auto;
		}
	}
</style>
