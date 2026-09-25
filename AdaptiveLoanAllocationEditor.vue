<template>
  <div class="allocation">
    <div class="allocation-heading">
      <strong>{{ title }}</strong>
      <el-button size="small" plain @click="addRow">Add allocation</el-button>
    </div>

    <div
      v-for="(row, index) in draft"
      :key="row.client_key || index"
      class="allocation-row"
    >
      <el-form-item :label="selectLabel" required>
        <el-select
          :model-value="row[referenceKey]"
          placeholder="Select"
          @update:model-value="setValue(index, referenceKey, $event)"
        >
          <el-option
            v-for="option in options"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          />
        </el-select>
      </el-form-item>

      <el-form-item label="Percentage" required>
        <FormField
          :model-value="row.percentage"
          :property="field('percentage', 'Percentage')"
          :form="row"
          @update:model-value="setValue(index, 'percentage', $event)"
        />
      </el-form-item>

      <el-button
        v-if="draft.length > 1"
        text
        type="danger"
        @click="removeRow(index)"
      >
        <v-icon>mdi-close</v-icon>
      </el-button>
    </div>

    <p :class="total === 100 ? 'total valid' : 'total invalid'">
      {{ totalLabel }}: {{ total }}%
    </p>
  </div>
</template>

<script>
/**
 * Percentage split editor, used for asset ownership (by Party) and
 * liability responsibility (by ApplicationParty). The rows must total
 * 100%; the main form checks that before the applicant can continue.
 *
 * The percentage is Saturn's built-in FormField. The owner dropdown is an
 * el-select of the options passed in, since they're the people on this
 * application (FormField's own option lists don't work in Saturn). FormField has no min/max, so the percentage isn't limited
 * to 0 to 100 as it's typed; the total shown below turns red until it's 100.
 */
export default {
  props: {
    modelValue: { type: Array, default: () => [] },
    /** Dropdown options for the owner or responsible applicant. */
    options: { type: Array, default: () => [] },
    /** Row field that holds the chosen option, e.g. party_id. */
    referenceKey: { type: String, required: true },
    title: { type: String, default: 'Allocation' },
    selectLabel: { type: String, default: 'Party' },
    totalLabel: { type: String, default: 'Total' },
    /** Set on new rows when given (e.g. "Borrower" for liabilities). */
    responsibilityType: { type: String, default: '' },
  },

  emits: ['update:modelValue'],

  data() {
    return { draft: this.copy(this.modelValue) };
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
  },

  computed: {
    total() {
      return this.draft.reduce((sum, row) => sum + (Number(row.percentage) || 0), 0);
    },
  },

  watch: {
    modelValue: {
      deep: true,
      handler(value) {
        this.draft = this.copy(value);
      },
    },
  },

  methods: {
    copy(value) {
      return JSON.parse(JSON.stringify(value || []));
    },

    key(prefix) {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    },

    /**
     * FormField's update event may send the value itself or
     * { property, data } (the Saturn guide isn't clear), so accept both.
     */
    valueOf(event) {
      if (event && typeof event === 'object' && 'property' in event && 'data' in event) {
        return event.data;
      }
      return event;
    },

    /**
     * FormField property config for the percentage (a number). Reused while
     * unchanged, so FormField isn't handed a new object on every keystroke.
     */
    field(name, label) {
      const config = { property: name, label, type: 'number' };
      const cacheKey = JSON.stringify(config);
      if (!this.fieldCache[cacheKey]) this.fieldCache[cacheKey] = config;
      return this.fieldCache[cacheKey];
    },

    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    setValue(index, key, event) {
      this.draft[index][key] = this.valueOf(event);
      this.notify();
    },

    /** Adds a row for whatever percentage is still unallocated. */
    addRow() {
      const row = {
        client_key: this.key('allocation'),
        id: null,
        percentage: Math.max(0, 100 - this.total),
      };
      row[this.referenceKey] = '';
      if (this.responsibilityType) row.responsibility_type = this.responsibilityType;
      this.draft.push(row);
      this.notify();
    },

    removeRow(index) {
      this.draft.splice(index, 1);
      this.notify();
    },
  },
};
</script>
