<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'

interface Totals {
  stars: number
  forks: number
  watchers: number
}

const props = withDefaults(defineProps<{
  // A list of GitHub repos. Supports "owner/repo" or full GitHub URLs.
  repos?: string[]
  // Optional GitHub token to increase rate limits. If not provided, will use import.meta.env.VITE_GITHUB_TOKEN.
  token?: string
  // Cache time in seconds
  cacheTtl?: number
  // Rounded corners
  rounded?: boolean
}>(), {
  repos: () => [
    'soybeanjs/soybean-ui',
    // 'soybeanjs/soybean-headless',
    'soybeanjs/soybean-admin',
    'soybeanjs/soybean-admin-element-plus',
    'soybeanjs/soybean-admin-antd',
    'flipped-aurora/gin-vue-admin',
  ],
  cacheTtl: 60 * 60, // 1 hour
  rounded: true,
})

const loading = ref(false)
const error = ref<string | null>(null)
const totals = ref<Totals>({ stars: 0, forks: 0, watchers: 0 })

function normalizeRepo(input: string): { owner: string, name: string } | null {
  const s = input.trim().replace(/\.git$/, '')
  // owner/repo
  const m1 = s.match(/^(?<owner>[\w.-]+)\/(?<name>[\w.-]+)$/)
  if (m1?.groups)
    return { owner: m1.groups.owner, name: m1.groups.name }

  // https://github.com/owner/repo
  const m2 = s.match(/^https?:\/\/(www\.)?github\.com\/(?<owner>[\w.-]+)\/(?<name>[\w.-]+)(?:\/.*)?$/i)
  if (m2?.groups)
    return { owner: m2.groups.owner, name: m2.groups.name }

  // git@github.com:owner/repo(.git)
  const m3 = s.match(/^git@github\.com:(?<owner>[\w.-]+)\/(?<name>[\w.-]+)$/i)
  if (m3?.groups)
    return { owner: m3.groups.owner, name: m3.groups.name }

  return null
}

function cacheKey(repos: string[]) {
  const key = repos.map(r => normalizeRepo(r))
    .filter(Boolean)
    .map(r => `${(r as any).owner}/${(r as any).name}`)
    .sort()
    .join(',')
  return `achv:gh:${key}`
}

async function fetchRepo(owner: string, name: string, token?: string) {
  const url = `https://api.github.com/repos/${owner}/${name}`
  const headers: Record<string, string> = { Accept: 'application/vnd.github+json' }
  const t = token || import.meta.env?.VITE_GITHUB_TOKEN
  if (t)
    headers.Authorization = `Bearer ${t}`

  const res = await fetch(url, { headers })
  if (!res.ok)
    throw new Error(`GitHub ${owner}/${name} ${res.status}`)
  const data = await res.json()

  // stargazers_count, forks_count always present; subscribers_count often present
  const stars = Number(data?.stargazers_count || 0)
  const forks = Number(data?.forks_count || 0)
  const watchers = Number(
    data?.subscribers_count
    ?? data?.watchers // legacy
    ?? data?.watchers_count // note: same as stargazers in REST v3
    ?? 0,
  )
  return { stars, forks, watchers }
}

async function load() {
  error.value = null
  loading.value = true
  try {
    const items = props.repos.map(normalizeRepo).filter(Boolean) as { owner: string, name: string }[]
    if (!items.length)
      throw new Error('无效的仓库列表')

    // try cache
    const key = cacheKey(props.repos)
    const raw = sessionStorage.getItem(key)
    if (raw) {
      try {
        const parsed = JSON.parse(raw) as { at: number, totals: Totals }
        if (Date.now() - parsed.at < (props.cacheTtl! * 1000)) {
          totals.value = parsed.totals
          loading.value = false
          return
        }
      }
      catch { /* ignore parse errors */ }
    }

    const results = await Promise.allSettled(items.map(it => fetchRepo(it.owner, it.name, props.token)))
    const agg: Totals = { stars: 0, forks: 0, watchers: 0 }
    const failed: string[] = []

    for (let i = 0; i < results.length; i++) {
      const r = results[i]
      const id = `${items[i].owner}/${items[i].name}`
      if (r.status === 'fulfilled') {
        agg.stars += r.value.stars
        agg.forks += r.value.forks
        agg.watchers += r.value.watchers
      }
      else {
        failed.push(id)
      }
    }

    totals.value = agg
    sessionStorage.setItem(key, JSON.stringify({ at: Date.now(), totals: agg }))

    if (failed.length) {
      error.value = `以下仓库获取失败: ${failed.join(', ')}`
    }
  }
  catch (e: any) {
    error.value = e?.message || '获取失败'
  }
  finally {
    loading.value = false
  }
}

load()
</script>

<template>
  <span class="inline-flex items-center">
    <span
      class="markdown-magic-link inline-flex items-center px-4 gap-1.5 leading-tight select-none"
      :class="props.rounded ? 'rounded-full' : 'rounded'"
      role="status"
      aria-live="polite"
      :aria-busy="loading ? 'true' : 'false'"
      :title="error || 'GitHub 统计汇总'"
    >
      <span v-if="loading" class="inline-flex items-center gap-1.5">
        <svg class="spin" viewBox="0 0 24 24" width="12" height="12" aria-hidden="true">
          <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" fill="none" opacity="0.25" />
          <path d="M22 12a10 10 0 0 0-10-10" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" fill="none" />
        </svg>
        <span class="num">加载中…</span>
      </span>

      <template v-else>
        <span class="inline-flex items-center gap-1" aria-label="Stars">
          <svg viewBox="0 0 24 24" width="1.1em" height="1.1em" aria-hidden="true">
            <path fill="currentColor" d="M12 2.5l2.9 5.89 6.5.94-4.7 4.58 1.1 6.48L12 17.77 6.2 20.4l1.1-6.49L2.6 9.33l6.5-.94L12 2.5z" />
          </svg>
          <span class="num">{{ totals.stars.toLocaleString() }}</span>
        </span>
        <span class="opacity-40 mx-1" aria-hidden="true">·</span>
        <span class="inline-flex items-center gap-1" aria-label="Forks">
          <svg viewBox="0 0 24 24" width="1.1em" height="1.1em">
            <g fill="none"><path d="M8 5a3 3 0 1 0-6 0a3 3 0 0 0 6 0m14 0a3 3 0 1 0-6 0a3 3 0 0 0 6 0m-7 14a3 3 0 1 0-6 0a3 3 0 0 0 6 0" /><path stroke="currentColor" stroke-linecap="square" stroke-width="2" d="M12 12H7a2 2 0 0 1-2-2V8m7 4h5a2 2 0 0 0 2-2V8.5M12 12v4M5 8a3 3 0 1 1 0-6a3 3 0 0 1 0 6Zm7 8a3 3 0 1 1 0 6a3 3 0 0 1 0-6ZM22 5a3 3 0 1 0-6 0a3 3 0 0 0 6 0Z" /></g>
          </svg>
          <span class="num">{{ totals.forks.toLocaleString() }}</span>
        </span>
        <span class="opacity-40 mx-1" aria-hidden="true">·</span>
        <span class="inline-flex items-center gap-1" aria-label="Watchers">
          <svg viewBox="0 0 24 24" width="1.1em" height="1.1em" aria-hidden="true">
            <path fill="currentColor" d="M12 5c5.23 0 9.27 3.11 10.94 6.43a1.6 1.6 0 0 1 0 1.14C21.27 15.89 17.23 19 12 19S2.73 15.89 1.06 12.57a1.6 1.6 0 0 1 0-1.14C2.73 8.11 6.77 5 12 5zm0 2C7.9 7 4.56 9.36 3.12 12 4.56 14.64 7.9 17 12 17s7.44-2.36 8.88-5C19.44 9.36 16.1 7 12 7zm0 2.5a2.5 2.5 0 1 1 0 5 2.5 2.5 0 0 1 0-5z" />
          </svg>
          <span class="num">{{ totals.watchers.toLocaleString() }}</span>
        </span>
      </template>
    </span>

    <span v-if="error" class="text-red-500 text-xs ml-2" role="alert">
      {{ error }}
    </span>
  </span>
</template>

<style scoped>
.num {
  @apply text-sm;
  font-variant-numeric: tabular-nums;
  font-weight: 600;
}

svg {
  display: block;
}

.spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
