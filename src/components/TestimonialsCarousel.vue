<template>
  <div
    class="glass-panel relative overflow-hidden rounded-3xl p-8"
    @mouseenter="pause"
    @mouseleave="resume"
  >
    <div class="flex items-start justify-between gap-4">
      <div>
        <p class="text-sm uppercase tracking-[0.3em] text-brand-200">Featured Story</p>
        <h3 class="mt-3 text-2xl font-semibold text-white">
          {{ active?.name }}
          <span class="ml-2 text-sm font-normal text-slate-300">· {{ active?.role }}</span>
        </h3>
        <p class="mt-1 text-xs uppercase text-brand-100/80">{{ active?.service }}</p>
      </div>
      <div class="flex flex-col items-end gap-2 text-right text-xs text-brand-100/80">
        <div class="flex items-center gap-1 text-brand-200">
          <template v-for="(star, index) in ratingStars.full" :key="`full-${index}`">
            <svg viewBox="0 0 20 20" fill="currentColor" class="h-4 w-4">
              <path d="M10 1.5l2.472 5.005 5.528.803-4 3.903.944 5.5L10 13.75l-4.944 2.96.944-5.5-4-3.903 5.528-.803z" />
            </svg>
          </template>
          <template v-if="ratingStars.half">
            <svg viewBox="0 0 20 20" class="h-4 w-4 text-brand-200">
              <defs>
                <linearGradient :id="gradientId" x1="0%" y1="0%" x2="100%" y2="0%">
                  <stop offset="50%" stop-color="currentColor" />
                  <stop offset="50%" stop-color="transparent" />
                </linearGradient>
              </defs>
              <path
                d="M10 1.5l2.472 5.005 5.528.803-4 3.903.944 5.5L10 13.75l-4.944 2.96.944-5.5-4-3.903 5.528-.803z"
                :fill="`url(#${gradientId})`"
                fill-opacity="1"
                stroke="currentColor"
                stroke-width="0.5"
              />
            </svg>
          </template>
          <template v-for="(star, index) in ratingStars.empty" :key="`empty-${index}`">
            <svg viewBox="0 0 20 20" fill="none" class="h-4 w-4 text-brand-200/30" stroke="currentColor" stroke-width="1">
              <path d="M10 1.5l2.472 5.005 5.528.803-4 3.903.944 5.5L10 13.75l-4.944 2.96.944-5.5-4-3.903 5.528-.803z" />
            </svg>
          </template>
        </div>
        <p>{{ active?.visitDate }}・{{ active?.rating?.toFixed(1) }} ★</p>
      </div>
    </div>

    <blockquote class="mt-6 text-lg leading-relaxed text-slate-200">
      “{{ active?.quote }}”
    </blockquote>

    <div class="mt-8 flex items-center justify-between">
      <div class="flex items-center gap-3">
        <div class="flex h-12 w-12 items-center justify-center rounded-full bg-brand-500/20 text-lg font-semibold text-brand-100">
          {{ initials }}
        </div>
        <div class="text-sm text-slate-300">
          <p class="font-medium text-white">{{ active?.name }}</p>
          <p>{{ active?.service }}</p>
        </div>
      </div>
      <div class="flex items-center gap-2">
        <button
          type="button"
          class="flex h-9 w-9 items-center justify-center rounded-full border border-white/20 bg-white/5 text-white transition hover:border-brand-300 hover:text-brand-100"
          @click="prev"
          aria-label="上一則評價"
        >
          <span aria-hidden="true">‹</span>
        </button>
        <button
          type="button"
          class="flex h-9 w-9 items-center justify-center rounded-full border border-white/20 bg-white/5 text-white transition hover:border-brand-300 hover:text-brand-100"
          @click="next"
          aria-label="下一則評價"
        >
          <span aria-hidden="true">›</span>
        </button>
      </div>
    </div>

    <div class="mt-6 flex justify-center gap-2">
      <button
        v-for="(item, index) in testimonials"
        :key="item.name"
        type="button"
        class="h-2 rounded-full transition"
        :class="[
          index === current
            ? 'w-8 bg-brand-200'
            : 'w-3 bg-white/20 hover:bg-brand-100/40'
        ]"
        @click="goTo(index)"
        :aria-label="`切換到 ${item.name} 的評價`"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';

type Testimonial = {
  name: string;
  role: string;
  quote: string;
  service: string;
  visitDate: string;
  rating?: number;
};

const props = defineProps<{ testimonials: Testimonial[] }>();

const current = ref(0);
let timer: number | null = null;

const normaliseIndex = (index: number) => {
  const length = props.testimonials.length;
  if (!length) return 0;
  return (index + length) % length;
};

const goTo = (index: number) => {
  current.value = normaliseIndex(index);
  if (props.testimonials.length <= 1) return;
  const wasActive = timer !== null;
  clear();
  if (wasActive) {
    start();
  }
};

const next = () => goTo(current.value + 1);
const prev = () => goTo(current.value - 1);

const start = () => {
  if (timer !== null || props.testimonials.length <= 1) return;
  timer = window.setInterval(() => {
    next();
  }, 6500);
};

const clear = () => {
  if (timer === null) return;
  window.clearInterval(timer);
  timer = null;
};

const pause = () => clear();
const resume = () => start();

onMounted(() => {
  start();
});

onBeforeUnmount(() => {
  clear();
});

const active = computed(() => props.testimonials[current.value] ?? null);

const ratingStars = computed(() => {
  const rating = active.value?.rating ?? 0;
  const full = Math.floor(rating);
  const hasHalf = rating - full >= 0.5;
  const empty = Math.max(0, 5 - full - (hasHalf ? 1 : 0));
  return {
    full: Array.from({ length: Math.min(full, 5) }),
    half: hasHalf,
    empty: Array.from({ length: empty })
  };
});

const initials = computed(() => {
  const name = active.value?.name?.trim() ?? '';
  if (!name) return '';
  const segments = name.split(/\s+/).filter(Boolean);
  if (segments.length <= 1) {
    return name.slice(0, 2);
  }
  return segments
    .map((segment) => segment.charAt(0))
    .join('')
    .slice(0, 2)
    .toUpperCase();
});

const gradientId = computed(() => `half-star-${current.value}`);
</script>
