<script lang="ts">
	import { UploadIcon } from "lucide-svelte";
	import Panel from "../visual/Panel.svelte";
	import clsx from "clsx";
	import { onMount } from "svelte";
	import { effects, files } from "$lib/store/index.svelte";
	import { converters } from "$lib/converters";
	import { goto } from "$app/navigation";
	import { page } from "$app/state";

	type Props = {
		class?: string;
		jpegify?: boolean;
	};

	const { class: classList, jpegify }: Props = $props();

	let uploaderButton = $state<HTMLButtonElement>();
	let fileInput = $state<HTMLInputElement>();

	let acceptedTypes = $state<string>();

	const setupFileInput = async () => {
		if (!fileInput) return;

		const filteredConverters = (
			await Promise.all(
				converters.map(async (c) => {
					if (await c.valid()) return c;
				}),
			)
		).filter((c) => typeof c !== "undefined");
		acceptedTypes = filteredConverters
			.map((c) => c.formatStrings((f) => f.fromSupported).join(","))
			.join(",");
	};

	const uploadFiles = async () => {
		if (!fileInput) return;
		fileInput.click();
	};

	const handleFileChange = (e: Event) => {
		if (!fileInput) return;
		if (page.url.pathname !== "/jpegify/") {
			const oldLength = files.files.length;
			files.add(fileInput.files);
			if (oldLength !== files.files.length) goto("/convert");
		} else {
			files.add(fileInput.files);
		}
	};

	onMount(() => {
		const handler = (e: Event) => {
			e.preventDefault();
			return false;
		};

		uploaderButton?.addEventListener("dragover", handler);
		uploaderButton?.addEventListener("dragenter", handler);
		uploaderButton?.addEventListener("dragleave", handler);
		uploaderButton?.addEventListener("drop", handler);

		void setupFileInput();

		return () => {
			uploaderButton?.removeEventListener("dragover", handler);
			uploaderButton?.removeEventListener("dragenter", handler);
			uploaderButton?.removeEventListener("dragleave", handler);
			uploaderButton?.removeEventListener("drop", handler);
		};
	});
</script>

<input
	bind:this={fileInput}
	type="file"
	multiple
	class="hidden"
	onchange={handleFileChange}
	accept={acceptedTypes}
/>

<button
	onclick={uploadFiles}
	bind:this={uploaderButton}
	class={clsx(
		`hover:scale-105 active:scale-100 ${$effects ? "" : "!scale-100"} duration-200 ${classList}`,
	)}
>
	<Panel
		class="flex justify-center items-center w-full h-full flex-col pointer-events-none"
	>
		<div
			class="w-16 h-16 bg-accent rounded-full flex items-center justify-center p-4"
		>
			<UploadIcon class="w-full h-full text-on-accent" />
		</div>
		<h2 class="text-center text-2xl font-semibold mt-4">
			Drop or click to {jpegify ? "JPEGIFY" : "convert"}
		</h2>
	</Panel>
</button>
