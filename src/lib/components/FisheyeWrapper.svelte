<script lang="ts">
    import { onMount } from "svelte";

    let { children } = $props();
    let containerEl: HTMLDivElement | undefined;

    onMount(() => {
        if (!containerEl) return;

        // Apply per-character warping for text elements
        const warpText = () => {
            const elements = containerEl!.querySelectorAll(
                "h1, h2, h3, p, span, div",
            );

            elements.forEach((el) => {
                const rect = el.getBoundingClientRect();
                const centerX = window.innerWidth / 2;
                const centerY = window.innerHeight / 2;

                const elCenterX = rect.left + rect.width / 2;
                const elCenterY = rect.top + rect.height / 2;

                const dx = elCenterX - centerX;
                const dy = elCenterY - centerY;
                const distance = Math.sqrt(dx * dx + dy * dy);
                const maxDist = Math.sqrt(
                    centerX * centerX + centerY * centerY,
                );
                const normalizedDist = distance / maxDist;

                // Apply barrel distortion scaling
                const scale = 1 + normalizedDist * normalizedDist * 0.15;
                const perspective = 1000 - normalizedDist * 300;
                const translateZ = -normalizedDist * 50;

                (el as HTMLElement).style.transform = `
                    perspective(${perspective}px)
                    translateZ(${translateZ}px)
                    scale(${scale})
                `;
                (el as HTMLElement).style.transformOrigin = "center center";
            });
        };

        warpText();
        window.addEventListener("resize", warpText);

        // Re-warp on scroll for dynamic effect
        window.addEventListener("scroll", warpText);

        return () => {
            window.removeEventListener("resize", warpText);
            window.removeEventListener("scroll", warpText);
        };
    });
</script>

<!-- SVG for true pixel-level barrel distortion -->
<svg width="0" height="0" style="position: absolute;">
    <defs>
        <!-- Radial displacement map -->
        <radialGradient id="fisheye-gradient" cx="50%" cy="50%" r="50%">
            <stop offset="0%" stop-color="#808080" />
            <stop offset="100%" stop-color="#000000" />
        </radialGradient>

        <filter id="true-fisheye" x="-20%" y="-20%" width="140%" height="140%">
            <!-- Create displacement map from radial gradient -->
            <feImage href="#fisheye-gradient" result="gradient" />

            <!-- Convert to displacement data -->
            <feColorMatrix
                in="gradient"
                type="matrix"
                values="0 0 0 0 0
                        0 0 0 0 0
                        0 0 0 0 0
                        0 0 0 1 -0.5"
                result="displacementMap"
            />

            <!-- Apply barrel distortion -->
            <feDisplacementMap
                in="SourceGraphic"
                in2="displacementMap"
                scale="40"
                xChannelSelector="A"
                yChannelSelector="A"
            />
        </filter>
    </defs>
</svg>

<div class="fisheye-wrapper" bind:this={containerEl}>
    <div class="fisheye-content">
        {@render children()}
    </div>
</div>

<style>
    .fisheye-wrapper {
        position: relative;
        width: 100%;
        min-height: 100vh;
        perspective: 1000px;
        perspective-origin: 50% 50%;
    }

    .fisheye-content {
        position: relative;
        width: 100%;
        min-height: 100vh;
        padding: 40px;

        /* Apply TRUE barrel distortion filter */
        filter: url(#true-fisheye);

        /* Enhance with transform */
        transform-style: preserve-3d;
        transform: scale(1.08);
        transform-origin: center center;
    }

    /* Individual element warping */
    .fisheye-content :global(h1),
    .fisheye-content :global(h2),
    .fisheye-content :global(h3),
    .fisheye-content :global(p) {
        transform-style: preserve-3d;
        backface-visibility: hidden;
        will-change: transform;
    }
</style>
