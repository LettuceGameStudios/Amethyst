<script setup lang="ts">
import { useFps } from "@vueuse/core";
import { onMounted, ref } from "vue";

import { amethyst } from "@/amethyst.js";
import SettingsSetting from "@/components/settings/SettingsSetting.vue";
import SliderInput from "@/components/v2/SliderInput.vue";
import SubtitleText from "@/components/v2/SubtitleText.vue";
import TitleText from "@/components/v2/TitleText.vue";
import ToggleSwitch from "@/components/v2/ToggleSwitch.vue";
import { smoothTween } from "@/logic/dom";

const handleToggleVsync = () => {
  window.electron.ipcRenderer.invoke("set-vsync", [performance.useVsync]);
};

const minFps = ref(Number.POSITIVE_INFINITY);
const maxFps = ref(Number.NEGATIVE_INFINITY);
const currentFps = ref(-1);
const audioLatency = ref(-1);
const tweenedFps = ref(-1);
const appMetrics = ref<Electron.ProcessMetric[]>([]);

onMounted(() => {
  const fpsCounter = useFps({ every: 30 });
  setInterval(() => {
    currentFps.value = fpsCounter.value;
    if (currentFps.value > maxFps.value) maxFps.value = currentFps.value;
    if (currentFps.value < minFps.value) minFps.value = currentFps.value;
    amethyst.player.getLatency().then((l) => audioLatency.value = l);
    smoothTween(tweenedFps.value, fpsCounter.value, 1000, (tweenedNumber) => tweenedFps.value = ~~tweenedNumber);
    window.electron.ipcRenderer.invoke<Electron.ProcessMetric[]>("get-app-metrics").then((data) => {
      appMetrics.value = data;
    });
  }, 1000);
});

const { performance } = amethyst.state.settings;
</script>

<template>
  <div class="w-full flex flex-col sm:flex-row gap-2">
    <div class="p-4 w-min rounded-8px bg-settings-setting-background text-text-title flex gap-4 items-center justify-between">
      <div class="flex-col flex justify-center h-full gap-2">
        <subtitle-text
          :text="$t('settings.performance.minimum_framerate')"
          class="text-12px"
        />
        <title-text :text="`${Number.isFinite(minFps) && minFps != -1 ? `${minFps} fps` : 'loading'}`" />
      </div>
      <div class="flex-col flex justify-center h-full gap-2">
        <subtitle-text
          :text="$t('settings.performance.current_framerate')"
          class="text-12px"
        />
        <title-text :text="`${Number.isFinite(tweenedFps) && tweenedFps != -1 ? `${tweenedFps + 1} fps` : 'loading'}`" />
      </div>
      <div class="flex-col flex justify-center h-full gap-2">
        <subtitle-text
          :text="$t('settings.performance.maximum_framerate')"
          class="text-12px"
        />
        <title-text :text="`${Number.isFinite(maxFps) && maxFps != -1 ? `${maxFps} fps` : 'loading'}`" />
      </div>
    </div>
    <div class="p-4 w-min rounded-8px bg-settings-setting-background text-text-title flex flex-col gap-4">
      <div class="flex-col flex justify-center h-full gap-2">
        <subtitle-text
          :text="$t('settings.performance.audio_latency')"
          class="text-12px"
        />
        <title-text :text="`${Number.isFinite(audioLatency) && audioLatency != -1 ? `${audioLatency.toFixed(1)} ms` : 'loading'}`" />
      </div>
    </div>
  </div>
  <settings-setting
    icon="ic:twotone-circle"
    :description="$t('settings.processing_concurrency.description')"
    :title="$t('settings.processing_concurrency.title')"
  >
    <slider-input
      v-model="performance.processingConcurrency"
      :min="1"
      :max="10"
      :step="1"
      suffix="threads"
    />
  </settings-setting>

  <settings-setting
    icon="ic:twotone-monitor-heart"
    :description="$t('settings.vsync.description')"
    :title="$t('settings.vsync.title')"
    :warning="$t('settings.vsync.warning')"
    :platforms="['desktop']"
  >
    <toggle-switch
      v-model="performance.useVsync"
      @change="handleToggleVsync"
    />
  </settings-setting>

  <settings-setting
    :platforms="['desktop']"
    icon="ic:twotone-pause"
    :description="$t('settings.pause_visuals.description')"
    :title="$t('settings.pause_visuals.title')"
  >
    <toggle-switch
      v-model="performance.pauseVisualsWhenUnfocused"
    />
  </settings-setting>
</template>
