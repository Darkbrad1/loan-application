<template>
  <section>
    <!-- Loan types (personal, auto, home, business) as big cards -->
    <div class="loan-grid flex flex-wrap gap-3" role="radiogroup" aria-label="Type of loan">
      <button
        v-for="loan in loans"
        :key="loan.id"
        type="button"
        role="radio"
        class="loan-card flex items-center gap-4 px-5 py-4 rounded-lg text-left"
        :style="['flex:1 1 240px;min-height:88px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827', (loanCategory === loan.id) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
        :aria-checked="loanCategory === loan.id"
        @click="$emit('select-category', loan.id)"
      >
        <span class="loan-icon flex items-center justify-center rounded-lg" :style="['width:52px;min-width:52px;max-width:52px;height:52px;min-height:52px;flex:0 0 52px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box', (loanCategory === loan.id) ? 'background:var(--brand);color:var(--brand-ink)' : 'background:var(--brand-tint);color:var(--brand-text)']"><v-icon size="28">{{ loan.icon }}</v-icon></span>
        <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">
          <b class="card-title block text-lg font-bold">{{ loan.title }}</b>
          <small class="card-note block text-sm text-gray-600">{{ loan.note }}</small>
        </span>
      </button>
    </div>

    <!-- The loans of that type, once a type is chosen -->
    <div v-if="loanCategory" class="product-block mt-6">
      <p class="question mb-3 text-base font-semibold" style="color:#111827">
        {{ products.length > 1 ? 'Which one fits you best?' : 'Your loan' }}
      </p>
      <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
        <button
          v-for="product in products"
          :key="product.id"
          type="button"
          role="radio"
          class="choice product-card flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
          :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (loanTypeId === product.id) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
          :aria-checked="loanTypeId === product.id"
          @click="$emit('select-product', product.id)"
        >
          <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (loanTypeId === product.id) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
          <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">
            <b class="card-title block text-lg font-bold">{{ product.name }}</b>
            <small v-if="rangeText(product)" class="card-note block text-sm text-gray-600">{{ rangeText(product) }}</small>
          </span>
        </button>
      </div>
      <p v-if="!products.length" class="hint mt-2 text-base text-gray-600">
        There are no loans of this type right now. Please choose another type.
      </p>
    </div>
  </section>
</template>

<script>
/**
 * Step 1 of the loan wizard: pick a loan category, then a specific
 * product within that category, both as big cards to tap. Fully
 * controlled by the parent: all state arrives as props and changes are
 * emitted upward. (The parent picks the product itself when a category
 * has only one.)
 */
export default {
  props: {
    /** Available loan categories with icon, title, and description. */
    loans: {
      type: Array,
      default: () => [],
    },
    /** Products already filtered to the selected category by the parent. */
    products: {
      type: Array,
      default: () => [],
    },
    /** Currently selected category ID, e.g. 'auto'. */
    loanCategory: {
      type: String,
      default: '',
    },
    /** Currently selected product (loan type) ID. */
    loanTypeId: {
      type: String,
      default: '',
    },
  },

  emits: ['select-category', 'select-product'],

  methods: {
    /** "EC$ 1,000 to EC$ 50,000, up to 60 months", from the product's limits. */
    rangeText(product) {
      const money = (value) => `EC$ ${Number(value).toLocaleString()}`;
      const parts = [];
      if (Number(product.maximum_amount) > 0) {
        parts.push(
          Number(product.minimum_amount) > 0
            ? `${money(product.minimum_amount)} to ${money(product.maximum_amount)}`
            : `Up to ${money(product.maximum_amount)}`
        );
      }
      if (Number(product.maximum_term_months) > 0) {
        parts.push(`up to ${product.maximum_term_months} months`);
      }
      return parts.join(', ');
    },
  },
};
</script>

<style scoped>
/*
 * No styles here on purpose: Saturn doesn't apply <style> blocks reliably.
 * Everything is styled in the template with Tailwind classes, plus inline
 * styles for the brand colours (CSS variables set on the form) and exact sizes.
 */
</style>
