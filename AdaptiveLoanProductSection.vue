<template>
  <section>
    <!-- Loan category cards (personal, auto, home, business) -->
    <div class="loan-grid flex flex-wrap gap-3">
      <button
        v-for="loan in loans"
        :key="loan.id"
        type="button"
        class="flex flex-col items-start gap-1 p-4 border-2 rounded-lg bg-white text-left"
        :class="loanCategory === loan.id ? 'selected border-blue-600 bg-blue-50' : 'border-gray-200'"
        style="flex: 1 1 240px"
        @click="$emit('select-category', loan.id)"
      >
        <v-icon>{{ loan.icon }}</v-icon>
        <b>{{ loan.title }}</b>
        <small>{{ loan.note }}</small>
      </button>
    </div>

    <!-- Product cards, only shown once a category is chosen -->
    <div v-if="loanCategory" class="context mt-5">
      <p class="product-question mb-3 text-base font-semibold">
        {{ products.length > 1 ? 'Which of these fits you best?' : 'Your loan' }}
      </p>
      <div class="product-list flex flex-col gap-3">
        <button
          v-for="product in products"
          :key="product.id"
          type="button"
          class="product-card flex flex-col items-start gap-1 w-full p-4 border-2 rounded-lg bg-white text-left text-base"
          :class="loanTypeId === product.id ? 'selected border-blue-600 bg-blue-50' : 'border-gray-200'"
          @click="$emit('select-product', product.id)"
        >
          <b class="block">{{ product.name }}</b>
          <small v-if="rangeText(product)" class="block text-sm text-gray-600">{{ rangeText(product) }}</small>
        </button>
      </div>
      <p v-if="!products.length" class="helper block w-full mt-1">
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
.product-question {
  margin: 0 0 12px;
  font-size: 16px;
  font-weight: 600;
}

.product-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.product-card {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
  padding: 16px;
  border: 2px solid rgba(0, 0, 0, 0.12);
  border-radius: 12px;
  background: #fff;
  font-size: 16px;
  text-align: left;
  cursor: pointer;
}

.product-card.selected {
  border-color: var(--brand, #1178bd);
  background: rgba(17, 120, 189, 0.06);
}
</style>