<script lang="ts">
	import { page } from '$app/state';
	import { goto } from '$app/navigation';
	import { marked } from 'marked';
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { LoaderCircle } from 'lucide-svelte';
	import * as Tabs from '$lib/components/ui/tabs';
	import { Textarea } from '$lib/components/ui/textarea';
	import { toast } from 'svelte-sonner';
	import type { PageServerData } from './$types';

	let isPublishing = $state(false);

	let { data }: { data: PageServerData } = $props();
	let enteredEditCode = $state('');
	let editCodeError = $state<string | null>(null);

	let markdownContent = $state(data.markdownContent);

	const id = page.params.id;

	async function publish() {
		if (!enteredEditCode) {
			toast.error('Edit code is required');
			editCodeError = 'Edit code is required';
			return;
		}

		isPublishing = true;
		try {
			const response = await fetch(`/api/doc/${id}`, {
				method: 'PUT',
				headers: {
					'Content-Type': 'application/json',
					'X-Edit-Code': enteredEditCode
				},
				body: JSON.stringify({ markdown: markdownContent })
			});

			if (response.ok) {
				toast.success('Document published successfully');
				await response.json();
				goto(`/view/${id}`);
			} else {
				toast.error('Failed to publish document');
			}
		} catch (error) {
			toast.error('An unexpected error occurred');
		} finally {
			isPublishing = false;
		}
	}
</script>

<div class="dark:bg-background-dark container mx-auto min-h-screen space-y-4 bg-background p-4">
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
