<script lang="ts">
    import { T, useThrelte } from "@threlte/core";
    import { interactivity, OrbitControls } from "@threlte/extras";
    import Background from "./Background.svelte";
    import Hand from "./Hand.svelte";

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
        maxPolarAngle={Math.PI / 1.5}
        minPolarAngle={Math.PI / 3}
    />
</T.PerspectiveCamera>

<T.DirectionalLight position={[5, 10, 5]} intensity={1.5} castShadow />
<T.AmbientLight intensity={0.5} />

<!-- Background Shader -->
<Background />

<!-- Game Layer -->
<Hand />

<!-- Debug Cube to verify rendering (Removed) -->
