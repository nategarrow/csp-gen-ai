<script lang="ts">
	interface Props {
		label: string;
		directive: string;
		sources: string[];
		onAdd: (source: string) => void;
		onRemove: (index: number) => void;
	}

	let { label, directive, sources, onAdd, onRemove }: Props = $props();
	let inputValue = $state('');

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Enter' && inputValue.trim()) {
			onAdd(inputValue.trim());
			inputValue = '';
		}
	}

	function handleAdd() {
		if (inputValue.trim()) {
			onAdd(inputValue.trim());
			inputValue = '';
		}
	}
</script>

<div class="mb-6">
	<label for={directive} class="mb-2 block text-sm font-medium text-subtext-color">
		{label} ({directive})
	</label>

	<div class="mb-2 flex gap-2">
		<input
			id={directive}
			type="text"
			bind:value={inputValue}
			onkeydown={handleKeydown}
			placeholder="Enter source (e.g., https://example.com, 'self', 'unsafe-inline')"
			class="flex-1 rounded-md border border-neutral-border bg-neutral-100 px-3 py-2 text-default-font placeholder:text-neutral-400 shadow-sm focus:border-brand-primary focus:ring-2 focus:ring-brand-primary focus:outline-none"
		/>
		<button
			onclick={handleAdd}
			class="rounded-md bg-brand-primary px-4 py-2 text-neutral-950 hover:bg-brand-700 focus:ring-2 focus:ring-brand-primary focus:ring-offset-2 focus:outline-none"
		>
			Add
		</button>
	</div>

	{#if sources.length > 0}
		<div class="space-y-1">
			{#each sources as source, index}
				<div class="flex items-center justify-between bg-neutral-100 px-3 py-2 rounded-md border border-neutral-border">
					<code class="text-sm text-default-font font-monospace-body">{source}</code>
					<button
						onclick={() => onRemove(index)}
						class="text-error-600 hover:text-error-700 focus:outline-none"
						aria-label="Remove {source}"
					>
						<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M6 18L18 6M6 6l12 12"
							></path>
						</svg>
					</button>
				</div>
			{/each}
		</div>
	{/if}
</div>
