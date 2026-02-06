<script lang="ts">
    import { T } from "@threlte/core";
    import Card from "./Card.svelte";

    interface Props {
        cards?: { rank: string; suit: string }[];
        cardScale?: number;
    }

    let {
        cards = [
            { rank: "A", suit: "♠" },
            { rank: "K", suit: "♥" },
            { rank: "Q", suit: "♦" },
            { rank: "J", suit: "♣" },
            { rank: "10", suit: "♠" },
        ],
        cardScale = 1.0
    }: Props = $props();

    // Config for the arc
    const spacing = 1.2;
    const radius = 10;
    const arcAngle = 0.5; // Radians
</script>

<T.Group position={[0, -2, 0]}>
    {#each cards as card, i}
        {@const angle =
            (i - (cards.length - 1) / 2) * (arcAngle / cards.length)}
        {@const x = Math.sin(angle) * radius}
        {@const y = Math.cos(angle) * radius - radius}
        {@const rotZ = -angle}

        <T.Group position={[x, y, i * 0.01]} rotation.z={rotZ}>
            <Card rank={card.rank} suit={card.suit} index={i} scale={cardScale} />
        </T.Group>
    {/each}
</T.Group>
