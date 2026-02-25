<template>
    <div class="app">
        <div class="aurora-bg" aria-hidden="true">
            <Aurora
                :color-stops="auroraStops"
                :amplitude="1.0"
                :blend="0.55"
                :speed="1.0"
                :intensity="1.0"
                class="w-full h-full"
            />
        </div>
        <main class="grid">
            <section class="card">
                <h2>Проверка системного шрифта</h2>

                <div class="row">
                    <button
                        type="button"
                        class="ghost"
                        :disabled="!supportsFontAccess"
                        @click="loadLocalFonts"
                    >
                        {{
                            supportsFontAccess
                                ? "Показать локальные шрифты"
                                : "Font Access API недоступен"
                        }}
                    </button>
                    <span v-if="fontAccessError" class="hint">{{
                        fontAccessError
                    }}</span>
                </div>

                <div v-if="fontAccessList.length" class="field">
                    <label>Локальные шрифты (Font Access)</label>
                    <select v-model="systemFontName">
                        <option
                            v-for="font in fontAccessList"
                            :key="font"
                            :value="font"
                        >
                            {{ font }}
                        </option>
                    </select>
                </div>

                <div class="field">
                    <label>Название шрифта (font-family)</label>
                    <div class="row">
                        <div class="input-status grow">
                            <input
                                v-model="systemFontName"
                                placeholder="Segoe UI"
                                class="input-with-status"
                            />
                            <span
                                class="status-dot"
                                :class="statusClass"
                                :title="statusLabel"
                                :aria-label="statusLabel"
                                role="status"
                            ></span>
                        </div>
                    </div>
                </div>

                <div class="field">
                    <label>Быстрый выбор</label>
                    <div class="chips">
                        <button
                            v-for="font in commonFonts"
                            :key="font"
                            class="chip"
                            type="button"
                            @click="systemFontName = font"
                        >
                            {{ font }}
                        </button>
                    </div>
                </div>

                <div class="field">
                    <label>Пример текста</label>
                    <textarea
                        v-model="sampleText"
                        class="preview-input"
                        :style="{ fontFamily: activeFontFamily }"
                    ></textarea>
                </div>

                <div class="row">
                    <code class="css-snippet">{{ systemFontCss }}</code>
                    <button
                        type="button"
                        :disabled="!systemFontName.trim()"
                        @click="copySnippet(systemFontCss, 'system')"
                    >
                        {{
                            copyStatus === "system"
                                ? "Скопировано"
                                : "Скопировать"
                        }}
                    </button>
                </div>

                <p class="hint">
                    Если вы вводите один шрифт, проверка показывает доступность
                    именно этого имени. Для стека указывайте шрифты по одному,
                    чтобы понять точный CSS-селектор.
                </p>
            </section>

            <section class="card">
                <h2>Локальный файл шрифта</h2>

                <div class="field">
                    <label>Загрузить файл (.ttf, .otf, .woff, .woff2)</label>
                    <input
                        type="file"
                        @change="onFileChange"
                        accept=".ttf,.otf,.woff,.woff2"
                    />
                </div>

                <div v-if="localFontError" class="error">
                    {{ localFontError }}
                </div>

                <template v-if="localFont">
                    <div class="meta">
                        <div class="meta-row">
                            <span>Файл</span>
                            <strong>{{ localFont.fileName }}</strong>
                        </div>
                        <div class="meta-row">
                            <span>Размер</span>
                            <strong>{{ localFont.fileSize }}</strong>
                        </div>
                        <div class="meta-row" v-if="localFont.fullName">
                            <span>Полное имя</span>
                            <strong>{{ localFont.fullName }}</strong>
                        </div>
                        <div class="meta-row" v-if="localFont.subfamily">
                            <span>Стиль</span>
                            <strong>{{ localFont.subfamily }}</strong>
                        </div>
                        <div class="meta-row" v-if="localFont.postScript">
                            <span>PostScript</span>
                            <strong>{{ localFont.postScript }}</strong>
                        </div>
                        <div class="meta-row" v-if="localFont.version">
                            <span>Версия</span>
                            <strong>{{ localFont.version }}</strong>
                        </div>
                    </div>

                    <div class="row">
                        <button type="button" @click="clearLocalFont">
                            Очистить загруженный шрифт
                        </button>
                    </div>

                    <p class="hint">
                        Это имя извлечено из метаданных файла. Если вы
                        установите шрифт в систему, используйте его в CSS. При
                        @font-face имя можно задать вручную.
                    </p>
                </template>

                <p v-else class="hint">
                    Загрузите файл шрифта, чтобы увидеть его внутреннее имя и
                    посмотреть превью.
                </p>
            </section>
        </main>

        <section class="specimen" :style="{ fontFamily: activeFontFamily }">
            <h2 class="specimen-title">Пробы</h2>
            <div class="specimen-body">
                <div class="specimen-left">
                    <div class="specimen-group">
                        <div class="specimen-line">
                            ABCDEFGHIJKLMNOPQRSTUVWXYZ
                        </div>
                        <div class="specimen-line specimen-lower">
                            abcdefghijklmnopqrstuvwxyz
                        </div>
                    </div>
                    <div class="specimen-group specimen-gap">
                        <div class="specimen-line">
                            АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ
                        </div>
                        <div class="specimen-line specimen-lower">
                            абвгдеёжзийклмнопрстуфхцчшщъыьэюя
                        </div>
                    </div>
                    <div class="specimen-line">0123456789</div>
                </div>
                <div class="specimen-right">
                    <div class="specimen-sample">{{ sampleText }}</div>
                </div>
            </div>
            <div class="specimen-weights">
                <div class="specimen-weight" style="font-weight: 300">
                    Light (300)
                </div>
                <div class="specimen-weight" style="font-weight: 400">
                    Regular (400)
                </div>
                <div class="specimen-weight" style="font-weight: 500">
                    Medium (500)
                </div>
                <div class="specimen-weight" style="font-weight: 600">
                    Semibold (600)
                </div>
                <div class="specimen-weight" style="font-weight: 700">
                    Bold (700)
                </div>
                <div class="specimen-weight" style="font-weight: 800">
                    Black (800)
                </div>
            </div>
        </section>
    </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from "vue";
import Aurora from "./components/Aurora.vue";
import opentype from "opentype.js";

type LocalFontInfo = {
    fileName: string;
    fileSize: string;
    detectedFamily: string;
    fullName: string;
    subfamily: string;
    postScript: string;
    version: string;
    previewFamily: string;
};

const systemFontName = ref("Segoe UI");
const sampleText = ref(
    "Твёрдый, как ъ, но и мягкий, словно ь, юноша из Бухары ищет фемину-москвичку для просмотра цветного экрана жизни.",
);

const auroraPresets = [
    ["#ff4d3d", "#2b0a0a", "#ff7a2f"], // spicy red
    ["#4db0ff", "#0a152b", "#6fd3ff"], // blue
    ["#7cff67", "#171d22", "#7cff67"], // green
    ["#7a5bff", "#1a0f2b", "#b08cff"], // purple
];

const auroraStops = ref(
    auroraPresets[Math.floor(Math.random() * auroraPresets.length)],
);
const commonFonts = [
    "Segoe UI",
    "Arial",
    "Times New Roman",
    "Calibri",
    "Cambria",
    "Consolas",
    "Tahoma",
    "Verdana",
    "Georgia",
];

const supportsFontCheck =
    typeof document !== "undefined" && "fonts" in document;
const supportsFontAccess =
    typeof window !== "undefined" &&
    "queryLocalFonts" in window &&
    typeof (window as any).queryLocalFonts === "function";

function normalizeFontFamily(name: string) {
    const trimmed = name.trim();
    if (!trimmed) return "";
    if (
        trimmed.includes(",") ||
        trimmed.includes('"') ||
        trimmed.includes("'")
    ) {
        return trimmed;
    }
    return `"${trimmed}"`;
}

const systemFontFamily = computed(() =>
    normalizeFontFamily(systemFontName.value),
);

const fontCheckText =
    "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
let fontCheckContext: CanvasRenderingContext2D | null = null;

function getFontCheckContext() {
    if (fontCheckContext) return fontCheckContext;
    if (typeof document === "undefined") return null;
    const canvas = document.createElement("canvas");
    fontCheckContext = canvas.getContext("2d");
    return fontCheckContext;
}

function checkFontAvailability(name: string) {
    const family = normalizeFontFamily(name);
    if (!family || !supportsFontCheck) return false;
    let apiAvailable = false;
    try {
        apiAvailable = document.fonts.check(`16px ${family}`);
    } catch {
        return false;
    }
    if (!apiAvailable) return false;
    const ctx = getFontCheckContext();
    if (!ctx) return apiAvailable;
    const size = "72px";
    const fallbacks = ["monospace", "sans-serif", "serif"];
    for (const fallback of fallbacks) {
        ctx.font = `${size} ${fallback}`;
        const baseline = ctx.measureText(fontCheckText).width;
        ctx.font = `${size} ${family}, ${fallback}`;
        const test = ctx.measureText(fontCheckText).width;
        if (test !== baseline) return true;
    }
    return false;
}

const systemFontAvailable = computed(() => {
    return checkFontAvailability(systemFontName.value);
});

const statusClass = computed(() => {
    if (!supportsFontCheck) return "warn";
    if (!systemFontName.value.trim()) return "warn";
    return systemFontAvailable.value ? "ok" : "bad";
});

const statusLabel = computed(() => {
    if (!supportsFontCheck) {
        return "Браузер не поддерживает проверку системных шрифтов.";
    }
    if (!systemFontName.value.trim()) {
        return "Введите название шрифта";
    }
    return systemFontAvailable.value ? "Шрифт найден" : "Шрифт не найден";
});

const systemFontCss = computed(() => {
    const family = systemFontFamily.value;
    if (!family) return "font-family: inherit;";
    return `font-family: ${family};`;
});

const localFont = ref<LocalFontInfo | null>(null);
const localFontFace = ref<FontFace | null>(null);
const localFontError = ref("");
const copyStatus = ref<"" | "system" | "local">("");
const fontAccessList = ref<string[]>([]);
const fontAccessError = ref("");

const localFontFamily = computed(() => {
    if (!localFont.value) return "";
    return normalizeFontFamily(localFont.value.previewFamily);
});

const localFontCss = computed(() => {
    if (!localFont.value) return "";
    const family = normalizeFontFamily(localFont.value.detectedFamily);
    if (!family) return "";
    return `font-family: ${family};`;
});

const lastValidSystemFontFamily = ref("");

watch(
    [systemFontFamily, systemFontAvailable],
    ([family, available]) => {
        if (available && family) {
            lastValidSystemFontFamily.value = family;
        }
    },
    { immediate: true },
);

const activeFontFamily = computed(() => {
    if (localFontFamily.value) return localFontFamily.value;
    if (systemFontAvailable.value) return systemFontFamily.value;
    return lastValidSystemFontFamily.value || "inherit";
});

function pickName(record?: Record<string, string>) {
    if (!record) return "";
    return (
        record["en"] ||
        record["en-US"] ||
        record["en-GB"] ||
        Object.values(record)[0] ||
        ""
    );
}

async function loadLocalFonts() {
    fontAccessError.value = "";
    if (!supportsFontAccess) {
        fontAccessError.value = "Font Access API не поддерживается.";
        return;
    }
    try {
        const fonts = await (window as any).queryLocalFonts();
        const names = Array.from(
            new Set(
                fonts
                    .map((font: any) => font.fullName || font.family)
                    .filter((name: string) => Boolean(name)),
            ),
        ).sort((a, b) => a.localeCompare(b));
        fontAccessList.value = names;
        if (names.length && !systemFontName.value.trim()) {
            systemFontName.value = names[0];
        }
    } catch (error) {
        console.error(error);
        fontAccessError.value =
            "Доступ к локальным шрифтам отклонён или недоступен.";
    }
}

function formatBytes(bytes: number) {
    if (bytes === 0) return "0 B";
    const k = 1024;
    const sizes = ["B", "KB", "MB", "GB"];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return `${(bytes / Math.pow(k, i)).toFixed(i === 0 ? 0 : 1)} ${sizes[i]}`;
}

async function onFileChange(event: Event) {
    localFontError.value = "";
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];
    if (!file) {
        localFont.value = null;
        return;
    }

    try {
        if (localFontFace.value) {
            try {
                document.fonts.delete(localFontFace.value);
            } catch {
                // ignore
            }
            localFontFace.value = null;
        }

        const buffer = await file.arrayBuffer();
        const parsed = opentype.parse(buffer);

        const family =
            pickName(parsed.names.fontFamily) ||
            pickName(parsed.names.preferredFamily);
        const fullName = pickName(parsed.names.fullName) || family;
        const subfamily =
            pickName(parsed.names.fontSubfamily) ||
            pickName(parsed.names.preferredSubfamily);
        const postScript = pickName(parsed.names.postScriptName);
        const version = pickName(parsed.names.version);

        const detectedFamily =
            family || fullName || file.name.replace(/\.[^/.]+$/, "");
        const previewFamily = detectedFamily || `LocalFont-${Date.now()}`;

        const fontFace = new FontFace(previewFamily, buffer);
        await fontFace.load();
        document.fonts.add(fontFace);
        localFontFace.value = fontFace;

        localFont.value = {
            fileName: file.name,
            fileSize: formatBytes(file.size),
            detectedFamily,
            fullName: fullName || detectedFamily,
            subfamily,
            postScript,
            version,
            previewFamily,
        };

        systemFontName.value = detectedFamily;
    } catch (error) {
        console.error(error);
        localFontError.value =
            "Не удалось прочитать шрифт. Попробуйте другой файл.";
        localFont.value = null;
    }
}

function clearLocalFont() {
    localFontError.value = "";
    if (localFontFace.value) {
        try {
            document.fonts.delete(localFontFace.value);
        } catch {
            // ignore
        }
        localFontFace.value = null;
    }
    localFont.value = null;
}

async function copySnippet(text: string, scope: "system" | "local") {
    if (!text) return;
    try {
        await navigator.clipboard.writeText(text);
        copyStatus.value = scope;
        window.setTimeout(() => {
            if (copyStatus.value === scope) copyStatus.value = "";
        }, 1400);
    } catch (error) {
        try {
            const area = document.createElement("textarea");
            area.value = text;
            area.style.position = "fixed";
            area.style.opacity = "0";
            document.body.appendChild(area);
            area.select();
            document.execCommand("copy");
            document.body.removeChild(area);
            copyStatus.value = scope;
            window.setTimeout(() => {
                if (copyStatus.value === scope) copyStatus.value = "";
            }, 1400);
        } catch (fallbackError) {
            console.error(fallbackError);
        }
    }
}
</script>
