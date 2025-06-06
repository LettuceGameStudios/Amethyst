<script setup lang="ts">
import type { KeyFilter } from "@vueuse/core";
import { onKeyDown, onKeyUp } from "@vueuse/core";
import { onMounted, ref, watch } from "vue";

import { amethyst } from "@/amethyst.js";
const props = defineProps<{
  button: KeyFilter;
}>();

const active = ref(false);

onMounted(() => {
  if (props.button === "CTRL") {
    watch(() => amethyst.shortcuts.isControlPressed.value, (value) => active.value = value);
  }
  else if (props.button === "SHIFT") {
    watch(() => amethyst.shortcuts.isShiftPressed.value, (value) => active.value = value);
  }
  else if (props.button === "ALT") {
    watch(() => amethyst.shortcuts.isAltPressed.value, (value) => active.value = value);
  }
  else {
    // Deals with lowercase and uppercase
    onKeyDown(props.button, () => active.value = true);
    onKeyDown(props.button.toString().toLocaleLowerCase(), () => active.value = true);
    onKeyUp(props.button, () => active.value = false);
    onKeyUp(props.button.toString().toLocaleLowerCase(), () => active.value = false);
  }

  // Fixes sticky buttons when a popup happens and we lose focus
  watch(() => amethyst.state.window.isFocused, (isFocused) => {
    if (!isFocused) active.value = false;
  });
});

</script>

<template>
  <kbd
    class="text-7px font-aseprite"
    :class="[active && 'active']"
  >
    <slot />
  </kbd>
</template>

<style scoped lang="postcss">
kbd {
  @apply mx-0.25 py-0.5 pt-1 px-1.25 rounded-3px border-solid border-1 border-surface-900 bg-surface-800;
  line-height: 1.5;
  box-shadow: 0px 2px 0px rgba(0, 0, 0, 0.5), inset 0px 0px 0px 2px rgb(var(--surface-600));

  &.active {
    @apply bg-accent text-black transform-gpu translate-y-2px;
    box-shadow: inset 0px 0px 0px 2px rgb(var(--accent));
  }
}
</style>
