<template>
  <div class="applicant-editor">
    <!-- ===== Someone who co-owns what secures the loan, but isn't borrowing ===== -->
    <template v-if="show('owner')">
      <el-form-item label="Is this a person or a business?">
        <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
          <button
            v-for="option in ownerKindOptions"
            :key="String(option.value)"
            type="button"
            role="radio"
            class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
            :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', ((draft.kind || 'PERSON') === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
            :aria-checked="(draft.kind || 'PERSON') === option.value"
            @click="set('kind', option.value)"
          >
            <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', ((draft.kind || 'PERSON') === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
            <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
          </button>
        </div>
      </el-form-item>
      <el-form-item v-if="draft.kind === 'ORGANIZATION'" label="Business name" required :error="need(draft.business_name)">
        <FormField
          :model-value="draft.business_name"
          :property="field('Party', 'legal_name', 'Business name', 'input')"
          :form="draft"
          @update:model-value="set('business_name', $event)"
        />
      </el-form-item>
      <div v-else class="field-grid flex flex-wrap" style="column-gap:16px">
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="First name" required :error="need(draft.first_name)">
          <FormField
            :model-value="draft.first_name"
            :property="field('Party', 'first_name', 'First name', 'input')"
            :form="draft"
            @update:model-value="set('first_name', $event)"
          />
        </el-form-item>
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Last name" required :error="need(draft.last_name)">
          <FormField
            :model-value="draft.last_name"
            :property="field('Party', 'last_name', 'Last name', 'input')"
            :form="draft"
            @update:model-value="set('last_name', $event)"
          />
        </el-form-item>
      </div>
      <el-form-item label="How are they related to you?" required :error="need(draft.relationship_to_applicant)">
        <template v-if="choices('relationship_to_applicant')">
          <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('relationship_to_applicant')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.relationship_to_applicant === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.relationship_to_applicant === option.value"
              @click="set('relationship_to_applicant', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.relationship_to_applicant === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.relationship_to_applicant"
            :property="field('ApplicationParty', 'relationship_to_applicant', 'How are they related to you?', 'select')"
            :form="draft"
            @update:model-value="set('relationship_to_applicant', $event)"
          />
        </template>
      </el-form-item>
      <div class="field-grid flex flex-wrap" style="column-gap:16px">
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Phone number" required :error="need(draft.phone)">
          <FormField
            :model-value="draft.phone"
            :property="field('Party', 'phone', 'Phone number', 'input')"
            :form="draft"
            @update:model-value="set('phone', $event)"
          />
        </el-form-item>
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Email (if they have one)">
          <FormField
            :model-value="draft.email"
            :property="field('Party', 'email', 'Email (if they have one)', 'input')"
            :form="draft"
            @update:model-value="set('email', $event)"
          />
        </el-form-item>
      </div>
      <el-form-item label="How should we contact them?" required :error="need(draft.preferred_contact_method)">
        <template v-if="choices('preferred_contact_method')">
          <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('preferred_contact_method')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.preferred_contact_method === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.preferred_contact_method === option.value"
              @click="set('preferred_contact_method', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.preferred_contact_method === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.preferred_contact_method"
            :property="field('ApplicationParty', 'preferred_contact_method', 'How should we contact them?', 'select')"
            :form="draft"
            @update:model-value="set('preferred_contact_method', $event)"
          />
        </template>
      </el-form-item>
    </template>

    <!-- ===== About: name and contact ===== -->
    <template v-if="show('about')">
      <div class="field-grid flex flex-wrap" style="column-gap:16px">
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="First name" required :error="need(draft.first_name)">
          <FormField
            :model-value="draft.first_name"
            :property="field('Party', 'first_name', 'First name', 'input')"
            :form="draft"
            @update:model-value="set('first_name', $event)"
          />
        </el-form-item>
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Last name" required :error="need(draft.last_name)">
          <FormField
            :model-value="draft.last_name"
            :property="field('Party', 'last_name', 'Last name', 'input')"
            :form="draft"
            @update:model-value="set('last_name', $event)"
          />
        </el-form-item>
      </div>
      <el-form-item label="Mobile phone number" required :error="need(draft.phone)">
        <FormField
          :model-value="draft.phone"
          :property="field('Party', 'phone', 'Mobile phone number', 'input')"
          :form="draft"
          @update:model-value="set('phone', $event)"
        />
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">We'll call or text you about your application.</small>
      </el-form-item>
      <el-form-item label="Email address" required :error="need(draft.email)">
        <FormField
          :model-value="draft.email"
          :property="field('Party', 'email', 'Email address', 'input')"
          :form="draft"
          @update:model-value="set('email', $event)"
        />
      </el-form-item>
      <el-form-item label="How should we contact you?" required :error="need(draft.preferred_contact_method)">
        <template v-if="choices('preferred_contact_method')">
          <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('preferred_contact_method')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.preferred_contact_method === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.preferred_contact_method === option.value"
              @click="set('preferred_contact_method', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.preferred_contact_method === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.preferred_contact_method"
            :property="field('ApplicationParty', 'preferred_contact_method', 'How should we contact you?', 'select')"
            :form="draft"
            @update:model-value="set('preferred_contact_method', $event)"
          />
        </template>
      </el-form-item>
      <el-form-item label="Date of birth" required :error="need(draft.date_of_birth)">
        <FormField
          :model-value="draft.date_of_birth"
          :property="field('Party', 'date_of_birth', 'Date of birth', 'date')"
          :form="draft"
          @update:model-value="set('date_of_birth', $event, 'date')"
        />
      </el-form-item>
      <el-form-item label="Are you married?">
        <template v-if="choices('marital_status')">
          <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('marital_status')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.marital_status === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.marital_status === option.value"
              @click="set('marital_status', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.marital_status === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.marital_status"
            :property="field('Party', 'marital_status', 'Are you married?', 'select')"
            :form="draft"
            @update:model-value="set('marital_status', $event)"
          />
        </template>
      </el-form-item>
    </template>

    <!-- ===== Home: address, housing, dependants ===== -->
    <template v-if="show('home')">
      <el-form-item label="Home address" required :error="need(draft.address)">
        <FormField
          :model-value="draft.address"
          :property="field('Party', 'address', 'Home address', 'textarea')"
          :form="draft"
          @update:model-value="set('address', $event)"
        />
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">House number, street, and village or town.</small>
      </el-form-item>
      <div class="field-grid flex flex-wrap" style="column-gap:16px">
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Country" required :error="need(draft.country)">
          <FormField
            :model-value="draft.country"
            :property="field('Party', 'country', 'Country', 'select')"
            :form="draft"
            @update:model-value="set('country', $event)"
          />
        </el-form-item>
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Parish" required :error="need(draft.parish)" v-if="inGrenada">
          <FormField
            :model-value="draft.parish"
            :property="field('Party', 'parish', 'Parish', 'select')"
            :form="draft"
            @update:model-value="set('parish', $event)"
          />
        </el-form-item>
      </div>
      <el-form-item label="Do you own or rent your home?" required :error="need(draft.housing_status)">
        <template v-if="choices('housing_status')">
          <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('housing_status')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.housing_status === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.housing_status === option.value"
              @click="set('housing_status', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.housing_status === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.housing_status"
            :property="field('ApplicationParty', 'housing_status', 'Do you own or rent your home?', 'select')"
            :form="draft"
            @update:model-value="set('housing_status', $event)"
          />
        </template>
      </el-form-item>
      <div class="field-grid flex flex-wrap" style="column-gap:16px">
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="When did you move there?" required :error="need(draft.address_since)">
          <FormField
            :model-value="draft.address_since"
            :property="field('Party', 'address_since', 'When did you move there?', 'date')"
            :form="draft"
            @update:model-value="set('address_since', $event, 'date')"
          />
        </el-form-item>
        <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="People who depend on you" required :error="need(draft.number_of_dependants)">
          <FormField
            :model-value="draft.number_of_dependants"
            :property="field('ApplicationParty', 'number_of_dependants', 'People who depend on you', 'number')"
            :form="draft"
            @update:model-value="set('number_of_dependants', $event)"
          />
          <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">For example children. Enter 0 if none.</small>
        </el-form-item>
      </div>

      <!-- Previous address: only under 2 years at this one -->
      <template v-if="needsPreviousAddress">
        <p class="subheading mt-6 mb-3 text-lg font-bold" style="color:#111827">Where did you live before?</p>
        <p class="hint mt-2 text-base text-gray-600">You've lived at your address for less than 2 years, so we need your previous one too.</p>
        <el-form-item label="Previous address" required :error="need(draft.previous_address)">
          <FormField
            :model-value="draft.previous_address"
            :property="field('Party', 'previous_address', 'Previous address', 'textarea')"
            :form="draft"
            @update:model-value="set('previous_address', $event)"
          />
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Country">
            <FormField
              :model-value="draft.previous_country"
              :property="field('Party', 'previous_country', 'Country', 'select')"
              :form="draft"
              @update:model-value="set('previous_country', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Parish" v-if="previousInGrenada">
            <FormField
              :model-value="draft.previous_parish"
              :property="field('Party', 'previous_parish', 'Parish', 'select')"
              :form="draft"
              @update:model-value="set('previous_parish', $event)"
            />
          </el-form-item>
        </div>
      </template>

      <!-- Optional extra, hidden until asked for -->
      <button v-if="!showMore && !draft.mailing_address" type="button" class="more-link inline-flex items-center gap-1 mb-5 py-1 text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer;text-align:left" @click="showMore = true">
        <v-icon size="20">mdi-plus</v-icon>
        My mail goes to a different address
      </button>
      <el-form-item label="Mailing address" v-else>
        <FormField
          :model-value="draft.mailing_address"
          :property="field('Party', 'mailing_address', 'Mailing address', 'textarea')"
          :form="draft"
          @update:model-value="set('mailing_address', $event)"
        />
      </el-form-item>
    </template>

    <!-- ===== Membership and citizenship ===== -->
    <template v-if="show('membership')">
      <el-form-item label="Are you a member of the credit union?">
        <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
          <button
            v-for="option in [{ value: true, label: 'Yes, I am a member' }, { value: false, label: 'Not yet' }]"
            :key="String(option.value)"
            type="button"
            role="radio"
            class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
            :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (Boolean(draft.is_member) === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
            :aria-checked="Boolean(draft.is_member) === option.value"
            @click="set('is_member', option.value)"
          >
            <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (Boolean(draft.is_member) === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
            <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
          </button>
        </div>
      </el-form-item>
      <el-form-item label="Member number" required :error="need(draft.member_number)" v-if="draft.is_member">
        <FormField
          :model-value="draft.member_number"
          :property="field('Party', 'member_number', 'Member number', 'input')"
          :form="draft"
          @update:model-value="set('member_number', $event)"
        />
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">It's on your passbook or member card.</small>
      </el-form-item>
      <p v-else class="soft-box info mb-5 p-4 rounded-lg bg-blue-50 text-blue-800">That's fine. You can still apply, and we'll help you join.</p>
      <el-form-item label="Which country are you a citizen of?" required :error="need(draft.citizenship)">
        <FormField
          :model-value="draft.citizenship"
          :property="field('Party', 'citizenship', 'Which country are you a citizen of?', 'select')"
          :form="draft"
          @update:model-value="set('citizenship', $event)"
        />
      </el-form-item>
      <el-form-item label="Residency status" required :error="need(draft.residency_status)">
        <template v-if="choices('residency_status')">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('residency_status')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.residency_status === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.residency_status === option.value"
              @click="set('residency_status', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.residency_status === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.residency_status"
            :property="field('Party', 'residency_status', 'Residency status', 'select')"
            :form="draft"
            @update:model-value="set('residency_status', $event)"
          />
        </template>
      </el-form-item>
      <button v-if="!showMore && !draft.tin" type="button" class="more-link inline-flex items-center gap-1 mb-5 py-1 text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer;text-align:left" @click="showMore = true">
        <v-icon size="20">mdi-plus</v-icon>
        Add a tax number (TIN)
      </button>
      <el-form-item label="Tax number (TIN)" v-else>
        <FormField
          :model-value="draft.tin"
          :property="field('Party', 'tin', 'Tax number (TIN)', 'input')"
          :form="draft"
          @update:model-value="set('tin', $event)"
        />
      </el-form-item>
    </template>

    <!-- ===== NIS number, photo ID, and their documents ===== -->
    <template v-if="show('ids')">
      <el-form-item label="NIS number" :required="!isStudent" :error="isStudent ? '' : need(draft.nis_number)">
        <FormField
          :model-value="draft.nis_number"
          :property="field('Party', 'nis_number', 'NIS number', 'input')"
          :form="draft"
          @update:model-value="set('nis_number', $event)"
        />
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">{{ isStudent ? "It's printed on your NIS card. Leave it empty if you don't have one yet." : "It's printed on your NIS card." }}</small>
      </el-form-item>

      <p class="subheading mt-6 mb-3 text-lg font-bold" style="color:#111827">Photo ID</p>
      <article
        v-for="(row, index) in draft.identifications"
        :key="row.client_key"
        class="item-card open mb-4 p-5 bg-white rounded-lg" style="border:2px solid var(--brand);box-shadow:0 0 0 3px var(--brand-tint)"
      >
        <div class="item-head head-open flex flex-wrap items-center gap-3" style="margin-bottom:20px;padding-bottom:16px;border-bottom:1px solid #e5e7eb">
          <span class="item-icon flex flex-col items-center justify-center rounded-lg" style="width:44px;min-width:44px;max-width:44px;height:44px;min-height:44px;flex:0 0 44px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;background:var(--brand-tint);color:var(--brand-text)"><v-icon>mdi-card-account-details-outline</v-icon></span>
          <div class="item-text" style="flex:1 1 160px;min-width:0">
            <strong class="item-name block text-lg font-bold" style="line-height:1.3;overflow-wrap:anywhere">{{ identificationTitle(row, index) }}</strong>
            <span class="item-sub block text-sm text-gray-600" v-if="row.is_primary && draft.identifications.length > 1">Your main ID</span>
          </div>
          <div class="item-actions flex gap-1" style="margin-left:auto">
            <button
              v-if="!row.is_primary && draft.identifications.length > 1"
              type="button"
              class="link-btn"
              @click="setPrimary(index)"
            >
              Make main
            </button>
            <button
              v-if="draft.identifications.length > 1"
              type="button"
              class="link-btn danger"
              @click="removeIdentification(index)"
            >
              Remove
            </button>
          </div>
        </div>
        <el-form-item label="Type of ID" required :error="need(row.identification_type)">
          <template v-if="choices('identification_type')">
            <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('identification_type')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (row.identification_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="row.identification_type === option.value"
                @click="setIdentification(index, 'identification_type', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (row.identification_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="row.identification_type"
              :property="field('PartyIdentification', 'identification_type', 'Type of ID', 'select')"
              :form="row"
              @update:model-value="setIdentification(index, 'identification_type', $event)"
            />
          </template>
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="ID number" required :error="need(row.identification_number)">
            <FormField
              :model-value="row.identification_number"
              :property="field('PartyIdentification', 'identification_number', 'ID number', 'input')"
              :form="row"
              @update:model-value="setIdentification(index, 'identification_number', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Expiry date">
            <FormField
              :model-value="row.expiry_date"
              :property="field('PartyIdentification', 'expiry_date', 'Expiry date', 'date')"
              :form="row"
              @update:model-value="setIdentification(index, 'expiry_date', $event, 'date')"
            />
            <small v-if="isExpired(row)" class="helper invalid block w-full mt-1 text-sm font-semibold text-red-700" style="flex:1 1 100%;line-height:1.45">
              This ID has expired. Please use one that's still valid.
            </small>
          </el-form-item>
        </div>

        <!-- Less common details, hidden until asked for -->
        <button v-if="!moreFor[row.client_key]" type="button" class="more-link inline-flex items-center gap-1 mb-5 py-1 text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer;text-align:left" @click="moreFor[row.client_key] = true">
          <v-icon size="20">mdi-plus</v-icon>
          Add issue date and country
        </button>
        <div v-else class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Issuing country">
            <FormField
              :model-value="row.issuing_country"
              :property="field('PartyIdentification', 'issuing_country', 'Issuing country', 'select')"
              :form="row"
              @update:model-value="setIdentification(index, 'issuing_country', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Issue date">
            <FormField
              :model-value="row.issue_date"
              :property="field('PartyIdentification', 'issue_date', 'Issue date', 'date')"
              :form="row"
              @update:model-value="setIdentification(index, 'issue_date', $event, 'date')"
            />
          </el-form-item>
        </div>

        <!-- Photo or scan of this ID, if the credit union asks for one -->
        <AdaptiveLoanDocumentRequirements
          title="Photo of this ID"
          :scope="documentScopes[`identification:${row.client_key}`]"
          :uploading-key="uploadingKey"
          :disabled="documentsDisabled"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />
      </article>

      <button type="button" class="add-button flex items-center justify-center gap-2 w-full bg-white rounded-lg text-lg font-semibold" style="min-height:56px;border:2px dashed #d1d5db;color:var(--brand-text);cursor:pointer" @click="addIdentification">
        <v-icon>mdi-plus</v-icon>
        Add another ID
      </button>
      <p class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">{{ identificationHint }}</p>

      <!-- Documents everyone gives, like the NIS card -->
      <AdaptiveLoanDocumentRequirements
        title="Your documents"
        :scope="documentScopes[`applicant:${draft.client_key}`]"
        :uploading-key="uploadingKey"
        :disabled="documentsDisabled"
        @stage-file="$emit('stage-file', $event)"
        @remove-file="$emit('remove-file', $event)"
        @request-file-upload="$emit('request-file-upload', $event)"
        @file-rejected="$emit('file-rejected', $event)"
      />
    </template>

    <!-- ===== Work ===== -->
    <template v-if="show('job')">
      <el-form-item label="What is your work situation?" required :error="need(draft.employment_status)">
        <template v-if="choices('employment_status')">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('employment_status')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.employment_status === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.employment_status === option.value"
              @click="set('employment_status', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.employment_status === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.employment_status"
            :property="field('ApplicationParty', 'employment_status', 'What is your work situation?', 'select')"
            :form="draft"
            @update:model-value="set('employment_status', $event)"
          />
        </template>
      </el-form-item>
      <template v-if="showEmploymentDetails">
        <el-form-item label="Type of job" required :error="need(draft.employment_type)">
          <template v-if="choices('employment_type')">
            <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('employment_type')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.employment_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.employment_type === option.value"
                @click="set('employment_type', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.employment_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.employment_type"
              :property="field('ApplicationParty', 'employment_type', 'Type of job', 'select')"
              :form="draft"
              @update:model-value="set('employment_type', $event)"
            />
          </template>
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Employer or business name">
            <FormField
              :model-value="draft.employer_name"
              :property="field('ApplicationParty', 'employer_name', 'Employer or business name', 'input')"
              :form="draft"
              @update:model-value="set('employer_name', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Job title">
            <FormField
              :model-value="draft.job_title"
              :property="field('ApplicationParty', 'job_title', 'Job title', 'input')"
              :form="draft"
              @update:model-value="set('job_title', $event)"
            />
          </el-form-item>
        </div>
        <el-form-item label="When did you start?" required :error="need(draft.employment_start_date)">
          <FormField
            :model-value="draft.employment_start_date"
            :property="field('ApplicationParty', 'employment_start_date', 'When did you start?', 'date')"
            :form="draft"
            @update:model-value="set('employment_start_date', $event, 'date')"
          />
        </el-form-item>
      </template>

      <!-- Previous job: only under 2 years in this one -->
      <template v-if="needsPreviousEmployment">
        <p class="subheading mt-6 mb-3 text-lg font-bold" style="color:#111827">Where did you work before?</p>
        <p class="hint mt-2 text-base text-gray-600">You started less than 2 years ago, so we need your previous job too.</p>
        <el-form-item label="Previous employer" required :error="need(draft.previous_employer_name)">
          <FormField
            :model-value="draft.previous_employer_name"
            :property="field('ApplicationParty', 'previous_employer_name', 'Previous employer', 'input')"
            :form="draft"
            @update:model-value="set('previous_employer_name', $event)"
          />
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Job title there">
            <FormField
              :model-value="draft.previous_job_title"
              :property="field('ApplicationParty', 'previous_job_title', 'Job title there', 'input')"
              :form="draft"
              @update:model-value="set('previous_job_title', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Years there">
            <FormField
              :model-value="draft.previous_employment_years"
              :property="field('ApplicationParty', 'previous_employment_years', 'Years there', 'number')"
              :form="draft"
              @update:model-value="set('previous_employment_years', $event)"
            />
          </el-form-item>
        </div>
      </template>
    </template>

    <!-- ===== Pay, other income, and (guarantors) the guarantee ===== -->
    <template v-if="show('pay')">
      <el-form-item label="Your pay before tax (EC$)" required :error="need(draft.gross_pay)">
        <FormField
          :model-value="draft.gross_pay"
          :property="field('ApplicationParty', 'gross_pay', 'Your pay before tax (EC$)', 'number')"
          :form="draft"
          @update:model-value="set('gross_pay', $event)"
        />
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">The amount on your payslip before anything is taken off. Enter 0 if you have no pay.</small>
      </el-form-item>
      <el-form-item label="How often are you paid?" v-if="Number(draft.gross_pay) > 0" required :error="need(draft.pay_frequency)">
        <template v-if="choices('pay_frequency')">
          <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in choices('pay_frequency')"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.pay_frequency === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="draft.pay_frequency === option.value"
              @click="set('pay_frequency', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.pay_frequency === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
            </button>
          </div>
        </template>
        <template v-else>
          <FormField
            :model-value="draft.pay_frequency"
            :property="field('ApplicationParty', 'pay_frequency', 'How often are you paid?', 'select')"
            :form="draft"
            @update:model-value="set('pay_frequency', $event)"
          />
        </template>
      </el-form-item>
      <el-form-item label="Business sales in a year (EC$)" v-if="isSelfEmployed">
        <FormField
          :model-value="draft.annual_revenue"
          :property="field('ApplicationParty', 'annual_revenue', 'Business sales in a year (EC$)', 'number')"
          :form="draft"
          @update:model-value="set('annual_revenue', $event)"
        />
      </el-form-item>

      <!-- NIS and income tax are worked out, not entered -->
      <div v-if="showDeductions" class="soft-box mb-5 p-4 rounded-lg bg-gray-100">
        <div class="sum-row flex justify-between gap-4 py-1">
          <span>Pay each month, before tax</span>
          <strong>{{ money(deductions.gross) }}</strong>
        </div>
        <div class="sum-row flex justify-between gap-4 py-1">
          <span>NIS (estimate)</span>
          <strong>− {{ money(deductions.nis) }}</strong>
        </div>
        <div class="sum-row flex justify-between gap-4 py-1">
          <span>Income tax (estimate)</span>
          <strong>− {{ money(deductions.incomeTax) }}</strong>
        </div>
        <div class="sum-row total flex justify-between gap-4 mt-2 pt-3 text-lg font-bold" style="border-top:2px solid #d1d5db">
          <span>Take-home pay each month</span>
          <strong>{{ money(deductions.net) }}</strong>
        </div>
      </div>

      <!-- Other income: asked as Yes/No first -->
      <el-form-item label="Do you get money from anywhere else?">
        <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
          <button
            v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
            :key="String(option.value)"
            type="button"
            role="radio"
            class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
            :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (hasOtherIncome === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
            :aria-checked="hasOtherIncome === option.value"
            @click="setOtherIncome(option.value)"
          >
            <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (hasOtherIncome === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
            <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
          </button>
        </div>
        <small class="helper block w-full mt-1 text-sm text-gray-600" style="flex:1 1 100%;line-height:1.45">For example rent, a pension, money sent from abroad, or a second job.</small>
      </el-form-item>

      <template v-if="hasOtherIncome">
        <article
          v-for="(income, index) in draft.incomes"
          :key="income.client_key"
          class="item-card open mb-4 p-5 bg-white rounded-lg" style="border:2px solid var(--brand);box-shadow:0 0 0 3px var(--brand-tint)"
        >
          <div class="item-head head-open flex flex-wrap items-center gap-3" style="margin-bottom:20px;padding-bottom:16px;border-bottom:1px solid #e5e7eb">
            <span class="item-icon flex items-center justify-center rounded-lg" style="width:44px;min-width:44px;max-width:44px;height:44px;min-height:44px;flex:0 0 44px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;background:var(--brand-tint);color:var(--brand-text)"><v-icon>mdi-cash-plus</v-icon></span>
            <div class="item-text" style="flex:1 1 160px;min-width:0">
              <strong class="item-name block text-lg font-bold" style="line-height:1.3;overflow-wrap:anywhere">{{ lookupLabel('income_type', income.income_type) || `Other money ${index + 1}` }}</strong>
            </div>
            <div class="item-actions flex gap-1" style="margin-left:auto">
              <button type="button" class="link-btn danger px-3 py-2 rounded text-base font-semibold text-red-700" style="background:transparent;border:0;cursor:pointer" @click="removeIncome(index)">Remove</button>
            </div>
          </div>
          <el-form-item label="Where does it come from?" required :error="need(income.income_type)">
            <template v-if="choices('income_type')">
              <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
                <button
                  v-for="option in choices('income_type')"
                  :key="String(option.value)"
                  type="button"
                  role="radio"
                  class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                  :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (income.income_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                  :aria-checked="income.income_type === option.value"
                  @click="setIncome(index, 'income_type', option.value)"
                >
                  <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (income.income_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                  <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
                </button>
              </div>
            </template>
            <template v-else>
              <FormField
                :model-value="income.income_type"
                :property="field('IncomeSource', 'income_type', 'Where does it come from?', 'select')"
                :form="income"
                @update:model-value="setIncome(index, 'income_type', $event)"
              />
            </template>
          </el-form-item>
          <div class="field-grid flex flex-wrap" style="column-gap:16px">
            <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Amount (EC$)" required :error="need(income.amount)">
              <FormField
                :model-value="income.amount"
                :property="field('IncomeSource', 'amount', 'Amount (EC$)', 'number')"
                :form="income"
                @update:model-value="setIncome(index, 'amount', $event)"
              />
            </el-form-item>
            <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="How often?" required :error="need(income.frequency)">
              <FormField
                :model-value="income.frequency"
                :property="field('IncomeSource', 'frequency', 'How often?', 'select')"
                :form="income"
                @update:model-value="setIncome(index, 'frequency', $event)"
              />
            </el-form-item>
          </div>
          <AdaptiveLoanDocumentRequirements
            title="Proof of this money"
            :scope="documentScopes[`income:${income.client_key}`]"
            :uploading-key="uploadingKey"
            :disabled="documentsDisabled"
            @stage-file="$emit('stage-file', $event)"
            @remove-file="$emit('remove-file', $event)"
            @request-file-upload="$emit('request-file-upload', $event)"
            @file-rejected="$emit('file-rejected', $event)"
          />
        </article>
        <button type="button" class="add-button flex items-center justify-center gap-2 w-full bg-white rounded-lg text-lg font-semibold" style="min-height:56px;border:2px dashed #d1d5db;color:var(--brand-text);cursor:pointer" @click="addIncome">
          <v-icon>mdi-plus</v-icon>
          Add more money you get
        </button>
      </template>

      <!-- Guarantors: the guarantee they're giving -->
      <template v-if="isGuarantor">
        <p class="subheading mt-6 mb-3 text-lg font-bold" style="color:#111827">The guarantee</p>
        <el-form-item label="Type of guarantee" required :error="need(draft.guarantee_type)">
          <template v-if="choices('guarantee_type')">
            <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('guarantee_type')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft.guarantee_type === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="draft.guarantee_type === option.value"
                @click="set('guarantee_type', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft.guarantee_type === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="draft.guarantee_type"
              :property="field('ApplicationParty', 'guarantee_type', 'Type of guarantee', 'select')"
              :form="draft"
              @update:model-value="set('guarantee_type', $event)"
            />
          </template>
        </el-form-item>
        <el-form-item label="Amount guaranteed (EC$)" required :error="need(draft.guarantee_amount)">
          <FormField
            :model-value="draft.guarantee_amount"
            :property="field('ApplicationParty', 'guarantee_amount', 'Amount guaranteed (EC$)', 'number')"
            :form="draft"
            @update:model-value="set('guarantee_amount', $event)"
          />
        </el-form-item>
      </template>
    </template>

    <!-- ===== Declarations: Yes/No questions ===== -->
    <template v-if="show('declarations')">
      <div v-for="question in declarationQuestions" :key="question.key" class="question-block mb-6">
        <p class="question mb-3 text-base font-semibold" style="color:#111827">{{ question.text }}</p>
        <div class="choice-list inline flex flex-col gap-3 w-full" role="radiogroup">
          <button
            v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
            :key="String(option.value)"
            type="button"
            role="radio"
            class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
            :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (draft[question.key] === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
            :aria-checked="draft[question.key] === option.value"
            @click="set(question.key, option.value)"
          >
            <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (draft[question.key] === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
            <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
          </button>
        </div>
        <small v-if="showErrors && draft[question.key] !== true && draft[question.key] !== false" class="helper invalid block w-full mt-1 text-sm font-semibold text-red-700" style="flex:1 1 100%;line-height:1.45">
          Please choose Yes or No.
        </small>
      </div>
      <el-form-item label="Please tell us the position and who holds it" required :error="need(draft.pep_details)" v-if="draft.is_pep">
        <FormField
          :model-value="draft.pep_details"
          :property="field('ApplicationParty', 'pep_details', 'Please tell us the position and who holds it', 'textarea')"
          :form="draft"
          @update:model-value="set('pep_details', $event)"
        />
      </el-form-item>
      <el-form-item label="Please tell us more about each Yes answer" required :error="need(draft.declaration_details)" v-if="anyDeclaration">
        <FormField
          :model-value="draft.declaration_details"
          :property="field('ApplicationParty', 'declaration_details', 'Please tell us more about each Yes answer', 'textarea')"
          :form="draft"
          @update:model-value="set('declaration_details', $event)"
        />
      </el-form-item>
    </template>

    <!-- ===== Agreement: big tick boxes ===== -->
    <template v-if="show('consent')">
      <div class="choice-list flex flex-col gap-3 w-full">
        <button
          v-for="statement in consentStatements"
          :key="statement.key"
          type="button"
          role="checkbox"
          class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
          :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827;line-height:1.35', (Boolean(draft[statement.key])) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
          :aria-checked="Boolean(draft[statement.key])"
          @click="set(statement.key, !draft[statement.key])"
        >
          <span class="choice-mark square flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (Boolean(draft[statement.key])) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
          <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ statement.text }}</span>
        </button>
      </div>
      <small v-if="showErrors && !allConsented" class="helper invalid block w-full mt-1 text-sm font-semibold text-red-700" style="flex:1 1 100%;line-height:1.45">Please tick every box to continue.</small>
    </template>
  </div>
</template>

<script>
/**
 * Editor for one applicant: personal details, split home address, NIS
 * number, identifications, employment and income (with estimated
 * statutory deductions calculated by the parent), and consent. Edits a
 * deep-cloned draft and emits a fresh copy on every change.
 *
 * The form shows one short screen at a time, so the `screen` prop picks
 * which part to show: about, home, membership, ids, job, pay,
 * declarations, consent, or owner (the Third Party Owner's short form).
 * These match APPLICANT_PARTS in the main form. With no screen, every
 * part is shown. Optional details stay hidden behind "+ Add ..." links.
 *
 * A Third Party Owner (owns collateral but isn't borrowing) gets a short
 * form instead: person or business, name, relationship, phone, and email.
 *
 * Questions with a short list of answers (see choices()) are shown as big
 * tappable answer cards; the rest use Saturn's built-in FormField.
 *
 * Every other input is Saturn's built-in FormField. Each field uses Saturn's own
 * definition of the property (Party, ApplicationParty, or
 * PartyIdentification) when there is one. Role and "Owner is a" are
 * el-selects, since the form decides their choices ("Primary Applicant" is
 * left out of Role).
 */
export default {
  props: {
    modelValue: {
      type: Object,
      default: () => ({}),
    },
    /** Which part to show (see above); "" shows everything. */
    screen: {
      type: String,
      default: '',
    },
    /** True for the person filling in the form (not someone they added). */
    isPrimary: {
      type: Boolean,
      default: true,
    },
    /** After a failed Next, shows "this is needed" under empty required boxes. */
    showErrors: {
      type: Boolean,
      default: false,
    },
    lookups: {
      type: Object,
      default: () => ({}),
    },
    /** Saturn's property definitions, keyed by resource name. */
    resourceProps: {
      type: Object,
      default: () => ({}),
    },
    roleOptions: {
      type: Array,
      default: () => [],
    },
    showRole: {
      type: Boolean,
      default: false,
    },
    /** Estimated { nis, nisBasis, incomeTax, net } from the parent form. */
    deductions: {
      type: Object,
      default: null,
    },
    /** Institution-wide minimum number of identifications. */
    minimumIdentifications: {
      type: Number,
      default: 1,
    },
    /** Document scopes from the parent (for identification scans). */
    documentScopes: {
      type: Object,
      default: () => ({}),
    },
    uploadingKey: {
      type: String,
      default: '',
    },
    documentsDisabled: {
      type: Boolean,
      default: false,
    },
  },

  emits: [
    'update:modelValue',
    // Document events are passed straight through to the parent form.
    'stage-file',
    'remove-file',
    'request-file-upload',
    'file-rejected',
  ],

  data() {
    return {
      draft: this.copy(this.modelValue),
      // Shows the optional extras (mailing address, tax number).
      showMore: false,
      // Shows an ID's issue date and country, by the ID's client key.
      moreFor: {},
      // "Do you get money from anywhere else?": yes if there's any already.
      hasOtherIncome: (this.modelValue.incomes || []).length ? true : null,
      ownerKindOptions: [
        { label: 'Person', value: 'PERSON' },
        { label: 'Business', value: 'ORGANIZATION' },
      ],
    };
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
  },

  computed: {
    /** The Yes/No questions on the declarations screen. */
    declarationQuestions() {
      return [
        {
          key: 'is_pep',
          text: 'Do you hold, or are you close family of someone who holds, a senior public position (for example a minister, judge, or senior officer)?',
        },
        { key: 'declared_bankruptcy', text: 'Have you ever been declared bankrupt?' },
        { key: 'declared_judgments', text: 'Has a court ever ordered you to pay a debt?' },
        { key: 'declared_arrears', text: 'Are you behind on any loan or bill payments right now?' },
        {
          key: 'declared_other_applications',
          text: 'Have you applied for any other loans that are still being decided?',
        },
      ];
    },

    /** The three agreement statements, each a big tick box. */
    consentStatements() {
      return [
        { key: 'consent_accuracy_confirmation', text: "Everything I've told you is true and complete." },
        { key: 'consent_credit_check', text: 'You may check my credit history.' },
        {
          key: 'consent_data_processing',
          text: 'You may use my information to process this application, as your privacy terms explain.',
        },
      ];
    },

    allConsented() {
      return this.consentStatements.every((statement) => Boolean(this.draft[statement.key]));
    },

    /** Matches THIRD_PARTY_OWNER_ROLE in the main form. */
    isThirdPartyOwner() {
      return String(this.draft.role || '').trim().toLowerCase() === 'third party owner';
    },

    /** Matches GUARANTOR_ROLE in the main form. */
    isGuarantor() {
      return String(this.draft.role || '').trim().toLowerCase() === 'guarantor';
    },

    /** Students may leave the NIS number empty. */
    isStudent() {
      return String(this.draft.employment_status || '').trim().toLowerCase() === 'student';
    },

    showEmploymentDetails() {
      const value = String(this.draft.employment_status || '').trim().toLowerCase();
      return value !== 'unemployed' && value !== 'retired' && value !== 'student';
    },

    isSelfEmployed() {
      const pattern = /self/i;
      return pattern.test(this.draft.employment_status || '') || pattern.test(this.draft.employment_type || '');
    },

    /** Under 2 years in the current job (matches needsPreviousEmployment in the main form). */
    needsPreviousEmployment() {
      const years = this.yearsSince(this.draft.employment_start_date);
      return this.showEmploymentDetails && years !== null && years < 2;
    },

    /** Under 2 years at this address (matches needsPreviousAddress in the main form). */
    needsPreviousAddress() {
      const years = this.yearsSince(this.draft.address_since);
      return years !== null && years < 2;
    },

    anyDeclaration() {
      const person = this.draft;
      return Boolean(
        person.declared_bankruptcy ||
          person.declared_judgments ||
          person.declared_arrears ||
          person.declared_other_applications
      );
    },

    inGrenada() {
      const country = String(this.draft.country || '').trim().toLowerCase();
      return !country || country === 'grenada';
    },

    previousInGrenada() {
      const country = String(this.draft.previous_country || '').trim().toLowerCase();
      return !country || country === 'grenada';
    },

    identificationHint() {
      const minimum = this.minimumIdentifications;
      const base =
        minimum > 1
          ? `At least ${minimum} forms of identification are required.`
          : 'At least one form of identification is required.';
      return `${base} Each must be a different type and not expired.`;
    },

    /** Show the estimate once there's a gross income to base it on. */
    showDeductions() {
      const gross = this.draft.gross_pay;
      return Boolean(this.deductions) && gross !== null && gross !== undefined && gross !== '';
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
    /** True when this part of the details is on screen. */
    show(part) {
      if (this.screen) return this.screen === part;
      if (part === 'owner') return this.isThirdPartyOwner;
      return !this.isThirdPartyOwner;
    },

    /** "This is needed" under an empty required box, after a failed Next. */
    need(value) {
      if (!this.showErrors) return '';
      const empty = value === null || value === undefined || value === '';
      return empty ? 'This is needed' : '';
    },

    /** Answers "Do you get money from anywhere else?". No clears the list. */
    setOtherIncome(value) {
      this.hasOtherIncome = value;
      if (value && !this.draft.incomes.length) {
        this.addIncome();
        return;
      }
      if (!value && this.draft.incomes.length) {
        this.draft.incomes = [];
        this.notify();
      }
    },

    copy(value) {
      const copy = JSON.parse(JSON.stringify(value || {}));
      if (!Array.isArray(copy.identifications)) copy.identifications = [];
      if (!Array.isArray(copy.incomes)) copy.incomes = [];
      return copy;
    },

    /** Whole years from a YYYY-MM-DD date until today, or null. */
    yearsSince(date) {
      if (!date) return null;
      const start = new Date(`${String(date).slice(0, 10)}T00:00:00`);
      if (Number.isNaN(start.getTime())) return null;
      const now = new Date();
      let years = now.getFullYear() - start.getFullYear();
      const anniversaryPassed =
        now.getMonth() > start.getMonth() ||
        (now.getMonth() === start.getMonth() && now.getDate() >= start.getDate());
      if (!anniversaryPassed) years--;
      return Math.max(0, years);
    },

    key(prefix) {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    },

    lookup(key) {
      return this.lookups[key] || [];
    },

    /**
     * The answers for a question as tappable cards, when Saturn's list is
     * short (2 to 7 answers). Otherwise null, and Saturn's dropdown is used.
     */
    choices(key) {
      const options = this.lookup(key);
      return options.length >= 2 && options.length <= 7 ? options : null;
    },

    lookupLabel(key, value) {
      const option = this.lookup(key).find((entry) => entry.value === value);
      return option ? option.label : value || '';
    },

    money(value) {
      return `EC$ ${Number(value).toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      })}`;
    },

    // ---- Saturn FormField helpers (the same in every section) ----

    /**
     * FormField's update event may send the value itself or
     * { property, data } (the Saturn guide isn't clear), so accept both.
     * Dates are kept as YYYY-MM-DD strings.
     */
    valueOf(event, kind) {
      let value = event;
      if (value && typeof value === 'object' && 'property' in value && 'data' in value) {
        value = value.data;
      }
      return kind === 'date' ? this.toDateString(value) : value;
    },

    /** A date as YYYY-MM-DD (the local date for Date objects), or ''. */
    toDateString(value) {
      if (!value) return '';
      if (value instanceof Date) {
        const pad = (number) => String(number).padStart(2, '0');
        return `${value.getFullYear()}-${pad(value.getMonth() + 1)}-${pad(value.getDate())}`;
      }
      return String(value).slice(0, 10);
    },

    /** Saturn's definition of one property of a resource, or null. */
    savedProperty(resourceName, name) {
      const rows = this.resourceProps[resourceName] || [];
      return rows.find((row) => String(row.property || row.key || row.name || '') === name) || null;
    },

    /**
     * FormField property config for one field. Uses Saturn's own definition
     * of the property (with our label) when there is one. Otherwise builds a
     * basic one from the Saturn guide; a dropdown becomes a text box, since
     * FormField's own option lists (lookup_type "values") don't work in
     * Saturn. Dropdowns whose choices the form decides use el-select
     * instead. Configs are reused while unchanged, so FormField isn't handed
     * a new object on every keystroke.
     */
    field(resourceName, name, label, kind) {
      const saved = this.savedProperty(resourceName, name);
      let config;
      if (saved) {
        config = Object.assign({}, saved, { property: name, label });
      } else if (kind === 'number' || kind === 'date') {
        config = { property: name, label, type: kind };
      } else if (kind === 'checkbox') {
        config = { property: name, label, type: 'boolean', input_properties: { type: 'check-box' } };
      } else {
        config = {
          property: name,
          label,
          type: 'string',
          input_properties: { type: kind === 'textarea' ? 'textarea' : 'input' },
        };
      }
      const cacheKey = JSON.stringify(config);
      if (!this.fieldCache[cacheKey]) this.fieldCache[cacheKey] = config;
      return this.fieldCache[cacheKey];
    },

    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    set(key, event, kind) {
      this.draft[key] = this.valueOf(event, kind);
      // Parish only applies in Grenada; clear it when the country changes.
      if (key === 'country' && !this.inGrenada) this.draft.parish = '';
      this.notify();
    },

    // ---- Identifications ----

    identificationTitle(row, index) {
      return (
        this.lookupLabel('identification_type', row.identification_type) ||
        `Identification ${index + 1}`
      );
    },

    isExpired(row) {
      return Boolean(row.expiry_date && row.expiry_date < this.toDateString(new Date()));
    },

    setIdentification(index, key, event, kind) {
      this.draft.identifications[index][key] = this.valueOf(event, kind);
      this.notify();
    },

    setPrimary(index) {
      this.draft.identifications.forEach((row, rowIndex) => {
        row.is_primary = rowIndex === index;
      });
      this.notify();
    },

    // ---- Other income ----

    setIncome(index, key, event) {
      this.draft.incomes[index][key] = this.valueOf(event);
      this.notify();
    },

    /** Matches createEmptyIncome() in the main form. */
    addIncome() {
      this.draft.incomes.push({
        client_key: this.key('income'),
        id: null,
        income_type: '',
        description: '',
        amount: null,
        frequency: '',
        document_ids: [],
      });
      this.notify();
    },

    removeIncome(index) {
      this.draft.incomes.splice(index, 1);
      this.notify();
    },

    addIdentification() {
      this.draft.identifications.push({
        client_key: this.key('ident'),
        id: null,
        identification_type: '',
        identification_number: '',
        issuing_country: 'Grenada',
        issue_date: '',
        expiry_date: '',
        is_primary: this.draft.identifications.length === 0,
        document_ids: [],
      });
      this.notify();
    },

    /** Removing the primary ID makes the first remaining one primary. */
    removeIdentification(index) {
      const removed = this.draft.identifications.splice(index, 1)[0];
      if (removed && removed.is_primary && this.draft.identifications.length) {
        this.draft.identifications[0].is_primary = true;
      }
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
