<script>
	let fileContent = '';
	let generatedCode = '';

	function handleFileUpload(event) {
		const file = event.target.files[0];
		if (!file) return;

		const reader = new FileReader();
		reader.onload = (e) => {
			fileContent = e.target.result;
			generateCode(fileContent);
		};
		reader.readAsText(file);
	}

	function generateCode(content) {
		try {
			const match = content.match(/APP_DATA\s*=\s*(\{[\s\S]*\});/);
			if (!match) {
				generatedCode = '// ❌ APP_DATA not found in file';
				return;
			}

			const parsed = eval('(' + match[1] + ')');
			const scenes = parsed.scenes || [];

			// 🧩 Generate only create360scene + createHotspot calls
			let result = '';

			// --- Create360scene ---
			for (const scene of scenes) {
				const { id, levels, initialViewParameters } = scene;
				result += `create360scene(\n  '${id}',\n  ${JSON.stringify(levels, null, 2)},\n  ${JSON.stringify(initialViewParameters, null, 2)}\n);\n\n`;
			}

			// --- CreateHotspot ---
			for (const scene of scenes) {
				const sourceId = scene.id;
				if (scene.linkHotspots?.length) {
					result += `// ${sourceId} hotspots\n`;
					for (const link of scene.linkHotspots) {
						const targetScene = scenes.find((s) => s.id === link.target);
						const label = targetScene ? targetScene.name : link.target;
						result += `createHotspot(\n  allScenes['${sourceId}'].scene,\n  { yaw: ${link.yaw}, pitch: ${link.pitch} },\n  allScenes['${link.target}'].scene,\n  '${label}',\n  '${link.target}'\n);\n\n`;
					}
				}
			}

			generatedCode = result.trim();
		} catch (err) {
			generatedCode = `// ❌ Error parsing file: ${err.message}`;
		}
	}

	function copyToClipboard() {
		navigator.clipboard.writeText(generatedCode);
		alert('✅ Code copied to clipboard!');
	}
</script>

<div class="min-h-screen bg-gray-900 p-6 text-white">
	<h1 class="mb-4 text-xl font-bold">🎥 360 Scene & Hotspot Generator</h1>
	<input
		type="file"
		accept=".js"
		on:change={handleFileUpload}
		class="mb-4 block w-full cursor-pointer rounded bg-gray-800 p-3"
	/>

	{#if generatedCode}
		<button
			class="mb-2 rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700"
			on:click={copyToClipboard}
		>
			Copy Code
		</button>

		<pre class="overflow-auto rounded bg-gray-800 p-4 text-sm">
      {generatedCode}
    </pre>
	{/if}
</div>
