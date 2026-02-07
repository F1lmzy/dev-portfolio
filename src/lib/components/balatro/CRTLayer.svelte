<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import * as THREE from 'three';

    // Shader parameters with default values matching the Godot shader
    let {
        scanlinesOpacity = 0.4,
        scanlinesWidth = 0.25,
        grilleOpacity = 0.3,
        resolution = new THREE.Vector2(640, 480),
        pixelate = true,
        roll = true,
        rollSpeed = 8.0,
        rollSize = 15.0,
        rollVariation = 1.8,
        distortIntensity = 0.05,
        noiseOpacity = 0.4,
        noiseSpeed = 5.0,
        staticNoiseIntensity = 0.06,
        aberration = 0.03,
        brightness = 1.4,
        discolor = true,
        warpAmount = 1.0,
        clipWarp = false,
        vignetteIntensity = 0.4,
        vignetteOpacity = 0.5
    } = $props();

    let canvas: HTMLCanvasElement;
    let renderer: THREE.WebGLRenderer;
    let scene: THREE.Scene;
    let camera: THREE.OrthographicCamera;
    let material: THREE.ShaderMaterial;
    let animationId: number;
    let startTime: number;

    // Vertex Shader
    const vertexShader = `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `;

    // Fragment Shader (Ported from Godot)
    const fragmentShader = `
        uniform float time;
        uniform vec2 resolution;
        uniform float scanlines_opacity;
        uniform float scanlines_width;
        uniform float grille_opacity;
        uniform bool pixelate;
        uniform bool roll;
        uniform float roll_speed;
        uniform float roll_size;
        uniform float roll_variation;
        uniform float distort_intensity;
        uniform float noise_opacity;
        uniform float noise_speed;
        uniform float static_noise_intensity;
        uniform float aberration;
        uniform float brightness;
        uniform bool discolor;
        uniform float warp_amount;
        uniform bool clip_warp;
        uniform float vignette_intensity;
        uniform float vignette_opacity;

        varying vec2 vUv;

        // Random function
        vec2 random(vec2 uv){
            uv = vec2( dot(uv, vec2(127.1,311.7) ),
                       dot(uv, vec2(269.5,183.3) ) );
            return -1.0 + 2.0 * fract(sin(uv) * 43758.5453123);
        }

        // Noise function
        float noise(vec2 uv) {
            vec2 uv_index = floor(uv);
            vec2 uv_fract = fract(uv);

            vec2 blur = smoothstep(0.0, 1.0, uv_fract);

            return mix( mix( dot( random(uv_index + vec2(0.0,0.0) ), uv_fract - vec2(0.0,0.0) ),
                             dot( random(uv_index + vec2(1.0,0.0) ), uv_fract - vec2(1.0,0.0) ), blur.x),
                        mix( dot( random(uv_index + vec2(0.0,1.0) ), uv_fract - vec2(0.0,1.0) ),
                             dot( random(uv_index + vec2(1.0,1.0) ), uv_fract - vec2(1.0,1.0) ), blur.x), blur.y) * 0.5 + 0.5;
        }

        // Warp function
        vec2 warp(vec2 uv){
            vec2 delta = uv - 0.5;
            float delta2 = dot(delta.xy, delta.xy);
            float delta4 = delta2 * delta2;
            float delta_offset = delta4 * warp_amount;
            
            return uv + delta * delta_offset;
        }

        // Border function
        float border (vec2 uv){
            float radius = min(warp_amount, 0.08);
            radius = max(min(min(abs(radius * 2.0), abs(1.0)), abs(1.0)), 1e-5);
            vec2 abs_uv = abs(uv * 2.0 - 1.0) - vec2(1.0, 1.0) + radius;
            float dist = length(max(vec2(0.0), abs_uv)) / radius;
            float square = smoothstep(0.96, 1.0, dist);
            return clamp(1.0 - square, 0.0, 1.0);
        }

        // Vignette function
        float vignette(vec2 uv){
            vec2 vuv = uv * (1.0 - uv.xy);
            float v = vuv.x * vuv.y * 15.0;
            return pow(clamp(v, 0.0, 1.0), vignette_intensity * vignette_opacity);
        }

        void main() {
            vec2 uv = warp(vUv);
            
            // To simulate an overlay without access to the background texture,
            // we will calculate the darkening effects (scanlines, vignette, grille)
            // and the additive effects (noise).
            
            // 1. Calculate Masking (Darkening)
            // Start with full transmission (White = 1.0)
            vec3 mask = vec3(1.0);
            
            // Grille
            if (grille_opacity > 0.0){
                float g_r = smoothstep(0.85, 0.95, abs(sin(uv.x * (resolution.x * 3.14159265))));
                float r = mix(1.0, g_r, grille_opacity);
                
                float g_g = smoothstep(0.85, 0.95, abs(sin(1.05 + uv.x * (resolution.x * 3.14159265))));
                float g = mix(1.0, g_g, grille_opacity);
                
                float g_b = smoothstep(0.85, 0.95, abs(sin(2.1 + uv.x * (resolution.x * 3.14159265))));
                float b = mix(1.0, g_b, grille_opacity);
                
                mask *= vec3(r, g, b);
            }
            
            // Scanlines
            float scanlines = 0.5;
            if (scanlines_opacity > 0.0)
            {
                scanlines = smoothstep(scanlines_width, scanlines_width + 0.5, abs(sin(uv.y * (resolution.y * 3.14159265))));
                mask *= mix(vec3(1.0), vec3(scanlines), scanlines_opacity);
            }
            
            // Vignette
            mask *= vignette(uv);
            
            // Border
            mask *= border(uv);
            
            // 2. Calculate Additive Noise
            vec3 noise_color = vec3(0.0);
            float roll_line = 0.0;
            
            if (roll || noise_opacity > 0.0)
            {
                float t = roll ? time : 0.0;
                roll_line = smoothstep(0.3, 0.9, sin(uv.y * roll_size - (t * roll_speed) ) );
                roll_line *= roll_line * smoothstep(0.3, 0.9, sin(uv.y * roll_size * roll_variation - (t * roll_speed * roll_variation) ) );
            }
            
            if (noise_opacity > 0.0)
            {
                float n = smoothstep(0.4, 0.5, noise(uv * vec2(2.0, 200.0) + vec2(10.0, (time * (noise_speed))) ) );
                roll_line *= n * scanlines * clamp(random((ceil(uv * resolution) / resolution) + vec2(time * 0.8, 0.0)).x + 0.8, 0.0, 1.0);
                noise_color += vec3(roll_line * noise_opacity);
            }
            
            if (static_noise_intensity > 0.0)
            {
                noise_color += clamp(random((ceil(uv * resolution) / resolution) + fract(time)).x, 0.0, 1.0) * static_noise_intensity;
            }
            
            // 3. Composite
            // We want to darken the background by 'mask' and add 'noise_color'.
            // In standard Alpha blending: Final = Src * (1 - Alpha) + Dest * Alpha.
            // If Dest is (0,0,0, 1-mask), then Final = Src * mask. (Darkening)
            // If Dest is (Noise, 1), then Final = Noise.
            
            // We can approximate this by outputting a color that blends well.
            // If we output Black with alpha = (1.0 - average(mask)), we get darkening.
            // But 'mask' is colored (Grille).
            // A simple alpha overlay cannot do component-wise multiplication (Grille).
            // We will average the mask for alpha.
            
            float alpha_mask = 1.0 - ((mask.r + mask.g + mask.b) / 3.0);
            
            // Add noise importance
            vec3 final_color = noise_color;
            float final_alpha = alpha_mask + length(noise_color); 
            
            // Clamp
            final_alpha = clamp(final_alpha, 0.0, 1.0);
            
            // Adjust final color to be black where we just want to darken
            // If we want to darken, color should be black.
            // If we want to add noise, color should be noise.
            // Weighted average?
            // Actually, if we just output vec4(noise_color, final_alpha) ??
            // If noise is 0, output (0,0,0, alpha_mask). -> Src * (1-alpha) + 0 -> Src * mask. Correct-ish.
            
            gl_FragColor = vec4(final_color, final_alpha);
            
            if (clip_warp)
            {
                gl_FragColor.a = border(uv);
            }
        }
    `;

    onMount(() => {
        init();
        animate();
        window.addEventListener('resize', onWindowResize);
    });

    onDestroy(() => {
        if (renderer) renderer.dispose();
        if (material) material.dispose();
        if (typeof window !== 'undefined') window.removeEventListener('resize', onWindowResize);
        if (animationId) cancelAnimationFrame(animationId);
    });

    function init() {
        if (!canvas) return;

        startTime = Date.now();
        const width = window.innerWidth;
        const height = window.innerHeight;

        scene = new THREE.Scene();
        camera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1);

        renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: false });
        renderer.setSize(width, height);
        renderer.setPixelRatio(window.devicePixelRatio);

        const geometry = new THREE.PlaneGeometry(2, 2);
        material = new THREE.ShaderMaterial({
            uniforms: {
                time: { value: 0 },
                resolution: { value: resolution },
                scanlines_opacity: { value: scanlinesOpacity },
                scanlines_width: { value: scanlinesWidth },
                grille_opacity: { value: grilleOpacity },
                pixelate: { value: pixelate },
                roll: { value: roll },
                roll_speed: { value: rollSpeed },
                roll_size: { value: rollSize },
                roll_variation: { value: rollVariation },
                distort_intensity: { value: distortIntensity },
                noise_opacity: { value: noiseOpacity },
                noise_speed: { value: noiseSpeed },
                static_noise_intensity: { value: staticNoiseIntensity },
                aberration: { value: aberration },
                brightness: { value: brightness },
                discolor: { value: discolor },
                warp_amount: { value: warpAmount },
                clip_warp: { value: clipWarp },
                vignette_intensity: { value: vignetteIntensity },
                vignette_opacity: { value: vignetteOpacity }
            },
            vertexShader,
            fragmentShader,
            transparent: true
        });

        const mesh = new THREE.Mesh(geometry, material);
        scene.add(mesh);
    }

    function onWindowResize() {
        if (!renderer) return;
        const width = window.innerWidth;
        const height = window.innerHeight;
        renderer.setSize(width, height);
        // We might want to update resolution uniform here if it's meant to match screen size
        // But the prop 'resolution' usually defines the "virtual" resolution (e.g. 640x480)
    }

    function animate() {
        animationId = requestAnimationFrame(animate);
        if (material) {
            material.uniforms.time.value = (Date.now() - startTime) / 1000;
        }
        if (renderer && scene && camera) {
            renderer.render(scene, camera);
        }
    }

    // Reactive updates for uniforms
    $effect(() => {
        if (material) {
            material.uniforms.resolution.value = resolution;
            material.uniforms.scanlines_opacity.value = scanlinesOpacity;
            material.uniforms.scanlines_width.value = scanlinesWidth;
            material.uniforms.grille_opacity.value = grilleOpacity;
            material.uniforms.pixelate.value = pixelate;
            material.uniforms.roll.value = roll;
            material.uniforms.roll_speed.value = rollSpeed;
            material.uniforms.roll_size.value = rollSize;
            material.uniforms.roll_variation.value = rollVariation;
            material.uniforms.distort_intensity.value = distortIntensity;
            material.uniforms.noise_opacity.value = noiseOpacity;
            material.uniforms.noise_speed.value = noiseSpeed;
            material.uniforms.static_noise_intensity.value = staticNoiseIntensity;
            material.uniforms.aberration.value = aberration;
            material.uniforms.brightness.value = brightness;
            material.uniforms.discolor.value = discolor;
            material.uniforms.warp_amount.value = warpAmount;
            material.uniforms.clip_warp.value = clipWarp;
            material.uniforms.vignette_intensity.value = vignetteIntensity;
            material.uniforms.vignette_opacity.value = vignetteOpacity;
        }
    });

</script>

<canvas
    bind:this={canvas}
    class="pointer-events-none fixed inset-0 z-50 w-full h-full block"
></canvas>

