<script lang="ts">
    import { Wllama, type WllamaChatMessage } from "@wllama/wllama";
    import wllamaSingle from "@wllama/wllama/src/single-thread/wllama.wasm?url";
    import wllamaMulti from "@wllama/wllama/src/multi-thread/wllama.wasm?url";

    const languages: string[] = [
        "English",
        "Chinese",
        "Arabic",
        "Japanese",
        "Russian",
    ];

    let wllama = new Wllama({
        "single-thread/wllama.wasm": wllamaSingle,
        "multi-thread/wllama.wasm": wllamaMulti,
    });

    let targetLanguage = $state(languages[languages.length - 1]);
    let sourceText = $state("");
    let translatedText = $state("");
    let isDropdownOpen = $state(false);

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

    function toggleDropdown() {
        isDropdownOpen = !isDropdownOpen;
    }

    function selectLanguage(language: string) {
        targetLanguage = language;
        isDropdownOpen = false;
    }
</script>

<div class="translator-container">
    <div class="panels">
        <div class="panel">
            <textarea
                bind:value={sourceText}
                onkeydown={handleKeydown}
                placeholder="Enter text to translate..."
            ></textarea>
        </div>

        <div class="panel">
            <textarea readonly>{translatedText}</textarea>
            <div class="language-dropdown">
                <button class="dropdown-trigger" onclick={toggleDropdown}>
                    {targetLanguage}
                </button>
                {#if isDropdownOpen}
                    <div class="dropdown-menu">
                        {#each languages as language}
                            <button onclick={() => selectLanguage(language)}>
                                {language}
                            </button>
                        {/each}
                    </div>
                {/if}
            </div>
        </div>
    </div>
    <p class="keyboard-hint">
        Press <kbd>Enter</kbd> to translate, <kbd>Shift + Enter</kbd> for new line
    </p>
</div>

<style>
    :root {
        --color-bg: #1d1d20;
        --color-text: #ffffff;
        --color-panel: #2e2e32;
        --color-muted: #424246;
        --color-accent: #505053;

        --spacing-sm: 6px;
        --spacing-md: 9px;
        --spacing-lg: 12px;

        --font-size-sm: 12px;
        --font-size-md: 14px;
        --font-size-lg: 16px;

        --shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
    }

    :global(body) {
        display: flex;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
        margin: 0;
        padding: var(--spacing-lg);
        background-color: var(--color-bg);
        font-family: Inter;
    }

    ::selection {
        background-color: var(--color-accent);
    }

    .translator-container {
        width: 100%;
        max-width: 900px;
    }

    .panels {
        display: flex;
        gap: var(--spacing-lg);
    }

    .panel {
        flex: 1;
        display: flex;
        flex-direction: column;
        min-height: 300px;
        padding: var(--spacing-lg);
        background-color: var(--color-panel);
        border-radius: var(--spacing-lg);
        box-shadow: var(--shadow);
    }

    textarea {
        flex: 1;
        width: 100%;
        min-height: 200px;
        resize: none;
        background: transparent;
        border: none;
        outline: none;
        color: var(--color-text);
        font-size: var(--font-size-lg);
        line-height: 1.25;
        font-family: inherit;
    }

    textarea::placeholder {
        color: var(--color-muted);
    }

    .language-dropdown {
        align-self: flex-start;
        position: relative;
    }

    .dropdown-trigger,
    .dropdown-menu button {
        padding: var(--spacing-md) var(--spacing-lg);
        background: transparent;
        border: none;
        outline: none;
        border-radius: var(--spacing-md);
        color: var(--color-text);
        font-size: var(--font-size-md);
        font-family: inherit;
        cursor: pointer;
        text-align: left;
    }

    .dropdown-trigger:hover,
    .dropdown-menu button:hover {
        background-color: var(--color-muted);
    }

    .dropdown-trigger:focus,
    .dropdown-menu button:hover {
        background-color: var(--color-accent);
    }

    .dropdown-menu {
        position: absolute;
        top: 100%;
        left: 0;
        margin-top: var(--spacing-sm);
        padding: var(--spacing-sm);
        background-color: var(--color-panel);
        border-radius: var(--spacing-md);
        box-shadow: var(--shadow);
    }

    .dropdown-menu button {
        display: block;
        min-width: 100%;
    }

    .keyboard-hint {
        width: 100%;
        margin-top: var(--spacing-lg);
        text-align: center;
        color: var(--color-muted);
        font-size: var(--font-size-md);
    }

    kbd {
        padding: 1px 4px;
        border-radius: var(--spacing-sm);
        background-color: var(--color-panel);
        color: var(--color-accent);
        font-size: var(--font-size-sm);
        font-family: inherit;
        box-shadow: var(--shadow);
    }
</style>
