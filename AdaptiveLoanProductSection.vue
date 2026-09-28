<template>
  <section>
    <!-- Loan types (personal, auto, home, business) as big cards -->
    <div class="loan-grid" role="radiogroup" aria-label="Type of loan">
      <button
        v-for="loan in loans"
        :key="loan.id"
        type="button"
        role="radio"
        class="loan-card"
        :class="{ selected: loanCategory === loan.id }"
        :aria-checked="loanCategory === loan.id"
        @click="$emit('select-category', loan.id)"
      >
        <span class="loan-icon"><v-icon size="28">{{ loan.icon }}</v-icon></span>
        <span>
          <b>{{ loan.title }}</b>
          <small>{{ loan.note }}</small>
        </span>
      </button>
    </div>

    <!-- The loans of that type, once a type is chosen -->
    <div v-if="loanCategory" class="product-block">
      <p class="question">
        {{ products.length > 1 ? 'Which one fits you best?' : 'Your loan' }}
      </p>
      <div class="choice-list" role="radiogroup">
        <button
          v-for="product in products"
          :key="product.id"
          type="button"
          role="radio"
          class="choice product-card"
          :class="{ selected: loanTypeId === product.id }"
          :aria-checked="loanTypeId === product.id"
          @click="$emit('select-product', product.id)"
        >
          <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
          <span>
            <b>{{ product.name }}</b>
            <small v-if="rangeText(product)">{{ rangeText(product) }}</small>
          </span>
        </button>
      </div>
      <p v-if="!products.length" class="hint">
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
/* Styled in the main form's stylesheet (.loan-card, .choice, .product-block). */
</style>
