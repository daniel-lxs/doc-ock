<script lang="ts">
	import { page } from '$app/state';
	import { goto } from '$app/navigation';
	import { marked } from 'marked';
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { LoaderCircle } from 'lucide-svelte';
	import * as Tabs from '$lib/components/ui/tabs';
	import { Textarea } from '$lib/components/ui/textarea';
	import type { PageServerData } from './$types';

	let isPublishing = $state(false);
	let error = $state<string | null>(null);

	let { data }: { data: PageServerData } = $props();
	let enteredEditCode = $state('');
	let editCodeError = $state<string | null>(null);

	let markdownContent = $state(data.markdownContent);

	const id = page.params.id;

	async function publish() {
		if (!enteredEditCode) {
			editCodeError = 'Edit code is required';
			return;
		}

		isPublishing = true;
		const response = await fetch(`/api/doc/${id}`, {
			method: 'PUT',
			headers: {
				'Content-Type': 'application/json',
				'X-Edit-Code': enteredEditCode
			},
			body: JSON.stringify({ markdown: markdownContent })
		});

		if (response.ok) {
			await response.json();
			goto(`/view/${id}`);
		} else {
			error = 'Failed to publish document';
		}
		isPublishing = false;
	}
</script>

<div class="container mx-auto min-h-screen space-y-4 bg-background p-4 dark:bg-background-dark">
	{#if error}
		<div class="alert alert-error rounded-lg bg-red-500/10 p-4 text-red-600 dark:text-red-400">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-6 w-6 shrink-0"
				fill="none"
				viewBox="0 0 24 24"
				stroke="currentColor"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
				/>
			</svg>
			<span>{error}</span>
		</div>
	{/if}

	{#if id}
		<div class="mb-4">
			<label class="mb-1 block text-sm font-medium" for="edit-code"> Edit Code </label>
			<Input
				type="text"
				id="edit-code"
				bind:value={enteredEditCode}
				placeholder="Enter edit code"
			/>
			{#if editCodeError}
				<p class="mt-1 text-sm text-red-500">{editCodeError}</p>
			{/if}
		</div>
	{/if}

	<Tabs.Root value="text" class="w-full">
		<Tabs.List>
			<Tabs.Trigger value="text">Text</Tabs.Trigger>
			<Tabs.Trigger value="preview">Preview</Tabs.Trigger>
			<Tabs.Trigger value="how">How</Tabs.Trigger>
		</Tabs.List>
		<Tabs.Content value="text">
			<Textarea rows={100} bind:value={markdownContent} />
		</Tabs.Content>
		<Tabs.Content value="preview">
			<div class="prose max-w-none dark:prose-invert">
				{@html marked.parse(markdownContent)}
			</div>
		</Tabs.Content>
		<Tabs.Content value="how">
			<div class="prose max-w-none dark:prose-invert">
				<h2>Markdown Guide</h2>
				<p>Here are some basic Markdown syntax examples:</p>
				<ul>
					<li><strong>Bold</strong>: `**text**`</li>
					<li><em>Italic</em>: `*text*`</li>
					<li><code>Code</code>: `` `code` ``</li>
					<li>
						<a href="/">Link</a>: `[Link](https://example.com)`
					</li>
					<li>List: `- Item`</li>
				</ul>
			</div>
		</Tabs.Content>
	</Tabs.Root>

	<Button variant="default" onclick={publish} disabled={isPublishing}>
		{#if isPublishing}
			<LoaderCircle class="mr-2 animate-spin" />
			Please wait
		{:else}
			Publish
		{/if}
	</Button>
</div>
