## CSS guidelines for the project

## 1. Basic Rules

This project uses Tailwind CSS as the primary styling solution, adhering to the "utility-first CSS" design philosophy. All developers must follow these guidelines to ensure code consistency and maintainability.

For third-party component libraries, use the recommended methods to modify styles. For example, when using Element Plus, apply deep selectors (::v-deep) for custom styling adjustments.

### 1.1 Prefer Tailwind Utility Classes

Always prioritize using Tailwind's utility classes for styling to maintain consistency, improve readability, and leverage Tailwind's optimization features.

✅ Recommended:

Use Tailwind's utility classes for layout, spacing, colors, and effects.

```html
<div class="flex items-center justify-between p-4 bg-white shadow rounded-lg">
  <!-- Content -->
</div>
```

❌ Not Recommended:

Avoid creating unnecessary custom classes for styles that can be handled by Tailwind's utility classes.

```html
<div class="custom-card">
  <!-- Content -->
</div>
```

### 1.2 Atomic Class Ordering

To maintain code readability, utility classes in the class attribute should follow this order:

Layout (e.g., display, position, top, left, z-index)

Flexbox/Grid (e.g., flex, grid, gap, justify, items)

Sizing (e.g., w-, h-, min-w-, max-h-)

Spacing (e.g., m-, p-, space-)

Background (e.g., bg-, bg-opacity-)

Borders (e.g., border-, rounded-)

Typography (e.g., text-, font-, tracking-)

State Variants (e.g., hover:, focus:, active:)

Miscellaneous (e.g., opacity-, shadow-, transition-)

✅ Recommended

```html
<button
  class="relative flex items-center w-full py-2 px-4 mb-3 bg-blue-500 hover:bg-blue-600 rounded-md text-white font-medium transition-colors"
>
  Submit
</button>
```

### 1.3 Responsive Design

Place responsive prefixes (sm:, md:, lg:, xl:, 2xl:) before the related property for better readability.

✅ Recommended

```html
<div class="w-full md:w-1/2 lg:w-1/3">
  <!-- Content -->
</div>
```

## 2 Styling Third-Party Components

### 2.1 Using Deep Selectors

When modifying third-party component styles, use deep selectors (:deep() or ::v-deep).

✅ Vue 3 SFC Example

```vue
<style lang="scss" scoped>
.custom-component {
  :deep(.el-input) {
    border-radius: 8px;

    .el-input__inner {
      height: 48px;
    }
  }
}
</style>
```

### 2.2 Avoid Global Style Pollution

Avoid global overrides; instead, scope styles within components.

❌ Not Recommended

```scss
:global(.ant-btn) {
  border-radius: 4px;
}
```

✅ Recommended

```scss
.my-page :deep(.ant-btn) {
  border-radius: 4px;
}
```
