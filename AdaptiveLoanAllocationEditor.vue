<template>
  <div class="allocation mt-2 mb-5">
    <p class="question mb-3 text-base font-semibold" style="color:#111827">{{ title }}</p>

    <!-- Simple mode: most people choose "Just me" and never see percentages -->
    <div v-if="simple && options.length > 1" class="choice-list inline" role="radiogroup">
      <button
        v-for="option in [{ value: 'me', label: 'Just me' }, { value: 'shared', label: sharedLabel }]"
        :key="option.value"
        type="button"
        role="radio"
        class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
        :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (mode === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
        :aria-checked="mode === option.value"
        @click="option.value === 'me' ? justMe() : shared()"
      >
        <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (mode === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
        <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
      </button>
    </div>
    <p v-else-if="simple" class="hint mt-2 text-base text-gray-600">{{ meLabel }}</p>

    <template v-if="!simple || mode === 'shared'">
      <p v-if="simple" class="hint mt-2 text-base text-gray-600">
        Choose each person and their share. The shares must add up to 100%.
      </p>
      <div
        v-for="(row, index) in draft"
        :key="row.client_key || index"
        class="allocation-row flex flex-wrap items-end gap-3"
      >
        <el-form-item :label="selectLabel" required>
          <el-select
            :model-value="row[referenceKey]"
            placeholder="Choose a person"
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

        <el-form-item label="Share (%)" required>
          <FormField
            :model-value="row.percentage"
            :property="field('percentage', 'Share (%)')"
            :form="row"
            @update:model-value="setValue(index, 'percentage', $event)"
          />
        </el-form-item>

        <el-form-item v-if="draft.length > 1" label=" ">
          <button type="button" class="link-btn danger px-3 py-2 rounded text-base font-semibold text-red-700" style="background:transparent;border:0;cursor:pointer" @click="removeRow(index)">Remove</button>
        </el-form-item>
      </div>

      <button type="button" class="more-link inline-flex items-center gap-1 mb-5 py-1 text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer;text-align:left" @click="addRow">
        <v-icon size="20">mdi-plus</v-icon>
        Add another person
      </button>

      <p class="total mt-2 text-base font-bold" :class="total === 100 ? 'text-green-700' : 'text-red-700'">
        {{ totalLabel }}: {{ total }}%
      </p>
    </template>
  </div>
</template>

<script>
/**
 * Percentage split editor, used for asset ownership (by Party) and
 * liability responsibility (by ApplicationParty). The rows must total
 * 100%; the main form checks that before the applicant can continue.
 *
 * In `simple` mode it first asks "Just me" or "Shared with someone else".
 * "Just me" is one row for `defaultValue` (the applicant) at 100%, and the
 * percentages are only shown for "Shared". With nobody else to choose from,
 * it just says who it belongs to.
 *
 * The percentage is Saturn's built-in FormField. The person dropdown is an
 * el-select of the options passed in, since they're the people on this
 * application (FormField's own option lists don't work in Saturn).
 * FormField has no min/max, so the percentage isn't limited to 0 to 100 as
 * it's typed; the total shown below turns red until it's 100.
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
    /** Ask "Just me" or "Shared" first (see above). */
    simple: { type: Boolean, default: false },
    /** The applicant's option value, used for "Just me". */
    defaultValue: { type: String, default: '' },
    sharedLabel: { type: String, default: 'Shared with someone else' },
    meLabel: { type: String, default: 'This belongs to you.' },
  },

  emits: ['update:modelValue'],

  data() {
    const draft = this.copy(this.modelValue);
    return {
      draft,
      // "me" or "shared" (simple mode only)
      mode: this.modeOf(draft),
    };
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
  },

  mounted() {
    // A new, unassigned row in simple mode belongs to the applicant.
    if (this.simple && this.mode === 'me' && this.defaultValue) {
      const row = this.draft[0];
      if (!row || row[this.referenceKey] !== this.defaultValue || Number(row.percentage) !== 100) {
        this.justMe();
      }
    }
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
        // Keep "Shared" open while it's being filled in.
        if (this.mode !== 'shared') this.mode = this.modeOf(this.draft);
      },
    },
  },

  methods: {
    copy(value) {
      return JSON.parse(JSON.stringify(value || []));
    },

    /** "me" when it's one row for the applicant (or not chosen yet), else "shared". */
    modeOf(rows) {
      if (rows.length > 1) return 'shared';
      const row = rows[0];
      if (!row) return 'me';
      const value = row[this.referenceKey];
      return !value || value === this.defaultValue ? 'me' : 'shared';
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

    newRow(value, percentage) {
      const row = {
        client_key: this.key('allocation'),
        id: null,
        percentage,
      };
      row[this.referenceKey] = value;
      if (this.responsibilityType) row.responsibility_type = this.responsibilityType;
      return row;
    },

    /** One row for the applicant at 100%, keeping the first row's saved ID. */
    justMe() {
      this.mode = 'me';
      const row = this.draft[0] || this.newRow('', 100);
      row[this.referenceKey] = this.defaultValue;
      row.percentage = 100;
      this.draft = [row];
      this.notify();
    },

    /** Opens the percentages, starting with the applicant and one more person. */
    shared() {
      this.mode = 'shared';
      if (this.draft.length < 2) {
        if (!this.draft.length) this.draft.push(this.newRow(this.defaultValue, 50));
        else this.draft[0].percentage = 50;
        this.draft.push(this.newRow('', 50));
        this.notify();
      }
    },

    setValue(index, key, event) {
      this.draft[index][key] = this.valueOf(event);
      this.notify();
    },

    /** Adds a row for whatever percentage is still unallocated. */
    addRow() {
      this.draft.push(this.newRow('', Math.max(0, 100 - this.total)));
      this.notify();
    },

    removeRow(index) {
      this.draft.splice(index, 1);
      this.notify();
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
