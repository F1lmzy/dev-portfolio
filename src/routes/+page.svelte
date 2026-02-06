<script lang="ts">
    import { Canvas } from "@threlte/core";
    import Scene from "$lib/components/balatro/Scene.svelte";
    import CRTLayer from "$lib/components/balatro/CRTLayer.svelte";

    // Dev mode settings
    let showSettings = $state(false);
    
    // Shader parameters
    let spinSpeed = $state(0.75);
    let spinAmount = $state(1.5);
    let contrast = $state(2.0);
    let spinRotation = $state(0.5);
    let pixelFilter = $state(3500.0);
    let uvScale = $state(125.0);
    
    // Card parameters
    let cardScale = $state(1.0);

    function resetToDefaults() {
        spinSpeed = 0.75;
        spinAmount = 1.5;
        contrast = 2.0;
        spinRotation = 0.5;
        pixelFilter = 3500.0;
        uvScale = 125.0;
        cardScale = 1.0;
    }
</script>

<div class="relative w-full h-screen bg-black overflow-hidden font-sans select-none">
    <!-- 3D Scene Layer -->
    <div class="absolute inset-0 z-0">
        <Canvas>
            <Scene
                {spinSpeed}
                {spinAmount}
                {contrast}
                {spinRotation}
                {pixelFilter}
                {uvScale}
                {cardScale}
            />
        </Canvas>
    </div>

    <!-- CRT Effects Layer -->
    <CRTLayer />

    <!-- Settings Button -->
    <button
        class="absolute bottom-4 left-4 z-50 bg-gray-800/80 hover:bg-gray-700/80 text-white p-3 rounded-lg shadow-lg border border-gray-600/50 backdrop-blur-sm transition-all"
        onclick={() => showSettings = !showSettings}
        aria-label="Toggle Settings"
    >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
        </svg>
    </button>

    <!-- Settings Panel -->
    {#if showSettings}
        <div class="absolute bottom-4 left-16 z-50 bg-gray-900/95 text-white p-6 rounded-lg shadow-2xl border border-gray-700/50 backdrop-blur-sm w-80 max-h-[80vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-lg font-bold text-yellow-400">Dev Mode Settings</h2>
                <button
                    class="text-gray-400 hover:text-white text-sm"
                    onclick={resetToDefaults}
                >
                    Reset
                </button>
            </div>

            <!-- Background Shader Settings -->
            <div class="mb-6">
                <h3 class="text-sm font-semibold text-blue-400 mb-3 uppercase tracking-wider">Background Shader</h3>
                
                <div class="space-y-3">
                    <div>
                        <label class="flex justify-between text-xs mb-1">
                            <span>Spin Speed</span>
                            <span class="text-gray-400">{spinSpeed.toFixed(2)}</span>
                        </label>
                        <input
                            type="range"
                            min="0"
                            max="3"
                            step="0.05"
                            bind:value={spinSpeed}
                            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                        />
                    </div>

                    <div>
                        <label class="flex justify-between text-xs mb-1">
                            <span>Spin Amount</span>
                            <span class="text-gray-400">{spinAmount.toFixed(2)}</span>
                        </label>
                        <input
                            type="range"
                            min="0"
                            max="3"
                            step="0.1"
                            bind:value={spinAmount}
                            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                        />
                    </div>

                    <div>
                        <label class="flex justify-between text-xs mb-1">
                            <span>Contrast</span>
                            <span class="text-gray-400">{contrast.toFixed(1)}</span>
                        </label>
                        <input
                            type="range"
                            min="0.5"
                            max="5"
                            step="0.1"
                            bind:value={contrast}
                            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                        />
                    </div>

                    <div>
                        <label class="flex justify-between text-xs mb-1">
                            <span>Spin Rotation</span>
                            <span class="text-gray-400">{spinRotation.toFixed(2)}</span>
                        </label>
                        <input
                            type="range"
                            min="0"
                            max="2"
                            step="0.05"
                            bind:value={spinRotation}
                            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                        />
                    </div>

                    <div>
                        <label class="flex justify-between text-xs mb-1">
                            <span>Pixel Filter (Density)</span>
                            <span class="text-gray-400">{pixelFilter.toFixed(0)}</span>
                        </label>
                        <input
                            type="range"
                            min="500"
                            max="10000"
                            step="100"
                            bind:value={pixelFilter}
                            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                        />
                    </div>

                    <div>
                        <label class="flex justify-between text-xs mb-1">
                            <span>UV Scale</span>
                            <span class="text-gray-400">{uvScale.toFixed(0)}</span>
                        </label>
                        <input
                            type="range"
                            min="10"
                            max="500"
                            step="5"
                            bind:value={uvScale}
                            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                        />
                    </div>
                </div>
            </div>

            <!-- Card Settings -->
            <div>
                <h3 class="text-sm font-semibold text-green-400 mb-3 uppercase tracking-wider">Card Scale</h3>
                
                <div>
                    <label class="flex justify-between text-xs mb-1">
                        <span>Card Scale</span>
                        <span class="text-gray-400">{cardScale.toFixed(2)}x</span>
                    </label>
                    <input
                        type="range"
                        min="0.5"
                        max="2"
                        step="0.05"
                        bind:value={cardScale}
                        class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer"
                    />
                </div>
            </div>
        </div>
    {/if}
</div>
