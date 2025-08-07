<script lang="ts">
	import DirectiveInput from '$lib/components/DirectiveInput.svelte';
	import { Dialog, Select } from 'bits-ui';

	// CSP directive state
	let directives = $state({
		'default-src': [] as string[],
		'connect-src': [] as string[],
		'font-src': [] as string[],
		'frame-src': [] as string[],
		'img-src': [] as string[],
		'media-src': [] as string[],
		'object-src': [] as string[],
		'script-src': [] as string[],
		'script-src-elem': [] as string[],
		'script-src-attr': [] as string[],
		'style-src': [] as string[],
		'style-src-elem': [] as string[],
		'frame-ancestors': [] as string[]
	});

	let generatedCSP = $state('');
	let selectedDirective = $state<string>('');
	let importDialogOpen = $state(false);
	let importCSPText = $state('');

	// Directive configurations
	const directiveConfigs = [
		{ key: 'default-src', label: 'Default Source' },
		{ key: 'connect-src', label: 'Connect Source' },
		{ key: 'font-src', label: 'Font Source' },
		{ key: 'frame-src', label: 'Frame Source' },
		{ key: 'img-src', label: 'Image Source' },
		{ key: 'media-src', label: 'Media Source' },
		{ key: 'object-src', label: 'Object Source' },
		{ key: 'script-src', label: 'Script Source' },
		{ key: 'script-src-elem', label: 'Script Source Element' },
		{ key: 'script-src-attr', label: 'Script Source Attribute' },
		{ key: 'style-src', label: 'Style Source' },
		{ key: 'style-src-elem', label: 'Style Source Element' },
		{ key: 'frame-ancestors', label: 'Frame Ancestors' }
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

	function parseCSP(cspString: string) {
		// Clear existing directives
		clearAll();
		
		// Remove 'Content-Security-Policy:' prefix if present
		const cleanCSP = cspString.replace(/^Content-Security-Policy:\s*/i, '').trim();
		
		// Split by semicolon to get individual directives
		const directiveParts = cleanCSP.split(';').map(part => part.trim()).filter(part => part);
		
		for (const part of directiveParts) {
			// Split each directive by spaces to get directive name and sources
			const tokens = part.split(/\s+/);
			if (tokens.length < 1) continue;
			
			const directiveName = tokens[0].toLowerCase();
			const sources = tokens.slice(1);
			
			// Check if this directive exists in our configuration
			if (directiveName in directives) {
				directives[directiveName as keyof typeof directives] = sources;
			}
		}
	}

	function handleImportCSP() {
		if (importCSPText.trim()) {
			parseCSP(importCSPText.trim());
			importDialogOpen = false;
			importCSPText = '';
		}
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
					<div class="flex gap-3">
						<Dialog.Root bind:open={importDialogOpen}>
							<Dialog.Trigger
								class="flex-1 rounded-md border border-brand-primary bg-transparent px-6 py-3 font-medium text-brand-primary transition-colors hover:bg-brand-primary hover:text-neutral-950 focus:ring-2 focus:ring-brand-primary focus:ring-offset-2 focus:outline-none"
							>
								Import CSP
							</Dialog.Trigger>
							<Dialog.Portal>
								<Dialog.Overlay class="fixed inset-0 bg-black/50 z-40" />
								<Dialog.Content
									class="fixed left-1/2 top-1/2 z-50 w-full max-w-lg -translate-x-1/2 -translate-y-1/2 rounded-lg border border-neutral-border bg-neutral-50 p-6 shadow-lg"
								>
									<Dialog.Title class="text-lg font-semibold text-default-font mb-2">
										Import Existing CSP
									</Dialog.Title>
									<Dialog.Description class="text-sm text-subtext-color mb-4">
										Paste your existing Content Security Policy below to import and edit it.
									</Dialog.Description>
									
									<textarea
										bind:value={importCSPText}
										placeholder="Paste your CSP here...\n\nExample:\ndefault-src 'self'; script-src 'self' https://example.com; style-src 'self' 'unsafe-inline';"
										class="w-full h-32 rounded-md border border-neutral-border bg-neutral-100 px-3 py-2 text-sm text-default-font placeholder:text-neutral-400 resize-none focus:border-brand-primary focus:ring-2 focus:ring-brand-primary focus:outline-none"
									></textarea>
									
									<div class="flex justify-end gap-3 mt-6">
										<Dialog.Close
											class="px-4 py-2 text-sm font-medium text-subtext-color hover:text-default-font transition-colors focus:outline-none"
										>
											Cancel
										</Dialog.Close>
										<button
											onclick={handleImportCSP}
											class="px-4 py-2 text-sm font-medium bg-brand-primary text-neutral-950 rounded-md hover:bg-brand-700 focus:ring-2 focus:ring-brand-primary focus:ring-offset-2 focus:outline-none transition-colors"
										>
											Import
										</button>
									</div>
								</Dialog.Content>
							</Dialog.Portal>
						</Dialog.Root>
						
						<button
							onclick={generateCSP}
							class="flex-1 rounded-md bg-success-600 px-6 py-3 font-medium text-neutral-950 transition-colors hover:bg-success-700 focus:ring-2 focus:ring-success-500 focus:ring-offset-2 focus:outline-none"
						>
							Generate CSP
						</button>
					</div>
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
