<template>
  <section class="review">
    <p class="review-intro">
      Please check everything below. If something is wrong, tap "Change" to fix it.
    </p>

    <article v-for="section in summary" :key="section.title" class="review-section">
      <header class="review-header">
        <h3>{{ section.title }}</h3>
        <el-button v-if="section.step" text type="primary" @click="$emit('edit-step', section.step)">
          Change
        </el-button>
      </header>

      <div v-for="(item, index) in section.items" :key="index" class="review-item">
        <strong v-if="section.items.length > 1 || item.heading !== section.title">{{ item.heading }}</strong>
        <div v-for="row in item.rows" :key="row[0]" class="review-row">
          <span>{{ row[0] }}</span>
          <span class="review-value">{{ row[1] }}</span>
        </div>
      </div>
    </article>

    <p class="helper">When everything looks right, tap "Send my application" below.</p>
  </section>
</template>

<script>
/**
 * Final review step. Shows everything the applicant entered, section by
 * section, with an "Edit" link back to each step. The main form builds the
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
.review-intro {
  margin-bottom: 16px;
}

.review-section {
  margin-bottom: 20px;
  padding: 12px 16px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

.review-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.review-header h3 {
  margin: 0;
}

.review-item {
  padding: 8px 0;
  border-top: 1px solid rgba(0, 0, 0, 0.06);
}

.review-item:first-of-type {
  border-top: none;
}

.review-row {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  padding: 3px 0;
}

.review-value {
  text-align: right;
  font-weight: 500;
}
</style>
