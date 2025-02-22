<script setup lang="ts">
import { useNuxtApp } from "#imports"
import { computed } from "vue"

import { camelCase } from "#shared/utils/case"
import type { AudioDetail, ImageDetail } from "#shared/types/media"
import { useSearchStore } from "~/stores/search"

import VLink from "~/components/VLink.vue"
import VButton from "~/components/VButton.vue"
import VIcon from "~/components/VIcon/VIcon.vue"

const props = defineProps<{
  sensitivity: AudioDetail["sensitivity"] | ImageDetail["sensitivity"]
  id: string
}>()

const emit = defineEmits<{ reveal: [] }>()

const searchStore = useSearchStore()
const backToSearchPath = computed(() => searchStore.backToSearchPath)

const { $sendCustomEvent } = useNuxtApp()
const handleBack = () => {
  $sendCustomEvent("GO_BACK_FROM_SENSITIVE_RESULT", {
    id: props.id,
    sensitivities: props.sensitivity.join(","),
  })
}
const handleShow = () => {
  emit("reveal")
}
</script>

<template>
  <div
    id="safety-wall"
    class="relative flex h-full w-full flex-grow items-center justify-center border-t border-default bg-default py-8 text-center"
  >
    <section class="mx-auto max-w-2xl px-8 text-sm leading-relaxed">
      <h1 class="heading-5 mb-2">
        {{ $t("sensitiveContent.singleResult.title") }}
      </h1>
      <p class="mb-2">
        {{ $t("sensitiveContent.singleResult.explanation") }}
      </p>
      <p v-for="reason in sensitivity" :key="reason">
        {{
          $t(`sensitiveContent.reasons.${camelCase(reason)}`, {
            openverse: "Openverse",
          })
        }}
      </p>
      <i18n-t
        scope="global"
        tag="p"
        class="mt-2"
        keypath="sensitiveContent.singleResult.learnMore"
      >
        <template #openverse>Openverse</template>
        <template #link>
          <VLink class="text-link hover:underline" href="/sensitive-content">{{
            $t("sensitiveContent.singleResult.link")
          }}</VLink>
          {{ " " }}
        </template>
      </i18n-t>

      <div
        class="mt-6 flex flex-col items-stretch justify-center gap-4 md:flex-row md:gap-6"
      >
        <VButton
          as="VLink"
          size="large"
          variant="filled-dark"
          class="label-bold"
          :href="backToSearchPath || '/'"
          @mousedown="handleBack"
        >
          {{ $t("singleResult.back") }}
        </VButton>
        <VButton
          size="large"
          variant="bordered-gray"
          class="label-bold"
          has-icon-end
          @click="handleShow"
        >
          {{ $t("sensitiveContent.singleResult.show") }}
          <VIcon name="eye-open" />
        </VButton>
      </div>
    </section>
  </div>
</template>
