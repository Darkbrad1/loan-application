<template>
  <section>
    <!-- Loan category cards (personal, auto, home, business) -->
    <div class="loan-grid">
      <button
        v-for="loan in loans"
        :key="loan.id"
        type="button"
        :class="{ selected: loanCategory === loan.id }"
        @click="$emit('select-category', loan.id)"
      >
        <v-icon>{{ loan.icon }}</v-icon>
        <b>{{ loan.title }}</b>
        <small>{{ loan.note }}</small>
      </button>
    </div>

    <!-- Product picker, only shown once a category is chosen -->
    <div v-if="loanCategory" class="context">
      <el-form-item label="Loan product" required>
        <el-select
          :model-value="loanTypeId"
          placeholder="Select a product"
          @update:model-value="$emit('select-product', $event)"
        >
          <el-option
            v-for="product in products"
            :key="product.id"
            :label="product.name"
            :value="product.id"
          />
        </el-select>
      </el-form-item>
    </div>
  </section>
</template>

<script scoped>
/**
 * Step 1 of the loan wizard: pick a loan category, then a specific
 * product within that category. Fully controlled by the parent —
 * all state arrives as props and changes are emitted upward.
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
};
</script>