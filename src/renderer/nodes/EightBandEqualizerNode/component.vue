<script setup lang="ts">
import type { Ref } from "vue";
import { onMounted, ref, watch } from "vue";

import CustomNode from "@/components/nodes/CustomNode.vue";
import SpectrumAnalyzerComposite from "@/components/visualizers/SpectrumAnalyzerComposite.vue";
import BandpassIcon from "@/icons/equalizer/BandpassIcon.vue";
import HighpassIcon from "@/icons/equalizer/HighpassIcon.vue";
import HighShelfIcon from "@/icons/equalizer/HighShelfIcon.vue";
import LowpassIcon from "@/icons/equalizer/LowpassIcon.vue";
import LowShelfIcon from "@/icons/equalizer/LowShelfIcon.vue";
import PeakIcon from "@/icons/equalizer/PeakIcon.vue";
import { getThemeColorHex } from "@/logic/color";

import type { AmethystEightBandEqualizerNode } from ".";
import EqualizerBand from "./EqualizerBand.vue";

const props = defineProps<{ node: AmethystEightBandEqualizerNode }>();

const componentKey = ref(0);

// hack to get it to refresh
const forceRerender = () => {
  componentKey.value += 1;
};

const frequencyResponseCanvas = ref() as Ref<HTMLCanvasElement>;

onMounted(() => {
  const ctx = frequencyResponseCanvas.value.getContext("2d")!,
    w = frequencyResponseCanvas.value.width,
    h = frequencyResponseCanvas.value.height,
    frequencies = props.node.calculateFrequencies(frequencyResponseCanvas.value),
    filterMagResponse = new Float32Array(frequencies.length),
    filterPhaseResponse = new Float32Array(frequencies.length),
    frequencyResponse = new Float32Array(frequencies.length);

  const renderFrequencyResponse = () => {
    frequencyResponse.fill(1);
    props.node.filters.forEach((filter) => {
      filter.getFrequencyResponse(
        frequencies,
        filterMagResponse,
        filterPhaseResponse,
      );
      for (let j = 0; j < frequencyResponse.length; j++) {
        frequencyResponse[j] *= filterMagResponse[j];
      }
    });
  };

  const draw = () => {
    renderFrequencyResponse();
    ctx.clearRect(0, 0, w, h);
    ctx.strokeStyle = getThemeColorHex("--primary-1000");
    ctx.lineWidth = 2;
    ctx.beginPath();
    const maxDb = 32;
    const minDb = -32;
    for (let x = 0; x < frequencyResponse.length; x++) {
      const gain = frequencyResponse[x];
      const db = 20 * Math.log10(gain);
      const y = h - ((db - minDb) / (maxDb - minDb)) * h;
      x === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
    }
    ctx.stroke();
  };

  draw();
  watch(() => componentKey.value, () => {
    console.log("cock");
    draw();
  });
});
// watch(() => props.node.frequencyPercent, percent => {
//   props.node.frequency = percentToLogValue(percent, props.node.MIN_FREQUENCY, props.node.MAX_FREQUENCY);
// });

const FILTER_TYPES = [
  // "allpass",
  "lowshelf",
  "highpass",
  "bandpass",
  // "notch",
  "peaking",
  "lowpass",
  "highshelf",
];

</script>

<template>
  <custom-node
    :node="node"
    title="8-Band equalizer"
  >
    <div
      :key="componentKey"
      class="grid grid-cols-4 gap-2 text-primary-900"
      @click="forceRerender()"
    >
      <div
        v-for="(filter, i) of node.filters"
        :key="i"
        class="div flex flex-col h-min w-min font-aseprite bg-surface-1000 p-2 rounded-4px gap-2"
        @mousedown.stop
      >
        <equalizer-band
          v-model="filter.frequency.value"
          :param="filter.frequency"
          :min="10"
          log
          :max="22050"
          :step="0.01"
          :digits="0"
          suffix="Hz"
        />
        <equalizer-band
          v-model="filter.gain.value"
          :param="filter.gain"
          :min="-24"
          :max="+24"
          :step="0.01"
          :digits="2"
          suffix="dB"
        />
        <equalizer-band
          v-model="filter.Q.value"
          :param="filter.Q"
          log
          :min="0.01"
          :max="10"
          :step="0.01"
          suffix="Q"
          :digits="2"
        />
        <div class="flex rounded-2px truncate">
          <button
            v-for="filterType of FILTER_TYPES"
            :key="filterType"
            class="text-11px cursor-pointer px-1 py-1 bg-surface-900"
            :class="[filter.type == filterType ? 'text-accent bg-accent/10' : 'text-surface-500']"
            @mousedown.stop
            @click="filter.type = filterType;"
          >
            <high-shelf-icon
              v-if="filterType == 'highshelf'"
              class="h-4 w-4"
            />
            <low-shelf-icon
              v-else-if="filterType == 'lowshelf'"
              class="h-4 w-4"
            />
            <lowpass-icon
              v-else-if="filterType == 'lowpass'"
              class="h-4 w-4"
            />
            <highpass-icon
              v-else-if="filterType == 'highpass'"
              class="h-4 w-4"
            />
            <peak-icon
              v-else-if="filterType == 'peaking'"
              class="h-4 w-4"
            />
            <bandpass-icon
              v-else-if="filterType == 'bandpass'"
              class="h-4 w-4"
            />
          </button>
        </div>
      </div>
    </div>
    <div class="relative w-full bg-surface-1000 rounded-4px">
      <spectrum-analyzer-composite
        class="h-168px w-full opacity-25"
        type="line"
        :node="node.pre"
      />
      <div class="absolute top-0 left-0 right-0">
        <spectrum-analyzer-composite
          class="h-168px w-full"
          :node="node.post"
          type="line"
        />
      </div>
      <canvas
        ref="frequencyResponseCanvas"
        class="h-168px w-full absolute top-0 left-0"
      />
    </div>
  </custom-node>
</template>
