<script setup lang="ts">
import type { Link } from '@/types'
import { CircleArrowRight } from 'lucide-vue-next'
import { toast } from 'vue-sonner'
import heroUrl from '@/assets/images/hero.svg?url'

const { title, description, github } = useAppConfig()

const createOpen = ref(false)
const formRef = ref<{ randomSlug: () => void } | null>(null)

function onShortenClick() {
  if (!getAuthToken()) {
    navigateTo('/dashboard/login')
    return
  }
  createOpen.value = true
}

watch([createOpen, formRef], ([open, form]) => {
  if (open && form)
    nextTick(() => form.randomSlug())
})

async function handleSuccess(link: Link) {
  createOpen.value = false
  const shortLink = `${window.location.origin}/${link.slug}`
  try {
    await navigator.clipboard.writeText(shortLink)
    toast.success(shortLink, { description: 'Copied to clipboard' })
  }
  catch {
    toast.success(shortLink)
  }
}
</script>

<template>
  <section class="flex flex-1 items-center">
    <div
      class="
        w-full py-16
        md:py-24
      "
    >
      <div
        class="
          mx-auto flex max-w-6xl flex-col items-center gap-12 px-6
          lg:flex-row lg:justify-between
        "
      >
        <div
          class="
            max-w-lg text-center
            lg:text-left
          "
        >
          <h1
            class="
              text-4xl font-medium text-balance
              md:text-5xl
              xl:text-6xl
            "
          >
            {{ title }}
          </h1>
          <p class="mt-6 text-lg text-pretty text-muted-foreground">
            {{ description }}
          </p>

          <div
            class="
              mt-10 flex flex-col items-center justify-center gap-2
              sm:flex-row
              lg:justify-start
            "
          >
            <Button
              size="lg"
              class="h-12 px-8 text-lg"
              @click="onShortenClick"
            >
              <span class="text-nowrap">Shorten Link</span>
              <CircleArrowRight aria-hidden="true" class="size-5" />
            </Button>
            <Button
              as-child
              size="lg"
              variant="ghost"
              class="h-12 px-5 text-lg text-muted-foreground"
            >
              <NuxtLink to="/dashboard">
                <span class="text-nowrap">{{ $t('dashboard.title') }}</span>
              </NuxtLink>
            </Button>
          </div>

          <p
            class="mt-8 text-sm text-muted-foreground"
          >
            Runs on Cloudflare · Built with Nuxt ·
            <a
              :href="github"
              target="_blank"
              rel="noopener noreferrer"
              class="
                underline underline-offset-4
                hover:text-primary
              "
            >Open source</a>
          </p>
        </div>

        <object
          type="image/svg+xml"
          :data="heroUrl"
          class="
            hidden aspect-square w-96 shrink-0
            md:block
            lg:w-[420px]
          "
          aria-label="Link sharing illustration"
          suppressHydrationWarning
        />
      </div>
    </div>

    <ResponsiveModal
      v-model:open="createOpen"
      :title="$t('links.create')"
    >
      <LazyDashboardLinksEditorForm
        ref="formRef"
        :link="{}"
        :is-edit="false"
        @success="handleSuccess"
      />

      <template #footer>
        <Button
          type="button"
          variant="secondary"
          @click="createOpen = false"
        >
          {{ $t('common.close') }}
        </Button>
        <Button type="submit" form="link-editor-form">
          {{ $t('common.save') }}
        </Button>
      </template>
    </ResponsiveModal>
  </section>
</template>
