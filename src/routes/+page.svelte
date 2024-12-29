<script lang="ts">
	import { enhance } from '$app/forms';
	import { marked } from 'marked';
	import { Button } from '$lib/components/ui/button';
	import { LoaderCircle } from 'lucide-svelte';
	import * as Tabs from '$lib/components/ui/tabs';
	import { Textarea } from '$lib/components/ui/textarea';
	import { Input } from '$lib/components/ui/input';
	import { Label } from '$lib/components/ui/label';
	import { CloudDownload } from 'lucide-svelte';
	import type { ActionResult } from '@sveltejs/kit';

	let url = $state('');
	let loading = $state(false);
	let isPublishing = $state(false);
	let error = $state<string | null>(null);
	let editCode = $state<string | null>(null);
	let documentId = $state<string | null>(null);
	let markdownContent = $state('');

	interface PublishResponse {
		success: boolean;
		documentId: string;
		editCode: string;
	}

	const submit = () => {
		loading = true;
		return async ({ result }: { result: ActionResult }) => {
			loading = false;
			if (result.type === 'success') {
				markdownContent = result.data?.markdown || '';
			} else if (result.type === 'error') {
				error = 'Failed to parse URL';
			}
		};
	};

	async function publish() {
		isPublishing = true;
		const response = await fetch('/api/doc', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({ markdown: markdownContent })
		});

		if (response.ok) {
			const data = (await response.json()) as PublishResponse;
			editCode = data.editCode;
			documentId = data.documentId;
		} else {
			error = 'Failed to publish document';
		}
		isPublishing = false;
	}

	function exportMarkdown() {
		const blob = new Blob([markdownContent], { type: 'text/plain;charset=utf-8' });
		const url = URL.createObjectURL(blob);
		const link = document.createElement('a');
		link.href = url;
		link.download = `document.txt`;
		document.body.appendChild(link);
		link.click();
		document.body.removeChild(link);
		URL.revokeObjectURL(url);
	}
</script>

<main class="container mx-auto max-w-2xl p-8">
	<div class="mb-8 flex items-center gap-4">
		<h1
			class="bg-gradient-to-r from-primary to-secondary bg-clip-text text-5xl font-bold text-transparent"
		>
			Doc Ock
		</h1>
		<CloudDownload class="h-10 w-10 text-primary" />
	</div>
	<p class="text-muted-foreground mb-8 text-lg">Convert any webpage to a clean markdown</p>

	<form method="POST" action="?/parse" use:enhance={submit} class="flex flex-col gap-6">
		<div class="flex w-full flex-col gap-1.5">
			<Label for="url">Enter URL</Label>
			<Input
				type="url"
				id="url"
				name="url"
				bind:value={url}
				placeholder="https://www.example.com"
				required
			/>
		</div>

		<div class="flex gap-4">
			<Button type="submit" disabled={loading}>
				{#if loading}
					<LoaderCircle class="mr-2 animate-spin" />
					Parsing...
				{:else}
					Parse
				{/if}
			</Button>
		</div>
	</form>

	{#if markdownContent}
		<div class="mt-8 space-y-4">
			<Tabs.Root value="text" class="w-full">
				<Tabs.List>
					<Tabs.Trigger value="text">Text</Tabs.Trigger>
					<Tabs.Trigger value="preview">Preview</Tabs.Trigger>
					<Tabs.Trigger value="how">How</Tabs.Trigger>
				</Tabs.List>
				<Tabs.Content value="text">
					<Textarea rows={20} bind:value={markdownContent} />
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

			<div class="flex gap-4">
				<Button variant="default" onclick={publish} disabled={isPublishing}>
					{#if isPublishing}
						<LoaderCircle class="mr-2 animate-spin" />
						Publishing...
					{:else}
						Publish
					{/if}
				</Button>
				<Button variant="outline" onclick={exportMarkdown}>Export</Button>
			</div>

			{#if editCode}
				<div
					class="mt-4 rounded-lg bg-green-50 p-4 text-green-700 dark:bg-green-900/10 dark:text-green-400"
				>
					<p>Your edit code: <strong>{editCode}</strong></p>
					<p class="mt-2 text-sm">
						Save this code to edit this document in the future. It will not be shown again.
					</p>
				</div>
				<Button variant="default" href={`/view/${documentId}`}>View Published Document</Button>
			{/if}
		</div>
	{/if}
</main>
