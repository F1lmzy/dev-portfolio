<script lang="ts">
    import { Canvas } from "@threlte/core";
    import Scene from "$lib/components/balatro/Scene.svelte";
    import CRTLayer from "$lib/components/balatro/CRTLayer.svelte";
</script>

<div
    class="relative w-full h-screen bg-black overflow-hidden font-sans select-none"
>
    <!-- 3D Scene Layer -->
    <div class="absolute inset-0 z-0">
        <Canvas>
            <Scene />
        </Canvas>
    </div>

    <!-- CRT Effects Layer -->
    <CRTLayer />

    <!-- UI Layer (2D Overlay) -->
    <div
        class="absolute inset-0 z-10 pointer-events-none p-8 flex flex-col justify-between"
    >
        <!-- Top Bar: Score -->
        <div class="flex justify-between items-start">
            <div
                class="bg-balatro-dark/90 p-4 border-4 border-white/20 rounded shadow-lg backdrop-blur-sm"
            >
                <h2
                    class="text-white text-sm uppercase tracking-widest mb-1 opacity-70"
                >
                    Round Score
                </h2>
                <div
                    class="text-4xl font-black text-balatro-red flex items-baseline gap-2"
                >
                    <span>1,250</span>
                    <span class="text-lg text-white font-normal opacity-50"
                        >/ 4,000</span
                    >
                </div>
            </div>

            <div
                class="bg-balatro-blue/90 p-4 border-4 border-white/20 rounded shadow-lg backdrop-blur-sm text-center"
            >
                <h2
                    class="text-white text-sm uppercase tracking-widest mb-1 opacity-70"
                >
                    Hands
                </h2>
                <div class="text-3xl font-black text-white">4</div>
            </div>
        </div>

        <!-- Bottom Bar: Controls -->
        <div class="flex justify-center gap-4 mb-8 pointer-events-auto">
            <button
                class="bg-balatro-red hover:bg-red-500 text-white font-bold py-4 px-12 rounded shadow-xl border-b-8 border-red-900 active:border-b-0 active:translate-y-2 transition-all text-xl uppercase tracking-wider"
            >
                Discard
            </button>
            <button
                class="bg-balatro-blue hover:bg-blue-500 text-white font-bold py-4 px-12 rounded shadow-xl border-b-8 border-blue-900 active:border-b-0 active:translate-y-2 transition-all text-xl uppercase tracking-wider"
            >
                Play Hand
            </button>
        </div>
    </div>
</div>
