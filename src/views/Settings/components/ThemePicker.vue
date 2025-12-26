<template>
    <div class="theme-picker">
        <div class="theme-picker__header">
            <div>
                <p class="theme-picker__title">Element Plus Theme Color</p>
                <p class="theme-picker__hint">Use Tailwind's official palette to recolor Element Plus instantly.</p>
            </div>
            <div class="theme-picker__current">
                <span class="theme-picker__chip" :style="{ backgroundColor: currentPrimary }"></span>
                <span class="theme-picker__code">{{ currentPrimary }}</span>
                <el-button size="small" link :disabled="isApplying" @click="initPrimaryColor(defaultPrimary)">
                    Reset
                </el-button>
            </div>
        </div>

        <div class="theme-picker__grid">
            <button
                v-for="color in colorFamilies"
                :key="color.name"
                type="button"
                class="theme-picker__item"
                :class="{ 'is-active': color.base === currentPrimary }"
                :disabled="isApplying"
                @click="selectColor(color)">
                <div
                    class="theme-picker__swatch"
                    :style="{
                        background: `linear-gradient(135deg, ${color.light}, ${color.vivid} 60%, ${color.dark})`
                    }">
                    <span class="theme-picker__badge">{{ color.name }}</span>
                </div>
                <div class="theme-picker__meta">
                    <span class="theme-picker__value">{{ color.base }}</span>
                    <span class="theme-picker__dots">
                        <span class="dot" :style="{ backgroundColor: color.light }"></span>
                        <span class="dot" :style="{ backgroundColor: color.base }"></span>
                        <span class="dot" :style="{ backgroundColor: color.dark }"></span>
                    </span>
                </div>
            </button>
        </div>
    </div>
</template>

<script setup>
    import { computed, onMounted } from 'vue';

    import colors from 'tailwindcss/colors';

    import { useElementTheme } from '../../../composables/useElementTheme';

    // Tailwind indigo-500
    const defaultPrimary = 'oklch(58.5% 0.233 277.117)';
    const { currentPrimary, isApplying, applyPrimaryColor, initPrimaryColor } = useElementTheme(defaultPrimary);

    const invalidKeys = new Set([
        'inherit',
        'current',
        'transparent',
        'black',
        'white',
        'lightBlue',
        'warmGray',
        'trueGray',
        'coolGray',
        'blueGray'
    ]);

    const colorFamilies = computed(() =>
        Object.entries(colors)
            .filter(([name, palette]) => {
                return !invalidKeys.has(name) && palette && typeof palette === 'object' && palette['500'];
            })
            .map(([name, palette]) => {
                const base = palette['500'];
                const light = palette['300'];
                const vivid = palette['600'];
                const dark = palette['700'];
                return {
                    name,
                    base,
                    light,
                    vivid,
                    dark,
                    palette
                };
            })
            .sort((a, b) => a.name.localeCompare(b.name))
    );

    const selectColor = async (color) => {
        await applyPrimaryColor(color.base, color.palette);
    };

    onMounted(async () => {
        await initPrimaryColor(defaultPrimary);
    });
</script>

<style>
    .theme-picker {
        border: 1px solid var(--el-border-color-light);
        border-radius: 14px;
        padding: 14px;
        background: var(--shell-surface, var(--el-bg-color-overlay));
        box-shadow: var(--el-box-shadow-light);
    }

    .theme-picker__header {
        display: flex;
        justify-content: space-between;
        gap: 12px;
        align-items: center;
        margin-bottom: 10px;
    }

    .theme-picker__title {
        margin: 0;
        font-size: 16px;
        font-weight: 700;
        color: var(--el-text-color-primary);
    }

    .theme-picker__hint {
        margin: 2px 0 0 0;
        font-size: 12px;
        color: var(--el-text-color-secondary);
    }

    .theme-picker__current {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        background: var(--el-color-primary-light-9);
        color: var(--el-text-color-primary);
        padding: 6px 10px;
        border-radius: 999px;
        border: 1px solid var(--el-color-primary-light-7);
    }

    .theme-picker__chip {
        width: 18px;
        height: 18px;
        border-radius: 50%;
        box-shadow: 0 0 0 3px var(--el-bg-color);
        border: 1px solid var(--el-border-color-lighter);
    }

    .theme-picker__code {
        font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace;
        font-size: 12px;
        color: var(--el-text-color-primary);
    }

    .theme-picker__grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(170px, 1fr));
        gap: 10px;
    }

    .theme-picker__item {
        all: unset;
        display: flex;
        flex-direction: column;
        gap: 8px;
        cursor: pointer;
        border: 1px solid var(--el-border-color-lighter);
        border-radius: 12px;
        padding: 8px;
        background: var(--el-bg-color);
        transition:
            border-color 0.15s ease,
            box-shadow 0.15s ease,
            transform 0.15s ease;
    }

    .theme-picker__item:hover {
        border-color: var(--el-color-primary);
        box-shadow: var(--el-box-shadow);
        transform: translateY(-2px);
    }

    .theme-picker__item.is-active {
        border-color: var(--el-color-primary);
        box-shadow: var(--el-box-shadow);
    }

    .theme-picker__item:disabled {
        cursor: not-allowed;
        opacity: 0.6;
    }

    .theme-picker__swatch {
        position: relative;
        height: 72px;
        border-radius: 10px;
        overflow: hidden;
        border: 1px solid color-mix(in oklch, var(--el-bg-color) 25%, transparent);
        box-shadow: inset 0 1px 0 color-mix(in oklch, var(--el-bg-color) 35%, transparent);
    }

    .theme-picker__badge {
        position: absolute;
        left: 10px;
        bottom: 8px;
        padding: 4px 8px;
        font-size: 12px;
        font-weight: 600;
        color: var(--el-text-color-primary);
        background: color-mix(in oklch, var(--el-bg-color-overlay) 82%, transparent);
        border-radius: 999px;
        backdrop-filter: blur(6px);
        text-transform: capitalize;
    }

    .theme-picker__meta {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 6px;
    }

    .theme-picker__value {
        font-size: 12px;
        color: var(--el-text-color-regular);
        font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace;
    }

    .theme-picker__dots {
        display: inline-flex;
        gap: 4px;
        align-items: center;
    }

    .theme-picker__dots .dot {
        width: 12px;
        height: 12px;
        border-radius: 50%;
        border: 1px solid var(--el-border-color-light);
    }

    @media (max-width: 768px) {
        .theme-picker__grid {
            grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
        }
        .theme-picker__header {
            flex-direction: column;
            align-items: flex-start;
        }
        .theme-picker__current {
            align-self: flex-start;
        }
    }
</style>
