DONE: See and update: [VSCode extensions](VScodeExtensions.md)

Also, add Svelte suggestions there but more easily - like where the settings.json is etc.

Getting Started With Vscode
This page aims to gather all resources related to Svelte and VS Code.

Install The Official Extension

https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode

Enable File Nesting

alt text

Keep your Explorer View compact by nesting related files together.


settings.json

{
	// ...
	"explorer.fileNesting.enabled": true,
	"explorer.fileNesting.expand": false,
	"explorer.fileNesting.patterns": {
		"+*.svelte": "+${capture}.js, +${capture}.server.js, +${capture}.ts, +${capture}.server.ts",
		"*.svelte": "${capture}.stories.svelte,${capture}.stories.ts"
	}
}
1
2
3
4
5
6
7
8
9
https://code.visualstudio.com/updates/v1_67#_explorer-file-nesting
https://github.com/antfu/vscode-file-nesting-config
Put Files First

Since Sveltekit uses a filesystem-based router, placing files first keeps related items grouped together within their folder.


settings.json

{
	// ...
	"explorer.sortOrder": "filesFirst"
}
1
2
3
4
Custom File labels

Sveltekit's filesystem-based router gives every file the same name. Which can be difficult to distinguish when multiple are open. The custom label feature can help clear this up. Open .vscode/settings.json and add the following.


settings.json (compact)

settings.json (expanded)

{
	// ...
	"workbench.editor.customLabels.patterns": {
		// ─── +page ────────────────────────────────────────────────────────────────
		"**/routes/**/*/+page.{svelte,[tj]s}": "${dirname} ❱ page",
		"**/routes/+page.{svelte,[tj]s}": "/ ❱ page",
		"**/routes/**/*/+page.server.[tj]s": "${dirname} ❱ page.server",
		"**/routes/+page.server.[tj]s": "/ ❱ page.server",

		// ─── +error ───────────────────────────────────────────────────────────────
		"**/routes/**/*/+error.svelte": "${dirname} ❱ error",
		"**/routes/+error.svelte": "/ ❱ error",

		// ─── +layout ──────────────────────────────────────────────────────────────
		"**/routes/**/*/+layout.{svelte,[tj]s}": "${dirname} ❱ layout",
		"**/routes/+layout.{svelte,[tj]s}": "/ ❱ layout",
		"**/routes/**/*/+layout.server.[tj]s": "${dirname} ❱ layout.server",
		"**/routes/+layout.server.[tj]s": "/ ❱ layout.server",

		// ─── +server ──────────────────────────────────────────────────────────────
		"**/routes/**/*/+server.[tj]s": "${dirname} ❱ API",
		"**/routes/+server.[tj]s": "/ ❱ API"
	}
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
Breakpoint Debugging

https://svelte.dev/docs/kit/debugging#Visual-Studio-Code

::

Explicit Extensions

{
	"javascript.preferences.importModuleSpecifierEnding": "js",
	"typescript.preferences.importModuleSpecifierEnding": "js"
}