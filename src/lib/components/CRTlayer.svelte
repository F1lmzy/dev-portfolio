<script lang="ts">
    import { onMount } from "svelte";

    let canvasElement: HTMLCanvasElement | undefined;
    let animationFrame: number;
    let time: number = 0;

    onMount(() => {
        if (!canvasElement) return;

        const canvas = canvasElement;
        const ctx = canvas.getContext("2d");
        if (!ctx) return;

        const resizeCanvas = () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        };

        resizeCanvas();
        window.addEventListener("resize", resizeCanvas);

        const animate = () => {
            time += 0.016;
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw scanlines
            drawScanlines(ctx, canvas.width, canvas.height, time);

            // Draw RGB chromatic aberration
            drawChromaticAberration(ctx, canvas.width, canvas.height, time);

            // Draw vignette
            drawVignette(ctx, canvas.width, canvas.height);

            // Draw screen flicker
            drawFlicker(ctx, canvas.width, canvas.height, time);

            // Draw CRT curvature overlay
            drawCRTCurvature(ctx, canvas.width, canvas.height);

            animationFrame = requestAnimationFrame(animate);
        };

        animate();

        return () => {
            window.removeEventListener("resize", resizeCanvas);
            cancelAnimationFrame(animationFrame);
        };
    });

    function drawScanlines(
        ctx: CanvasRenderingContext2D,
        width: number,
        height: number,
        time: number,
    ) {
        const scanlineSpacing = 4;
        const scanlineOpacity = 0.15;
        const offset = (time * 60) % (scanlineSpacing * 2);

        // Horizontal scanlines
        ctx.strokeStyle = `rgba(0, 0, 0, ${scanlineOpacity})`;
        ctx.lineWidth = 1;

        for (let y = -offset; y < height; y += scanlineSpacing) {
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(width, y);
            ctx.stroke();
        }

        // Add alternating darker lines for more authentic CRT look
        ctx.strokeStyle = `rgba(0, 0, 0, ${scanlineOpacity * 0.5})`;
        for (
            let y = -offset + scanlineSpacing / 2;
            y < height;
            y += scanlineSpacing
        ) {
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(width, y);
            ctx.stroke();
        }
    }

    function drawChromaticAberration(
        ctx: CanvasRenderingContext2D,
        width: number,
        height: number,
        time: number,
    ) {
        // Subtle RGB shift at edges
        const intensity = 0.02 + Math.sin(time * 0.5) * 0.01;

        // Create subtle color fringing at screen edges
        const edgeGradient = ctx.createLinearGradient(0, 0, width, 0);
        edgeGradient.addColorStop(0, `rgba(255, 0, 0, ${intensity})`);
        edgeGradient.addColorStop(0.5, "rgba(0, 0, 0, 0)");
        edgeGradient.addColorStop(1, `rgba(0, 255, 255, ${intensity})`);

        ctx.fillStyle = edgeGradient;
        ctx.fillRect(0, 0, width, height);
    }

    function drawFlicker(
        ctx: CanvasRenderingContext2D,
        width: number,
        height: number,
        time: number,
    ) {
        // Random flicker effect
        const flicker = Math.random() > 0.97 ? 0.03 : 0;
        const brightness = 0.01 + flicker + Math.sin(time * 2.5) * 0.005;

        ctx.fillStyle = `rgba(255, 255, 255, ${brightness})`;
        ctx.fillRect(0, 0, width, height);
    }

    function drawVignette(
        ctx: CanvasRenderingContext2D,
        width: number,
        height: number,
    ) {
        const vignette = ctx.createRadialGradient(
            width / 2,
            height / 2,
            0,
            width / 2,
            height / 2,
            Math.max(width, height) / 1.3,
        );
        vignette.addColorStop(0, "rgba(0, 0, 0, 0)");
        vignette.addColorStop(0.7, "rgba(0, 0, 0, 0.1)");
        vignette.addColorStop(1, "rgba(0, 0, 0, 0.6)");

        ctx.fillStyle = vignette;
        ctx.fillRect(0, 0, width, height);
    }

    function drawCRTCurvature(
        ctx: CanvasRenderingContext2D,
        width: number,
        height: number,
    ) {
        // Draw curved screen edges overlay
        const edgeThickness = 30;

        // Top and bottom curves
        ctx.fillStyle = "rgba(0, 0, 0, 0.8)";

        // Top curve
        ctx.beginPath();
        ctx.moveTo(0, 0);
        ctx.quadraticCurveTo(width / 2, edgeThickness, width, 0);
        ctx.lineTo(width, 0);
        ctx.lineTo(0, 0);
        ctx.fill();

        // Bottom curve
        ctx.beginPath();
        ctx.moveTo(0, height);
        ctx.quadraticCurveTo(width / 2, height - edgeThickness, width, height);
        ctx.lineTo(width, height);
        ctx.lineTo(0, height);
        ctx.fill();

        // Left curve
        ctx.beginPath();
        ctx.moveTo(0, 0);
        ctx.quadraticCurveTo(edgeThickness, height / 2, 0, height);
        ctx.lineTo(0, height);
        ctx.lineTo(0, 0);
        ctx.fill();

        // Right curve
        ctx.beginPath();
        ctx.moveTo(width, 0);
        ctx.quadraticCurveTo(width - edgeThickness, height / 2, width, height);
        ctx.lineTo(width, height);
        ctx.lineTo(width, 0);
        ctx.fill();
    }
</script>

<svelte:head>
    <style>
        /* Global fisheye and CRT effect styles */
        html {
            overflow-x: hidden;
        }

        body {
            margin: 0;
            padding: 0;
            overflow-x: hidden;
            position: relative;
            background: #000;
        }

        /* Apply fisheye barrel distortion using clip-path */
        body > div:first-child {
            position: relative;
            width: 100vw;
            min-height: 100vh;
            /* Subtle barrel distortion effect */
            clip-path: ellipse(98% 97% at 50% 50%);
            transform: scale(1.02);
            transform-origin: center center;
        }
    </style>
</svelte:head>

<div class="crt-container">
    <canvas
        bind:this={canvasElement}
        class="crt-layer"
        aria-label="CRT visual effects overlay"
    ></canvas>
</div>

<!-- Fisheye lens overlay with proper barrel distortion visual -->
<div class="fisheye-overlay">
    <div class="fisheye-inner"></div>
</div>

<style>
    .crt-container {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        pointer-events: none;
        z-index: 9999;
        overflow: hidden;
    }

    canvas {
        width: 100%;
        height: 100%;
        pointer-events: none;
    }

    .fisheye-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
        z-index: 10000;
        overflow: hidden;
    }

    .fisheye-inner {
        position: absolute;
        top: -2%;
        left: -2%;
        width: 104%;
        height: 104%;
        border-radius: 8% / 6%;
        background: radial-gradient(
            ellipse 85% 90% at 50% 50%,
            transparent 0%,
            transparent 45%,
            rgba(0, 0, 0, 0.03) 55%,
            rgba(0, 0, 0, 0.08) 65%,
            rgba(0, 0, 0, 0.15) 75%,
            rgba(0, 0, 0, 0.25) 85%,
            rgba(0, 0, 0, 0.5) 95%,
            rgba(0, 0, 0, 0.8) 100%
        );
        box-shadow:
            inset 0 0 200px rgba(0, 0, 0, 0.6),
            inset 0 0 100px rgba(0, 0, 0, 0.4),
            inset 0 0 50px rgba(0, 0, 0, 0.3),
            /* Curved glass reflection effect */ inset -20px -20px 100px
                rgba(255, 255, 255, 0.02),
            inset 20px 20px 100px rgba(255, 255, 255, 0.02);
    }

    /* Enhanced global styles for barrel distortion effect */
    :global(body) {
        perspective: 1200px;
        perspective-origin: 50% 50%;
    }

    /* Apply subtle transform to content for fisheye simulation */
    :global(body > div:first-of-type) {
        backface-visibility: hidden;
        -webkit-backface-visibility: hidden;
    }

    /* CRT screen bulge effect on main content */
    :global(body::before) {
        content: "";
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
        z-index: 1;
        background: radial-gradient(
            circle at 50% 50%,
            transparent 0%,
            transparent 60%,
            rgba(0, 0, 0, 0.02) 70%,
            rgba(0, 0, 0, 0.05) 80%,
            rgba(0, 0, 0, 0.1) 90%,
            rgba(0, 0, 0, 0.2) 100%
        );
        /* Screen edge shadow to enhance barrel effect */
        box-shadow: inset 0 0 150px rgba(0, 0, 0, 0.3);
        border-radius: 2%;
    }

    /* Apply transform-based distortion for more pronounced fisheye */
    :global(body > div:first-child > *) {
        transform-style: preserve-3d;
    }
</style>
