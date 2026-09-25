<template>
  <section class="review-block">
    <div v-for="row in rows" :key="row.label" class="review-row">
      <span>{{ row.label }}</span>
      <strong>{{ row.value }}</strong>
    </div>
  </section>
</template>

<script>
/**
 * Final review step. Shows a read-only summary of the application before
 * submission. The application number isn't shown: it's assigned by the
 * submit workflow and appears on the receipt.
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
  },

  computed: {
    rows() {
      const form = this.application;
      const money = (value) =>
        value === null || value === undefined || value === ''
          ? '—'
          : `EC$ ${Number(value).toLocaleString(undefined, {
              minimumFractionDigits: 2,
              maximumFractionDigits: 2,
            })}`;

      const first = this.applicants[0] || {};
      const primary =
        `${first.first_name || ''} ${first.last_name || ''}`.trim() ||
        'Primary Applicant';

      return [
        { label: 'Product', value: form.loan_name || 'Not selected' },
        { label: 'Requested amount', value: money(form.requested_loan_amount) },
        {
          label: 'Term',
          value: form.requested_loan_term
            ? `${form.requested_loan_term} months`
            : '—',
        },
        { label: 'Primary applicant', value: primary },
        { label: 'Applicants', value: String(this.applicants.length) },
        {
          label: 'Assets / Liabilities / Expenses',
          value: `${(form.assets || []).length} / ${(form.liabilities || []).length} / ${(form.expenses || []).length}`,
        },
        {
          label: 'Collateral',
          // Collateral is marked on each asset.
          value: this.requiresCollateral
            ? `${(form.assets || []).filter((asset) => asset.collateral && asset.collateral.enabled).length} asset(s)`
            : 'Not required',
        },
      ];
    },
  },
};
</script>