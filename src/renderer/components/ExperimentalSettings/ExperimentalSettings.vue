<template>
  <FtSettingsSection
    :title="$t('Settings.Experimental Settings.Experimental Settings')"
  >
    <p class="experimental-warning">
      {{ $t('Settings.Experimental Settings.Warning') }}
    </p>
    <FtFlexBox>
      <FtToggleSwitch
        tooltip-position="top"
        :label="$t('Settings.Experimental Settings.Replace HTTP Cache')"
        compact
        :default-value="replaceHttpCache"
        :disabled="replaceHttpCacheLoading"
        :tooltip="$t('Tooltips.Experimental Settings.Replace HTTP Cache')"
        @change="handleRestartPrompt"
      />
    </FtFlexBox>
    <FtFlexBox v-if="isLinux">
      <FtToggleSwitch
        tooltip-position="top"
        :label="$t('Settings.Experimental Settings.Hardware Acceleration')"
        compact
        :default-value="useHardwareAcceleration"
        :disabled="useHardwareAccelerationLoading"
        :tooltip="$t('Tooltips.Experimental Settings.Hardware Acceleration')"
        @change="handleHardwareAccelerationRestartPrompt"
      />
    </FtFlexBox>
    <FtPrompt
      v-if="showRestartPrompt"
      :label="$t('Settings[\'The app needs to restart for changes to take effect. Restart and apply change?\']')"
      :option-names="[$t('Yes, Restart'), $t('Cancel')]"
      :option-values="['restart', 'cancel']"
      @click="handleReplaceHttpCache"
    />
    <FtPrompt
      v-if="showHardwareAccelerationRestartPrompt"
      :label="$t('Settings[\'The app needs to restart for changes to take effect. Restart and apply change?\']')"
      :option-names="[$t('Yes, Restart'), $t('Cancel')]"
      :option-values="['restart', 'cancel']"
      @click="handleUseHardwareAcceleration"
    />
  </FtSettingsSection>
</template>

<script setup>
import { onMounted, ref } from 'vue'

import FtSettingsSection from '../FtSettingsSection/FtSettingsSection.vue'
import FtFlexBox from '../ft-flex-box/ft-flex-box.vue'
import FtToggleSwitch from '../FtToggleSwitch/FtToggleSwitch.vue'
import FtPrompt from '../FtPrompt/FtPrompt.vue'

const isLinux = process.platform === 'linux'

const replaceHttpCacheLoading = ref(true)
const replaceHttpCache = ref(false)
const showRestartPrompt = ref(false)

const useHardwareAccelerationLoading = ref(true)
const useHardwareAcceleration = ref(false)
const showHardwareAccelerationRestartPrompt = ref(false)

onMounted(async () => {
  if (process.env.IS_ELECTRON) {
    replaceHttpCache.value = await window.ftElectron.getReplaceHttpCache()
    if (isLinux) {
      useHardwareAcceleration.value = await window.ftElectron.getUseHardwareAcceleration()
    }
  }

  replaceHttpCacheLoading.value = false
  useHardwareAccelerationLoading.value = false
})

/**
 * @param {boolean} value
 */
function handleRestartPrompt(value) {
  replaceHttpCache.value = value
  showRestartPrompt.value = true
}

/**
 * @param {'restart' | 'cancel' | null} value
 */
function handleReplaceHttpCache(value) {
  showRestartPrompt.value = false

  if (value === null || value === 'cancel') {
    replaceHttpCache.value = !replaceHttpCache.value
    return
  }

  if (process.env.IS_ELECTRON) {
    window.ftElectron.toggleReplaceHttpCache()
  }
}

/**
 * @param {boolean} value
 */
function handleHardwareAccelerationRestartPrompt(value) {
  useHardwareAcceleration.value = value
  showHardwareAccelerationRestartPrompt.value = true
}

/**
 * @param {'restart' | 'cancel' | null} value
 */
function handleUseHardwareAcceleration(value) {
  showHardwareAccelerationRestartPrompt.value = false

  if (value === null || value === 'cancel') {
    useHardwareAcceleration.value = !useHardwareAcceleration.value
    return
  }

  if (process.env.IS_ELECTRON) {
    window.ftElectron.toggleUseHardwareAcceleration()
  }
}
</script>

<style scoped src="./ExperimentalSettings.css" />
