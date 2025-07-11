<template>
  <Card
    ref="cardRef"
    :data-testid="`template-workflow-${template.name}`"
    class="w-64 template-card rounded-2xl overflow-hidden cursor-pointer shadow-elevation-2 dark-theme:bg-dark-elevation-1.5 h-full"
    :pt="{
      body: { class: 'p-0 h-full flex flex-col' }
    }"
    @click="$emit('loadWorkflow', template.name)"
  >
    <template #header>
      <div class="flex items-center justify-center">
        <div class="relative overflow-hidden rounded-t-lg">
          <template v-if="template.mediaType === 'audio'">
            <AudioThumbnail :src="baseThumbnailSrc" />
          </template>
          <template v-else-if="template.thumbnailVariant === 'compareSlider'">
            <CompareSliderThumbnail
              :base-image-src="baseThumbnailSrc"
              :overlay-image-src="overlayThumbnailSrc"
              :alt="title"
              :is-hovered="isHovered"
              :is-video="
                template.mediaType === 'video' ||
                template.mediaSubtype === 'webp'
              "
            />
          </template>
          <template v-else-if="template.thumbnailVariant === 'hoverDissolve'">
            <HoverDissolveThumbnail
              :base-image-src="baseThumbnailSrc"
              :overlay-image-src="overlayThumbnailSrc"
              :alt="title"
              :is-hovered="isHovered"
              :is-video="
                template.mediaType === 'video' ||
                template.mediaSubtype === 'webp'
              "
            />
          </template>
          <template v-else>
            <DefaultThumbnail
              :src="baseThumbnailSrc"
              :alt="title"
              :is-hovered="isHovered"
              :is-video="
                template.mediaType === 'video' ||
                template.mediaSubtype === 'webp'
              "
              :hover-zoom="
                template.thumbnailVariant === 'zoomHover'
                  ? UPSCALE_ZOOM_SCALE
                  : DEFAULT_ZOOM_SCALE
              "
            />
          </template>
          <ProgressSpinner
            v-if="loading"
            class="absolute inset-0 z-1 w-3/12 h-full"
          />
        </div>
      </div>
    </template>
    <template #content>
      <div class="flex items-center px-4 py-3">
        <div class="flex-1 flex flex-col">
          <h3 class="line-clamp-2 text-lg font-normal mb-0" :title="title">
            {{ title }}
          </h3>
          <p class="line-clamp-2 text-sm text-muted grow" :title="description">
            {{ description }}
          </p>
        </div>
      </div>
    </template>
  </Card>
</template>

<script setup lang="ts">
import { useElementHover } from '@vueuse/core'
import Card from 'primevue/card'
import ProgressSpinner from 'primevue/progressspinner'
import { computed, ref } from 'vue'

import AudioThumbnail from '@/components/templates/thumbnails/AudioThumbnail.vue'
import CompareSliderThumbnail from '@/components/templates/thumbnails/CompareSliderThumbnail.vue'
import DefaultThumbnail from '@/components/templates/thumbnails/DefaultThumbnail.vue'
import HoverDissolveThumbnail from '@/components/templates/thumbnails/HoverDissolveThumbnail.vue'
import { useTemplateWorkflows } from '@/composables/useTemplateWorkflows'
import { TemplateInfo } from '@/types/workflowTemplateTypes'

const UPSCALE_ZOOM_SCALE = 16 // for upscale templates, exaggerate the hover zoom
const DEFAULT_ZOOM_SCALE = 5

const { sourceModule, loading, template } = defineProps<{
  sourceModule: string
  categoryTitle: string
  loading: boolean
  template: TemplateInfo
}>()

const cardRef = ref<HTMLElement | null>(null)
const isHovered = useElementHover(cardRef)

const { getTemplateThumbnailUrl, getTemplateTitle, getTemplateDescription } =
  useTemplateWorkflows()

// Determine the effective source module to use (from template or prop)
const effectiveSourceModule = computed(
  () => template.sourceModule || sourceModule
)

const baseThumbnailSrc = computed(() =>
  getTemplateThumbnailUrl(
    template,
    effectiveSourceModule.value,
    effectiveSourceModule.value === 'default' ? '1' : ''
  )
)

const overlayThumbnailSrc = computed(() =>
  getTemplateThumbnailUrl(
    template,
    effectiveSourceModule.value,
    effectiveSourceModule.value === 'default' ? '2' : ''
  )
)

const description = computed(() =>
  getTemplateDescription(template, effectiveSourceModule.value)
)
const title = computed(() =>
  getTemplateTitle(template, effectiveSourceModule.value)
)

defineEmits<{
  loadWorkflow: [name: string]
}>()
</script>
