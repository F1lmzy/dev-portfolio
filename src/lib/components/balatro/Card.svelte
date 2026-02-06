<script lang="ts">
    import { T } from "@threlte/core";
    import { interactivity } from "@threlte/extras";
    import { spring } from "svelte/motion";
    import { CanvasTexture } from "three";
    import { onMount } from "svelte";

    // Enable Threlte interactivity
    interactivity();

    export let rank = "A";
    export let suit = "♠";
    export let index = 0;
    export let selected = false;

    let canvas: HTMLCanvasElement;
    let texture: CanvasTexture | undefined;

    // Spring animations for position and rotation
    const rotationSpring = spring(
        { x: 0, y: 0, z: 0 },
        { stiffness: 0.1, damping: 0.4 },
    );
    const scaleSpring = spring(1, { stiffness: 0.2, damping: 0.5 });
    const positionSpring = spring(
        { x: 0, y: 0, z: 0 },
        { stiffness: 0.1, damping: 0.6 },
    );

    // Generate card texture
    function createCardTexture() {
        if (typeof document === "undefined") return;

        canvas = document.createElement("canvas");
        canvas.width = 256;
        canvas.height = 384;
        const ctx = canvas.getContext("2d");
        if (!ctx) return;

        // Background
        ctx.fillStyle = "#f0f0f0";
        ctx.fillRect(0, 0, 256, 384);

        // Border
        ctx.strokeStyle = "#1a1a1a"; // Dark border
        ctx.lineWidth = 16;
        ctx.strokeRect(8, 8, 240, 368);

        // Inner design
        ctx.fillStyle = suit === "♥" || suit === "♦" ? "#ff4d4d" : "#1a1a1a";
        ctx.font = 'bold 80px "Courier New", monospace';
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";

        // Center Suit
        ctx.font = 'bold 120px "Courier New", monospace';
        ctx.fillText(suit, 128, 192);

        // Corner Rank
        ctx.font = 'bold 40px "Courier New", monospace';
        ctx.fillText(rank, 40, 50);
        ctx.fillText(suit, 40, 90);

        // Bottom Corner Rank (Rotated)
        ctx.save();
        ctx.translate(216, 334);
        ctx.rotate(Math.PI);
        ctx.fillText(rank, 0, 0);
        ctx.fillText(suit, 0, 40);
        ctx.restore();

        texture = new CanvasTexture(canvas);
        texture.needsUpdate = true; // Ensure texture is flagged for update
    }

    onMount(() => {
        createCardTexture();
    });

    function handlePointerEnter() {
        scaleSpring.set(1.1);
        positionSpring.set({ x: 0, y: 0.5, z: 0.5 }); // Lift up
    }

    function handlePointerLeave() {
        scaleSpring.set(1);
        rotationSpring.set({ x: 0, y: 0, z: 0 });
        positionSpring.set({ x: 0, y: 0, z: 0 });
    }

    function handlePointerMove(e: any) {
        // Calculate tilt based on where mouse is on the card
        // e.point is the intersection point in 3D space
        // We can just use a simple approximation if we don't have the UVs handy easily from the event
        // For now, let's just do a subtle random wobble or look at camera
        // Actually, Threlte's event gives us UV.

        if (e.uv) {
            const x = (e.uv.x - 0.5) * 2; // -1 to 1
            const y = (e.uv.y - 0.5) * 2; // -1 to 1
            rotationSpring.set({ x: -y * 0.5, y: x * 0.5, z: 0 });
        }
    }
</script>

<T.Group
    position.x={$positionSpring.x}
    position.y={$positionSpring.y + (selected ? 0.8 : 0)}
    position.z={$positionSpring.z}
    rotation.x={$rotationSpring.x}
    rotation.y={$rotationSpring.y}
    rotation.z={$rotationSpring.z}
    scale={$scaleSpring}
>
    <T.Mesh
        onpointerenter={handlePointerEnter}
        onpointerleave={handlePointerLeave}
        onpointermove={handlePointerMove}
        onclick={() => (selected = !selected)}
        castShadow
    >
        <!-- Card Shape -->
        <T.BoxGeometry args={[2, 3, 0.05]} />

        <!-- Materials: Front, Back, Sides -->
        <!-- Front (Material Index 4 for BoxGeometry? usually: right, left, top, bottom, front, back) -->
        <!-- Actually standard BoxGeometry UV mapping is face-specific. -->
        <!-- We can use attach to assign materials to specific material slots -->

        <!-- 0: Right, 1: Left, 2: Top, 3: Bottom, 4: Front, 5: Back -->

        <!-- Sides -->
        <T.MeshStandardMaterial attach="material-0" color="#dddddd" />
        <T.MeshStandardMaterial attach="material-1" color="#dddddd" />
        <T.MeshStandardMaterial attach="material-2" color="#dddddd" />
        <T.MeshStandardMaterial attach="material-3" color="#dddddd" />

        <!-- Front (Face) -->
        {#if texture}
            <T.MeshStandardMaterial
                attach="material-4"
                map={texture}
                roughness={0.4}
                metalness={0.1}
            />
        {:else}
            <T.MeshStandardMaterial attach="material-4" color="#ffffff" />
        {/if}

        <!-- Back -->
        <T.MeshStandardMaterial
            attach="material-5"
            color="#ff4d4d"
            roughness={0.6}
        />
    </T.Mesh>
</T.Group>
