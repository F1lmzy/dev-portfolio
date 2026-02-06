<script lang="ts">
    import { T } from "@threlte/core";
    import { interactivity } from "@threlte/extras";
    import { spring } from "svelte/motion";
    import { CanvasTexture, MeshStandardMaterial } from "three";

    // Enable Threlte interactivity
    interactivity();

    interface Props {
        rank?: string;
        suit?: string;
        index?: number;
        selected?: boolean;
        scale?: number;
    }

    let { 
        rank = "A", 
        suit = "♠", 
        index = 0, 
        selected = $bindable(false),
        scale: cardScale = 1.0
    }: Props = $props();

    // Generate card texture immediately
    function createCardTexture() {
        if (typeof document === "undefined") return undefined;

        const canvas = document.createElement("canvas");
        canvas.width = 256;
        canvas.height = 384;
        const ctx = canvas.getContext("2d");
        if (!ctx) return undefined;

        // Background
        ctx.fillStyle = "#f0f0f0";
        ctx.fillRect(0, 0, 256, 384);

        // Border
        ctx.strokeStyle = "#888888"; // Grey border
        ctx.lineWidth = 4;
        ctx.strokeRect(2, 2, 252, 380);

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

        const texture = new CanvasTexture(canvas);
        texture.needsUpdate = true;
        return texture;
    }

    // Create materials array - only run once
    let materials = $state<MeshStandardMaterial[]>([]);
    let initialized = $state(false);
    
    $effect(() => {
        if (initialized) return;
        initialized = true;
        
        const texture = createCardTexture();
        materials = [
            new MeshStandardMaterial({ color: "#dddddd" }), // right
            new MeshStandardMaterial({ color: "#dddddd" }), // left
            new MeshStandardMaterial({ color: "#dddddd" }), // top
            new MeshStandardMaterial({ color: "#dddddd" }), // bottom
            new MeshStandardMaterial({ 
                map: texture || undefined,
                color: texture ? undefined : "#ffffff",
                roughness: 0.4,
                metalness: 0.1 
            }), // front
            new MeshStandardMaterial({ color: "#ff4d4d", roughness: 0.6 }), // back
        ];
    });

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
    scale={$scaleSpring * cardScale}
>
    <T.Mesh
        onpointerenter={handlePointerEnter}
        onpointerleave={handlePointerLeave}
        onpointermove={handlePointerMove}
        onclick={() => (selected = !selected)}
        castShadow
        material={materials}
    >
        <!-- Card Shape -->
        <T.BoxGeometry args={[2, 3, 0.05]} />
    </T.Mesh>
</T.Group>
