<script setup lang="ts">
import type { mastodon } from 'masto'
import { fetchAccountByHandle } from '~/composables/cache'

type WatcherType = [acc?: mastodon.v1.Account, h?: string, v?: boolean]

defineOptions({
  inheritAttrs: false,
})

const props = defineProps<{
  account?: mastodon.v1.Account
  handle?: string
  disabled?: boolean
}>()

const hoverCard = ref()
const targetIsVisible = ref(false)
const account = ref<mastodon.v1.Account | null | undefined>(props.account)

useIntersectionObserver(
  hoverCard,
  ([{ intersectionRatio }]) => {
    targetIsVisible.value = intersectionRatio > 0.1
  },
)
watch(
  () => [props.account, props.handle, targetIsVisible.value] satisfies WatcherType,
  ([newAccount, newHandle, newVisible], oldProps) => {
    if (newAccount) {
      account.value = newAccount
      return
    }

    if (!newVisible || process.test)
      return

    if (newHandle) {
      const [_oldAccount, oldHandle, _oldVisible] = oldProps ?? [undefined, undefined, false]
      if (!oldHandle || newHandle !== oldHandle || !account.value) {
        // new handle can be wrong: using server instead of webDomain
        fetchAccountByHandle(newHandle).then((acc) => {
          if (newHandle === props.handle)
            account.value = acc
        })
      }

      return
    }

    account.value = undefined
  }, { immediate: true, flush: 'post' },
)

const userSettings = useUserSettings()
</script>

<template>
  <span ref="hoverCard">
    <VMenu v-if="!disabled && account && !getPreferences(userSettings, 'hideAccountHoverCard')" placement="bottom-start" :delay="{ show: 500, hide: 100 }" v-bind="$attrs" :close-on-content-click="false">
      <slot />
      <template #popper>
        <AccountHoverCard v-if="account" :account="account" />
      </template>
    </VMenu>
    <slot v-else />
  </span>
</template>
