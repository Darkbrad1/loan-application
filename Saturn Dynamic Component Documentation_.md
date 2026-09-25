# Saturn Dynamic Component Documentation\_

## **🚀 Getting Started**

Dynamic components in Saturn allow you to write Vue 3 single-file components that will be compiled and rendered at runtime. You have access to Tailwind CSS classes, Element Plus, Vuetify, Resource/NovaAI APIs, ECharts, Three.js (`THREE`), TresJS (`TresJS`), PostProcessing (`PostProcessing`), Lodash (`_`), Moment.js (`moment`), and QrScanner (`QrScanner`).

## res**📐 Basic Component Structure**

```vue
<template>
  <div class="p-4 bg-white rounded-lg shadow">
    <h2 class="text-xl font-bold">{{ title }}</h2>
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: 'Hello World',
      message: 'This is a dynamic component'
    }
  }
}
</script>
```

## **🎨 Tailwind CSS Classes**

All commonly used Tailwind utility classes are available. Here are the main categories:

### **Spacing (Padding & Margin)**

Use`p-{size}`,`m-{size}`,`px-{size}`,`py-{size}`, etc.

**Available sizes:**&#x30;, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24, 32, 40, 48, 56, 64, 80, 96

```vue
<div class="p-4 m-2 px-6 py-3">Content</div>
```

### **Layout & Flexbox**

* `flex`,`grid`,`block`,`inline-block`,`hidden`
* `flex-row`,`flex-col`,`flex-wrap`
* `justify-start`,`justify-center`,`justify-between`
* `items-start`,`items-center`,`items-end`
* `gap-{size}`- spacing between flex/grid items

```vue
<div class="flex items-center justify-between gap-4">
  <span>Left</span>
  <span>Right</span>
</div>
```

### **Width & Height**

* `w-full`,`w-1/2`,`w-1/3`,`w-{size}`
* `h-full`,`h-screen`,`h-{size}`
* `min-w-{size}`,`max-w-{size}`

### **Colors**

Available colors: red, blue, green, yellow, purple, pink, gray, indigo, teal, orange, amber, emerald, and more

Shades: 50, 100, 200, 300, 400, 500, 600, 700, 800, 900

```vue
<div class="bg-blue-500 text-white border-gray-300">
  Colored content
</div>
```

#### **Brand Colors (Custom)**

* `bg-brand-primary-color`,`text-brand-secondary-color`
* Available: primary-color, tertiary-color, secondary-color, primary-color-dim

### **Typography**

* **Size:**`text-xs`,`text-sm`,`text-base`,`text-lg`,`text-xl`,`text-2xl`,`text-3xl`,`text-4xl`,`text-5xl`,`text-6xl`
* **Weight:**`font-light`,`font-normal`,`font-medium`,`font-semibold`,`font-bold`
* **Alignment:**`text-left`,`text-center`,`text-right`

### **Borders & Shadows**

* `border`,`border-2`,`border-4`
* `rounded`,`rounded-lg`,`rounded-full`
* `shadow`,`shadow-md`,`shadow-lg`

### **Responsive Design**

Use breakpoint prefixes:`sm:`,`md:`,`lg:`,`xl:`,`2xl:`

```vue
<div class="w-full md:w-1/2 lg:w-1/3">
  Responsive width
</div>
```

**📚 Full Reference:**&#x53;ee`/documentation/TAILWIND_CLASSES_DYNAMIC_COMPONENTS.md`for a complete list of available Tailwind classes.

## **🧩 UI Components**

**🌳 Tree-Shaking Optimized:**&#x4F;nly commonly used components are included to keep bundle size small. If you need additional components, contact support.

### **Element Plus**

Import and use Element Plus components directly:

```vue
<template>
  <el-button type="primary" @click="handleClick">Click Me</el-button>
  <el-input v-model="inputValue" placeholder="Enter text"></el-input>
  <el-message :message="'Success!'" type="success" />
</template>

<script>
import { ElMessage } from 'element-plus';

export default {
  data() {
    return { inputValue: '' }
  },
  methods: {
    handleClick() {
      ElMessage.success('Button clicked!');
    }
  }
}
</script>
```

#### **Available Element Plus Components (53 components):**

**Form Components:**

* `ElButton`- Button
* `ElInput`- Text input
* `ElSelect`+`ElOption`- Dropdown select
* `ElCheckbox`,`ElCheckboxGroup`- Checkboxes
* `ElRadioGroup`,`ElRadioButton`- Radio buttons (use`value`, not`label`, for the selected value)
* `ElSwitch`- Toggle switch
* `ElDatePicker`- Date picker
* `ElTimePicker`- Time picker
* `ElForm`,`ElFormItem`- Form container
* `ElUpload`- File upload

**Data Display:**

* `ElTable`,`ElTableColumn`- Data table
* `ElPagination`- Pagination
* `ElCard`- Card container
* `ElTag`- Tag/label
* `ElAlert`- Alert message
* `ElIcon`- Icons
* `ElBadge`- Badge/notification indicator
* `ElAvatar`- Avatar
* `ElEmpty`- Empty state
* `ElDescriptions`,`ElDescriptionsItem`- Description list
* `ElImage`- Image with lazy load

**Layout:**

* `ElRow`,`ElCol`- Grid layout
* `ElSpace`- Spacing container
* `ElDivider`- Divider line

**Feedback & Dialogs:**

* `ElMessage`- Toast message
* `ElMessageBox`- Confirm/Alert dialog
* `ElNotification`- Notification
* `ElLoading`- Loading indicator
* `ElDialog`- Dialog/modal
* `ElDrawer`- Side panel drawer
* `ElPopover`- Popover
* `ElTooltip`- Tooltip
* `ElProgress`- Progress bar

**Navigation:**

* `ElDropdown`,`ElDropdownMenu`,`ElDropdownItem`- Dropdown menu
* `ElTabs`,`ElTabPane`- Tabs
* `ElBreadcrumb`,`ElBreadcrumbItem`- Breadcrumb navigation
* `ElSteps`,`ElStep`- Step progress indicator
* `ElLink`- Link
* `ElBacktop`- Back to top button

**Others:**

* `ElCollapse`,`ElCollapseItem`- Collapsible accordion

### **Vuetify**

Vuetify components are also available:

```vue
<template>
  <v-card>
    <v-card-title>Card Title</v-card-title>
    <v-card-text>Card content</v-card-text>
    <v-card-actions>
      <v-btn color="primary">Action</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import { useDisplay } from 'vuetify';

export default {
  setup() {
    const display = useDisplay();
    return { display }
  }
}
</script>
```

#### **Available Vuetify Components:**

**Layout & Containers:**

* `VCard`,`VCardTitle`,`VCardText`,`VCardActions`- Card components
* `VContainer`- Responsive container
* `VRow`,`VCol`- Grid layout
* `VDivider`- Divider line
* `VSpacer`- Flexible spacer

**Form Components:**

* `VBtn`- Button
* `VTextField`- Text input
* `VSelect`- Dropdown select
* `VCheckbox`- Checkbox
* `VRadio`,`VRadioGroup`- Radio buttons
* `VSwitch`- Toggle switch

**Data Display:**

* `VAvatar`- Avatar/profile image
* `VImg`- Image
* `VIcon`- Material Design Icons
* `VChip`- Chip/tag
* `VAlert`- Alert message
* `VList`,`VListItem`,`VListItemTitle`- Lists

**Feedback & Dialogs:**

* `VDialog`- Dialog/modal
* `VSnackbar`- Toast notification (like ElMessage)
* `VOverlay`- Full-screen overlay
* `VProgressCircular`- Circular progress
* `VProgressLinear`- Linear progress bar

**Navigation:**

* `VMenu`- Context menu
* `VTooltip`- Tooltip
* `VTabs`,`VTab`- Tabs navigation

**Composables:**

* `useDisplay`- Responsive breakpoints
* `useTheme`- Theme utilities

### **Message Boxes & Dialogs - Detailed Examples**

#### **Element Plus Messages & Dialogs**

```vue
<template>
  <div>
    <el-button @click="showMessage">Show Message</el-button>
    <el-button @click="showNotification">Show Notification</el-button>
    <el-button @click="showConfirm">Show Confirm</el-button>
    <el-button @click="dialogVisible = true">Show Dialog</el-button>

    <!-- Element Plus Dialog -->
    <el-dialog v-model="dialogVisible" title="My Dialog" width="500px">
      <p>Dialog content goes here</p>
      <template #footer>
        <el-button @click="dialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="handleSubmit">Submit</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { ElMessage, ElMessageBox, ElNotification } from 'element-plus';

export default {
  data() {
    return {
      dialogVisible: false
    }
  },
  methods: {
    showMessage() {
      // Simple message toast
      ElMessage.success('Operation successful!');
      // or: ElMessage.error('Something went wrong!');
      // or: ElMessage.warning('Warning message');
      // or: ElMessage.info('Info message');
    },
    showNotification() {
      // Notification with more details
      ElNotification({
        title: 'Success',
        message: 'This is a detailed notification',
        type: 'success',
        position: 'top-right'
      });
    },
    async showConfirm() {
      // Confirmation dialog
      try {
        await ElMessageBox.confirm(
          'Are you sure you want to delete this?',
          'Confirmation',
          {
            confirmButtonText: 'Yes',
            cancelButtonText: 'No',
            type: 'warning'
          }
        );
        ElMessage.success('Deleted!');
      } catch {
        ElMessage.info('Cancelled');
      }
    },
    handleSubmit() {
      this.dialogVisible = false;
      ElMessage.success('Form submitted!');
    }
  }
}
</script>
```

#### **Vuetify Snackbar & Dialog**

```vue
<template>
  <div>
    <v-btn @click="showSnackbar">Show Snackbar</v-btn>
    <v-btn @click="dialog = true">Show Dialog</v-btn>

    <!-- Vuetify Snackbar (Toast) -->
    <v-snackbar
      v-model="snackbar"
      :timeout="3000"
      color="success"
      location="top"
    >
      {{ snackbarText }}
      <template v-slot:actions>
        <v-btn color="white" variant="text" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>

    <!-- Vuetify Dialog -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title class="text-h5">Dialog Title</v-card-title>
        <v-card-text>
          Dialog content goes here. You can add forms, text, or any other content.
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="grey" variant="text" @click="dialog = false">
            Cancel
          </v-btn>
          <v-btn color="primary" variant="text" @click="handleDialogSubmit">
            Submit
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Vuetify Overlay (for custom loading) -->
    <v-overlay v-model="loading" class="align-center justify-center">
      <v-progress-circular indeterminate size="64"></v-progress-circular>
    </v-overlay>
  </div>
</template>

<script>
export default {
  data() {
    return {
      snackbar: false,
      snackbarText: '',
      dialog: false,
      loading: false
    }
  },
  methods: {
    showSnackbar() {
      this.snackbarText = 'Operation successful!';
      this.snackbar = true;
    },
    handleDialogSubmit() {
      this.dialog = false;
      this.showSnackbar();
    }
  }
}
</script>
```

**💡 Quick Comparison:**

* **Element Plus:**`ElMessage`(programmatic) vs**Vuetify:**`VSnackbar`(template-based)
* **Element Plus:**`ElMessageBox`(async confirms) vs**Vuetify:**`VDialog`(custom modals)
* Both`ElDialog`and`VDialog`work great for custom dialogs!

### **⚠️ Vue 3 Syntax - Important!**

**This app uses Vue 3!**&#x4D;ake sure to use Vue 3 syntax, not Vue 2. Here are the key differences:

#### **Element Plus - Vue 3 Examples**

```vue
<template>
  <!-- Card with Slots -->
  <el-card>
    <template #header>
      <!-- ✅ Vue 3: Use #header or v-slot:header -->
      {{ title }}
    </template>
    <div>{{ content }}</div>
    <div style="margin-top: 16px;">
      <el-button type="primary" @click="dialogVisible = true">
        Open Dialog
      </el-button>
    </div>
  </el-card>

  <!-- Dialog with v-model (Vue 3) -->
  <el-dialog v-model="dialogVisible" title="My Dialog" width="500px">
    <!-- ✅ Vue 3: Use v-model, not :visible.sync -->
    <p>Dialog content goes here</p>
    <template #footer>
      <!-- ✅ Vue 3: Use #footer, not slot="footer" -->
      <el-button @click="dialogVisible = false">Cancel</el-button>
      <el-button type="primary" @click="handleSubmit">Confirm</el-button>
    </template>
  </el-dialog>

  <!-- Drawer -->
  <el-drawer v-model="drawerVisible" title="Settings" direction="rtl" size="50%">
    <p>Drawer content</p>
  </el-drawer>

  <!-- Tabs -->
  <el-tabs v-model="activeTab">
    <el-tab-pane label="Tab 1" name="first">Content 1</el-tab-pane>
    <el-tab-pane label="Tab 2" name="second">Content 2</el-tab-pane>
  </el-tabs>
</template>

<script>
export default {
  data() {
    return {
      title: 'Card Title',
      content: 'Card content',
      dialogVisible: false,
      drawerVisible: false,
      activeTab: 'first'
    }
  },
  methods: {
    handleSubmit() {
      this.dialogVisible = false;
      // Your submit logic
    }
  }
}
</script>
```

#### **Vuetify - Vue 3 Examples**

```vue
<template>
  <!-- Card -->
  <v-card>
    <v-card-title>Card Title</v-card-title>
    <v-card-text>Card content</v-card-text>
    <v-card-actions>
      <v-btn color="primary" @click="dialog = true">Open Dialog</v-btn>
    </v-card-actions>
  </v-card>

  <!-- Dialog with v-model -->
  <v-dialog v-model="dialog" max-width="500px">
    <!-- ✅ Vue 3: Use v-model, not :value and @input -->
    <v-card>
      <v-card-title>Dialog Title</v-card-title>
      <v-card-text>
        Dialog content goes here
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="grey" variant="text" @click="dialog = false">
          Cancel
        </v-btn>
        <v-btn color="primary" variant="text" @click="handleConfirm">
          Confirm
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <!-- Snackbar (Toast) -->
  <v-snackbar v-model="snackbar" :timeout="3000" color="success">
    {{ snackbarText }}
    <template v-slot:actions>
      <v-btn color="white" variant="text" @click="snackbar = false">
        Close
      </v-btn>
    </template>
  </v-snackbar>

  <!-- Tabs -->
  <v-tabs v-model="tab">
    <v-tab value="one">Tab 1</v-tab>
    <v-tab value="two">Tab 2</v-tab>
  </v-tabs>
</template>

<script>
export default {
  data() {
    return {
      dialog: false,
      snackbar: false,
      snackbarText: 'Success!',
      tab: 'one'
    }
  },
  methods: {
    handleConfirm() {
      this.dialog = false;
      this.snackbarText = 'Action confirmed!';
      this.snackbar = true;
    }
  }
}
</script>
```

#### **🚫 Common Vue 2 Mistakes to Avoid**

| ❌ Vue 2 (Wrong)          | ✅ Vue 3 (Correct)                        |
| ------------------------ | ---------------------------------------- |
| `:visible.sync="dialog"` | `v-model="dialog"`                       |
| `<div slot="header">`    | `<template #header>`                     |
| `<span slot="footer">`   | `<template #footer>`                     |
| `slot-scope="{ row }"`   | `#default="{ row }"`or`v-slot="{ row }"` |
| `@hook:mounted`          | `@vue:mounted`                           |

### **🔥 Composition API Support**

**Yes!**&#x44;ynamic components fully support Vue 3's Composition API with the`setup()`function.

#### **Composition API with Options API**

```vue
<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">{{ title }}</h2>
    <el-button type="primary" @click="increment">
      Count: {{ count }}
    </el-button>
    <el-button @click="loadData" :loading="loading">
      Load Data
    </el-button>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import { ElMessage } from 'element-plus';

export default {
  setup() {
    // Reactive state
    const count = ref(0);
    const title = ref('Composition API Example');
    const loading = ref(false);
    const items = ref([]);

    // Computed properties
    const doubleCount = computed(() => count.value * 2);

    // Methods
    const increment = () => {
      count.value++;
      ElMessage.success(`Count is now ${count.value}`);
    };

    const loadData = async () => {
      loading.value = true;
      try {
        const resource = new Resource(this, 'MyResource');
        items.value = await resource.list({ limit: 10 });
        ElMessage.success('Data loaded!');
      } catch (error) {
        ElMessage.error('Failed to load data');
      } finally {
        loading.value = false;
      }
    };

    // Lifecycle hooks
    onMounted(() => {
      console.log('Component mounted!');
    });

    // Return everything you want to expose to the template
    return {
      count,
      title,
      loading,
      items,
      doubleCount,
      increment,
      loadData
    };
  }
}
</script>
```

#### **Using Composables**

```vue
<template>
  <v-card>
    <v-card-title>Responsive Design</v-card-title>
    <v-card-text>
      <p>Screen size: {{ screenSize }}</p>
      <p>Is Mobile: {{ isMobile }}</p>
      <p>Current Theme: {{ isDark ? 'Dark' : 'Light' }}</p>
    </v-card-text>
    <v-card-actions>
      <v-btn @click="toggleTheme">Toggle Theme</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import { computed } from 'vue';
import { useDisplay, useTheme } from 'vuetify';

export default {
  setup() {
    // Use Vuetify composables
    const display = useDisplay();
    const theme = useTheme();

    // Computed from composables
    const screenSize = computed(() => display.name.value);
    const isMobile = computed(() => display.mobile.value);
    const isDark = computed(() => theme.global.current.value.dark);

    const toggleTheme = () => {
      theme.global.name.value = isDark.value ? 'light' : 'dark';
    };

    return {
      screenSize,
      isMobile,
      isDark,
      toggleTheme
    };
  }
}
</script>
```

#### **Combining with Options API**

```vue
<template>
  <div>
    <p>Setup value: {{ setupValue }}</p>
    <p>Data value: {{ dataValue }}</p>
    <el-button @click="combinedMethod">Click Me</el-button>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const setupValue = ref('From setup()');
    
    return {
      setupValue
    };
  },
  data() {
    return {
      dataValue: 'From data()'
    };
  },
  methods: {
    combinedMethod() {
      // Can access both setup and data values
      console.log(this.setupValue);
      console.log(this.dataValue);
    }
  }
}
</script>
```

#### **Working with Resource API**

```vue
<template>
  <div class="p-4">
    <el-table :data="projects" v-loading="loading">
      <el-table-column prop="name" label="Project"></el-table-column>
      <el-table-column prop="status" label="Status"></el-table-column>
    </el-table>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { ElMessage } from 'element-plus';

export default {
  setup() {
    const projects = ref([]);
    const loading = ref(false);

    const fetchProjects = async () => {
      loading.value = true;
      try {
        // Note: 'this' is not available in setup()
        // For Resource API, you'll need to use it via methods or use getCurrentInstance
        const resource = new Resource(null, 'Project');
        projects.value = await resource.list({ limit: 20 });
      } catch (error) {
        ElMessage.error('Failed to fetch projects');
      } finally {
        loading.value = false;
      }
    };

    onMounted(() => {
      fetchProjects();
    });

    return {
      projects,
      loading,
      fetchProjects
    };
  }
}
</script>
```

**💡 Composition API Tips:**

* Use`ref()`for primitive values,`reactive()`for objects
* Access ref values with`.value`in script, but not in template
* All composables from Vue, Vuetify, and Element Plus work perfectly
* You can mix`setup()`with`data()`,`methods()`, etc.
* `onMounted`,`onBeforeUnmount`, etc. replace lifecycle hooks

**⚠️ Note:**`<script setup>`syntax is NOT currently supported. Use the`setup()`function inside`export default`instead.

## **📝 FormField Component**

The`FormField`component is available in dynamic components. It provides a unified way to render various form input types based on property configuration.

**✨ No Import Required!**&#x54;he`FormField`component is automatically available in all dynamic components. Just use it directly in your template!

### **Basic Usage**

```vue
<template>
  <div class="p-4">
    <FormField
      v-model="formData.name"
      :property="nameProperty"
      :form="formData"
      size="default"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      formData: {
        name: ''
      },
      nameProperty: {
        property: 'name',
        label: 'Name',
        type: 'string',
        input_properties: {
          type: 'input'
        }
      }
    };
  }
};
</script>
```

### **Props Reference**

| **Prop**         | **Type** | **Default** | **Description**                               |
| ---------------- | -------- | ----------- | --------------------------------------------- |
| `modelValue`     | Any      | -           | The v-model value for the field               |
| `property`       | Object   | {}          | Property configuration object (see below)     |
| `form`           | Object   | {}          | The parent form data object                   |
| `resource`       | Object   | {}          | Resource context                              |
| `resourceDetail` | Object   | {}          | Resource detail configuration                 |
| `size`           | String   | "default"   | Input size: "small", "default", "large"       |
| `setting`        | String   | "form"      | Form setting: "form", "form-add", "form-edit" |
| `layout`         | String   | "column"    | Layout direction                              |

### **Property Configuration**

The`property`prop is the key configuration object that determines what type of input is rendered.

```vue
{
  property: 'field_name',           // Field identifier
  label: 'Field Label',             // Display label
  type: 'string',                   // Data type: string, number, boolean, date, array, object
  input_properties: {
    type: 'input'                   // Input type (see below)
  },
  // Optional: for select/lookup fields
  map: {
    type: 'model',
    resource: 'ResourceName'
  },
  lookup_type: 'lookup'             // For dropdown fields
}
```

### **Supported Input Types**

| **input\_properties.type** | **Description**                               |
| -------------------------- | --------------------------------------------- |
| `input`                    | Standard text input (default for string type) |
| `textarea`                 | Multi-line text area                          |
| `richtexteditor`           | Rich text editor with formatting              |
| `html`                     | HTML editor                                   |
| `code_editor`              | Monaco-based code editor for string fields    |
| `colorpicker`              | Color picker                                  |
| `signature`                | Signature pad                                 |
| `time`                     | Time picker                                   |
| `check-box`                | Checkbox                                      |
| `switch`                   | Toggle switch (also used for boolean type)    |
| `file_upload`              | File upload                                   |
| `typed_file_upload`        | Typed file upload with categories             |
| `dynamic-component`        | Nested dynamic component                      |

### **Examples**

#### **Text Input**

```
<template>
  <FormField
    v-model="formData.title"
    :property="titleProperty"
    :form="formData"
  />
</template>

<script>
export default {
  data() {
    return {
      formData: { title: '' },
      titleProperty: {
        property: 'title',
        label: 'Title',
        type: 'string',
        input_properties: { type: 'input' }
      }
    };
  }
};
</script>
```

#### **Number Input**

```
<FormField
  v-model="formData.quantity"
  :property="{
    property: 'quantity',
    label: 'Quantity',
    type: 'number'
  }"
  :form="formData"
/>
```

#### **Text Area**

```
<FormField
  v-model="formData.description"
  :property="{
    property: 'description',
    label: 'Description',
    type: 'string',
    input_properties: { type: 'textarea' }
  }"
  :form="formData"
/>
```

#### **Code Editor**

```
<FormField
  v-model="formData.template"
  :property="{
    property: 'template',
    label: 'Template',
    type: 'string',
    input_properties: { type: 'code_editor' }
  }"
  :form="formData"
/>
```

#### **Switch/Boolean**

```
<FormField
  v-model="formData.active"
  :property="{
    property: 'active',
    label: 'Active',
    type: 'boolean'
  }"
  :form="formData"
/>
```

#### **Date Picker**

```
<FormField
  v-model="formData.dueDate"
  :property="{
    property: 'dueDate',
    label: 'Due Date',
    type: 'date'
  }"
  :form="formData"
/>
```

#### **Color Picker**

```
<FormField
  v-model="formData.color"
  :property="{
    property: 'color',
    label: 'Color',
    type: 'string',
    input_properties: { type: 'colorpicker' }
  }"
  :form="formData"
/>
```

#### **Select with Static Options**

```
<FormField
  v-model="formData.status"
  :property="{
    property: 'status',
    label: 'Status',
    type: 'string',
    lookup_type: 'values',
    map: {
      values: [
        { value: 'pending', label: 'Pending' },
        { value: 'active', label: 'Active' },
        { value: 'completed', label: 'Completed' }
      ]
    }
  }"
  :form="formData"
/>
```

### **Complete Form Example**

```
<template>
  <div class="p-4">
    <el-card>
      <template #header>
        <span class="font-bold text-lg">User Profile</span>
      </template>
      
      <el-form label-position="top">
        <el-form-item label="Full Name">
          <FormField
            v-model="formData.name"
            :property="properties.name"
            :form="formData"
          />
        </el-form-item>
        
        <el-form-item label="Email">
          <FormField
            v-model="formData.email"
            :property="properties.email"
            :form="formData"
          />
        </el-form-item>
        
        <el-form-item label="Bio">
          <FormField
            v-model="formData.bio"
            :property="properties.bio"
            :form="formData"
          />
        </el-form-item>
        
        <el-form-item label="Active">
          <FormField
            v-model="formData.active"
            :property="properties.active"
            :form="formData"
          />
        </el-form-item>
        
        <el-form-item>
          <el-button type="primary" @click="handleSubmit">
            Save Profile
          </el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import { ElMessage } from 'element-plus';

export default {
  data() {
    return {
      formData: {
        name: '',
        email: '',
        bio: '',
        active: true
      },
      properties: {
        name: {
          property: 'name',
          label: 'Full Name',
          type: 'string',
          input_properties: { type: 'input' }
        },
        email: {
          property: 'email',
          label: 'Email',
          type: 'string',
          input_properties: { type: 'input' }
        },
        bio: {
          property: 'bio',
          label: 'Bio',
          type: 'string',
          input_properties: { type: 'textarea' }
        },
        active: {
          property: 'active',
          label: 'Active',
          type: 'boolean'
        }
      }
    };
  },
  methods: {
    async handleSubmit() {
      try {
        const resource = new Resource(this, 'Users');
        await resource.create(this.formData);
        ElMessage.success('Profile saved successfully!');
      } catch (error) {
        ElMessage.error('Failed to save profile');
      }
    }
  }
};
</script>
```

### **Events**

| **Event**           | **Payload**        | **Description**                |
| ------------------- | ------------------ | ------------------------------ |
| `update:modelValue` | { property, data } | Emitted when value changes     |
| `onInput`           | { property, data } | Emitted on input               |
| `onBlur`            | -                  | Emitted when field loses focus |
| `uploads`           | { file, details }  | Emitted for file uploads       |

**💡 Tips:**

* Always pass the`form`prop to enable cross-field dependencies
* Use`type: 'boolean'`for automatic switch rendering
* Use`type: 'number'`for numeric input with validation
* Use`type: 'date'`for date picker functionality
* For dropdowns, set`lookup_type`and provide values via`map`

## **🧾 ResourceFormDialog Component**

The`ResourceFormDialog`component is available in dynamic components. It renders Saturn’s standard resource create/edit dialog and is typically opened imperatively via a ref.

**✨ No Import Required!**`ResourceFormDialog`is automatically available in all dynamic components.

### **Basic Usage**

```
<template>
  <div class="p-4">
    <el-button type="primary" @click="openCreate">New</el-button>

    <ResourceFormDialog
      ref="rfd"
      :object="resourceClass"
      :fields="customFields"
      :resourceDetail="resourceDetail"
      :sections="resourceDetail.sections || []"
      :openOnCreate="false"
      :formFillerEnabled="false"
      :isInline="false"
      :isSmallTable="false"
      :parent_data="parentData"
      :property="boundedProperty"
      :setting="setting"
      :interceptor="interceptor"
      :createFunction="createFunction"
      :updateFunction="updateFunction"
      @created="onCreated"
      @updated="onUpdated"
      @closed="onClosed"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      // IMPORTANT:
      // - object must be a Saturn resource “class” (the same kind used elsewhere in the app)
      // - it should have: type, props, and usually create(form)/update(id, changes)
      resourceClass: null,
      customFields: [],
      resourceDetail: { sections: [] },
      parentData: null,
      boundedProperty: null,
      setting: {},
    };
  },
  methods: {
    openCreate() {
      this.$refs.rfd.open({ mode: "add", data: {} });
    },
    openEdit(row) {
      // Optimistic loading: pass available data immediately, fetch fresh in background
      this.$refs.rfd.open({
        mode: "edit",
        data: row,
        fetchFresh: () => this.resourceClass.get(row.id)
      });
    },
    openSingleField(row) {
      this.$refs.rfd.open({
        mode: "edit",
        data: row,
        property: "status",
        displaySection: false,
      });
    },
    onCreated(created) {},
    onUpdated(updated) {},
    onClosed() {},
  },
};
</script>
```

### **Props Reference**

| **Prop**            | **Type** | **Default** | **Description**                                                                                                                                                                          |
| ------------------- | -------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `object`            | Object   | -           | **Required.**&#x53;aturn resource "class" (must provide`type`,`props`, and usually`create(form)`/`update(id, changes)`).                                                                 |
| `fields`            | Array    | undefined   | Custom form fields array. When provided, overrides`object.props`for determining which fields to display.                                                                                 |
| `resourceDetail`    | Object   | {}          | Rendering/layout configuration (section layout, step titles, restrictions, etc.). Also supports global form component override via`resourceDetail.render_details.form_component`.        |
| `sections`          | Array    | \[]         | Section definitions used by wizard/sections layout.                                                                                                                                      |
| `openOnCreate`      | Boolean  | false       | If true, navigates to the created resource after a successful create.                                                                                                                    |
| `formFillerEnabled` | Boolean  | false       | Enables the form prefiller UI and related behavior.                                                                                                                                      |
| `property`          | Object   | -           | Optional “bounded property” context (used for certain dynamic lookup relationships).                                                                                                     |
| `parent_data`       | Object   | -           | Optional parent record used when creating bounded/related items.                                                                                                                         |
| `isSmallTable`      | Boolean  | false       | Adjusts create behavior for small table contexts (reload/add-another behavior).                                                                                                          |
| `isInline`          | Boolean  | false       | Marks the dialog as used in an inline resource manager context (affects resets/local storage behaviors).                                                                                 |
| `setting`           | Object   | {}          | Optional context object passed through lookup/prefill logic. You can usually omit it.                                                                                                    |
| `interceptor`       | Function | -           | Optional hook to transform data before persistence. Signature:`(formOrUpdateForm) => nextFormOrUpdateForm`.                                                                              |
| `createFunction`    | Function | -           | Optional override for creating instead of`object.create(form)`. Signature:`(form) => created`.**Note:**&#x77;hen provided, Saturn’s default inline-manager + upload handling is skipped. |
| `updateFunction`    | Function | -           | Optional override for updating instead of`object.update(id, changes)`. Signature:`(form) => updated`.                                                                                    |

### **Interceptor Hook (pre-save transform)**

`interceptor`lets you transform data**right before**the dialog persists it. It’s useful for trimming strings, normalizing values, enforcing formats, or removing UI-only fields.

**⚠️ Important behavior:**

* On**create**(`mode: "add"`), the interceptor receives the**full form**and must return the next full form.
* On**update**(`mode: "edit"`), the interceptor receives a**patch object**(only the changed fields) and must return the next patch object.
* If you pass a custom`updateFunction`, the default update path may bypass the interceptor’s patch. In that case, apply your normalization inside`updateFunction`too.

#### **Example: trim names + convert empty strings to null**

```
<template>
  <ResourceFormDialog
    ref="rfd"
    :object="resourceClass"
    :interceptor="interceptor"
    @created="onCreated"
    @updated="onUpdated"
  />
</template>

<script>
export default {
  methods: {
    interceptor(payload) {
      // payload is:
      // - FULL form on create
      // - PATCH (changed fields only) on update
      const next = { ...payload };

      if (typeof next.first_name === "string") next.first_name = next.first_name.trim();
      if (typeof next.last_name === "string") next.last_name = next.last_name.trim();

      // normalize clears (often helpful on update)
      Object.keys(next).forEach((k) => {
        if (next[k] === "") next[k] = null;
      });

      return next;
    },

    openCreate() {
      this.$refs.rfd.open({ mode: "add", data: {} });
    },

    openEdit(row) {
      this.$refs.rfd.open({ mode: "edit", data: row });
    },

    onCreated(created) {},
    onUpdated(updated) {},
  },
};
</script>
```

### **open(...) API**

The`open()`method accepts an options object with the following fields:

#### **Basic Options**

| **Field**        | **Type** | **Description**                                                                                    |
| ---------------- | -------- | -------------------------------------------------------------------------------------------------- |
| `mode`           | string   | `"add"`or`"edit"`                                                                                  |
| `data`           | object   | Existing record data (used for edit mode)                                                          |
| `property`       | string   | Optional: open only a single field by property name                                                |
| `displaySection` | boolean  | Whether to show section UI when opening a single field                                             |
| `fetchFresh`     | function | Async function to fetch fresh data in background (optimistic loading). Example:`() => api.get(id)` |

#### **Dynamic Configuration (Override Props)**

These parameters allow you to dynamically override component props when opening the dialog:

| **Field**              | **Type** | **Description**                                      |
| ---------------------- | -------- | ---------------------------------------------------- |
| `fields`               | array    | Custom fields array (overrides`object.props`)        |
| `sections`             | array    | Custom sections array                                |
| `sectionLayout`        | string   | `"wizard"`or`"sections"`                             |
| `stepOneTitle`         | string   | Title for the first step/section                     |
| `stepOneDescription`   | string   | Description for the first step/section               |
| `numberOfFormColumns`  | number   | Number of form columns (1-4)                         |
| `resourceLabel`        | string   | Custom label for the resource (used in dialog title) |
| `wizardStepNavigation` | boolean  | Whether to show wizard step navigation               |
| `fieldRestrictions`    | array    | Field restriction rules                              |

#### **Dynamic Configuration Example**

```
// Open with dynamic configuration
this.$refs.rfd.open({
  mode: 'add',
  data: {},
  // Dynamic overrides
  fields: customFields,
  sections: customSections,
  sectionLayout: 'wizard',
  stepOneTitle: 'Get Started',
  numberOfFormColumns: 2,
  resourceLabel: 'Custom Form'
});
```

**💡 Note:**&#x44;ynamic configuration values passed to`open()`take precedence over component props. They are reset when the dialog closes.

### **Events**

| **Event** | **Payload**             | **Description**                    |
| --------- | ----------------------- | ---------------------------------- |
| `created` | created record (object) | Emitted after a successful create. |
| `updated` | updated record (object) | Emitted after a successful update. |
| `closed`  | -                       | Emitted when the dialog closes.    |

### **Notes**

* **Field "setting"**: inside the dialog, fields run in`"form-add"`/`"form-edit"`mode (for defaults/visibility) regardless of the dialog-level`setting`object.
* **Single-field edit**:`open({ property: "someField", displaySection: false })`can be used to edit just one field.

### **Resource Form Component Override**

Resource settings can override which Saturn UI component opens for create/update dialogs:

```
resourceDetail.render_details.form_component = {
  default: "MySaturnForm",
  create: "MySaturnCreateForm",
  update: "MySaturnUpdateForm"
};
```

* `add`mode resolves to`create`
* `edit`/`update`mode resolves to`update`
* If only`default`is set, it is used for both create and update
* These values must be Saturn UI Component names
* System component mapping remains controlled by manifest (`resource_component_map`)

### **Individual Override Props**

ResourceFormDialog also supports individual props that override`resourceDetail`values:

| **Prop**               | **Type** | **Overrides**                                          |
| ---------------------- | -------- | ------------------------------------------------------ |
| `sectionLayout`        | String   | `resourceDetail.render_details.section_layout`         |
| `stepOneTitle`         | String   | `resourceDetail.render_details.step_one_title`         |
| `stepOneDescription`   | String   | `resourceDetail.render_details.step_one_description`   |
| `numberOfFormColumns`  | Number   | `resourceDetail.render_details.numberOfFormColumns`    |
| `resourceLabel`        | String   | `resourceDetail.render_details.resource_label`         |
| `wizardStepNavigation` | Boolean  | `resourceDetail.render_details.wizard_step_navigation` |
| `fieldRestrictions`    | Array    | `resourceDetail.field_restrictions`                    |

These props take precedence over the corresponding`resourceDetail`values when both are provided.

## **📋 ResourceFormInline Component**

The`ResourceFormInline`component renders Saturn's resource create/edit form inline (without a dialog wrapper). This is useful when you want to embed a form directly in a page or within a dynamic component. It also supports`v-model`for external draft-state control.

**✨ No Import Required!**`ResourceFormInline`is automatically available in all dynamic components.

### **Basic Usage**

```
<template>
  <div class="p-4">
    <ResourceFormInline
      ref="inlineForm"
      v-model="draftForm"
      :object="resourceClass"
      :fields="customFields"
      :resourceDetail="resourceDetail"
      :sections="resourceDetail.sections || []"
      :show-actions="true"
      :show-cancel="true"
      :auto-init="false"
      @created="onCreated"
      @updated="onUpdated"
      @closed="onClosed"
      @cancelled="onCancelled"
    />

    <el-button @click="openCreateForm">Create New</el-button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      draftForm: {},
      resourceClass: null,
      customFields: [],
      resourceDetail: { sections: [] },
    };
  },
  methods: {
    openCreateForm() {
      this.$refs.inlineForm.open({ mode: "add", data: {} });
    },
    openEditForm(row) {
      this.$refs.inlineForm.open({ mode: "edit", data: row });
    },
    onCreated(created) {
      console.log('Created:', created);
    },
    onUpdated(updated) {
      console.log('Updated:', updated);
    },
    onClosed() {
      console.log('Form closed');
    },
    onCancelled() {
      console.log('Form cancelled');
    }
  }
};
</script>
```

### **Props Reference**

ResourceFormInline accepts all the same props as ResourceFormDialog, plus additional props for inline behavior,`v-model`, and individual overrides for`resourceDetail`values.

#### **Core Props (same as ResourceFormDialog)**

| **Prop**         | **Type** | **Default** | **Description**                                                                                                                                                                    |
| ---------------- | -------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `object`         | Object   | -           | **Required.**&#x53;aturn resource "class" (must provide`type`,`props`, and usually`create(form)`/`update(id, changes)`).                                                           |
| `fields`         | Array    | undefined   | Custom form fields array. When provided, overrides`object.props`for determining which fields to display.**Note:**&#x57;hen`fields`is provided, the form auto-initializes on mount. |
| `resourceDetail` | Object   | {}          | Optional rendering/layout configuration, including`render_details.form_component`for resource form component selection.                                                            |
| `sections`       | Array    | \[]         | Section definitions used by wizard/sections layout.                                                                                                                                |
| `parent_data`    | Object   | -           | Optional parent record used when creating bounded/related items.                                                                                                                   |
| `interceptor`    | Function | -           | Optional hook to transform data before persistence.                                                                                                                                |
| `createFunction` | Function | -           | Optional override for creating instead of`object.create(form)`.                                                                                                                    |
| `updateFunction` | Function | -           | Optional override for updating instead of`object.update(id, changes)`.                                                                                                             |

#### **Inline-Specific Props**

| **Prop**         | **Type** | **Default** | **Description**                                                                          |
| ---------------- | -------- | ----------- | ---------------------------------------------------------------------------------------- |
| `modelValue`     | Object   | undefined   | **v-model.**&#x52;eceives live form edits and can seed initial form data.                |
| `showActions`    | Boolean  | true        | Whether to show the Save/Cancel/Back/Next buttons.                                       |
| `showCancel`     | Boolean  | true        | Whether to show the Cancel button.                                                       |
| `showAddAnother` | Boolean  | true        | Whether to show the "Add Another" checkbox in add mode.                                  |
| `saveLabel`      | String   | "Save"      | Custom label for the Save button.                                                        |
| `cancelLabel`    | String   | "Cancel"    | Custom label for the Cancel button.                                                      |
| `autoInit`       | Boolean  | false       | If true, automatically initializes the form on mount using`initialMode`and`initialData`. |
| `initialMode`    | String   | "add"       | Initial mode when`autoInit`is true. Either "add" or "edit".                              |
| `initialData`    | Object   | {}          | Initial data when`autoInit`is true.                                                      |

#### **Individual Override Props**

These props allow you to override specific`resourceDetail`values without needing to pass the full object:

| **Prop**               | **Type** | **Overrides**                                                          |
| ---------------------- | -------- | ---------------------------------------------------------------------- |
| `sectionLayout`        | String   | `resourceDetail.render_details.section_layout`("wizard" or "sections") |
| `stepOneTitle`         | String   | `resourceDetail.render_details.step_one_title`                         |
| `stepOneDescription`   | String   | `resourceDetail.render_details.step_one_description`                   |
| `numberOfFormColumns`  | Number   | `resourceDetail.render_details.numberOfFormColumns`                    |
| `resourceLabel`        | String   | `resourceDetail.render_details.resource_label`                         |
| `wizardStepNavigation` | Boolean  | `resourceDetail.render_details.wizard_step_navigation`                 |
| `fieldRestrictions`    | Array    | `resourceDetail.field_restrictions`                                    |

### **Usage Patterns**

#### **External save button (no built-in actions)**

```
<template>
  <div>
    <ResourceFormInline
      ref="inlineForm"
      v-model="draftForm"
      :object="resourceObject"
      :fields="fields"
      :sections="sections"
      :show-actions="false"
      :auto-init="true"
      initial-mode="edit"
    />

    <el-button type="primary" @click="saveFromParent">Save</el-button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      draftForm: {},
      resourceObject: { type: 'MyResource' },
      fields: [],
      sections: []
    };
  },
  methods: {
    saveFromParent() {
      this.$refs.inlineForm.saveForm();
    }
  }
};
</script>
```

#### **With resourceDetail (standard usage)**

```
<ResourceFormInline
  :object="resourceObject"
  :resource-detail="resourceDetail"
  :sections="sections"
  :data="existingData"
  @created="onCreated"
/>
```

#### **With individual props (no resourceDetail needed)**

```
<ResourceFormInline
  :object="resourceObject"
  :sections="sections"
  section-layout="sections"
  step-one-title="Basic Information"
  :number-of-form-columns="1"
  @created="onCreated"
/>
```

#### **With resourceDetail + overrides**

```
<ResourceFormInline
  :object="resourceObject"
  :resource-detail="resourceDetail"
  :sections="sections"
  :number-of-form-columns="1"
  step-one-title="Custom Title"
  @created="onCreated"
/>
```

#### **Auto-initialize on mount**

```
<ResourceFormInline
  :object="resourceObject"
  :sections="sections"
  :auto-init="true"
  initial-mode="add"
  :initial-data="{}"
  @created="onCreated"
/>
```

#### **With custom fields (overrides object.props)**

```
<template>
  <ResourceFormInline
    :object="resourceObject"
    :fields="customFields"
    :sections="sections"
    :auto-init="true"
    @created="onCreated"
  />
</template>

<script>
export default {
  data() {
    return {
      resourceObject: { type: 'MyResource' },
      customFields: [
        {
          property: 'name',
          label: 'Name',
          type: 'string',
          required: true,
          input_properties: { type: 'input' }
        },
        {
          property: 'email',
          label: 'Email',
          type: 'string',
          email: true,
          input_properties: { type: 'input' }
        },
        {
          property: 'status',
          label: 'Status',
          type: 'string',
          lookup_type: 'values',
          map: {
            values: [
              { value: 'active', label: 'Active' },
              { value: 'inactive', label: 'Inactive' }
            ]
          }
        }
      ],
      sections: []
    };
  },
  methods: {
    onCreated(created) {
      console.log('Created:', created);
    }
  }
};
</script>
```

### **open(...) API**

The`open()`method accepts an options object with the following fields:

#### **Basic Options**

| **Field**        | **Type** | **Description**                                                                       |
| ---------------- | -------- | ------------------------------------------------------------------------------------- |
| `mode`           | string   | `"add"`or`"edit"`                                                                     |
| `data`           | object   | Existing record data (used for edit mode). If omitted, current`v-model`value is used. |
| `property`       | string   | Optional: open only a single field by property name                                   |
| `displaySection` | boolean  | Whether to show section UI when opening a single field                                |

#### **Dynamic Configuration (Override Props)**

These parameters allow you to dynamically override component props when calling`open()`:

| **Field**              | **Type** | **Description**                               |
| ---------------------- | -------- | --------------------------------------------- |
| `fields`               | array    | Custom fields array (overrides`object.props`) |
| `sections`             | array    | Custom sections array                         |
| `sectionLayout`        | string   | `"wizard"`or`"sections"`                      |
| `stepOneTitle`         | string   | Title for the first step/section              |
| `stepOneDescription`   | string   | Description for the first step/section        |
| `numberOfFormColumns`  | number   | Number of form columns (1-4)                  |
| `resourceLabel`        | string   | Custom label for the resource                 |
| `wizardStepNavigation` | boolean  | Whether to show wizard step navigation        |
| `fieldRestrictions`    | array    | Field restriction rules                       |

#### **Dynamic Configuration Example**

```
// Open inline form with dynamic configuration
this.$refs.inlineForm.open({
  mode: 'add',
  data: {},
  // Dynamic overrides
  fields: [
    { key: 'name', label: 'Name', type: 'string', required: true },
    { key: 'email', label: 'Email', type: 'string', email: true }
  ],
  sections: [
    { name: 'Contact Info', properties: ['name', 'email'] }
  ],
  sectionLayout: 'sections',
  numberOfFormColumns: 1
});
```

**💡 Note:**&#x44;ynamic configuration values passed to`open()`take precedence over component props. They are reset when`close()`is called.

### **Events**

| **Event**           | **Payload**             | **Description**                                 |
| ------------------- | ----------------------- | ----------------------------------------------- |
| `created`           | created record (object) | Emitted after a successful create.              |
| `updated`           | updated record (object) | Emitted after a successful update.              |
| `closed`            | -                       | Emitted when the form is closed/reset.          |
| `cancelled`         | -                       | Emitted when the user clicks Cancel.            |
| `form-ready`        | { form, formRef }       | Emitted when the form is initialized and ready. |
| `update:modelValue` | current form object     | Emitted on form changes for`v-model`syncing.    |

### **Exposed Methods**

| **Method**                                                   | **Description**                                                                                 |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `open({ mode, data, property, displaySection, fetchFresh })` | Initialize and open the form. Optionally provide`fetchFresh`for optimistic loading (see below). |
| `close()`                                                    | Close and reset the form.                                                                       |
| `saveForm()`                                                 | Programmatically trigger form submission.                                                       |
| `getFormData()`                                              | Get the current form data.                                                                      |
| `getFormRef()`                                               | Get the Element Plus form ref for validation.                                                   |
| `validate(callback)`                                         | Validate the form.                                                                              |

### **Optimistic Loading with****`fetchFresh`**

When editing large records, fetching all data before opening the dialog can cause noticeable delays. Use`fetchFresh`to open the dialog immediately with available data, then refresh in the background.

**✨ How it works:**

* Dialog opens instantly with the`data`you provide
* A small "Syncing..." indicator appears in the header
* Background fetch refreshes data without blocking the UI
* Fields the user has already touched are not overwritten
* If the fetch fails, an inline error with retry appears

#### **Usage Example**

```
// OLD: Wait for fetch, then open (slow)
async handleEdit(row) {
  const fullData = await this.resourceClass.get(row.id);
  this.$refs.form.open({ mode: "edit", data: fullData });
}

// NEW: Open immediately, fetch in background (fast)
async handleEdit(row) {
  this.$refs.form.open({
    mode: "edit",
    data: row,  // available data (from table/grid)
    fetchFresh: () => this.resourceClass.get(row.id)  // background refresh
  });
}
```

#### **open() Options Reference**

| **Option**       | **Type** | **Required** | **Description**                                                                                           |
| ---------------- | -------- | ------------ | --------------------------------------------------------------------------------------------------------- |
| `mode`           | String   | Yes          | `"add"`for create,`"edit"`for update                                                                      |
| `data`           | Object   | Yes          | Record data. For edit, pass available fields; missing fields will be fetched.                             |
| `property`       | String   | No           | Single field edit mode (shows only this field).                                                           |
| `displaySection` | Boolean  | No           | Show/hide section headers. Default:`true`                                                                 |
| `fetchFresh`     | Function | No           | Async function that returns fresh record data. Only used in edit mode. Example:`() => api.get(record.id)` |

**💡 When to use ResourceFormInline vs ResourceFormDialog:**

* Use**ResourceFormDialog**when you want a modal popup experience.
* Use**ResourceFormInline**when you want to embed the form directly in the page (e.g., in a card, panel, or dynamic component).
* Both components share the same underlying form logic and support the same features.

## **👁️ ResourceViewInline Component**

The`ResourceViewInline`component renders resource data in a read-only inline details layout. It is based on the same details/sections rendering approach as`ResourceView`/`ResourceDetails`, but intentionally excludes workflow actions, tabs, and inline edit controls.

**✨ No Import Required!**`ResourceViewInline`is automatically available in all dynamic components.

### **Basic Usage**

```
<template>
  <div class="p-4">
    <ResourceViewInline
      v-model="record"
      :fields="fields"
      :sections="sections"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      record: {
        id: 'dep-1',
        first_name: 'Ava',
        last_name: 'Stone',
        status: 'active',
        documents: []
      },
      fields: [
        { property: 'first_name', label: 'First Name', type: 'string' },
        { property: 'last_name', label: 'Last Name', type: 'string' },
        { property: 'status', label: 'Status', type: 'string' },
        {
          property: 'documents',
          label: 'Documents',
          type: 'array',
          input_properties: { type: 'file_upload' }
        }
      ],
      sections: [
        { name: 'Profile', properties: ['first_name', 'last_name', 'status'] },
        { name: 'Files', properties: ['documents'] }
      ]
    };
  }
};
</script>
```

### **Props Reference**

| **Prop**     | **Type** | **Required** | **Description**                                                              |
| ------------ | -------- | ------------ | ---------------------------------------------------------------------------- |
| `modelValue` | Object   | Yes          | **v-model.**&#x54;he resource record to render.                              |
| `fields`     | Array    | Yes          | Field/property schema used for labels, types, mappings, and render metadata. |
| `sections`   | Array    | Yes          | Section definitions used to group fields in the inline details view.         |

### **Notes**

* **Read-only by design:**&#x74;his component does not show section edit/save/cancel controls.
* **No workflow action UI:**&#x73;tatus action buttons/dropdowns are not rendered.
* **No tabs:**&#x61;ll rendered content is inline in a single details flow.
* **v-model compatibility:**`update:modelValue`is declared for compatibility, but this view does not emit updates.

## **🗺️ Map View Templates**

Saturn map views support configurable marker/hover/click rendering. For**marker**,**hover**, and**click**you can use a single template mode:`custom_code`. Map`custom_code`templates are rendered through`InlineComponent`.

### **Template Context**

Map templates receive record context through`item`.

* `item`: current resource row for the selected marker.
* `resourceDetail`: current resource detail metadata when available.
* `property`: mapped property context (for inline/mapped usage).

### **Custom Code Template Example**

```
<template>
  <div class="space-y-1 p-2 rounded bg-slate-50 border border-slate-200">
    <div class="font-semibold">{{ item.name || item.title || item.id }}</div>
    <div class="text-xs text-slate-500">{{ item.status || item.description || 'No details' }}</div>
  </div>
</template>
```

### **Click Action Behavior**

* `show_data`: opens the in-map popup card using field/template content.
* `navigate`: opens Saturn resource view for the clicked record.
* `open_drawer`: opens Saturn drawer/dialog for the clicked record.

## **📅 Calendar Event Templates**

Calendar view now supports custom event card templates (similar to Kanban card templates). Enable`Use Custom Template`in the Calendar view config under**Event Card**.

### **Color Method Configuration**

Calendar event color uses an explicit`Color Method`selector:`field`or`color_function`.

| **Field**        | **Type**          | **Description**                                                  |
| ---------------- | ----------------- | ---------------------------------------------------------------- |
| `color_method`   | String            | `field`or`color_function`. Legacy views default to`field`.       |
| `color`          | String            | Property key used when`color_method = field`.                    |
| `color_function` | String (Function) | Custom function source used when`color_method = color_function`. |

Color function signature:`(property_value, obj, ctx) => string`

`ctx`contains:`{ _, moment, user, _user, view, resource, mode }`. If function output is invalid, calendar falls back to the default brand color.

```
(property_value, obj, ctx) => {
  if (obj?.priority === "high") return "#dc2626";
  if (typeof property_value === "string" && property_value.trim() !== "") {
    return property_value;
  }
  return "var(--brand-primary-color)";
}
```

### **Template Props**

| **Prop**    | **Type** | **Description**                                                                  |
| ----------- | -------- | -------------------------------------------------------------------------------- |
| `event`     | Object   | Mapped calendar event object used for rendering.                                 |
| `item`      | Object   | Original resource record (raw API item).                                         |
| `title`     | String   | Resolved event title (name field fallback: record id).                           |
| `timeLabel` | String   | Formatted time label, including end time when present.                           |
| `showTime`  | Boolean  | True only when start has an explicit time value.                                 |
| `color`     | String   | Resolved event color based on`color_method`(field/function) with brand fallback. |
| `resource`  | String   | Current resource type.                                                           |
| `view`      | Object   | Current view definition object.                                                  |

### **Starter Template Example**

```
<template>
  <div class="w-full rounded-md border border-white/25 px-2 py-1 text-white">
    <div class="truncate text-xs font-semibold">{{ title }}</div>
    <div v-if="showTime && timeLabel" class="text-[11px] opacity-90">
      {{ timeLabel }}
    </div>
    <div class="truncate text-[10px] opacity-80">#{{ item?.id }}</div>
  </div>
</template>

<script>
export default {
  props: {
    event: { type: Object, default: () => ({}) },
    item: { type: Object, default: () => ({}) },
    title: { type: String, default: "" },
    timeLabel: { type: String, default: "" },
    showTime: { type: Boolean, default: false },
    color: { type: String, default: "" },
    resource: { type: String, default: "" },
    view: { type: Object, default: () => ({}) }
  }
}
</script>
```

## **📑 SectionManager Component**

The`SectionManager`component is available in dynamic components. It provides a complete UI for managing form sections with drag-and-drop reordering, section layout options (wizard or sections), and a live preview.

**✨ No Import Required!**`SectionManager`is automatically available in all dynamic components.

### **Basic Usage**

```
<template>
  <div class="p-4">
    <SectionManager
      v-model:sections="sections"
      v-model:sectionLayout="sectionLayout"
      v-model:stepOneTitle="stepOneTitle"
      :properties="properties"
      @update:sections="onSectionsChange"
      @update:sectionLayout="onLayoutChange"
      @update:stepOneTitle="onTitleChange"
      @afterUpdate="onAfterUpdate"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      sections: [
        {
          name: 'Personal Info',
          properties: ['first_name', 'last_name', 'email']
        },
        {
          name: 'Address',
          properties: ['street', 'city', 'state', 'zip']
        }
      ],
      sectionLayout: 'wizard', // 'wizard' or 'sections'
      stepOneTitle: 'Get Started',
      properties: [
        { key: 'first_name', label: 'First Name' },
        { key: 'last_name', label: 'Last Name' },
        { key: 'email', label: 'Email' },
        { key: 'street', label: 'Street' },
        { key: 'city', label: 'City' },
        { key: 'state', label: 'State' },
        { key: 'zip', label: 'ZIP Code' }
      ]
    };
  },
  methods: {
    onSectionsChange(sections) {
      console.log('Sections updated:', sections);
    },
    onLayoutChange(layout) {
      console.log('Layout changed:', layout);
    },
    onTitleChange(title) {
      console.log('Step one title changed:', title);
    },
    onAfterUpdate(value) {
      console.log('After update:', value);
    }
  }
};
</script>
```

### **Props Reference**

| **Prop**        | **Type** | **Default** | **Description**                                                                                              |
| --------------- | -------- | ----------- | ------------------------------------------------------------------------------------------------------------ |
| `sections`      | Array    | \[]         | **v-model.**&#x54;he sections array. Each section has`name`(string) and`properties`(array of property keys). |
| `properties`    | Array    | *Required*  | Array of resource property objects. Each should have at least`key`and optionally`label`.                     |
| `sectionLayout` | String   | "sections"  | **v-model.**&#x4C;ayout type:`"sections"`(stacked) or`"wizard"`(step-by-step).                               |
| `stepOneTitle`  | String   | ""          | **v-model.**&#x54;itle for the first step/section (defaults to "Get Started" in the preview).                |
| `hideAddEdit`   | Boolean  | false       | When true, hides the internal AddEditSection dialog (for external control scenarios).                        |

### **Events**

| **Event**              | **Payload** | **Description**                                                       |
| ---------------------- | ----------- | --------------------------------------------------------------------- |
| `update:sections`      | Array       | Emitted when sections are added, edited, deleted, or reordered.       |
| `update:sectionLayout` | String      | Emitted when the layout type changes.                                 |
| `update:stepOneTitle`  | String      | Emitted when the step one title changes.                              |
| `afterUpdate`          | Any         | Emitted after section updates complete (useful for triggering saves). |

### **Exposed Methods**

You can call these methods via a template ref:

| **Method**                        | **Description**                                                |
| --------------------------------- | -------------------------------------------------------------- |
| `openAddSection()`                | Programmatically opens the "Add Section" dialog.               |
| `openEditSection(section, index)` | Programmatically opens the edit dialog for a specific section. |

### **Section Object Structure**

```
{
  name: "Section Name",           // Display name for the section
  properties: ["field1", "field2"] // Array of property keys to include
}
```

### **Complete Example with Resource Integration**

```
<template>
  <div class="p-6">
    <el-card>
      <template #header>
        <div class="flex justify-between items-center">
          <span class="font-bold">Form Builder</span>
          <el-button type="primary" size="small" @click="saveConfiguration">
            Save Configuration
          </el-button>
        </div>
      </template>
      
      <SectionManager
        ref="sectionManager"
        v-model:sections="formConfig.sections"
        v-model:sectionLayout="formConfig.layout"
        v-model:stepOneTitle="formConfig.stepOneTitle"
        :properties="resourceProperties"
        @update:sections="markDirty"
        @update:sectionLayout="markDirty"
        @update:stepOneTitle="markDirty"
      />
    </el-card>
  </div>
</template>

<script>
import { ElMessage } from 'element-plus';

export default {
  data() {
    return {
      isDirty: false,
      formConfig: {
        sections: [],
        layout: 'wizard',
        stepOneTitle: 'Basic Information'
      },
      resourceProperties: []
    };
  },
  methods: {
    markDirty() {
      this.isDirty = true;
    },
    async loadResourceProperties() {
      try {
        const resource = new Resource(this, 'MyResource');
        const props = await resource.loadResourceProps('MyResource');
        this.resourceProperties = props.map(p => ({
          key: p.property,
          label: p.label || p.property
        }));
      } catch (error) {
        ElMessage.error('Failed to load resource properties');
      }
    },
    async saveConfiguration() {
      try {
        // Save your configuration
        const resource = new Resource(this, 'FormConfiguration');
        await resource.update('config-id', this.formConfig);
        this.isDirty = false;
        ElMessage.success('Configuration saved!');
      } catch (error) {
        ElMessage.error('Failed to save configuration');
      }
    },
    // Programmatically add a section
    addNewSection() {
      this.$refs.sectionManager.openAddSection();
    }
  },
  mounted() {
    this.loadResourceProperties();
  }
};
</script>
```

### **Features**

* **Drag-and-Drop Reordering:**&#x53;ections can be reordered by dragging.
* **Add/Edit/Delete:**&#x46;ull CRUD operations for sections.
* **Layout Toggle:**&#x53;witch between "Sections" (stacked) and "Wizard" (step-by-step) layouts.
* **Live Preview:**&#x53;ee how your sections will appear in the form.
* **Step 1 Title:**&#x43;ustomize the title for the initial step.
* **Property Tags:**&#x56;isual display of which fields are in each section.

**💡 Tips:**

* Fields not assigned to any section will appear in "Step 1" by default.
* Use the`wizard`layout for multi-step forms with clear progression.
* Use the`sections`layout for single-page forms with collapsible groups.
* The`properties`prop should match your resource schema for proper field labeling.

## **📋 FormQuestionCreator Component**

The`FormQuestionCreator`component provides a complete interface for designing form fields (properties) and organizing them into sections. It combines the property management capabilities of ManageSchemaBody with SectionManager for a full form building experience.

**✨ No Import Required!**`FormQuestionCreator`is automatically available in all dynamic components.

### **Basic Usage**

```
<template>
  <div class="p-4">
    <FormQuestionCreator v-model="formConfig" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      formConfig: {
        fields: [],
        sections: [],
        sectionLayout: 'sections',
        stepOneTitle: 'Get Started',
        stepOneDescription: '',
        numberOfFormColumns: 1,
        wizardStepNavigation: true,
        fieldRestrictions: {}
      }
    };
  }
};
</script>
```

### **Data Structure (v-model)**

The component binds to an object with the following structure:

```
{
  fields: [
    {
      key: "first_name",
      label: "First Name",
      type: "string",
      required: true,
      input_properties: { type: "input" },
      number_of_columns: { default: 1, full_width: false },
      // ... other property configuration options
    },
    {
      key: "email",
      label: "Email Address",
      type: "string",
      required: true,
      email: true,
      input_properties: { type: "input" }
    }
  ],
  sections: [
    {
      name: "Personal Info",
      description: "Basic personal details",
      number_of_columns: 2,
      properties: ["first_name", "email"],
      placeInTab: false,
      disableEditing: false
    }
  ],
  sectionLayout: 'sections',    // 'sections' or 'wizard'
  stepOneTitle: 'Get Started',  // Title for step 1
  stepOneDescription: '',       // Description for step 1
  numberOfFormColumns: 1,       // Default columns (1-4)
  wizardStepNavigation: true,   // Show wizard step navigation
  fieldRestrictions: {}         // Field-level restrictions
}
```

### **Props Reference**

| **Prop**     | **Type** | **Default** | **Description**                                                                           |
| ------------ | -------- | ----------- | ----------------------------------------------------------------------------------------- |
| `modelValue` | Object   | See below   | **v-model.**&#x54;he configuration object containing fields, sections, and form settings. |

#### **v-model Properties**

| **Property**           | **Type** | **Default**  | **Description**                                                                |
| ---------------------- | -------- | ------------ | ------------------------------------------------------------------------------ |
| `fields`               | Array    | `[]`         | Array of field/property definitions.                                           |
| `sections`             | Array    | `[]`         | Array of section definitions for grouping fields.                              |
| `sectionLayout`        | String   | `'sections'` | Layout mode:`'sections'`(stacked) or`'wizard'`(step-by-step).                  |
| `stepOneTitle`         | String   | `''`         | Title for the first step/section.                                              |
| `stepOneDescription`   | String   | `''`         | Description text for the first step/section.                                   |
| `numberOfFormColumns`  | Number   | `1`          | Default number of columns for the form layout (1-4).                           |
| `wizardStepNavigation` | Boolean  | `true`       | Whether to show step navigation controls in wizard mode.                       |
| `fieldRestrictions`    | Object   | `{}`         | Field-level restrictions. Keys are field keys, values are restriction objects. |

### **Events**

| **Event**           | **Payload** | **Description**                                                   |
| ------------------- | ----------- | ----------------------------------------------------------------- |
| `update:modelValue` | Object      | Emitted when the form configuration (fields or sections) changes. |

### **Features**

* **Property Management:**&#x41;dd, edit, delete, duplicate, and reorder properties with drag-and-drop.
* **Full Property Editor:**&#x55;ses the same property editor dialog as the Resource Schema editor with all configuration options.
* **Form Settings:**&#x43;onfigure layout mode, column count, step titles/descriptions, wizard navigation, and field restrictions.
* **Section Organization:**&#x47;roup properties into logical sections with customizable names, descriptions, and column layouts.
* **Layout Options:**&#x43;hoose between "Sections" (stacked) or "Wizard" (step-by-step) layouts.
* **Visibility Controls:**&#x54;oggle property visibility for forms, tables, drawers, and views.
* **Copy/Paste:**&#x43;opy properties to clipboard and paste them to quickly duplicate configurations.

### **Complete Example**

```
<template>
  <div class="min-h-screen bg-gray-100 p-6">
    <div class="max-w-6xl mx-auto">
      <div class="bg-white rounded-lg shadow-lg p-6">
        <div class="flex justify-between items-center mb-6">
          <h1 class="text-2xl font-bold">Form Builder</h1>
          <el-button type="primary" @click="saveForm">
            Save Form
          </el-button>
        </div>

        <FormQuestionCreator v-model="formConfig" />

        <!-- Preview the configuration -->
        <div class="mt-6 p-4 bg-gray-50 rounded">
          <h3 class="font-bold mb-2">Configuration Preview</h3>
          <pre class="text-xs">{{ JSON.stringify(formConfig, null, 2) }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      formConfig: {
        fields: [],
        sections: [],
        sectionLayout: 'sections',
        stepOneTitle: 'Step 1',
        stepOneDescription: '',
        numberOfFormColumns: 2,
        wizardStepNavigation: true,
        fieldRestrictions: {}
      }
    };
  },
  methods: {
    async saveForm() {
      try {
        const resource = new Resource(this, 'FormDefinition');
        await resource.create({
          name: 'My Custom Form',
          config: this.formConfig
        });
        ElMessage.success('Form saved successfully!');
      } catch (error) {
        ElMessage.error('Failed to save form');
      }
    }
  }
};
</script>
```

**💡 Tips:**

* The`FormQuestionCreator`focuses on defining form structure. Form-level metadata (name, description) should be managed separately.
* Properties use the same configuration options as Saturn's resource properties, including types, validation, mappings, and visibility settings.
* Use the section "Number of Columns" setting to control how fields are laid out within each section.

## **📊 ECharts - Data Visualization**

**🎨 Apache ECharts:**&#x41; powerful, interactive charting and visualization library. Perfect for dashboards and data analytics!

ECharts is fully available in dynamic components using the`<v-chart>`component wrapper.

**✨ No Import Required!**&#x54;he`<v-chart>`component is automatically available in all dynamic components. You don't need to import or declare it in your component - just use it directly in your template!

**🗺️ Geo Maps Supported:**`MapChart`,`GeoComponent`, and`echarts.registerMap`are available for map and`geo`option based charts. You can use`echarts.registerMap(...)`directly or import`{ registerMap }`from`echarts/core`.

### **Available Chart Types (14 Types)**

**Basic Charts:**

* `BarChart`- Bar/column charts
* `LineChart`- Line/area charts
* `PieChart`- Pie/donut charts
* `ScatterChart`- Scatter plots
* `RadarChart`- Radar/spider charts

**Advanced Charts:**

* `GaugeChart`- Gauge/meter charts
* `CandlestickChart`- Stock/OHLC charts
* `HeatmapChart`- Heatmap visualizations
* `TreemapChart`- Hierarchical treemaps
* `FunnelChart`- Funnel charts

**Relationship Charts:**

* `SankeyChart`- Flow diagrams
* `GraphChart`- Network/relationship graphs

**Geographic & Special:**

* `MapChart`- Geographic maps
* `EffectScatterChart`- Animated scatter with ripple effects

### **ECharts Components (12 Components)**

**Core Components:**

* `GridComponent`- Chart grid/coordinate system
* `TooltipComponent`- Hover tooltips
* `AxisPointerComponent`- Axis-triggered hover and crosshair support
* `LegendComponent`- Chart legends
* `TitleComponent`- Chart titles
* `DatasetComponent`- Data management

**Interaction & Analysis:**

* `ToolboxComponent`- Interactive tools
* `DataZoomComponent`- Zoom/scroll controls
* `MarkLineComponent`- Reference lines/areas
* `VisualMapComponent`- Visual mapping/color scales

**Geographic:**

* `GeoComponent`- Geographic coordinate system
* `RadarComponent`- Radar coordinate system

### **Basic Example - Bar Chart**

```
<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">Monthly Sales</h2>
    <v-chart 
      :option="chartOptions" 
      :autoresize="true" 
      style="height: 400px;" 
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      chartOptions: {
        title: {
          text: 'Sales Overview'
        },
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun']
        },
        yAxis: {
          type: 'value'
        },
        series: [{
          name: 'Sales',
          type: 'bar',
          data: [820, 932, 901, 934, 1290, 1330],
          itemStyle: {
            color: '#0d9488'
          }
        }]
      }
    }
  }
}
</script>
```

### **Geo Map Example**

```
<template>
  <div class="p-4">
    <v-chart
      v-if="mapReady"
      :option="mapOptions"
      :autoresize="true"
      style="height: 420px;"
    />
  </div>
</template>

<script>
import { registerMap } from 'echarts/core';

export default {
  data() {
    return {
      mapReady: false,
      mapOptions: {
        tooltip: { trigger: 'item' },
        visualMap: {
          min: 0,
          max: 1000,
          left: 20,
          bottom: 20,
          calculable: true
        },
        geo: {
          map: 'world',
          roam: true,
          emphasis: {
            label: { show: false }
          }
        },
        series: [{
          name: 'Activity',
          type: 'map',
          map: 'world',
          geoIndex: 0,
          data: [
            { name: 'Canada', value: 420 },
            { name: 'United States of America', value: 860 }
          ]
        }]
      }
    }
  },
  async mounted() {
    const res = await fetch('https://raw.githubusercontent.com/apache/echarts-website/asf-site/examples/data/asset/geo/world.json');
    registerMap('world', await res.json());
    this.mapReady = true;
  }
}
</script>
```

### **Line Chart with Dynamic Data**

```
<template>
  <div class="p-4">
    <el-button @click="loadChartData" type="primary">Load Data</el-button>
    <v-chart :option="lineOptions" :autoresize="true" style="height: 350px;" class="mt-4" />
  </div>
</template>

<script>
import { ElMessage } from 'element-plus';

export default {
  data() {
    return {
      lineOptions: {
        title: { text: 'Resource Trends' },
        tooltip: { trigger: 'axis' },
        legend: { data: ['Active', 'Pending'] },
        xAxis: {
          type: 'category',
          data: []
        },
        yAxis: { type: 'value' },
        series: [
          { name: 'Active', type: 'line', data: [], smooth: true },
          { name: 'Pending', type: 'line', data: [], smooth: true }
        ]
      }
    }
  },
  methods: {
    async loadChartData() {
      try {
        const resource = new Resource(this, 'Analytics');
        const data = await resource.list({ limit: 7 });
        
        // Update chart with real data
        this.lineOptions.xAxis.data = data.map(d => d.date);
        this.lineOptions.series[0].data = data.map(d => d.active);
        this.lineOptions.series[1].data = data.map(d => d.pending);
        
        ElMessage.success('Chart data loaded!');
      } catch (error) {
        ElMessage.error('Failed to load chart data');
      }
    }
  },
  mounted() {
    this.loadChartData();
  }
}
</script>
```

### **Pie Chart Example**

```
<template>
  <v-chart :option="pieOptions" :autoresize="true" style="height: 400px;" />
</template>

<script>
export default {
  data() {
    return {
      pieOptions: {
        title: {
          text: 'Status Distribution',
          left: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)'
        },
        legend: {
          orient: 'vertical',
          left: 'left'
        },
        series: [{
          name: 'Status',
          type: 'pie',
          radius: '60%',
          data: [
            { value: 335, name: 'Completed', itemStyle: { color: '#10b981' } },
            { value: 234, name: 'In Progress', itemStyle: { color: '#3b82f6' } },
            { value: 135, name: 'Pending', itemStyle: { color: '#f59e0b' } },
            { value: 48, name: 'Cancelled', itemStyle: { color: '#ef4444' } }
          ],
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          }
        }]
      }
    }
  }
}
</script>
```

### **Gauge Chart Example**

```
<template>
  <v-chart :option="gaugeOptions" :autoresize="true" style="height: 300px;" />
</template>

<script>
export default {
  data() {
    return {
      gaugeOptions: {
        series: [{
          type: 'gauge',
          progress: { show: true, width: 18 },
          axisLine: { lineStyle: { width: 18 } },
          axisTick: { show: false },
          splitLine: { length: 15, lineStyle: { width: 2, color: '#999' } },
          axisLabel: { distance: 25, color: '#999', fontSize: 12 },
          anchor: { show: true, showAbove: true, size: 25 },
          title: { show: true, offsetCenter: [0, '70%'] },
          detail: {
            valueAnimation: true,
            fontSize: 32,
            offsetCenter: [0, '40%'],
            formatter: '{value}%'
          },
          data: [{ value: 75, name: 'Completion' }]
        }]
      }
    }
  }
}
</script>
```

### **Radar Chart Example**

```
<template>
  <v-chart :option="radarOptions" :autoresize="true" style="height: 400px;" />
</template>

<script>
export default {
  data() {
    return {
      radarOptions: {
        title: { text: 'Skills Assessment' },
        radar: {
          indicator: [
            { name: 'Communication', max: 100 },
            { name: 'Technical', max: 100 },
            { name: 'Problem Solving', max: 100 },
            { name: 'Teamwork', max: 100 },
            { name: 'Leadership', max: 100 }
          ]
        },
        series: [{
          name: 'Skills',
          type: 'radar',
          data: [
            { value: [85, 95, 80, 90, 75], name: 'Employee A' },
            { value: [70, 85, 90, 85, 80], name: 'Employee B' }
          ]
        }]
      }
    }
  }
}
</script>
```

### **Treemap Chart Example**

```
<template>
  <v-chart :option="treemapOptions" :autoresize="true" style="height: 400px;" />
</template>

<script>
export default {
  data() {
    return {
      treemapOptions: {
        title: { text: 'Resource Distribution' },
        series: [{
          type: 'treemap',
          data: [
            {
              name: 'Development',
              value: 450,
              children: [
                { name: 'Frontend', value: 200 },
                { name: 'Backend', value: 150 },
                { name: 'DevOps', value: 100 }
              ]
            },
            {
              name: 'Marketing',
              value: 300,
              children: [
                { name: 'Digital', value: 180 },
                { name: 'Content', value: 120 }
              ]
            },
            { name: 'Sales', value: 250 }
          ]
        }]
      }
    }
  }
}
</script>
```

### **Heatmap Chart Example**

```
<template>
  <v-chart :option="heatmapOptions" :autoresize="true" style="height: 400px;" />
</template>

<script>
export default {
  data() {
    return {
      heatmapOptions: {
        title: { text: 'Activity Heatmap' },
        tooltip: { position: 'top' },
        grid: { height: '50%', top: '10%' },
        xAxis: {
          type: 'category',
          data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
        },
        yAxis: {
          type: 'category',
          data: ['Morning', 'Afternoon', 'Evening', 'Night']
        },
        visualMap: {
          min: 0,
          max: 100,
          calculable: true,
          orient: 'horizontal',
          left: 'center',
          bottom: '15%'
        },
        series: [{
          type: 'heatmap',
          data: [
            [0, 0, 45], [0, 1, 78], [0, 2, 65], [0, 3, 23],
            [1, 0, 52], [1, 1, 89], [1, 2, 71], [1, 3, 18],
            // ... more data points
          ],
          label: { show: true }
        }]
      }
    }
  }
}
</script>
```

### **Funnel Chart Example**

```
<template>
  <v-chart :option="funnelOptions" :autoresize="true" style="height: 400px;" />
</template>

<script>
export default {
  data() {
    return {
      funnelOptions: {
        title: { text: 'Sales Funnel' },
        tooltip: { trigger: 'item', formatter: '{b}: {c} ({d}%)' },
        series: [{
          type: 'funnel',
          left: '10%',
          width: '80%',
          label: { formatter: '{b}: {c}' },
          data: [
            { value: 1000, name: 'Visitors' },
            { value: 750, name: 'Leads' },
            { value: 400, name: 'Opportunities' },
            { value: 200, name: 'Proposals' },
            { value: 80, name: 'Customers' }
          ]
        }]
      }
    }
  }
}
</script>
```

**📖 More Chart Types Available:**

* **Sankey Charts:**&#x46;low diagrams showing relationships and proportions
* **Graph Charts:**&#x4E;etwork/node relationship visualizations
* **Candlestick Charts:**&#x46;inancial OHLC stock charts
* **Scatter & EffectScatter:**&#x44;ata distribution with optional animations
* [Official ECharts Examples Gallery →](https://echarts.apache.org/examples/en/index.html)
* [vue-echarts Documentation →](https://github.com/ecomfe/vue-echarts)

**💡 Pro Tips:**

* **No imports needed:**`<v-chart>`is globally available - just use it!
* Always use`:autoresize="true"`prop for responsive behavior
* Combine charts with`Resource`API for dynamic data loading
* Use`ElLoading`or`VProgressCircular`while fetching chart data
* Charts automatically work with Tailwind utility classes for layout
* Update chart data by modifying the options object - Vue's reactivity handles the rest

## **🧊 Three.js, TresJS & Digital Twin Rendering**

**🎥 3D Rendering:**&#x54;hree.js is available for custom WebGL scenes, TresJS provides Vue bindings for Three.js, Cientos adds high-level scene helpers, and PostProcessing provides common render effects.

The installed 3D packages are available in dynamic components:

* `THREE`and`Three`: the full`three`module.
* `TresJS`: exports from`@tresjs/core`, including`TresCanvas`,`useRenderLoop`,`useTres`, and texture/loader helpers.
* `TresCientos`and`Cientos`: exports from`@tresjs/cientos`, including controls, loaders, environment helpers, text, HTML overlays, and scene utilities.
* `TresCanvas`: available directly in templates.
* `TresPostProcessing`: exports from`@tresjs/post-processing`, with template components such as`EffectComposer`,`UnrealBloom`,`OutlinePmndrs`, and`SMAA`.
* `PostProcessing`and`postprocessing`: exports from`postprocessing`, including`EffectComposer`,`RenderPass`,`EffectPass`, and effects such as`BloomEffect`.
* `CameraControls`: smooth app-grade camera navigation.
* `ThreeMeshBVH`: accelerated raycasting, picking, spatial queries, and measurements for large models.
* `MeshOptimizer`,`MeshoptDecoder`, and`MeshoptEncoder`: meshopt-compressed glTF support.
* `Draco3D`: Draco encoder/decoder module access.
* `ThreeAddons`: common Three.js addon classes such as`OrbitControls`,`MapControls`,`GLTFLoader`,`DRACOLoader`,`KTX2Loader`,`CSS2DRenderer`,`CSS3DRenderer`,`RoomEnvironment`,`Line2`, and official post-processing passes.

**✨ No Import Required!**&#x55;se`THREE`,`TresJS`,`Cientos`,`TresPostProcessing`,`PostProcessing`,`CameraControls`,`ThreeMeshBVH`, and`ThreeAddons`directly in your script. Import syntax is also supported for`three`,`three/addons`,`@tresjs/core`,`@tresjs/cientos`,`@tresjs/post-processing`,`postprocessing`,`camera-controls`,`three-mesh-bvh`,`meshoptimizer`, and`draco3d`.

### **Basic Three.js Scene**

```
<template>
  <div class="p-4">
    <div ref="canvasHost" style="height: 320px; width: 100%;"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      renderer: null,
      scene: null,
      camera: null,
      cube: null,
      animationFrame: null
    };
  },
  mounted() {
    const host = this.$refs.canvasHost;
    const width = host.clientWidth;
    const height = host.clientHeight;

    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color('#f8fafc');

    this.camera = new THREE.PerspectiveCamera(55, width / height, 0.1, 100);
    this.camera.position.z = 4;

    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.setSize(width, height);
    this.renderer.setPixelRatio(window.devicePixelRatio || 1);
    host.appendChild(this.renderer.domElement);

    const geometry = new THREE.BoxGeometry(1.4, 1.4, 1.4);
    const material = new THREE.MeshNormalMaterial();
    this.cube = new THREE.Mesh(geometry, material);
    this.scene.add(this.cube);

    const animate = () => {
      this.cube.rotation.x += 0.01;
      this.cube.rotation.y += 0.015;
      this.renderer.render(this.scene, this.camera);
      this.animationFrame = requestAnimationFrame(animate);
    };

    animate();
  },
  beforeUnmount() {
    if (this.animationFrame) cancelAnimationFrame(this.animationFrame);
    if (this.cube) {
      this.cube.geometry.dispose();
      this.cube.material.dispose();
    }
    if (this.renderer) {
      this.renderer.dispose();
      this.renderer.domElement.remove();
    }
  }
};
</script>
```

### **Import Syntax (Optional)**

```
<script>
import * as THREE from 'three';
import { BoxGeometry, MeshNormalMaterial, Mesh } from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';
import CameraControls from 'camera-controls';
import { TresCanvas, useRenderLoop } from '@tresjs/core';
import { OrbitControls, Html, useGLTF } from '@tresjs/cientos';
import { EffectComposer as TresEffectComposer, UnrealBloom } from '@tresjs/post-processing';
import { EffectComposer, RenderPass, EffectPass, BloomEffect } from 'postprocessing';
import { acceleratedRaycast, computeBoundsTree, disposeBoundsTree } from 'three-mesh-bvh';
import { MeshoptDecoder } from 'meshoptimizer';

export default {
  mounted() {
    const geometry = new BoxGeometry(1, 1, 1);
    const material = new MeshNormalMaterial();
    const mesh = new Mesh(geometry, material);

    console.log(
      THREE,
      GLTFLoader,
      DRACOLoader,
      CameraControls,
      TresCanvas,
      useRenderLoop,
      OrbitControls,
      Html,
      useGLTF,
      TresEffectComposer,
      UnrealBloom,
      EffectComposer,
      RenderPass,
      EffectPass,
      BloomEffect,
      acceleratedRaycast,
      computeBoundsTree,
      disposeBoundsTree,
      MeshoptDecoder,
      mesh
    );
  }
};
</script>
```

### **TresCanvas + Cientos Template**

```
<template>
  <TresCanvas clear-color="#f8fafc" style="height: 320px;">
    <TresPerspectiveCamera :position="[0, 0, 5]" />
    <OrbitControls make-default />
    <Environment preset="city" />
    <TresMesh :rotation="[0.5, 0.5, 0]">
      <TresBoxGeometry :args="[1.5, 1.5, 1.5]" />
      <TresMeshNormalMaterial />
    </TresMesh>
    <EffectComposer>
      <SMAA />
      <UnrealBloom :strength="0.2" />
    </EffectComposer>
  </TresCanvas>
</template>

<script>
export default {};
</script>
```

### **Digital Twin Model Loading**

```
<script>
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';
import { KTX2Loader } from 'three/addons/loaders/KTX2Loader.js';
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';
import { MeshoptDecoder } from 'meshoptimizer';
import { computeBoundsTree, disposeBoundsTree, acceleratedRaycast } from 'three-mesh-bvh';

export default {
  async mounted() {
    THREE.BufferGeometry.prototype.computeBoundsTree = computeBoundsTree;
    THREE.BufferGeometry.prototype.disposeBoundsTree = disposeBoundsTree;
    THREE.Mesh.prototype.raycast = acceleratedRaycast;

    const loader = new GLTFLoader();
    loader.setMeshoptDecoder(MeshoptDecoder);

    const dracoLoader = new DRACOLoader();
    dracoLoader.setDecoderPath('/draco/');
    loader.setDRACOLoader(dracoLoader);

    const ktx2Loader = new KTX2Loader();
    ktx2Loader.setTranscoderPath('/basis/');
    ktx2Loader.detectSupport(this.renderer);
    loader.setKTX2Loader(ktx2Loader);

    const pmrem = new THREE.PMREMGenerator(this.renderer);
    this.scene.environment = pmrem.fromScene(new RoomEnvironment()).texture;

    const gltf = await loader.loadAsync('/models/facility.glb');
    gltf.scene.traverse((object) => {
      if (object.isMesh) {
        object.geometry.computeBoundsTree();
        object.castShadow = true;
        object.receiveShadow = true;
      }
    });

    this.scene.add(gltf.scene);
  }
};
</script>
```

**💡 Tips:**

* Use fixed or parent-controlled heights for WebGL containers; a zero-height host will render a blank canvas.
* Dispose geometries, materials, renderers, and animation frames in`beforeUnmount()`.
* Use`GLTFLoader`for digital-twin assets and prefer`.glb`with Draco, meshopt, and KTX2 compression for large scenes.
* Use`ThreeMeshBVH`for picking, measurement tools, section tools, and raycasting against large facility models.
* Use`CSS2DRenderer`,`CSS3DRenderer`, or Cientos`Html`for asset labels, sensor badges, and annotations.
* Post-processing effects need an initialized renderer, scene, and camera before creating an imperative composer.

## **🔌 Resource API**

The`Resource`class is automatically available in your script section for CRUD operations and custom API endpoints on any resource type.

### **Available in Script**

Access via the injected`Resource`parameter:

```vue
<script>
export default {
  async mounted() {
    // Resource is automatically available
    const projects = new Resource(this, 'Project');
    const data = await projects.list();
    console.log(data);
  }
}
</script>
```

### **Core Methods**

| **Method**                                                          | **Description**                                                                              | **Example**                                                                      |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `list(params)`                                                      | Get a list of resources                                                                      | `await resource.list({ limit: 10 })`                                             |
| `get(id)`                                                           | Get a single resource by ID                                                                  | `await resource.get('123')`                                                      |
| `create(data)`                                                      | Create a new resource                                                                        | `await resource.create({ name: 'Test' })`                                        |
| `update(id, data)`                                                  | Update an existing resource                                                                  | `await resource.update('123', { name: 'Updated' })`                              |
| `delete(id)`                                                        | Delete a resource                                                                            | `await resource.delete('123')`                                                   |
| `export(params)`                                                    | Export resources                                                                             | `await resource.export({ format: 'csv' })`                                       |
| `count()`                                                           | Get total count                                                                              | `await resource.count()`                                                         |
| `request(method, path, options)`                                    | Call a custom endpoint with full control over method, path, payload, params, and return type | `await resource.request('post', 'archive/123', { data: { reason: 'manual' } })`  |
| `customGet/customPost/customPut/customPatch/customDelete`           | Shorthand helpers for custom endpoints                                                       | `await resource.customPost('archive/123', { reason: 'manual' })`                 |
| `invokeWorkflowAction(resourceId, actionId, data, getFullResponse)` | Execute a single workflow action for a resource and return the workflow status.              | `await resource.invokeWorkflowAction(projectId, actionId, { note: 'approved' })` |

### **Advanced Methods**

| **Method**                                | **Description**                                                                                 | **Example**                                                                                                                                                                                                                             |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `loadResourceProps(resource)`             | Load resource property definitions                                                              | `await resource.loadResourceProps('Project')`                                                                                                                                                                                           |
| `get_histogram_data(data, filter)`        | Get histogram analytics data. Supports preset modes like`this month`and explicit`period`ranges. | `await new Resource(this).get_histogram_data({ source: 'Project', mode: 'this month', interval: 'day', aggregations: [{ aggrigation: 'count', field: 'id', source: 'Project', time_axis: 'created_at' }], time: moment().format() })`   |
| `get_source_histogram_data(data, filter)` | Get histogram source drill-down data                                                            | `await new Resource(this).get_source_histogram_data({ source: 'Project', series_index: 0, data_index: 2, aggregations: [{ aggrigation: 'count', field: 'id', source: 'Project', time_axis: 'created_at' }], time: moment().format() })` |
| `term(data, filter)`                      | Get grouped term aggregation data for a field, with optional filters merged into the payload.   | `await new Resource(this).term({ source: 'Project', field: 'status.keyword', size: 10 }, 'archived = false')`                                                                                                                           |
| `updateSchema(name, schema)`              | Update resource schema                                                                          | `await resource.updateSchema('Project', schemaRows)`                                                                                                                                                                                    |

### **Analytics Method Examples**

```
<script>
export default {
  async mounted() {
    // Histogram request using a preset mode instead of an explicit period range.
    const histogram = await new Resource(this).get_histogram_data(
      {
        source: 'Project',
        mode: 'this month',
        interval: 'day',
        aggregations: [
          {
            aggrigation: 'count',
            field: 'id',
            source: 'Project',
            time_axis: 'created_at'
          }
        ],
        time: moment().format()
      },
      'archived = false'
    );

    // Term aggregation request for the top statuses.
    const statusBreakdown = await new Resource(this).term(
      {
        source: 'Project',
        field: 'status.keyword',
        size: 10
      },
      'archived = false'
    );

    console.log(histogram, statusBreakdown);
  }
}
</script>
```

For histogram requests, use`mode`for presets like`this month`,`this year`, or`last 30 days`. Use`period`when you need a custom`[start, end]`range instead.

In histogram aggregations,`time_axis`is the date field used for bucketing and`field`is the value field being aggregated. For example, a count over time can use`field: 'id'`with`time_axis: 'created_at'`.

### **Custom Endpoint Examples**

Custom endpoint paths can be either resource-relative or absolute:

```
<script>
export default {
  async mounted() {
    const projectResource = new Resource(this, 'Project');

    // Resource-relative GET: /projects/stats
    const stats = await projectResource.customGet('stats', {
      params: { include_archived: true }
    });

    // Resource-relative: /projects/archive/123
    await projectResource.customPost('archive/123', {
      reason: 'manual'
    });

    // Absolute workflow action endpoint via the inherited helper.
    const workflowStatus = await projectResource.invokeWorkflowAction(
      'project-123',
      'action-456',
      { note: 'approved' }
    );

    const body = {
      source: 'Project',
      field: 'status.keyword'
    };

    // Absolute path: /analytics/term
    const analytics = await new Resource(this).customPost(
      '/analytics/term',
      body,
      { returnType: 'rawData' }
    );

    console.log(stats, workflowStatus, analytics);
  }
}
</script>
```

`request()`and the custom verb helpers return`response.data.data`by default when available. Use`returnType: 'rawData'`for`response.data`or`returnType: 'response'`for the full Axios response.

### **Example: Fetch and Display Data**

```
<template>
  <div class="p-4">
    <h2 class="text-xl font-bold mb-4">Projects</h2>
    <div v-for="project in projects" :key="project.id" 
         class="p-3 mb-2 bg-gray-100 rounded">
      {{ project.name }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      projects: []
    }
  },
  async mounted() {
    const projectResource = new Resource(this, 'Project');
    this.projects = await projectResource.list({ limit: 20 });
  }
}
</script>
```

## **🧭 Dashboard Filters API**

Inline dashboard dynamic components receive two injected props:`dashboardFilters`and`dashboardFilterApi`. Use these to create/edit/remove dashboard-level filters and react to changes.

### **Available Props**

| **Prop**             | **Type** | **Description**                                                                    |
| -------------------- | -------- | ---------------------------------------------------------------------------------- |
| `dashboardFilters`   | array    | Current dashboard filter pills.                                                    |
| `dashboardFilterApi` | object   | Functions for creating/updating/removing filters and subscribing to filter events. |

### **`dashboardFilterApi`****Methods**

| **Method**                | **Description**                                                                                                                                      |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `createFilter(payload)`   | Create a dashboard filter. Use`origin: 'inline_api'`by default. For mapped fields, pass display labels in`values`and execution IDs in`query_values`. |
| `updateFilter(id, patch)` | Patch an existing filter by ID.                                                                                                                      |
| `removeFilter(id)`        | Remove a filter by ID.                                                                                                                               |
| `clearFilters()`          | Remove all dashboard filters.                                                                                                                        |
| `subscribe(listener)`     | Listen for filter changes. Returns an unsubscribe function.                                                                                          |

### **Example: Add Mapped Filter From Inline Component**

```
<template>
  <div class="p-4">
    <el-button type="primary" @click="filterCluster77">
      Filter Cluster = Saturn HQ
    </el-button>
  </div>
</template>

<script>
export default {
  props: {
    dashboardFilters: { type: Array, default: () => [] },
    dashboardFilterApi: { type: Object, default: () => ({}) }
  },
  methods: {
    filterCluster77() {
      this.dashboardFilterApi.createFilter({
        resource: 'ClusterMetrics',
        field: 'cluster',
        operator: 'is_one_of',
        values: ['Saturn HQ'],
        query_values: ['77'],
        origin: 'inline_api'
      });
    }
  }
}
</script>
```

If`query_values`is omitted for mapped fields, the dashboard resolves labels to IDs automatically. Existing filters that only contain`values`still run as before.

### **Example: Subscribe To Filter Changes**

```
<script>
export default {
  props: {
    dashboardFilters: { type: Array, default: () => [] },
    dashboardFilterApi: { type: Object, default: () => ({}) }
  },
  data() {
    return { unsubscribe: null };
  },
  mounted() {
    this.unsubscribe = this.dashboardFilterApi.subscribe((event, context) => {
      console.log('Filter event:', event.type, context.filters);
    });
  },
  beforeUnmount() {
    if (this.unsubscribe) this.unsubscribe();
  }
}
</script>
```

## **🤖 NovaAI API**

The`NovaAI`class provides methods for interacting with AI models and agents.

### **Available Methods**

| **Method**                                   | **Description**                                    | **Parameters**                                                          |
| -------------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------------------- |
| `prompt(prompt, options)`                    | Send a single prompt to an AI model                | `prompt`(string),`options`(object)                                      |
| `chatPrompt(body, onMeta, onChunk, onAudio)` | Send a chat prompt with streaming response support | `body`(object/FormData), callbacks for metadata, text chunks, and audio |
| `extractMetaBlocks(text)`                    | Extract meta blocks from response text             | `text`(string)                                                          |
| `extractTags(text, tagName)`                 | Extract specific tags from text                    | `text`(string),`tagName`(string)                                        |

### **prompt() Options**

| **Option** | **Type** | **Default**                     | **Description**                                           |
| ---------- | -------- | ------------------------------- | --------------------------------------------------------- |
| `provider` | string   | "openai"                        | AI provider to use                                        |
| `model`    | string   | "gpt-4.1"                       | Model to use for the prompt                               |
| `context`  | string   | "You are a coding assistant..." | System context for the AI                                 |
| `options`  | object   | {}                              | Additional model options (temperature, max\_tokens, etc.) |
| `files`    | array    | \[]                             | Files to include with the prompt                          |

### **Example: Simple Prompt**

```
<template>
  <div class="p-4">
    <el-input 
      v-model="userPrompt" 
      placeholder="Ask something..."
      @keyup.enter="askAI"
    />
    <el-button @click="askAI" :loading="loading">Send</el-button>
    
    <div v-if="response" class="mt-4 p-3 bg-gray-100 rounded">
      <div v-html="response"></div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userPrompt: '',
      response: '',
      loading: false
    }
  },
  methods: {
    async askAI() {
      if (!this.userPrompt) return;
      
      this.loading = true;
      try {
        const novaAI = new NovaAI(this);
        this.response = await novaAI.prompt(this.userPrompt, {
          provider: 'openai',
          model: 'gpt-4',
          context: 'You are a helpful assistant that responds in HTML format.'
        });
      } catch (error) {
        console.error('AI Error:', error);
      } finally {
        this.loading = false;
      }
    }
  }
}
</script>
```

### **Example: Streaming Chat with Callbacks**

```
<script>
export default {
  data() {
    return {
      streamingResponse: '',
      metadata: []
    }
  },
  methods: {
    async streamChat() {
      const novaAI = new NovaAI(this);
      
      await novaAI.chatPrompt(
        {
          prompt: 'Tell me about Vue 3',
          agent: 'coding-assistant',
          stream: true
        },
        // onMeta callback
        (meta) => {
          console.log('Metadata:', meta);
          this.metadata.push(meta);
        },
        // onChunk callback
        (chunk) => {
          this.streamingResponse += chunk;
        },
        // onAudio callback (optional)
        (audioChunk, isIntermediate, metadata) => {
          if (audioChunk) {
            console.log('Audio chunk:', audioChunk.seq);
            // Process audio chunk
          } else {
            console.log('Audio segment complete');
          }
        }
      );
    }
  }
}
</script>
```

### **Example: With File Upload**

```
<script>
export default {
  methods: {
    async analyzeFile(file) {
      const novaAI = new NovaAI(this);
      
      const response = await novaAI.prompt(
        'Analyze this file and provide insights',
        {
          provider: 'openai',
          model: 'gpt-4',
          files: [file],
          options: {
            temperature: 0.7,
            max_tokens: 1000
          }
        }
      );
      
      return response;
    }
  }
}
</script>
```

**💡 Note:**&#x54;he`NovaAI`class inherits from`Service`, so it also has access to all Service methods:`list()`,`get()`,`create()`,`update()`,`delete()`.

## **🔧 Lodash - Utility Library**

[Lodash](https://lodash.com/docs)is a modern JavaScript utility library delivering modularity, performance, and extras. It's available globally as`_`in all dynamic components.

**✨ No Import Required!**&#x4C;odash is automatically available as`_`in all dynamic components. You can also use import syntax if preferred.

### **Usage Examples**

```
<script>
export default {
  data() {
    return {
      users: [
        { name: 'John', age: 25, active: true },
        { name: 'Jane', age: 30, active: false },
        { name: 'Bob', age: 35, active: true }
      ]
    };
  },
  computed: {
    // Filter active users
    activeUsers() {
      return _.filter(this.users, { active: true });
    },
    // Sort by age
    sortedByAge() {
      return _.sortBy(this.users, 'age');
    },
    // Get unique ages
    uniqueAges() {
      return _.uniq(_.map(this.users, 'age'));
    },
    // Group by active status
    groupedByStatus() {
      return _.groupBy(this.users, 'active');
    }
  },
  methods: {
    // Debounce a search function
    debouncedSearch: _.debounce(function(query) {
      console.log('Searching for:', query);
    }, 300),
    
    // Deep clone an object
    cloneUser(user) {
      return _.cloneDeep(user);
    },
    
    // Check if value is empty
    isEmpty(value) {
      return _.isEmpty(value);
    }
  }
};
</script>
```

### **Import Syntax (Optional)**

You can also use import syntax if you prefer explicit imports:

```
<script>
// Default import
import _ from 'lodash';

// Named imports
import { map, filter, sortBy } from 'lodash';

// Subpath import
import assignWith from 'lodash/assignWith';

export default {
  computed: {
    filteredItems() {
      return filter(this.items, item => item.active);
    }
  },
  methods: {
    mergeDefaults(target, source) {
      return assignWith(target, source, (objValue, srcValue) => {
        return objValue === undefined ? srcValue : objValue;
      });
    }
  }
};
</script>
```

### **Common Functions**

| **Function**    | **Description**             | **Example**                         |
| --------------- | --------------------------- | ----------------------------------- |
| `_.map()`       | Transform array elements    | `_.map(users, 'name')`              |
| `_.filter()`    | Filter array by condition   | `_.filter(users, { active: true })` |
| `_.find()`      | Find first matching element | `_.find(users, { name: 'John' })`   |
| `_.sortBy()`    | Sort array by property      | `_.sortBy(users, 'age')`            |
| `_.groupBy()`   | Group by property           | `_.groupBy(users, 'role')`          |
| `_.uniq()`      | Remove duplicates           | `_.uniq([1, 2, 2, 3])`              |
| `_.cloneDeep()` | Deep clone object           | `_.cloneDeep(obj)`                  |
| `_.debounce()`  | Debounce function calls     | `_.debounce(fn, 300)`               |
| `_.throttle()`  | Throttle function calls     | `_.throttle(fn, 1000)`              |
| `_.isEmpty()`   | Check if value is empty     | `_.isEmpty([])`                     |
| `_.get()`       | Safe property access        | `_.get(obj, 'a.b.c', 'default')`    |
| `_.set()`       | Set nested property         | `_.set(obj, 'a.b.c', value)`        |

## **📅 Moment.js - Date & Time**

[Moment.js](https://momentjs.com/docs/)is a powerful library for parsing, validating, manipulating, and formatting dates. It's available globally as`moment`in all dynamic components.

**✨ No Import Required!**&#x4D;oment.js is automatically available as`moment`in all dynamic components.

### **Usage Examples**

```
<template>
  <div class="p-4">
    <p>Current time: {{ currentTime }}</p>
    <p>Formatted date: {{ formattedDate }}</p>
    <p>Relative time: {{ relativeTime }}</p>
    <p>Days until deadline: {{ daysUntilDeadline }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      deadline: '2025-12-31',
      createdAt: '2025-01-01T10:30:00'
    };
  },
  computed: {
    currentTime() {
      return moment().format('MMMM Do YYYY, h:mm:ss a');
      // Output: "January 2nd 2026, 3:45:00 pm"
    },
    formattedDate() {
      return moment(this.createdAt).format('MMM D, YYYY');
      // Output: "Jan 1, 2025"
    },
    relativeTime() {
      return moment(this.createdAt).fromNow();
      // Output: "a year ago"
    },
    daysUntilDeadline() {
      return moment(this.deadline).diff(moment(), 'days');
      // Output: number of days
    }
  },
  methods: {
    formatDate(date, format = 'YYYY-MM-DD') {
      return moment(date).format(format);
    },
    addDays(date, days) {
      return moment(date).add(days, 'days').format('YYYY-MM-DD');
    },
    isBeforeToday(date) {
      return moment(date).isBefore(moment(), 'day');
    },
    getStartOfMonth() {
      return moment().startOf('month').format('YYYY-MM-DD');
    }
  }
};
</script>
```

### **Import Syntax (Optional)**

```
<script>
import moment from 'moment';

export default {
  methods: {
    formatDate(date) {
      return moment(date).format('LL');
    }
  }
};
</script>
```

### **Common Formats**

| **Format**        | **Output Example**        | **Description**           |
| ----------------- | ------------------------- | ------------------------- |
| `'YYYY-MM-DD'`    | 2025-01-15                | ISO date format           |
| `'MM/DD/YYYY'`    | 01/15/2025                | US date format            |
| `'DD/MM/YYYY'`    | 15/01/2025                | European date format      |
| `'MMMM Do, YYYY'` | January 15th, 2025        | Full month with ordinal   |
| `'MMM D, YYYY'`   | Jan 15, 2025              | Short month format        |
| `'h:mm a'`        | 3:30 pm                   | 12-hour time              |
| `'HH:mm:ss'`      | 15:30:45                  | 24-hour time with seconds |
| `'llll'`          | Wed, Jan 15, 2025 3:30 PM | Full localized datetime   |

### **Common Methods**

| **Method**   | **Description**          | **Example**                     |
| ------------ | ------------------------ | ------------------------------- |
| `format()`   | Format date to string    | `moment().format('YYYY-MM-DD')` |
| `fromNow()`  | Relative time from now   | `moment(date).fromNow()`        |
| `add()`      | Add time to date         | `moment().add(7, 'days')`       |
| `subtract()` | Subtract time from date  | `moment().subtract(1, 'month')` |
| `diff()`     | Difference between dates | `moment(a).diff(b, 'days')`     |
| `isBefore()` | Check if date is before  | `moment(a).isBefore(b)`         |
| `isAfter()`  | Check if date is after   | `moment(a).isAfter(b)`          |
| `startOf()`  | Start of time unit       | `moment().startOf('month')`     |
| `endOf()`    | End of time unit         | `moment().endOf('year')`        |
| `isValid()`  | Check if date is valid   | `moment(date).isValid()`        |

## **📷 QR Scanner - QR Code Scanning**

[QR Scanner](https://github.com/nimiq/qr-scanner)is a lightweight library for scanning QR codes using a device camera or from images. It's available globally as`QrScanner`in all dynamic components.

**✨ No Import Required!**&#x51;rScanner is automatically available as`QrScanner`in all dynamic components.

**⚠️ Camera Permission Required**The browser will prompt users for camera access when starting the scanner. Make sure to handle permission denied scenarios.

### **Basic Camera Scanning**

```
<template>
  <div class="p-4">
    <div class="mb-4">
      <video ref="videoEl" class="w-full max-w-md rounded-lg"></video>
    </div>
    <div class="flex gap-2 mb-4">
      <el-button type="primary" @click="startScanner" :disabled="scanning">
        Start Scanner
      </el-button>
      <el-button @click="stopScanner" :disabled="!scanning">
        Stop Scanner
      </el-button>
    </div>
    <div v-if="result" class="p-4 bg-green-50 rounded-lg">
      <strong>Scanned Result:</strong> {{ result }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      scanner: null,
      scanning: false,
      result: null
    };
  },
  methods: {
    async startScanner() {
      try {
        this.scanner = new QrScanner(
          this.$refs.videoEl,
          result => {
            this.result = result.data;
            // Optionally stop after first scan:
            // this.stopScanner();
          },
          {
            highlightScanRegion: true,
            highlightCodeOutline: true
          }
        );
        await this.scanner.start();
        this.scanning = true;
      } catch (error) {
        console.error('Failed to start scanner:', error);
        alert('Could not access camera. Please check permissions.');
      }
    },
    stopScanner() {
      if (this.scanner) {
        this.scanner.stop();
        this.scanning = false;
      }
    }
  },
  beforeUnmount() {
    // Clean up scanner when component is destroyed
    if (this.scanner) {
      this.scanner.destroy();
    }
  }
}
</script>
```

### **Scan from Image File**

```
<template>
  <div class="p-4">
    <el-upload
      action="#"
      :auto-upload="false"
      :show-file-list="false"
      accept="image/*"
      @change="handleImageUpload">
      <el-button type="primary">Upload Image with QR Code</el-button>
    </el-upload>
    <div v-if="result" class="mt-4 p-4 bg-green-50 rounded-lg">
      <strong>Scanned Result:</strong> {{ result }}
    </div>
    <div v-if="error" class="mt-4 p-4 bg-red-50 rounded-lg text-red-600">
      {{ error }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      result: null,
      error: null
    };
  },
  methods: {
    async handleImageUpload(file) {
      this.error = null;
      this.result = null;
      try {
        const result = await QrScanner.scanImage(file.raw);
        this.result = result;
      } catch (err) {
        this.error = 'No QR code found in image';
      }
    }
  }
}
</script>
```

### **Check Camera Availability**

```
<script>
export default {
  data() {
    return {
      hasCamera: false
    };
  },
  async mounted() {
    // Check if device has a camera
    this.hasCamera = await QrScanner.hasCamera();
  }
}
</script>
```

### **Common Methods**

| **Method**                                | **Description**                            | **Example**                                        |
| ----------------------------------------- | ------------------------------------------ | -------------------------------------------------- |
| `new QrScanner(video, callback, options)` | Create scanner instance on video element   | `new QrScanner(videoEl, r => console.log(r.data))` |
| `scanner.start()`                         | Start scanning (async, returns Promise)    | `await scanner.start()`                            |
| `scanner.stop()`                          | Stop scanning                              | `scanner.stop()`                                   |
| `scanner.destroy()`                       | Clean up scanner resources                 | `scanner.destroy()`                                |
| `QrScanner.scanImage(source)`             | Scan QR code from image (static method)    | `await QrScanner.scanImage(imageFile)`             |
| `QrScanner.hasCamera()`                   | Check if device has camera (static method) | `await QrScanner.hasCamera()`                      |
| `scanner.setCamera(facingMode)`           | Switch camera (e.g., front/back)           | `scanner.setCamera('environment')`                 |

### **Scanner Options**

| **Option**             | **Type** | **Description**                           |
| ---------------------- | -------- | ----------------------------------------- |
| `highlightScanRegion`  | Boolean  | Show a box around the scan region         |
| `highlightCodeOutline` | Boolean  | Highlight detected QR code outline        |
| `preferredCamera`      | String   | 'environment' (back) or 'user' (front)    |
| `maxScansPerSecond`    | Number   | Limit scan frequency (default: unlimited) |

## **🧱 Custom UI Components & Composables**

Saturn allows you to create reusable custom components and composables that become available in all dynamic components throughout the application.

**✨ Create Once, Use Everywhere!**&#x43;ustom components and composables you create are automatically registered and available in all dynamic components without any imports needed.

### **Accessing the UI Components Manager**

Navigate to**Settings → UI Components**to create and manage your custom components and composables.

### **Component Types**

| **Type**     | **Description**                                                    | **Use Case**                                      |
| ------------ | ------------------------------------------------------------------ | ------------------------------------------------- |
| `Component`  | A Vue Single File Component (SFC) with template, script, and style | Reusable UI elements like cards, buttons, widgets |
| `Composable` | A JavaScript function using Vue's Composition API                  | Shared logic, state management, utility functions |

### **Creating a Custom Component**

When you select**Component**as the type, you write a standard Vue SFC:

```
<template>
  <div class="p-4 border rounded-lg shadow-sm bg-white">
    <h3 class="text-lg font-semibold mb-2">{{ title }}</h3>
    <p class="text-gray-600">{{ description }}</p>
    <div class="mt-4">
      <slot></slot>
    </div>
    <div class="mt-4 flex gap-2">
      <el-button size="small" type="primary" @click="$emit('action')">
        {{ actionLabel }}
      </el-button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      default: 'Card Title'
    },
    description: {
      type: String,
      default: 'Card description goes here'
    },
    actionLabel: {
      type: String,
      default: 'Click Me'
    }
  },
  emits: ['action']
}
</script>

<style scoped>
/* Your custom styles */
</style>
```

### **Axis Tooltip Example**

```
<template>
  <v-chart
    :option="chartOptions"
    :autoresize="true"
    style="height: 320px;"
  />
</template>

<script>
export default {
  data() {
    return {
      chartOptions: {
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            type: 'line',
            data: [120, 132, 101, 134, 90],
            smooth: true
          }
        ]
      }
    };
  }
};
</script>
```

If you name this component`MyCard`, you can use it in any dynamic component:

```
<template>
  <div class="grid grid-cols-3 gap-4">
    <!-- Use your custom component by name -->
    <MyCard 
      title="Project Alpha" 
      description="A great project"
      actionLabel="View Details"
      @action="handleAction"
    >
      <p>Additional content via slot</p>
    </MyCard>
    
    <!-- Or use kebab-case -->
    <my-card title="Project Beta" />
  </div>
</template>
```

### **Creating a Custom Composable**

When you select**Composable**as the type, you write a JavaScript function that uses Vue's Composition API:

```
function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  const doubleCount = computed(() => count.value * 2);
  
  const increment = () => {
    count.value++;
  };
  
  const decrement = () => {
    count.value--;
  };
  
  const reset = () => {
    count.value = initialValue;
  };
  
  return {
    count,
    doubleCount,
    increment,
    decrement,
    reset
  };
}
```

**⚠️ Important:**&#x43;omposable definitions should be a function expression. Do not include`export`statements. The function name becomes the composable name.

#### **Using Custom Composables**

Access your composables via the`composables`object:

```
<template>
  <div class="p-4">
    <p>Count: {{ count }}</p>
    <p>Double: {{ doubleCount }}</p>
    <div class="flex gap-2 mt-4">
      <el-button @click="decrement">-</el-button>
      <el-button @click="increment">+</el-button>
      <el-button @click="reset">Reset</el-button>
    </div>
  </div>
</template>

<script>
export default {
  setup() {
    // Access your custom composable via the composables object
    const { count, doubleCount, increment, decrement, reset } = composables.useCounter(10);
    
    return {
      count,
      doubleCount,
      increment,
      decrement,
      reset
    };
  }
}
</script>
```

### **Advanced Composable Example**

Create a composable for fetching and managing resource data:

```
function useResourceList(resourceName, defaultParams = {}) {
  const items = ref([]);
  const loading = ref(false);
  const error = ref(null);
  const total = ref(0);
  
  const fetchItems = async (params = {}) => {
    loading.value = true;
    error.value = null;
    
    try {
      const resource = new Resource(null, resourceName);
      const response = await resource.list({ ...defaultParams, ...params });
      items.value = response.data || response;
      total.value = response.total || items.value.length;
    } catch (e) {
      error.value = e.message || 'Failed to fetch items';
    } finally {
      loading.value = false;
    }
  };
  
  const refresh = () => fetchItems();
  
  // Auto-fetch on creation
  onMounted(() => {
    fetchItems();
  });
  
  return {
    items,
    loading,
    error,
    total,
    fetchItems,
    refresh
  };
}
```

Then use it in your components:

```
<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error" class="text-red-500">{{ error }}</div>
    <div v-else>
      <p>Total: {{ total }}</p>
      <div v-for="item in items" :key="item.id">
        {{ item.name }}
      </div>
    </div>
    <el-button @click="refresh">Refresh</el-button>
  </div>
</template>

<script>
export default {
  setup() {
    const { items, loading, error, total, refresh } = composables.useResourceList('Project', { limit: 20 });
    
    return { items, loading, error, total, refresh };
  }
}
</script>
```

### **Available Vue Composition API**

The following Vue Composition API functions are available in composables:

**Reactivity:**

* `ref`,`reactive`
* `computed`
* `readonly`,`shallowRef`
* `shallowReactive`,`shallowReadonly`
* `toRef`,`toRefs`
* `triggerRef`,`customRef`
* `markRaw`,`toRaw`

**Lifecycle:**

* `onMounted`
* `onUnmounted`
* `onBeforeMount`
* `onBeforeUnmount`

**Watchers:**

* `watch`
* `watchEffect`

**Dependency Injection:**

* `provide`
* `inject`

**Type Guards:**

* `isRef`
* `isReactive`
* `isReadonly`
* `isProxy`

**💡 Best Practices:**

* Name components with PascalCase (e.g.,`MyCard`,`DataTable`)
* Name composables with "use" prefix (e.g.,`useCounter`,`useFetch`)
* Keep components focused on a single responsibility
* Use composables for shared logic that doesn't need a UI
* Components and composables are loaded on app startup, so they're always available

## **🔗 Custom Page Props**

Custom pages rendered through`/custom/{slug}`,`CustomPageRenderer`, or the custom page preview receive route context props. Use these to read query params from URLs such as`/custom/reports/summary?status=open&tag=a&tag=b`.

### **Available Props**

| **Prop**            | **Type** | **Description**                                                                      |
| ------------------- | -------- | ------------------------------------------------------------------------------------ |
| `queryParams`       | `object` | Normalized query params. Single values are strings; repeated keys are string arrays. |
| `pageSlug`          | `string` | The current custom page slug.                                                        |
| `customPageContext` | `object` | Route context with`slug`,`path`,`fullPath`, and`query`.                              |

### **Example with****`<script setup>`**

```
<template>
  <section class="p-6">
    <h1 class="text-xl font-semibold">{{ statusLabel }} reports</h1>
    <p class="text-gray-600">Slug: {{ pageSlug }}</p>
    <p class="text-gray-600">Tags: {{ tags.join(', ') || 'none' }}</p>
  </section>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  queryParams: {
    type: Object,
    default: () => ({})
  },
  pageSlug: {
    type: String,
    default: ''
  },
  customPageContext: {
    type: Object,
    default: () => ({})
  }
})

const statusLabel = computed(() => props.queryParams.status || 'all')
const tags = computed(() => {
  const value = props.queryParams.tag
  return Array.isArray(value) ? value : value ? [value] : []
})
</script>
```

### **Example with Options API**

```
<template>
  <div class="p-6">
    <pre>{{ queryParams }}</pre>
  </div>
</template>

<script>
export default {
  props: {
    queryParams: {
      type: Object,
      default: () => ({})
    },
    pageSlug: String,
    customPageContext: Object
  }
}
</script>
```

## **👤 User & Authentication**

The current authenticated user and token are automatically available:

| **Variable** | **Description**                   | **Type**                                            |
| ------------ | --------------------------------- | --------------------------------------------------- |
| `user`       | Current authenticated user object | Object with user properties (id, email, name, etc.) |
| `token`      | Authentication token              | String                                              |

### **Example**

```
<template>
  <div class="p-4">
    <p>Welcome, {{ user?.first_name }}!</p>
    <p>Email: {{ user?.email }}</p>
  </div>
</template>

<script>
export default {
  mounted() {
    console.log('User:', user);
    console.log('Token:', token);
  }
}
</script>
```

## **💡 Complete Example**

Here's a comprehensive example combining everything:

```
<template>
  <div class="p-6 bg-white rounded-lg shadow-lg">
    <h1 class="text-2xl font-bold text-brand-primary-color mb-4">
      My Projects Dashboard
    </h1>
    
    <div class="flex gap-4 mb-4">
      <el-input 
        v-model="searchTerm" 
        placeholder="Search projects..."
        class="w-64"
      />
      <el-button type="primary" @click="loadProjects">
        Refresh
      </el-button>
    </div>

    <div v-if="loading" class="text-center py-4">
      Loading...
    </div>

    <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <v-card v-for="project in filteredProjects" :key="project.id">
        <v-card-title>{{ project.name }}</v-card-title>
        <v-card-text>
          <p class="text-gray-600">{{ project.description }}</p>
        </v-card-text>
        <v-card-actions>
          <v-btn 
            size="small" 
            color="primary" 
            @click="viewProject(project.id)"
          >
            View Details
          </v-btn>
        </v-card-actions>
      </v-card>
    </div>
  </div>
</template>

<script>
import { ElMessage } from 'element-plus';

export default {
  data() {
    return {
      projects: [],
      searchTerm: '',
      loading: false
    }
  },
  computed: {
    filteredProjects() {
      if (!this.searchTerm) return this.projects;
      return this.projects.filter(p => 
        p.name.toLowerCase().includes(this.searchTerm.toLowerCase())
      );
    }
  },
  methods: {
    async loadProjects() {
      this.loading = true;
      try {
        const projectResource = new Resource(this, 'Project');
        this.projects = await projectResource.list({ limit: 50 });
        ElMessage.success('Projects loaded successfully!');
      } catch (error) {
        ElMessage.error('Failed to load projects');
      } finally {
        this.loading = false;
      }
    },
    viewProject(id) {
      ElMessage.info(`Viewing project ${id}`);
      // Navigate or show details
    }
  },
  mounted() {
    this.loadProjects();
  }
}
</script>
```

## **⚠️ Important Notes**

* **Async Operations:**&#x41;lways use`async/await`or`.then()`for API calls
* **Error Handling:**&#x57;rap API calls in try/catch blocks
* **Component Lifecycle:**&#x55;se`mounted()`for initial data loading
* **Reactivity:**&#x44;efine all reactive data in the`data()`function
* **Template Required:**&#x41;ll components must have a`<template>`section

### **Dynamic Component Form Field (dynamic-component)**

When you configure a resource property and choose the`Dynamic Component`input type (with`type: "dynamic-component"`in`input_property_types`), Saturn will:

* Use the`Dynamic Component`editor to let you maintain a Vue SFC string in`input_properties.component_definition`.
* Render that SFC at form runtime through a dedicated form field component that wraps`<dynamic-component>`and wires it to the form model via`v-model`.
* Pass useful context props such as`resourceDetail`,`property`,`options`,`options_data`,`utils`,`item`, and`page_loaded`into your dynamic component so you can build rich, context-aware form widgets.

Mapped string fields can also use`dynamic-component`. When a field would normally resolve to a mapped dropdown or inline resource manager, the explicit`input_properties.type = "dynamic-component"`setting takes precedence and renders the custom component instead.

Your dynamic component should follow the standard Vue 3`v-model`contract:

```
<template>
  <div class="w-full">
    <el-input
      v-model="localValue"
      placeholder="Enter a value"
      @input="emitChange"
    />
  </div>
</template>

<script>
export default {
  props: {
    // Context props passed from the host (resource/property context)
    resourceDetail: { type: Object },
    property: Object,
    options: { type: Object },
    options_data: { type: Object },
    utils: {
      type: Object,
      default: () => ({}),
    },
    item: {
      type: Object,
      description: 'Contains resource data',
      default: () => ({}),
    },
    page_loaded: {
      type: Boolean,
      default: false,
    },

    // Value bound from the form via v-model
    modelValue: {
      type: [String, Number, Boolean, Array, Object],
      default: null,
    },
  },

  emits: ['update:modelValue'],

  data() {
    return {
      localValue: this.modelValue,
    };
  },

  watch: {
    modelValue(newVal) {
      this.localValue = newVal;
    },
  },

  methods: {
    emitChange() {
      this.$emit('update:modelValue', this.localValue);
    },
  },
}
</script>
```

#### **Handling Async Dependent Data**

Some fields are resolved after the first render (for example dynamic source lookups). Use`page_loaded`(or`utils.page_loaded`) before reading deep properties from`item`.

```
<template>
  <div v-if="!page_loaded">Loading related data...</div>
  <div v-else>
    {{ item?.dependents?.length || 0 }} dependents
  </div>
</template>
```

**✅ Result:**&#x54;he form field will behave like any built-in input component (text, select, etc.), with two-way binding between the form model and your dynamic component.

#### **Use @input for real-time form modeling**

* Many UI libraries (including Element Plus) only fire`@change`when the input loses focus. This means the form value updates*after*the user clicks away.
* For dynamic**form fields**you almost always want the form model to update as the user types. Prefer`@input`(or the library's equivalent for "on each keystroke") inside your dynamic component, and emit`update:modelValue`from there.
* Pattern: bind your visual control with`v-model="localValue"`and call a method like`emitChange()`on`@input`to propagate the latest value back to the host form.

## **📚 Additional Resources**

* [Tailwind CSS Documentation](https://tailwindcss.com/docs)
* [Element Plus Documentation](https://element-plus.org/)
* [Vuetify Documentation](https://vuetifyjs.com/)
* [Vue 3 Documentation](https://vuejs.org/guide)
