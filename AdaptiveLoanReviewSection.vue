<template>
  <section class="review flex flex-col gap-4">
    <article v-for="section in summary" :key="section.title" class="check-section rounded-lg overflow-hidden bg-white" style="border:2px solid #e5e7eb">
      <header class="check-header flex items-center justify-between gap-3 px-5 py-3 bg-gray-50" style="border-bottom:1px solid #e5e7eb">
        <h3 class="check-title text-lg font-bold" style="margin:0">{{ section.title }}</h3>
        <button
          v-if="section.step"
          type="button"
          class="link-btn px-3 py-2 rounded text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer"
          @click="$emit('edit-step', section.step)"
        >
          Change
        </button>
      </header>

      <div v-for="(item, index) in section.items" :key="index" class="check-item px-5 py-3">
        <p v-if="section.items.length > 1 || item.heading !== section.title" class="check-heading">
          {{ item.heading }}
        </p>
        <div v-for="row in item.rows" :key="row[0]" class="check-row flex flex-wrap justify-between gap-4 py-1">
          <span class="check-label text-gray-600" style="flex:1 1 180px">{{ row[0] }}</span>
          <span class="check-value font-semibold text-right" style="flex:1 1 180px;overflow-wrap:anywhere">{{ row[1] || '—' }}</span>
        </div>
      </div>
    </article>

    <p class="hint mt-2 text-base text-gray-600">When everything looks right, tap "Send my application" below.</p>
  </section>
</template>

<script>
/**
 * Final review step. Shows everything the applicant entered, section by
 * section, with a "Change" button back to each step. The main form builds the
 * content (reviewSummary), with dropdown values already turned into labels,
 * so this component only lays it out.
 *
 * The application number isn't shown: it's assigned by the submit workflow
 * and appears on the receipt.
 */
export default {
  props: {
    application: {
      type: Object,
      required: true,
    },
    applicants: {
      type: Array,
      default: () => [],
    },
    requiresCollateral: {
      type: Boolean,
      default: false,
    },
    /** Sections of { title, step, items: [{ heading, rows: [[label, value]] }] }. */
    summary: {
      type: Array,
      default: () => [],
    },
  },

  emits: ['edit-step'],
};
</script>

<style scoped>
/*
 * No styles here on purpose: Saturn doesn't apply <style> blocks reliably.
 * Everything is styled in the template with Tailwind classes, plus inline
 * styles for the brand colours (CSS variables set on the form) and exact sizes.
 */
</style>
