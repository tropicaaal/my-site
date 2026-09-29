<script>
    let {
        value,
        digits = undefined,
        pad = "left",
        brokenChance = 0.12,
        scale = 2,
    } = $props();

    function randomTip() {
        const TIP_VARIANTS = [1, 2, 3, 4, 5];
        return TIP_VARIANTS[Math.floor(Math.random() * TIP_VARIANTS.length)];
    }

    function lightSuffix(hasLeftLit, hasRightLit) {
        if (hasLeftLit && hasRightLit) return "_light_both";
        if (hasLeftLit) return "_light_left";
        if (hasRightLit) return "_light_right";
        return "";
    }

    function randomBrokenName(hasLeftLit, hasRightLit) {
        const withStrings = Math.random() < 0.4 ? "_with_strings" : "";
        return `bulb_broken${withStrings}${lightSuffix(hasLeftLit, hasRightLit)}`;
    }

    function buildSlots(value, digits, pad) {
        const raw = String(value);
        const realChars = [];
        for (const ch of raw) {
            if (ch >= "0" && ch <= "9") realChars.push(ch);
            else if (ch === ":") realChars.push("colon");
        }

        let slots = [...realChars];
        if (digits && digits > slots.length) {
            const padCount = digits - slots.length;
            const padSlots = Array.from({ length: padCount }, () => null);
            slots =
                pad === "left"
                    ? [...padSlots, ...slots]
                    : [...slots, ...padSlots];
        }
        return slots;
    }

    function resolveSprites(slots, brokenChance) {
        return slots.map((token, i) => {
            const leftIsLitDigit =
                slots[i - 1] !== undefined && slots[i - 1] !== null;
            const rightIsLitDigit =
                slots[i + 1] !== undefined && slots[i + 1] !== null;

            if (token === null) {
                if (Math.random() < brokenChance) {
                    return {
                        name: randomBrokenName(leftIsLitDigit, rightIsLitDigit),
                        alt: "",
                    };
                }
                const tip = randomTip();
                return {
                    name: `bulb_tip${tip}_empty${lightSuffix(leftIsLitDigit, rightIsLitDigit)}`,
                    alt: "",
                };
            }

            const tip = randomTip();
            return {
                name: `bulb_tip${tip}_${token}`,
                alt: token === "colon" ? ":" : token,
            };
        });
    }

    let slots = $derived(buildSlots(value, digits, pad));
    let tubes = $state([]);

    $effect(() => {
        tubes = resolveSprites(slots, brokenChance);
    });
</script>

<span aria-label={value} class="nixies">
    {#each tubes as tube, i (i)}
        <img
            aria-hidden="true"
            class="tube"
            src={`/nixie/${tube.name}.png`}
            alt={tube.alt}
            width={6 * scale}
            height={10 * scale}
        />
    {/each}
</span>

<style>
    /* workaround for vite's bullshit */
    :global(.nixies) {
        vertical-align: middle;
        display: inline-flex;
        gap: 1px;
    }
</style>
