<script lang="ts" setup>
import { createPopper } from "@popperjs/core/lib/popper-lite"
import offsetPopper from "@popperjs/core/lib/modifiers/offset"
import arrow from "@popperjs/core/lib/modifiers/arrow"
import preventOverflow from "@popperjs/core/lib/modifiers/preventOverflow.js"
import flip from "@popperjs/core/lib/modifiers/flip"
import {
  Instance,
  VirtualElement,
  State,
  Modifier,
} from "@popperjs/core/lib/types"
import { Placement } from "@popperjs/core"

interface Props {
  value: boolean
  placement?: Placement
  offset?: [number, number]
  sameWidth?: boolean
  fixed?: boolean
  disabled?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  placement: "bottom-start",
  offset: () => [0, 8],
  sameWidth: false,
  fixed: false,
  disabled: false,
})

const resizeObserver = ref<ResizeObserver | null>(null)
const wrapper = ref<Element | VirtualElement | null>(null)
const popper = ref<Instance | HTMLElement | null>(null)

watch(
  () => props.value,
  (value: boolean) => {
    if (popper.value === null) return

    if ("setOptions" in popper.value) {
      popper.value.setOptions((options) => ({
        ...options,
        modifiers: [
          ...(options.modifiers ?? []),
          { name: "eventListeners", enabled: value },
        ],
      }))
    }

    if ("update" in popper.value && value) popper.value?.update()
  }
)

onBeforeUnmount(() => {
  if (popper.value === null) return

  if ("destroy" in popper.value) popper.value.destroy()
})

const sameWidth: Modifier<string, Record<string, string>> = {
  name: "sameWidth",
  enabled: true,
  phase: "beforeWrite",
  requires: ["computeStyles"],
  fn: ({ state }: { state: State }) => {
    state.styles.popper.width = `${state.rects.reference.width}px`
  },
}

onMounted(() => {
  if (popper.value === null || wrapper.value === null) return

  const modifiers = [
    {
      name: "computeStyles",
      options: {
        roundOffsets: ({ x, y }: { x: number; y: number }) => ({
          x: Math.round(x + 2),
          y: Math.round(y + 2),
        }),
      },
    },
    { ...offsetPopper, options: { offset: props.offset } },
    { ...preventOverflow, options: { padding: 8 } },
    { ...arrow },
    { ...flip, options: { padding: 12 } },
  ] as Modifier<string, Record<string, string>>[]

  if (props.sameWidth) {
    modifiers.push(sameWidth)
  }

  if (popper.value instanceof HTMLElement) {
    popper.value = createPopper(wrapper.value, popper.value, {
      placement: props.placement,
      strategy: props.fixed ? "fixed" : "absolute",
      modifiers,
    })
  }

  resizeObserver.value = new ResizeObserver((entries) => {
    update()
  })

  resizeObserver.value.observe(wrapper.value as Element)
})

const update = () => {
  if (popper.value === null) return

  if ("update" in popper.value) popper?.value.update()
}
</script>

<template>
  <div class="base-popper">
    <div ref="wrapper" class="wrapper">
      <slot />
    </div>

    <div ref="popper" class="popper">
      <transition name="fade">
        <div v-show="value">
          <slot name="content" />
        </div>
      </transition>
    </div>
  </div>
</template>

<style scoped>
.base-popper__content {
  z-index: 50;
}

/* Content animation */

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
