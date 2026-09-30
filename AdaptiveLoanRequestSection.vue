<template>
  <section>
    <template v-if="show('main')">
      <template v-if="revolving">
        <el-form-item label="What limit would you like? (EC$)" required :error="need(draft.requested_credit_limit)">
          <FormField
            :model-value="draft.requested_credit_limit"
            :property="fields.requested_credit_limit"
            :form="draft"
            @update:model-value="set('requested_credit_limit', $event)"
          />
          <small v-if="selectedProduct" class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">You can ask for {{ money(amountMinimum) }} to {{ money(amountMaximum) }}.</small>
        </el-form-item>
      </template>
      <template v-else>
      <el-form-item label="How much would you like to borrow? (EC$)" required :error="need(draft.requested_loan_amount)">
        <FormField
          :model-value="draft.requested_loan_amount"
          :property="fields.requested_loan_amount"
          :form="draft"
          @update:model-value="set('requested_loan_amount', $event)"
        />
        <small v-if="selectedProduct" class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">You can borrow from {{ money(amountMinimum) }} to {{ money(amountMaximum) }}.</small>
      </el-form-item>
      <el-form-item label="How many months do you need to pay it back?" required :error="need(draft.requested_loan_term)">
        <FormField
          :model-value="draft.requested_loan_term"
          :property="fields.requested_loan_term"
          :form="draft"
          @update:model-value="set('requested_loan_term', $event)"
        />
        <small v-if="selectedProduct" class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">From {{ termMinimum }} to {{ termMaximum }} months (up to {{ yearsLabel(termMaximum) }}).</small>
      </el-form-item>
      <el-form-item label="How often would you like to pay?">
        <template v-if="choices('repayment_frequency')">
          <div class="choice-list inline flex flex-wrap gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('repayment_frequency')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.repayment_frequency === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.repayment_frequency === option.value"
              @click="set('repayment_frequency', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.repayment_frequency === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.repayment_frequency"
            :property="fields.repayment_frequency"
            :form="draft"
            @update:model-value="set('repayment_frequency', $event)"
          />
        </template>
      </el-form-item>
      </template>
      <el-form-item :label="revolving ? 'What will you use it for?' : 'What is the loan for?'" required :error="need(draft.loan_purpose)">
        <FormField
          :model-value="draft.loan_purpose"
          :property="fields.loan_purpose"
          :form="draft"
          @update:model-value="set('loan_purpose', $event)"
        />
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">{{ revolving ? 'A sentence is enough, for example "Everyday spending and emergencies".' : loanCategory === 'student' ? 'A sentence is enough, for example "Tuition and books for my nursing degree".' : 'A sentence is enough, for example "To buy a used car for work".' }}</small>
      </el-form-item>
    </template>

    <template v-if="show('details')">
      <template v-if="loanCategory === 'automotive'">
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Make">
            <FormField
              :model-value="draft.vehicle_make"
              :property="fields.vehicle_make"
              :form="draft"
              @update:model-value="set('vehicle_make', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Model">
            <FormField
              :model-value="draft.vehicle_model"
              :property="fields.vehicle_model"
              :form="draft"
              @update:model-value="set('vehicle_model', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Year">
            <FormField
              :model-value="draft.vehicle_year"
              :property="fields.vehicle_year"
              :form="draft"
              @update:model-value="set('vehicle_year', $event)"
            />
          </el-form-item>
        </div>
        <el-form-item label="Is it new or used?">
          <template v-if="choices('vehicle_condition')">
            <div class="choice-list inline flex flex-wrap gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('vehicle_condition')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.vehicle_condition === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.vehicle_condition === option.value"
                @click="set('vehicle_condition', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.vehicle_condition === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.vehicle_condition"
              :property="fields.vehicle_condition"
              :form="draft"
              @update:model-value="set('vehicle_condition', $event)"
            />
          </template>
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Registration number (if it has one)">
            <FormField
              :model-value="draft.vehicle_registration_number"
              :property="fields.vehicle_registration_number"
              :form="draft"
              @update:model-value="set('vehicle_registration_number', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Chassis number (VIN)">
            <FormField
              :model-value="draft.vehicle_chassis_number"
              :property="fields.vehicle_chassis_number"
              :form="draft"
              @update:model-value="set('vehicle_chassis_number', $event)"
            />
          </el-form-item>
        </div>
      </template>

      <template v-if="loanCategory === 'property'">
        <el-form-item label="Address of the property">
          <FormField
            :model-value="draft.property_address"
            :property="fields.property_address"
            :form="draft"
            @update:model-value="set('property_address', $event)"
          />
        </el-form-item>
        <el-form-item label="What kind of property is it?">
          <template v-if="choices('property_type')">
            <div class="choice-list inline flex flex-wrap gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('property_type')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.property_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.property_type === option.value"
                @click="set('property_type', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.property_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.property_type"
              :property="fields.property_type"
              :form="draft"
              @update:model-value="set('property_type', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="What is it worth? (EC$)">
          <FormField
            :model-value="draft.property_value"
            :property="fields.property_value"
            :form="draft"
            @update:model-value="set('property_value', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">Your best guess is fine.</small>
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Block and parcel">
            <FormField
              :model-value="draft.property_block_and_parcel"
              :property="fields.property_block_and_parcel"
              :form="draft"
              @update:model-value="set('property_block_and_parcel', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Deed number">
            <FormField
              :model-value="draft.property_deed_number"
              :property="fields.property_deed_number"
              :form="draft"
              @update:model-value="set('property_deed_number', $event)"
            />
          </el-form-item>
        </div>
      </template>

      <!--
        Purchase details (auto and home). With a purchase price, the vehicle
        or property being bought is added on the Assets step as collateral.
      -->
      <template v-if="isPurchaseCategory">
        <p class="subheading mt-6 mb-3 text-lg font-bold" style="color:#111827">Are you buying it?</p>
        <p class="hint mt-2 text-base text-gray-600">
          If you're not buying it (for example, you're refinancing), leave the price empty.
        </p>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Price (EC$)">
            <FormField
              :model-value="draft.purchase_price"
              :property="fields.purchase_price"
              :form="draft"
              @update:model-value="set('purchase_price', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Your down payment (EC$)">
            <FormField
              :model-value="draft.down_payment_amount"
              :property="fields.down_payment_amount"
              :form="draft"
              @update:model-value="set('down_payment_amount', $event)"
            />
          </el-form-item>
        </div>
        <el-form-item label="Where is the down payment coming from?" required :error="need(draft.source_of_funds)" v-if="Number(draft.down_payment_amount) > 0">
          <template v-if="choices('source_of_funds')">
            <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('source_of_funds')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.source_of_funds === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.source_of_funds === option.value"
                @click="set('source_of_funds', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.source_of_funds === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.source_of_funds"
              :property="fields.source_of_funds"
              :form="draft"
              @update:model-value="set('source_of_funds', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Tell us more about the down payment" v-if="Number(draft.down_payment_amount) > 0" :required="String(draft.source_of_funds).toLowerCase() === 'other'">
          <FormField
            :model-value="draft.source_of_funds_details"
            :property="fields.source_of_funds_details"
            :form="draft"
            @update:model-value="set('source_of_funds_details', $event)"
          />
        </el-form-item>
        <el-form-item label="Who are you buying it from?">
          <template v-if="choices('seller_type')">
            <div class="choice-list inline flex flex-wrap gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('seller_type')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.seller_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.seller_type === option.value"
                @click="set('seller_type', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.seller_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.seller_type"
              :property="fields.seller_type"
              :form="draft"
              @update:model-value="set('seller_type', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Seller name">
          <FormField
            :model-value="draft.seller_name"
            :property="fields.seller_name"
            :form="draft"
            @update:model-value="set('seller_name', $event)"
          />
        </el-form-item>
        <p v-if="loanToValue !== null" class="soft-box info mb-5 p-4 rounded-lg bg-blue-50 text-blue-800">
          The loan is {{ loanToValue }}% of the price.
        </p>
      </template>

      <!-- The business is saved as its own Party record -->
      <template v-if="loanCategory === 'organization'">
        <el-form-item label="Business name" required :error="need(draft.business_name)">
          <FormField
            :model-value="draft.business_name"
            :property="fields.business_name"
            :form="draft"
            @update:model-value="set('business_name', $event)"
          />
        </el-form-item>
        <el-form-item label="Type of business">
          <template v-if="choices('business_type')">
            <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('business_type')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.business_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.business_type === option.value"
                @click="set('business_type', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.business_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.business_type"
              :property="fields.business_type"
              :form="draft"
              @update:model-value="set('business_type', $event)"
            />
          </template>
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Registration number">
            <FormField
              :model-value="draft.business_registration_number"
              :property="fields.business_registration_number"
              :form="draft"
              @update:model-value="set('business_registration_number', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Date it started">
            <FormField
              :model-value="draft.business_incorporation_date"
              :property="fields.business_incorporation_date"
              :form="draft"
              @update:model-value="set('business_incorporation_date', $event, 'date')"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Number of employees">
            <FormField
              :model-value="draft.business_employee_count"
              :property="fields.business_employee_count"
              :form="draft"
              @update:model-value="set('business_employee_count', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Yearly sales (EC$)">
            <FormField
              :model-value="draft.business_annual_revenue"
              :property="fields.business_annual_revenue"
              :form="draft"
              @update:model-value="set('business_annual_revenue', $event)"
            />
            <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">The business's total sales or income for its last full year.</small>
          </el-form-item>
        </div>
      </template>

      <template v-if="loanCategory === 'credit_card'">
        <el-form-item label="Name to print on the card" required :error="need(draft.name_on_card)">
          <FormField
            :model-value="draft.name_on_card"
            :property="fields.name_on_card"
            :form="draft"
            @update:model-value="set('name_on_card', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">As it should appear on the card, for example &quot;JANE A SMITH&quot;.</small>
        </el-form-item>
        <el-form-item label="How would you like to get your card?">
          <template v-if="choices('card_collection_method')">
            <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('card_collection_method')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.card_collection_method === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.card_collection_method === option.value"
                @click="set('card_collection_method', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.card_collection_method === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.card_collection_method"
              :property="fields.card_collection_method"
              :form="draft"
              @update:model-value="set('card_collection_method', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Would you like to secure it with your savings?">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.is_secured_by_savings === true === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.is_secured_by_savings === true === option.value"
              @click="set('is_secured_by_savings', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.is_secured_by_savings === true === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">Your savings or shares with us are held against it, which can mean a better rate.</small>
        </el-form-item>
        <el-form-item label="How much of your savings? (EC$)" required :error="need(draft.secured_savings_amount)" v-if="draft.is_secured_by_savings">
          <FormField
            :model-value="draft.secured_savings_amount"
            :property="fields.secured_savings_amount"
            :form="draft"
            @update:model-value="set('secured_savings_amount', $event)"
          />
        </el-form-item>
      </template>

      <template v-if="loanCategory === 'overdraft'">
        <el-form-item label="Which account is the overdraft for?" required :error="need(draft.linked_account_number)">
          <FormField
            :model-value="draft.linked_account_number"
            :property="fields.linked_account_number"
            :form="draft"
            @update:model-value="set('linked_account_number', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">The account number, from your passbook or statement.</small>
        </el-form-item>
        <el-form-item label="Would you like to secure it with your savings?">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.is_secured_by_savings === true === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.is_secured_by_savings === true === option.value"
              @click="set('is_secured_by_savings', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.is_secured_by_savings === true === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">Your savings or shares with us are held against it, which can mean a better rate.</small>
        </el-form-item>
        <el-form-item label="How much of your savings? (EC$)" required :error="need(draft.secured_savings_amount)" v-if="draft.is_secured_by_savings">
          <FormField
            :model-value="draft.secured_savings_amount"
            :property="fields.secured_savings_amount"
            :form="draft"
            @update:model-value="set('secured_savings_amount', $event)"
          />
        </el-form-item>
      </template>

      <template v-if="loanCategory === 'student'">
        <el-form-item label="Name of the school, college, or university" required :error="need(draft.institution_name)">
          <FormField
            :model-value="draft.institution_name"
            :property="fields.institution_name"
            :form="draft"
            @update:model-value="set('institution_name', $event)"
          />
        </el-form-item>
        <el-form-item label="Which country is it in?">
          <FormField
            :model-value="draft.institution_country"
            :property="fields.institution_country"
            :form="draft"
            @update:model-value="set('institution_country', $event)"
          />
        </el-form-item>
        <el-form-item label="What will you study?" required :error="need(draft.program_name)">
          <FormField
            :model-value="draft.program_name"
            :property="fields.program_name"
            :form="draft"
            @update:model-value="set('program_name', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">For example &quot;Nursing&quot; or &quot;Business administration&quot;.</small>
        </el-form-item>
        <el-form-item label="What level is the course?">
          <template v-if="choices('program_level')">
            <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('program_level')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.program_level === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.program_level === option.value"
                @click="set('program_level', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.program_level === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.program_level"
              :property="fields.program_level"
              :form="draft"
              @update:model-value="set('program_level', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Student ID number (if you have one)">
          <FormField
            :model-value="draft.student_id_number"
            :property="fields.student_id_number"
            :form="draft"
            @update:model-value="set('student_id_number', $event)"
          />
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="When does the course start?">
            <FormField
              :model-value="draft.enrollment_start_date"
              :property="fields.enrollment_start_date"
              :form="draft"
              @update:model-value="set('enrollment_start_date', $event, 'date')"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="When will you finish?" required :error="need(draft.expected_graduation_date)">
            <FormField
              :model-value="draft.expected_graduation_date"
              :property="fields.expected_graduation_date"
              :form="draft"
              @update:model-value="set('expected_graduation_date', $event, 'date')"
            />
          </el-form-item>
        </div>
        <el-form-item label="Tuition fees" required :error="need(draft.tuition_amount)">
          <FormField
            :model-value="draft.tuition_amount"
            :property="fields.tuition_amount"
            :form="draft"
            @update:model-value="set('tuition_amount', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">The total for the course, from the school's fee letter.</small>
        </el-form-item>
        <el-form-item label="Which currency are the fees in?">
          <template v-if="choices('tuition_currency')">
            <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('tuition_currency')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.tuition_currency === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.tuition_currency === option.value"
                @click="set('tuition_currency', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.tuition_currency === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.tuition_currency"
              :property="fields.tuition_currency"
              :form="draft"
              @update:model-value="set('tuition_currency', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Other costs, like books and living costs">
          <FormField
            :model-value="draft.other_study_costs"
            :property="fields.other_study_costs"
            :form="draft"
            @update:model-value="set('other_study_costs', $event)"
          />
        </el-form-item>
        <el-form-item label="How should the money be paid out?">
          <template v-if="choices('disbursement_schedule')">
            <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('disbursement_schedule')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.disbursement_schedule === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.disbursement_schedule === option.value"
                @click="set('disbursement_schedule', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.disbursement_schedule === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.disbursement_schedule"
              :property="fields.disbursement_schedule"
              :form="draft"
              @update:model-value="set('disbursement_schedule', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Should we pay the school directly?">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.pay_to_institution === true === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.pay_to_institution === true === option.value"
              @click="set('pay_to_institution', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.pay_to_institution === true === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </el-form-item>
        <el-form-item label="Months after you finish before payments start">
          <FormField
            :model-value="draft.grace_period_months"
            :property="fields.grace_period_months"
            :form="draft"
            @update:model-value="set('grace_period_months', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">Leave empty if you're not sure.</small>
        </el-form-item>
      </template>

    </template>
  </section>
</template>

<script>
/**
 * Every field on this step: [form key, label, input kind, resource, Saturn
 * property]. Most are Application properties. The vehicle and property
 * identifiers are saved on the Asset being bought, and the business details
 * on the business's own Party record, so their FormField uses those
 * resources' definitions.
 */
const FIELDS = [
    ["requested_loan_amount", "Requested amount (EC$)", "number", "Application", "requested_loan_amount"],
    ["requested_loan_term", "Term (months)", "number", "Application", "requested_loan_term"],
    ["repayment_frequency", "Repayment frequency", "select", "Application", "repayment_frequency"],
    ["loan_purpose", "Purpose of the loan", "textarea", "Application", "loan_purpose"],
    ["vehicle_make", "Make", "input", "Application", "vehicle_make"],
    ["vehicle_model", "Model", "input", "Application", "vehicle_model"],
    ["vehicle_year", "Year", "input", "Application", "vehicle_year"],
    ["vehicle_condition", "Condition", "select", "Application", "vehicle_condition"],
    ["vehicle_registration_number", "Registration number", "input", "Asset", "registration_number"],
    ["vehicle_chassis_number", "Chassis number (VIN)", "input", "Asset", "chassis_number"],
    ["property_address", "Property address", "input", "Application", "property_address"],
    ["property_type", "Property type", "select", "Application", "property_type"],
    ["property_value", "Estimated value (EC$)", "number", "Application", "property_value"],
    ["property_block_and_parcel", "Block and parcel", "input", "Asset", "block_and_parcel"],
    ["property_deed_number", "Deed number", "input", "Asset", "deed_number"],
    ["purchase_price", "Purchase price (EC$)", "number", "Application", "purchase_price"],
    ["down_payment_amount", "Down payment (EC$)", "number", "Application", "down_payment_amount"],
    ["source_of_funds", "Where is the down payment coming from?", "select", "Application", "source_of_funds"],
    ["source_of_funds_details", "Down payment details", "textarea", "Application", "source_of_funds_details"],
    ["seller_type", "Seller type", "select", "Application", "seller_type"],
    ["seller_name", "Seller name", "input", "Application", "seller_name"],
    ["business_name", "Business name", "input", "Party", "business_name"],
    ["business_registration_number", "Registration number", "input", "Party", "registration_number"],
    ["business_type", "Business type", "select", "Party", "business_type"],
    ["business_incorporation_date", "Incorporation date", "date", "Party", "incorporation_date"],
    ["business_employee_count", "Number of employees", "number", "ApplicationParty", "number_of_employees"],
    ["business_annual_revenue", "Yearly sales (EC$)", "number", "ApplicationParty", "annual_revenue"],
    ["requested_credit_limit", "Limit (EC$)", "number", "Application", "requested_credit_limit"],
    ["is_secured_by_savings", "Secured by savings", "checkbox", "Application", "is_secured_by_savings"],
    ["secured_savings_amount", "Savings held against it (EC$)", "number", "Application", "secured_savings_amount"],
    ["linked_account_number", "Account number", "input", "Application", "linked_account_number"],
    ["name_on_card", "Name on the card", "input", "Application", "name_on_card"],
    ["card_collection_method", "How to get the card", "select", "Application", "card_collection_method"],
    ["institution_name", "School", "input", "Application", "institution_name"],
    ["institution_country", "Country of the school", "select", "Application", "institution_country"],
    ["program_name", "Course", "input", "Application", "program_name"],
    ["program_level", "Level", "select", "Application", "program_level"],
    ["student_id_number", "Student ID", "input", "Application", "student_id_number"],
    ["enrollment_start_date", "Course start", "date", "Application", "enrollment_start_date"],
    ["expected_graduation_date", "Expected finish", "date", "Application", "expected_graduation_date"],
    ["tuition_amount", "Tuition fees", "number", "Application", "tuition_amount"],
    ["tuition_currency", "Currency", "select", "Application", "tuition_currency"],
    ["other_study_costs", "Other study costs", "number", "Application", "other_study_costs"],
    ["disbursement_schedule", "Payout schedule", "select", "Application", "disbursement_schedule"],
    ["pay_to_institution", "Pay the school directly", "checkbox", "Application", "pay_to_institution"],
    ["grace_period_months", "Grace period (months)", "number", "Application", "grace_period_months"],
];

/**
 * "Your request" step: amount, term, and purpose (or, for credit cards and
 * overdrafts, the limit), then the details for the kind of loan: vehicle,
 * property, business, card, overdraft, or studies, plus the purchase
 * details for auto and property loans.
 * Every input is Saturn's built-in FormField, using Saturn's own definition
 * of each property when it exists (see FIELDS for which resource).
 *
 * The amount and term limits aren't enforced by the inputs; the main form
 * checks them when the applicant presses Continue.
 */
export default {
    props: {
        modelValue: { type: Object, default: () => ({}) },
        /**
         * The kind of loan: property, automotive, personal, organization,
         * credit_card, overdraft, or student (the main form maps the
         * LoanCategory code to this).
         */
        loanCategory: { type: String, default: "" },
        /** Credit cards and overdrafts: ask for a limit, not an amount and term. */
        revolving: { type: Boolean, default: false },
        selectedProduct: { type: Object, default: null },
        lookups: { type: Object, default: () => ({}) },
        /** Saturn's property definitions for the Application resource. */
        applicationProps: { type: Array, default: () => [] },
        /** Saturn's property definitions, keyed by resource name. */
        resourceProps: { type: Object, default: () => ({}) },
        /** Which short screen to show: "main" (amount) or "details"; "" shows both. */
        screen: { type: String, default: "" },
        /** After a failed Next, shows "this is needed" under empty required boxes. */
        showErrors: { type: Boolean, default: false },
        amountMinimum: { type: Number, default: 0 },
        amountMaximum: { type: Number, default: 999999999 },
        termMinimum: { type: Number, default: 1 },
        termMaximum: { type: Number, default: 600 },
    },
    emits: ["update:modelValue"],
    data() {
        return { draft: this.copy(this.modelValue) };
    },
    computed: {
        /**
         * FormField property configs keyed by form key. Built once (and
         * again only if the props change), so FormField isn't handed a new
         * object on every keystroke.
         */
        fields() {
            const result = {};
            FIELDS.forEach((row) => {
                result[row[0]] = this.field(row[3], row[4], row[1], row[2]);
            });
            return result;
        },

        isPurchaseCategory() {
            return this.loanCategory === "automotive" || this.loanCategory === "property";
        },

        /** The requested amount as a % of the purchase price (loan-to-value). */
        loanToValue() {
            const price = Number(this.draft.purchase_price);
            const amount = Number(this.draft.requested_loan_amount);
            if (!(price > 0) || !(amount > 0)) return null;
            return Math.round((amount / price) * 1000) / 10;
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
        /**
         * The answers for a question as tappable cards, when Saturn's list is
         * short (2 to 7 answers). Otherwise null, and Saturn's dropdown is used.
         */
        choices(key) {
            const options = this.lookups[key] || [];
            return options.length >= 2 && options.length <= 7 ? options : null;
        },

        copy(value) {
            return JSON.parse(JSON.stringify(value || {}));
        },

        /**
         * FormField's update event may send the value itself or
         * { property, data } (the Saturn guide isn't clear), so accept both.
         * Dates are kept as YYYY-MM-DD strings.
         */
        valueOf(event, kind) {
            let value = event;
            if (
                value &&
                typeof value === "object" &&
                "property" in value &&
                "data" in value
            ) {
                value = value.data;
            }
            if (kind !== "date") return value;
            if (!value) return "";
            if (value instanceof Date) {
                const pad = (number) => String(number).padStart(2, "0");
                return `${value.getFullYear()}-${pad(value.getMonth() + 1)}-${pad(value.getDate())}`;
            }
            return String(value).slice(0, 10);
        },

        set(key, event, kind) {
            this.draft[key] = this.valueOf(event, kind);
            this.$emit("update:modelValue", this.copy(this.draft));
        },

        /** Saturn's definition of one property of a resource, or null. */
        savedProperty(resourceName, name) {
            const rows =
                resourceName === "Application" && this.applicationProps.length
                    ? this.applicationProps
                    : this.resourceProps[resourceName] || [];
            return (
                rows.find(
                    (row) =>
                        String(row.property || row.key || row.name || "") === name,
                ) || null
            );
        },

        /**
         * The FormField property config for one field. Uses Saturn's own
         * definition (with our label) when there is one; otherwise builds
         * one of the kinds shown in the Saturn guide. FormField's own option
         * lists (lookup_type "values") don't work in Saturn, so a dropdown
         * with no Saturn definition is a text box.
         */
        field(resourceName, name, label, kind) {
            const saved = this.savedProperty(resourceName, name);
            if (saved) {
                return Object.assign({}, saved, { property: name, label });
            }
            if (kind === "number" || kind === "date") {
                return { property: name, label, type: kind };
            }
            return {
                property: name,
                label,
                type: "string",
                input_properties: {
                    type: kind === "textarea" ? "textarea" : "input",
                },
            };
        },

        /** True when this part is on screen. */
        show(part) {
            return !this.screen || this.screen === part;
        },

        /** "This is needed" under an empty required box, after a failed Next. */
        need(value) {
            if (!this.showErrors) return "";
            return value === null || value === undefined || value === ""
                ? "This is needed"
                : "";
        },

        /** Months as years, e.g. 60 -> "5 years". */
        /** "6 months", "1 year", or "2.5 years". */
        yearsLabel(months) {
            const count = Number(months);
            if (count < 12) return `${count} ${count === 1 ? "month" : "months"}`;
            const years = Math.round((count / 12) * 10) / 10;
            return `${years} ${years === 1 ? "year" : "years"}`;
        },

        money(value) {
            return `EC$ ${Number(value).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
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
