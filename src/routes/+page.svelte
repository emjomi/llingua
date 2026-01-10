<script lang="ts">
    import { Wllama } from "@wllama/wllama";

    const languages: string[] = [
        "English",
        "Chinese",
        "Arabic",
        "Japanese",
        "Russian",
    ];

    let wllama = new Wllama({
        "single-thread/wllama.wasm":
            "./node_modules/@wllama/wllama/src/single-thread/wllama.wasm",
        "multi-thread/wllama.wasm":
            "./node_modules/@wllama/wllama/src/multi-thread/wllama.wasm",
    });

    let sourceLanguage = $state(languages[0]);
    let targetLanguage = $state(languages[languages.length - 1]);
    let sourceText = $state("");
    let translatedText = $state("");

    async function translate() {
        translatedText = "";
        if (!wllama.isModelLoaded()) {
            await wllama.loadModelFromHF(
                "tencent/HY-MT1.5-1.8B-GGUF",
                "HY-MT1.5-1.8B-Q4_K_M.gguf",
            );
        }
        const stream = await wllama.createCompletion(
            `Here's the translation of:\n\n${sourceText}\n\nfrom ${sourceLanguage} to ${targetLanguage}:\n\n`,
            {
                nPredict: 50,
                sampling: {
                    temp: 0.5,
                    top_k: 40,
                    top_p: 0.9,
                },
                stream: true,
            },
        );
        for await (const chunk of stream) {
            translatedText = chunk.currentText;
        }
    }

    async function handleKeydown(event: KeyboardEvent) {
        if (event.key === "Enter" && !event.shiftKey) {
            event.preventDefault();
            await translate();
        }
    }
</script>

<div>
    <select bind:value={sourceLanguage}>
        {#each languages as language}
            <option value={language}>{language}</option>
        {/each}
    </select>
    <select bind:value={targetLanguage}>
        {#each languages as language}
            <option value={language}>{language}</option>
        {/each}
    </select>
</div>

<div>
    <textarea bind:value={sourceText} onkeydown={handleKeydown}></textarea>
    <textarea readonly>{translatedText}</textarea>
</div>
