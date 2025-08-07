<script lang="ts">
	import DirectiveInput from '$lib/components/DirectiveInput.svelte';
	import { Select } from 'bits-ui';

	// CSP directive state
	let directives = $state({
		'default-src': [] as string[],
		'script-src': [] as string[],
		'script-src-elem': [] as string[],
		'script-src-attr': [] as string[],
		'style-src': [] as string[],
		'style-src-elem': [] as string[],
		'style-src-attr': [] as string[],
		'img-src': [] as string[],
		'connect-src': [] as string[],
		'font-src': [] as string[],
		'object-src': [] as string[],
		'media-src': [] as string[],
		'frame-src': [] as string[],
		'child-src': [] as string[],
		'worker-src': [] as string[],
		'manifest-src': [] as string[],
		'base-uri': [] as string[],
		'form-action': [] as string[]
	});

	let generatedCSP = $state('');
	let selectedDirective = $state<string>('');

	// Directive configurations
	const directiveConfigs = [
		{ key: 'default-src', label: 'Default Source' },
		{ key: 'script-src', label: 'Script Source' },
		{ key: 'script-src-elem', label: 'Script Source Element' },
		{ key: 'script-src-attr', label: 'Script Source Attribute' },
		{ key: 'style-src', label: 'Style Source' },
		{ key: 'style-src-elem', label: 'Style Source Element' },
		{ key: 'style-src-attr', label: 'Style Source Attribute' },
		{ key: 'img-src', label: 'Image Source' },
		{ key: 'connect-src', label: 'Connect Source' },
		{ key: 'font-src', label: 'Font Source' },
		{ key: 'object-src', label: 'Object Source' },
		{ key: 'media-src', label: 'Media Source' },
		{ key: 'frame-src', label: 'Frame Source' },
		{ key: 'child-src', label: 'Child Source' },
		{ key: 'worker-src', label: 'Worker Source' },
		{ key: 'manifest-src', label: 'Manifest Source' },
		{ key: 'base-uri', label: 'Base URI' },
		{ key: 'form-action', label: 'Form Action' }
	];

	function addSource(directive: string, source: string) {
		directives[directive as keyof typeof directives] = [
			...directives[directive as keyof typeof directives],
			source
		];
	}

	function removeSource(directive: string, index: number) {
		const sources = directives[directive as keyof typeof directives];
		directives[directive as keyof typeof directives] = sources.filter((_, i) => i !== index);
	}

	function generateCSP() {
		const cspParts: string[] = [];

		for (const [directive, sources] of Object.entries(directives)) {
			if (sources.length > 0) {
				cspParts.push(`${directive} ${sources.join(' ')}`);
			}
		}

		generatedCSP = cspParts.join('; ');
	}

	function clearAll() {
		for (const key of Object.keys(directives)) {
			directives[key as keyof typeof directives] = [];
		}
		generatedCSP = '';
	}
</script>

<svelte:head>
	<title>CSP Generator</title>
	<meta name="description" content="Generate Content Security Policy (CSP) headers easily" />
</svelte:head>

<div class="min-h-screen bg-default-background py-8">
	<div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
		<div class="mb-8 text-center">
			<h1 class="mb-2 text-3xl font-bold text-default-font">CSP Generator</h1>
			<p class="text-subtext-color">Generate Content Security Policy headers with ease</p>
		</div>

		<div class="grid grid-cols-1 gap-8 lg:grid-cols-2">
			<!-- Left Column: Input Fields -->
			<div class="rounded-lg border border-neutral-border bg-neutral-50 p-6 shadow-md">
				<div class="mb-6 flex items-center justify-between">
					<h2 class="text-xl font-semibold text-default-font">
						{selectedDirective
							? `${directiveConfigs.find((c) => c.key === selectedDirective)?.label} (${selectedDirective})`
							: 'CSP Directives'}
					</h2>

					<Select.Root type="single" bind:value={selectedDirective}>
						<Select.Trigger
							class="flex items-center justify-between gap-2 rounded-md border border-neutral-border bg-neutral-100 px-3 py-2 text-sm text-default-font hover:bg-neutral-200 focus:ring-2 focus:ring-brand-primary focus:outline-none min-w-[200px]"
						>
							{selectedDirective
								? `${directiveConfigs.find((c) => c.key === selectedDirective)?.label} (${selectedDirective})`
								: 'Show All Directives'}
							<svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M19 9l-7 7-7-7"
								></path>
							</svg>
						</Select.Trigger>
						<Select.Portal>
							<Select.Content
								class="z-50 max-h-60 overflow-y-auto rounded-md border border-neutral-border bg-neutral-50 shadow-lg p-1"
							>
								<Select.Viewport>
									<Select.Item
										value=""
										class="cursor-pointer rounded px-3 py-2 text-sm text-default-font hover:bg-neutral-100 focus:bg-neutral-100 data-[highlighted]:bg-neutral-100"
									>
										Show All Directives
									</Select.Item>
									{#each directiveConfigs as config}
										<Select.Item
											value={config.key}
											class="cursor-pointer rounded px-3 py-2 text-sm text-default-font hover:bg-neutral-100 focus:bg-neutral-100 data-[highlighted]:bg-neutral-100"
										>
											{config.label} ({config.key})
										</Select.Item>
									{/each}
								</Select.Viewport>
							</Select.Content>
						</Select.Portal>
					</Select.Root>
				</div>

				{#if selectedDirective && selectedDirective !== ''}
					<!-- Single Directive View -->
					{@const config = directiveConfigs.find((c) => c.key === selectedDirective)}
					{#if config}
						<DirectiveInput
							label={config.label}
							directive={config.key}
							sources={directives[config.key as keyof typeof directives]}
							onAdd={(source) => addSource(config.key, source)}
							onRemove={(index) => removeSource(config.key, index)}
						/>
					{/if}
				{:else}
					<!-- All Directives View -->
					<div class="max-h-96 space-y-4 overflow-y-auto">
						{#each directiveConfigs as config}
							<DirectiveInput
								label={config.label}
								directive={config.key}
								sources={directives[config.key as keyof typeof directives]}
								onAdd={(source) => addSource(config.key, source)}
								onRemove={(index) => removeSource(config.key, index)}
							/>
						{/each}
					</div>
				{/if}

				<div class="mt-6 border-t border-neutral-border pt-6">
					<button
						onclick={generateCSP}
						class="w-full rounded-md border border-success-600 px-6 py-3 font-medium text-neutral-950 transition-colors hover:bg-success-700 focus:ring-2 focus:ring-success-500 focus:ring-offset-2 focus:outline-none cursor-pointer"
					>
						Generate CSP
					</button>
				</div>
			</div>

			<!-- Right Column: Generated CSP -->
			<div class="rounded-lg border border-neutral-border bg-neutral-50 p-6 shadow-md">
				<h2 class="mb-6 text-xl font-semibold text-default-font">Generated CSP</h2>

				{#if generatedCSP}
					<div class="mb-4">
						<div class="mb-2 block text-sm font-medium text-subtext-color">
							Content-Security-Policy Header:
						</div>
						<pre
							class="overflow-x-auto rounded-md border border-neutral-border bg-neutral-100 p-4 text-sm break-all whitespace-pre-wrap text-default-font font-monospace-body">{generatedCSP}</pre>
					</div>

					<div class="mb-4">
						<div class="mb-2 block text-sm font-medium text-subtext-color">HTML Meta Tag:</div>
						<pre
							class="overflow-x-auto rounded-md border border-neutral-border bg-neutral-100 p-4 text-sm break-all whitespace-pre-wrap text-default-font font-monospace-body">&lt;meta http-equiv="Content-Security-Policy" content="{generatedCSP}"&gt;</pre>
					</div>
				{:else}
					<div class="py-12 text-center">
						<div class="mb-4 text-neutral-400">
							<svg class="mx-auto h-12 w-12" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
								></path>
							</svg>
						</div>
						<p class="text-subtext-color">
							Add some directives and click "Generate CSP" to see your policy here
						</p>
					</div>
				{/if}

				<div class="mt-6 border-t border-neutral-border pt-6">
					<button
						onclick={clearAll}
						class="w-full rounded-md bg-error-600 px-6 py-3 font-medium text-neutral-950 transition-colors hover:bg-error-700 focus:ring-2 focus:ring-error-500 focus:ring-offset-2 focus:outline-none"
					>
						Clear All
					</button>
				</div>
			</div>
		</div>
	</div>
</div>
