<template>
  <section class="review flex flex-col gap-4">
    <article
      v-for="section in summary"
      :key="section.title"
      class="check-section p-4 border rounded-lg bg-white"
    >
      <header class="flex items-center justify-between gap-3 pb-2 border-b">
        <h3 class="text-lg font-semibold m-0">{{ section.title }}</h3>
        <el-button v-if="section.step" type="primary" plain @click="$emit('edit-step', section.step)">
          Change
        </el-button>
      </header>

      <div v-for="(item, index) in section.items" :key="index" class="check-item pt-3">
        <p
          v-if="section.items.length > 1 || item.heading !== section.title"
          class="font-semibold mb-1"
        >
          {{ item.heading }}
        </p>
        <div
          v-for="row in item.rows"
          :key="row[0]"
          class="check-row flex justify-between gap-4 py-1 border-b"
        >
          <span class="text-gray-600">{{ row[0] }}</span>
          <span class="font-medium text-right">{{ row[1] }}</span>
        </div>
      </div>
    </article>

    <p class="text-base">When everything looks right, tap "Send my application" below.</p>
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
/* Layout uses Tailwind classes in the template (Saturn doesn't always apply
   component styles). The old "review-row" class names are avoided because the
   main form still styles those as dark blocks. */
</style>
