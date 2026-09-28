<template>
  <section class="review">
    <article
      v-for="section in summary"
      :key="section.title"
      class="check-section"
    >
      <header class="check-header">
        <h3 class="check-title">{{ section.title }}</h3>
        <el-button v-if="section.step" type="primary" plain @click="$emit('edit-step', section.step)">
          Change
        </el-button>
      </header>

      <div v-for="(item, index) in section.items" :key="index" class="check-item">
        <p
          v-if="section.items.length > 1 || item.heading !== section.title"
          class="check-heading"
        >
          {{ item.heading }}
        </p>
        <div
          v-for="row in item.rows"
          :key="row[0]"
          class="check-row"
        >
          <span class="check-label">{{ row[0] }}</span>
          <span class="check-value">{{ row[1] }}</span>
        </div>
      </div>
    </article>

    <p class="check-footer">When everything looks right, tap "Send my application" below.</p>
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
/* Class names avoid the old "review-row"/"review-block", which the main form
   styled as dark blocks. */
.review {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.check-section {
  padding: 16px;
  border: 1px solid #dfe7ec;
  border-radius: 12px;
  background: #fff;
}

.check-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid #dfe7ec;
}

.check-title {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
}

.check-item {
  padding-top: 12px;
}

.check-heading {
  margin: 0 0 4px;
  font-weight: 600;
}

.check-row {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  padding: 4px 0;
  border-bottom: 1px solid #f1f5f9;
}

.check-label {
  color: #64748b;
}

.check-value {
  font-weight: 500;
  text-align: right;
}

.check-footer {
  margin: 0;
  font-size: 16px;
}
</style>
