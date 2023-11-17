<template>
  <div :class="ui.wrapper">
    <RAccordionRoot
      :default-value="defaultOpenedValue"
      :type="multiple ? 'multiple' : 'single'"
      :collapsible="true"
    >
      <template v-for="(item, index) in items" :key="item.label">
        <RAccordionItem class="" :disabled="item.disabled" :value="item.label" v-slot="{ open }">
          <RAccordionHeader asChild class="">
            <RAccordionTrigger asChild>
              <slot :item="item" :index="index" :open="open" :close="close">
                <UButton v-bind="{ ...omit(ui.default, ['openIcon', 'closeIcon']), ...attrs, ...omit(item, ['slot', 'disabled', 'content', 'defaultOpen']) }">
                <template #trailing>
                  <UIcon
                    :name="!open ? openIcon : closeIcon ? closeIcon : openIcon"
                    :class="[
                      open && !closeIcon ? '-rotate-180' : '',
                      uiButton.icon.size[item.size || uiButton.default.size],
                      ui.item.icon
                    ]"
                  />
                </template>
              </UButton>
            </slot>
            </RAccordionTrigger>
          </RAccordionHeader>
          <Transition
            v-bind="ui.transition"
            @enter="onEnter"
            @after-enter="onAfterEnter"
            @before-leave="onBeforeLeave"
            @leave="onLeave"
          >
            <RAccordionContent v-show="open" :asChild="multiple" forceMount>
              <div :class="[ui.item.base, ui.item.size, ui.item.color, ui.item.padding]">
                <slot :name="item.slot || 'item'" :item="item" :index="index" :open="open" :close="close">
                  {{ item.content }}
                </slot>
              </div>
            </RAccordionContent>
          </Transition>
        </RAccordionItem>
      </template>
    </RAccordionRoot>
  </div>
</template>

<script lang="ts">
import { ref, computed, toRef, defineComponent } from 'vue'
import type { PropType } from 'vue'
import { AccordionContent as RAccordionContent, AccordionHeader as RAccordionHeader, AccordionItem as RAccordionItem, AccordionRoot as RAccordionRoot, AccordionTrigger as RAccordionTrigger } from 'radix-vue'
import UIcon from '../elements/Icon.vue'
import UButton from '../elements/Button.vue'
import { useUI } from '../../composables/useUI'
import { mergeConfig, omit } from '../../utils'
import type { AccordionItem, Strategy } from '../../types'
// @ts-expect-error
import appConfig from '#build/app.config'
import { accordion, button } from '#ui/ui.config'

const config = mergeConfig<typeof accordion>(appConfig.ui.strategy, appConfig.ui.accordion, accordion)

const configButton = mergeConfig<typeof button>(appConfig.ui.strategy, appConfig.ui.button, button)

export default defineComponent({
  components: {
    UIcon,
    UButton,
    RAccordionContent,
    RAccordionHeader,
    RAccordionItem,
    RAccordionRoot,
    RAccordionTrigger
  },
  inheritAttrs: false,
  props: {
    items: {
      type: Array as PropType<AccordionItem[]>,
      default: () => []
    },
    defaultOpen: {
      type: Boolean,
      default: false
    },
    openIcon: {
      type: String,
      default: () => config.default.openIcon
    },
    closeIcon: {
      type: String,
      default: () => config.default.closeIcon
    },
    multiple: {
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
  setup (props) {
    const { ui, attrs } = useUI('accordion', toRef(props, 'ui'), config, toRef(props, 'class'))

    const uiButton = computed<Partial<typeof configButton>>(() => configButton)

    const buttonRefs = ref<{ open: boolean, close: (e: EventTarget) => {} }[]>([])

    const defaultOpenedValue = computed(() => {
      if (props.defaultOpen) {
        if (!props.multiple) {
          console.error('Invalid prop: "defaultOpen" should be used with "multiple" prop')
        }
        return props.items.map((item) => item.label)
      }
      const defaultOpened = props.items.find((item) => item.defaultOpen)

      const defaultValue = defaultOpened ? defaultOpened.label : undefined

      return props.multiple ? [defaultValue] : defaultValue
    })

    function onEnter (el: HTMLElement, done) {
      el.style.height = '0'
      el.offsetHeight // Trigger a reflow, flushing the CSS changes
      el.style.height = el.scrollHeight + 'px'

      el.addEventListener('transitionend', done, { once: true })
    }

    function onBeforeLeave (el: HTMLElement) {
      el.style.height = el.scrollHeight + 'px'
      el.offsetHeight // Trigger a reflow, flushing the CSS changes
    }

    function onAfterEnter (el: HTMLElement) {
      el.style.height = 'auto'
    }

    function onLeave (el: HTMLElement, done) {
      el.style.height = '0'

      el.addEventListener('transitionend', done, { once: true })
    }

    return {
      // eslint-disable-next-line vue/no-dupe-keys
      ui,
      uiButton,
      attrs,
      buttonRefs,
      omit,
      onEnter,
      onBeforeLeave,
      onAfterEnter,
      onLeave,
      defaultOpenedValue
    }
  }
})
</script>
