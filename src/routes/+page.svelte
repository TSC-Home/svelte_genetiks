<script lang="ts">
	import { onMount } from 'svelte';

	const AS: { [key: string]: string | string[] } = {
		UUU: 'Phe',
		UUC: 'Phe',
		UUA: 'Leu',
		UUG: 'Leu',
		CUU: 'Leu',
		CUC: 'Leu',
		CUA: 'Leu',
		CUG: 'Leu',
		AUU: 'Ile',
		AUC: 'Ile',
		AUA: 'Ile',
		AUG: ['Met', 'start'],
		GUU: 'Val',
		GUC: 'Val',
		GUA: 'Val',
		GUG: 'Val',
		UCU: 'Ser',
		UCC: 'Ser',
		UCA: 'Ser',
		UCG: 'Ser',
		CCU: 'Pro',
		CCC: 'Pro',
		CCA: 'Pro',
		CCG: 'Pro',
		ACU: 'Thr',
		ACC: 'Thr',
		ACA: 'Thr',
		ACG: 'Thr',
		GCU: 'Ala',
		GCC: 'Ala',
		GCA: 'Ala',
		GCG: 'Ala',
		UAU: 'Tyr',
		UAC: 'Tyr',
		UAA: 'stopp',
		UAG: 'stopp',
		CAU: 'His',
		CAC: 'His',
		CAA: 'Gln',
		CAG: 'Gln',
		AAU: 'Asn',
		AAC: 'Asn',
		AAA: 'Lys',
		AAG: 'Lys',
		GAU: 'Asp',
		GAC: 'Asp',
		GAA: 'Glu',
		GAG: 'Glu',
		UGU: 'Cys',
		UGC: 'Cys',
		UGA: 'stopp',
		UGG: 'Trp',
		CGU: 'Arg',
		CGC: 'Arg',
		CGA: 'Arg',
		CGG: 'Arg',
		AGU: 'Ser',
		AGC: 'Ser',
		AGA: 'Arg',
		AGG: 'Arg',
		GGU: 'Gly',
		GGC: 'Gly',
		GGA: 'Gly',
		GGG: 'Gly'
	};

	let mRNA: string = '';
	let output: string[] = [];
	let result_combined: any = '';

	function translate(): void {
		const splited = mRNA.match(/\w{3}|\w+$/g) || [];
		const tempOutput: string[] = [];
		let last_stopp_index = 0;
		let last_start_index = 0;

		function translator(words: string): string | string[] {
			if (AS[words]) {
				return AS[words];
			} else {
				return 'Not found';
			}
		}

		splited.forEach((word, i) => {
			const objected = translator(word);
			let stringified = Array.isArray(objected) ? objected[0] : objected;

			if (tempOutput.includes('stopp')) {
				last_stopp_index = tempOutput.length - tempOutput.reverse().indexOf('stopp');
				tempOutput.reverse();
			}

			if (tempOutput.includes('start')) {
				last_start_index = tempOutput.length - tempOutput.reverse().indexOf('start');
				tempOutput.reverse();
			}

			if (word === 'AUG') {
				if (last_start_index <= last_stopp_index) {
					stringified = AS['AUG'][1] as string;
				} else if (last_stopp_index > last_start_index) {
					stringified = AS['AUG'][1] as string;
				} else if (last_stopp_index < last_start_index) {
					stringified = AS['AUG'][0] as string;
				}
			}

			if (last_start_index > last_stopp_index || word === 'AUG') {
				tempOutput.push(stringified);
				if (stringified === 'stopp') {
					tempOutput.push(' ');
				}
			}
		});

		const start_indices = tempOutput
			.map((x, i) => (x === 'start' ? i : -1))
			.filter((i) => i !== -1);
		const stop_indices = tempOutput.map((x, i) => (x === 'stopp' ? i : -1)).filter((i) => i !== -1);

		const result: string[][] = [];

		start_indices.forEach((start, index) => {
			const stop = stop_indices[index];
			if (start < stop) {
				result.push(tempOutput.slice(start + 1, stop));
			}
		});
		console.log(result);
		(result_combined = result.map((sublist) => sublist.join(' - '))), console.log(result_combined);
	}

	function handleFileUpload(event: Event): void {
		const target = event.target as HTMLInputElement;
		if (target.files && target.files.length > 0) {
			const file = target.files[0];
			const reader = new FileReader();
			reader.onload = function (e: ProgressEvent<FileReader>) {
				if (e.target) {
					mRNA = e.target.result as string;
					translate();
				}
			};
			reader.readAsText(file);
		}
	}
</script>

<div class="flex min-h-screen flex-col items-center justify-center bg-gray-100">
	<h1 class="mb-8 text-4xl font-bold">mRNA Translator</h1>
	<div class="w-96">
		<textarea
			class="mb-4 w-full rounded border border-gray-300 p-4"
			bind:value={mRNA}
			placeholder="Enter mRNA sequence here..."
		></textarea>
		<a
			href="https://github.com/Twyggy/code-for-molecular-genetics/blob/main/mRNA%20sequenz.txt"
			class="mb-4 flex w-full rounded border border-gray-300 p-4 hover:border-blue-500 active:bg-blue-600/30"
		>
			Cklick to download a Example .txt file.
		</a>
		<button
			class="mb-4 w-full rounded bg-blue-500 py-2 text-white hover:bg-blue-600"
			on:click={translate}>Translate</button
		>
		<label for="file-upload" class="mb-4 flex w-full rounded border border-gray-300 p-4">
			<p>Or Cklick to upload a .txt file</p>
			<input
				id="file-upload"
				class="hidden"
				type="file"
				accept=".txt"
				on:change={handleFileUpload}
			/>
		</label>
		<div class="rounded border border-gray-300 bg-white p-4">
			<h2 class="mb-2 text-xl font-bold">Amino Acid Sequence:</h2>
			<p>
				{#each result_combined as result}
					{result}
					<br />
				{/each}
			</p>
		</div>
		<div class=" mt-2 flex -space-x-2">
			<a
				target="_blank"
				class="duration-200 hover:z-50 hover:scale-110"
				title="GitHub Twyggy"
				href="https://github.com/twyggy"
			>
				<img
					src="https://github.com/twyggy.png"
					class="size-8 rounded-full border-2 border-white"
					alt="GitHub Twyggy"
				/>
			</a>
			<a
				target="_blank"
				class="duration-200 hover:z-50 hover:scale-110"
				title="GitHub tsc-home"
				href="https://github.com/tsc-home"
			>
				<img
					src="https://github.com/tsc-home.png"
					class="size-8 rounded-full border-2 border-white"
					alt="GitHub tsc-home"
				/>
			</a>
		</div>
		<div class="mt-2 flex">
			<a href="https://synthetix.me/blog/creating-an-mrna-translator" target="_blank">
				<img
					alt="Blog Post mRNA Translator with Svelte and TypeScript"
					src="https://og.synthetix.me/mRNA%20Translator%20with%20Svelte%20and%20TypeScript"
					class="rounded-md border-2 border-white duration-100 hover:rotate-3 hover:scale-110"
				/>
			</a>
		</div>
	</div>
</div>
