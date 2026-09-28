<template>
  <section class="review">
    <article v-for="section in summary" :key="section.title" class="check-section">
      <header class="check-header">
        <h3 class="check-title">{{ section.title }}</h3>
        <button
          v-if="section.step"
          type="button"
          class="link-btn"
          @click="$emit('edit-step', section.step)"
        >
          Change
        </button>
      </header>

      <div v-for="(item, index) in section.items" :key="index" class="check-item">
        <p v-if="section.items.length > 1 || item.heading !== section.title" class="check-heading">
          {{ item.heading }}
        </p>
        <div v-for="row in item.rows" :key="row[0]" class="check-row">
          <span class="check-label">{{ row[0] }}</span>
          <span class="check-value">{{ row[1] || '—' }}</span>
        </div>
      </div>
    </article>

    <p class="hint">When everything looks right, tap "Send my application" below.</p>
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
/* Styled in the main form's stylesheet (.check-section, .check-row, ...). */
</style>
