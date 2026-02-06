<script lang="ts">
    import { T, useThrelte } from "@threlte/core";
    import { interactivity, OrbitControls } from "@threlte/extras";
    import Background from "./Background.svelte";
    import Hand from "./Hand.svelte";

    interface Props {
        spinSpeed?: number;
        spinAmount?: number;
        contrast?: number;
        spinRotation?: number;
        pixelFilter?: number;
        uvScale?: number;
        cardScale?: number;
    }

    let {
        spinSpeed = 0.75,
        spinAmount = 1.5,
        contrast = 2.0,
        spinRotation = 0.5,
        pixelFilter = 3500.0,
        uvScale = 125.0,
        cardScale = 1.0
    }: Props = $props();

    interactivity();

    const { camera } = useThrelte();
</script>

<T.PerspectiveCamera
    makeDefault
    position={[0, 0, 10]}
    fov={50}
>
    <OrbitControls
        enableZoom={false}
        enablePan={false}
        enableRotate={false}
    />
</T.PerspectiveCamera>

<T.DirectionalLight position={[5, 10, 5]} intensity={1.5} castShadow />
<T.AmbientLight intensity={0.5} />

<!-- Background Shader -->
<Background
    {spinSpeed}
    {spinAmount}
    {contrast}
    {spinRotation}
    {pixelFilter}
    {uvScale}
/>

<!-- Game Layer -->
<Hand {cardScale} />
