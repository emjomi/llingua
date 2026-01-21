<script lang="ts">
    import { Wllama, type WllamaChatMessage } from "@wllama/wllama";

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
        const messages: WllamaChatMessage[] = [
            {
                role: "system",
                content: `Translate user message into ${targetLanguage}, without additional explanation.`,
            },
            { role: "user", content: sourceText },
        ];
        const stream = await wllama.createChatCompletion(messages, {
            nPredict: 4096,
            sampling: {
                top_k: 20,
                top_p: 0.6,
                penalty_repeat: 1.05,
                temp: 0.7,
            },
            stream: true,
        });
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

<select bind:value={targetLanguage}>
    {#each languages as language}
        <option value={language}>{language}</option>
    {/each}
</select>

<div>
    <textarea bind:value={sourceText} onkeydown={handleKeydown}></textarea>
    <textarea readonly>{translatedText}</textarea>
</div>
