<template>
  <DialogRoot :open="isOpen" @update:open="(e) => !preventClose && close(e)">
    <DialogTrigger asChild>
      <slot name="trigger" />
    </DialogTrigger>
    <DialogPortal>
      <Transition
        :enter-active-class="overlayTransition.enter"
        :enter-from-class="overlayTransition.enterFrom"
        :enter-to-class="overlayTransition.enterTo"
        :leave-active-class="overlayTransition.leave"
        :leave-from-class="overlayTransition.leaveFrom"
        :leave-to-class="overlayTransition.leaveTo"
      >
        <DialogOverlay v-if="overlay && isOpen" forceMount :class="[ui.overlay.base, ui.overlay.background]" />
      </Transition>
      <Transition
        :enter-active-class="contentTransition.enter"
        :enter-from-class="contentTransition.enterFrom"
        :enter-to-class="contentTransition.enterTo"
        :leave-active-class="contentTransition.leave"
        :leave-from-class="contentTransition.leaveFrom"
        :leave-to-class="contentTransition.leaveTo"
      >
        <DialogContent v-if="isOpen" forceMount :class="[ui.content, fullscreen ? 'w-screen' : ui.width]">
          <div :class="[ui.container, !fullscreen && ui.padding]">
            <div :class="[
                  ui.base,
                  ui.background,
                  ui.ring,
                  ui.shadow,
                  fullscreen ? 'w-screen' : ui.width,
                  fullscreen ? 'h-screen' : ui.height,
                  fullscreen ? 'rounded-none' : ui.rounded
                ]">
              <!-- <DialogClose>Close</DialogClose>
              <DialogTitle>Title</DialogTitle>
              <DialogDescription>Description</DialogDescription> -->
              <slot />
            </div>
          </div>
        </DialogContent>
      </Transition>
    </DialogPortal>
  </DialogRoot>
</template>

<script lang="ts">
import { computed, toRef, defineComponent } from 'vue'
import type { PropType } from 'vue'
import { DialogClose,  DialogContent,  DialogDescription,  DialogOverlay,  DialogPortal,  DialogRoot,  DialogTitle,  DialogTrigger } from 'radix-vue'
import { useUI } from '../../composables/useUI'
import { mergeConfig } from '../../utils'
import type { Strategy } from '../../types'
// @ts-expect-error
import appConfig from '#build/app.config'
import { modal } from '#ui/ui.config'

const config = mergeConfig<typeof modal>(appConfig.ui.strategy, appConfig.ui.modal, modal)

export default defineComponent({
  components: {
    DialogClose,
    DialogContent,
    DialogDescription,
    DialogOverlay,
    DialogPortal,
    DialogRoot,
    DialogTitle,
    DialogTrigger
  },
  inheritAttrs: false,
  props: {
    modelValue: {
      type: Boolean,
      default: false
    },
    appear: {
      type: Boolean,
      default: false
    },
    overlay: {
      type: Boolean,
      default: true
    },
    transition: {
      type: Boolean,
      default: true
    },
    preventClose: {
      type: Boolean,
      default: false
    },
    fullscreen: {
      type: Boolean,
      default: false
    },
    class: {
      type: [String, Object, Array] as PropType<any>,
      default: undefined
    },
    ui: {
      type: Object as PropType<Partial<typeof config & { strategy?: Strategy }>>,
      default: undefined
    }
  },
  emits: ['update:modelValue', 'close'],
  setup (props, { emit }) {
    const { ui, attrs } = useUI('modal', toRef(props, 'ui'), config, toRef(props, 'class'))

    const isOpen = computed({
      get () {
        return props.modelValue
      },
      set (value) {
        emit('update:modelValue', value)
      }
    })

    const contentTransition = computed(() => {
      if (!props.transition) {
        return {}
      }

      return {
        ...ui.value.transition
      }
    })

    const overlayTransition = computed(() => {
      if (!props.transition) {
        return {}
      }

      return {
        ...ui.value.overlay.transition
      }
    })

    function close (value: boolean) {
      isOpen.value = value

      emit('close')
    }

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      attrs,
      isOpen,
      contentTransition,
      overlayTransition,
      close
    }
  }
})
</script>
