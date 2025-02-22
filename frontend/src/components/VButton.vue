<script setup lang="ts">
import { watch, computed, useAttrs } from "vue"

import { skipToContentTargetId } from "#shared/constants/window"
import type {
  ButtonConnections,
  ButtonForm,
  ButtonSize,
  ButtonType,
  ButtonVariant,
} from "#shared/types/button"
import { warn } from "~/utils/console"

import VLink from "~/components/VLink.vue"

/**
 * A button component that behaves just like a regular HTML `button` element
 * aside from pre-applied styles based on the passed in variant.
 *
 * All props available for the basic `button` component are available here as
 * well, including an `as` prop which allows for component polymorphism. The
 * most common use case for this prop is to turn the `VButton` component into
 * an `anchor` element, so that you can render a link instead of a `button`.
 *
 * The accessibility helpers on this component are critical and are completely
 * adapted from Reakit's Button, Clickable, and Tabbable component implementations.
 */
const props = withDefaults(
  defineProps<{
    /**
     * Passed to `component :is` to allow the button to *appear* as a button but
     * work like another element (like an `anchor`). May only be either `button` or `VLink`.
     * `anchor` is only supported for the `VSkipToContentButton` component that uses
     * a hash URL (`skipToContentTargetId`).
     *
     * We do not support other elements because their use cases are marginal, and they
     * add complexity that we can avoid otherwise.
     *
     * We also don't allow any old Vue component because Vue does not have ref-forwarding,
     * so we wouldn't be able to detect the type of the DOM node that is ultimately rendered
     * by any Vue component passed.
     *
     * @default 'button'
     */
    as?: ButtonForm
    /**
     * The variant of the button.
     *
     * Plain removes all styles except the focus ring. The button
     * should set a border color, otherwise the browser default is used.
     * Plain--avoid removes _all_ styles including the focus ring.
     */
    variant: ButtonVariant
    /**
     * Allows for programmatically setting the pressed state of a button,
     * i.e., in the case of a button opening a menu.
     */
    pressed?: boolean
    /**
     * The size of the button. `disabled` removes all internal padding allowing
     * the consumer of the component to determine the padding.
     *
     * @default 'medium'
     */
    size: ButtonSize
    /**
     * Whether the button is disabled. Used alone this will only
     * visually effect the button but will not "truly" disable the
     * button unless the `focusable` prop is also set to `false`.
     *
     * @default false
     */
    disabled?: boolean
    /**
     * Whether the button is focusable when disabled. Should be `false`
     * in almost all cases except when a button needs to be focusable
     * while still being disabled (in the case of a form submit button
     * that is disabled due to an incomplete form for example).
     *
     * @default false
     */
    focusableWhenDisabled?: boolean
    /**
     * The HTML `type` attribute for the button. Ignored if `as` is
     * passed as anything other than `"button"`.
     *
     * @default 'button'
     */
    type?: ButtonType
    /**
     * Whether the button is connected to another control and needs to have no rounded
     * borders at that edge.
     *
     * @default []
     */
    connections?: ButtonConnections[]
    /**
     * Whether the button has an icon at the inline start of the button.
     *
     * @default false
     */
    hasIconStart?: boolean
    /**
     * Whether the button has an icon at the inline end of the button.
     *
     * @default false
     */
    hasIconEnd?: boolean
    /**
     * If the button is only an icon, width is set to height, and padding is removed.
     */
    iconOnly?: boolean
  }>(),
  {
    as: "button",
    disabled: false,
    focusableWhenDisabled: false,
    type: "button",
    connections: () => [],
    hasIconStart: false,
    hasIconEnd: false,
    iconOnly: false,
    pressed: undefined,
  }
)

const buttonComponent = computed(() =>
  props.as === "VLink" ? VLink : props.as
)

defineEmits<{
  click: [MouseEvent]
  mousedown: [MouseEvent]
  keydown: [KeyboardEvent]
  focus: [FocusEvent]
  blur: [FocusEvent]
}>()

const attrs = useAttrs()
const typeAttribute = computed<ButtonType | undefined>(() =>
  ["VLink", "a"].includes(props.as) ? undefined : props.type
)

const connectionStyles = computed(() =>
  props.connections.map((connection) => `connection-${connection}`).join(" ")
)

const isActive = computed(() => {
  return props.pressed || attrs["aria-pressed"] || attrs["aria-expanded"]
})
const variantClass = computed(() => {
  if (
    isActive.value &&
    ["bordered-white", "bordered-tx", "transparent-dark"].includes(
      props.variant
    )
  ) {
    return `${props.variant}-pressed`
  }
  return props.variant
})

const isPlainDangerous = computed(() => {
  return props.variant === "plain--avoid"
})
const isFocusSlimFilled = computed(() => {
  return props.variant.startsWith("filled-")
})
const isFocusSlimTx = computed(() => {
  return (
    props.variant.startsWith("bordered-") ||
    props.variant.startsWith("transparent-") ||
    props.variant === "plain"
  )
})

const supportsDisabledAttribute = computed(() => props.as !== "VLink")

const ariaDisabled = computed<true | undefined>(
  () =>
    // If disabled and focusable then use `aria-disabled` instead of the `disabled` attribute to allow
    // the button to still be tabbed into by screen reader users
    (props.disabled && props.focusableWhenDisabled) || undefined
)

const disabledAttribute = computed<true | undefined>(() => {
  const trulyDisabled = props.disabled && !props.focusableWhenDisabled

  return (trulyDisabled && supportsDisabledAttribute.value) || undefined
})

const linkProps = computed(() =>
  props.as === "VLink" ? { href: attrs.href } : {}
)

watch(
  () => props.as,
  (as) => {
    if (
      ["a", "NuxtLink"].includes(as) &&
      attrs.href !== `#${skipToContentTargetId}`
    ) {
      warn(
        `Please use \`VLink\` with an \`href\` prop instead of ${as} for the button component`
      )
    }
  },
  { immediate: true }
)
</script>

<template>
  <Component
    :is="buttonComponent"
    :type="typeAttribute"
    class="group/button button flex appearance-none items-center justify-center rounded-sm no-underline"
    :class="[
      variantClass,
      connectionStyles,
      size,
      {
        'icon-only': iconOnly,
        'icon-start': hasIconStart,
        'icon-end': hasIconEnd,
        border: !isPlainDangerous,
        'focus-visible:outline-tx': isPlainDangerous,
        'focus-slim-filled': isFocusSlimFilled,
        'focus-slim-tx': isFocusSlimTx,
      },
    ]"
    :aria-pressed="pressed"
    :aria-disabled="ariaDisabled"
    :disabled="disabledAttribute"
    v-bind="linkProps"
    @click="$emit('click', $event)"
    @mousedown="$emit('mousedown', $event)"
    @keydown="$emit('keydown', $event)"
    @focus="$emit('focus', $event)"
    @blur="$emit('blur', $event)"
  >
    <!--
      @slot The content of the button
    -->
    <slot />
  </Component>
</template>

<style scoped>
.button[disabled="disabled"],
.button[aria-disabled="true"] {
  @apply cursor-not-allowed;
}

a.button {
  @apply no-underline hover:no-underline;
}

.connection-start {
  @apply rounded-s-none;
}
.connection-end {
  @apply rounded-e-none;
}
.connection-top {
  @apply rounded-se-none rounded-ss-none;
}
.connection-bottom {
  @apply rounded-ee-none rounded-es-none;
}

.filled-pink-8 {
  @apply border-tx bg-primary text-over-dark hover:bg-primary-hover hover:text-over-dark;
}
.filled-dark {
  @apply border-tx bg-tertiary text-over-dark hover:bg-tertiary-hover hover:text-over-dark disabled:opacity-70;
}
.filled-gray {
  @apply border-tx bg-secondary text-default hover:bg-secondary-hover hover:text-over-dark;
}
.filled-white {
  @apply border-tx bg-default text-default hover:bg-secondary-hover hover:text-over-dark;
}
.bordered-white {
  @apply border-bg-ring bg-default text-default hover:border-transparent-hover hover:focus-visible:border-tx;
}
.bordered-white-pressed {
  @apply border-tx bg-tertiary text-over-dark hover:border-hover hover:bg-tertiary-hover hover:focus-visible:border-tx;
}
.bordered-gray {
  @apply border-default bg-default text-default hover:border-hover focus-visible:border-tx hover:focus-visible:border-tx;
}
.bordered-tx {
  @apply border-tx bg-tx text-default hover:border-transparent-hover hover:focus-visible:border-tx dark:hover:border-[--color-gray-9];
}
.bordered-tx-pressed {
  @apply border-tx bg-tertiary text-over-dark hover:border-hover;
}

.transparent-tx {
  @apply border-tx;
}
.transparent-gray {
  @apply border-tx bg-tx text-default hover:bg-transparent-hover disabled:text-disabled;
}
.transparent-dark {
  @apply border-tx bg-tx text-default hover:bg-tertiary hover:text-over-dark;
}
.transparent-dark-pressed {
  @apply border-tx bg-tertiary text-over-dark hover:border-hover;
}

.filled-pink-8[aria-disabled="true"],
.filled-gray[aria-disabled="true"],
.filled-dark[aria-disabled="true"] {
  @apply bg-disabled;
}
.filled-pink-8[aria-disabled="true"],
.filled-dark[aria-disabled="true"] {
  @apply text-over-negative;
}
.filled-gray[aria-disabled="true"],
.filled-white[aria-disabled="true"] {
  @apply text-disabled;
}

.icon-only {
  @apply flex-none;
}
.small {
  @apply h-8 px-2 py-0;
}
.small.icon-only {
  @apply w-8 p-0;
}
.small.icon-start {
  @apply gap-x-1 ps-1;
}
.small.icon-end {
  @apply gap-x-1 pe-1;
}

.medium {
  @apply h-10 px-3 py-0;
}
.medium.icon-only {
  @apply w-10 p-0;
}
.medium.icon-start {
  @apply gap-x-2 ps-2;
}
.medium.icon-end {
  @apply gap-x-2 pe-2;
}

.large {
  @apply h-12 px-5 py-0;
}
.large.icon-only {
  @apply w-12 p-0;
}
.large.icon-start {
  @apply gap-x-2 ps-4;
}
.large.icon-end {
  @apply gap-x-2 pe-4;
}

.larger {
  @apply h-16;
}
.larger.icon-only {
  @apply w-16;
}
</style>
