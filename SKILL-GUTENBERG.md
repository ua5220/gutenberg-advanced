---
name: gutenberg-blocks
description: |
  Master SKILL for WordPress Gutenberg block development.
  Routes to:
  - 40+ Components (Block Editor + WordPress)
  - Data Store Management (useSelect, useDispatch)
  - Custom Data Stores
  - Block Development Guide
  - Best Practices & Patterns
---

# Gutenberg Block Development Master Router

**Комплексний маршрутизатор для всіх аспектів Gutenberg розробки.**

---

## 🚀 Quick Start (30 seconds)

**Виберіть потрібного гайда:**

| Що робити? | Перейти на | Килькість |
|---|---|---|
| **Components & UI** | [COMPONENTS-API.md](./COMPONENTS-API.md) | 40+ |
| **Data & State** | [DATA-STORE.md](./DATA-STORE.md) | useSelect, useDispatch |
| **Block Development** | Below | Full guide |
| **Best Practices** | Below | 10 rules |

---

## 🗐️ 1) COMPONENTS & CONTROLS

### 📦 Block Editor Components (9)

**Import from `@wordpress/block-editor`:**

1. **useBlockProps** - Required for all blocks
   ```javascript
   const blockProps = useBlockProps();
   return <div {...blockProps}>Content</div>;
   ```

2. **RichText** - Text editing
   ```javascript
   <RichText
     tagName="p"
     value={content}
     onChange={(value) => setAttributes({ content: value })}
   />
   ```

3. **InspectorControls** - Right panel settings
   ```javascript
   <InspectorControls>
     <PanelBody>Controls</PanelBody>
   </InspectorControls>
   ```

4. **BlockControls** - Toolbar above block
5. **MediaUpload** - Image/video selector
6. **InnerBlocks** - Nested blocks support
7. **URLInput** - URL field
8. **PanelColorSettings** - Color picker
9. **AlignmentToolbar** - Text alignment

**🔗 Reference:** [COMPONENTS-API.md#block-editor-components](./COMPONENTS-API.md#block-editor-components)

### 📋 WordPress Components (20+)

**Import from `@wordpress/components`:**

**Text Controls:**
- TextControl, TextareaControl, URLInput

**Selection Controls:**
- SelectControl, RadioControl, CheckboxControl, ToggleControl

**Range Controls:**
- RangeControl, FontSizePicker

**Color Controls:**
- ColorPicker, ColorPalette, PanelColorSettings

**Layout Controls:**
- Button, Icon, Modal, Notice, Spinner, Placeholder, Popover

**Advanced Controls:**
- TabPanel, DateTimePicker, FormTokenField

**🔗 Reference:** [COMPONENTS-API.md#wordpress-components](./COMPONENTS-API.md#wordpress-components)

---

## 📄 2) DATA & STATE MANAGEMENT

### useSelect Hook (Get Data) ✅

```javascript
const posts = useSelect((select) => 
  select(coreStore).getEntityRecords('postType', 'post')
);
```

**3 Main Patterns:**
1. Get entity records (posts, users, categories)
2. Get editor state (current post, selected block)
3. Get loading/saving status

### useDispatch Hook (Modify Data) ✅

```javascript
const { saveEntityRecord } = useDispatch(coreStore);
await saveEntityRecord('postType', 'post', { id: 1, title: 'New' });
```

**Common Actions:**
1. saveEntityRecord - Save post/data
2. deleteEntityRecord - Delete post/data
3. editEntityRecord - Stage changes
4. insertBlock - Add block
5. updateBlockAttributes - Change block properties

### Core Stores (5)

| Store | Purpose | Usage |
|-------|---------|-------|
| `core` | Posts, users, taxonomies | `select(coreStore).getEntityRecords(...)` |
| `core/block-editor` | Block editor state | `select('core/block-editor').getSelectedBlock()` |
| `core/editor` | Post editor state | `select('core/editor').getCurrentPost()` |
| `core/notices` | Admin notifications | `dispatch('core/notices').createNotice(...)` |
| `core/preferences` | User preferences | `select('core/preferences').get(...)` |

### Custom Data Stores ✅

```javascript
const store = createReduxStore('my-plugin/store', {
  reducer: (state = {}, action) => state,
  actions: { setItem: (item) => ({ type: 'SET', item }) },
  selectors: { getItem: (state) => state.item },
});

register(store);
```

**🔗 Reference:** [DATA-STORE.md](./DATA-STORE.md)

---

## 🔨 3) BLOCK DEVELOPMENT

### Block Structure

```
my-block/
├── block.json          ✅ Metadata
├── index.js            ✅ Registration
├── edit.js             ✅ Editor component
├── save.js             ✅ Frontend save
├── render.php          ✅ Server-side render
├── style.scss          ✅ Frontend + editor
├── editor.scss         ✅ Editor only
└── view.js             ✅ Frontend JavaScript
```

### block.json Template

```json
{
  "apiVersion": 3,
  "name": "my-plugin/my-block",
  "title": "My Block",
  "category": "widgets",
  "icon": "smiley",
  "attributes": {
    "content": { "type": "string", "source": "html" },
    "columns": { "type": "number", "default": 3 }
  },
  "supports": {
    "html": false,
    "align": ["wide", "full"],
    "color": { "background": true, "text": true }
  },
  "editorScript": "file:./index.js",
  "editorStyle": "file:./editor.css",
  "style": "file:./style.css"
}
```

### Edit Component

```javascript
import { useBlockProps, RichText, InspectorControls } from '@wordpress/block-editor';
import { PanelBody, RangeControl } from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
  const blockProps = useBlockProps();
  const { content, columns } = attributes;

  return (
    <>
      <InspectorControls>
        <PanelBody>
          <RangeControl
            label="Columns"
            value={columns}
            onChange={(value) => setAttributes({ columns: value })}
            min={1}
            max={6}
          />
        </PanelBody>
      </InspectorControls>
      <div {...blockProps}>
        <RichText
          tagName="p"
          value={content}
          onChange={(value) => setAttributes({ content: value })}
        />
      </div>
    </>
  );
}
```

### Save Component

```javascript
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function Save({ attributes }) {
  const blockProps = useBlockProps.save();
  const { content } = attributes;

  return (
    <div {...blockProps}>
      <RichText.Content tagName="p" value={content} />
    </div>
  );
}
```

### Dynamic Block (Server-Side Render)

```php
<?php
// render.php - Server-side rendering
echo wp_kses_post( $attributes['content'] );
?>
```

```javascript
// save.js - Return null for dynamic blocks
export default function Save() {
  return null;
}
```

---

## 🌟 4) INNERBLOCKS (CONTAINER BLOCKS)

```javascript
import { InnerBlocks } from '@wordpress/block-editor';

// Edit
export function Edit() {
  return (
    <InnerBlocks
      allowedBlocks={['core/paragraph', 'core/heading']}
      template={[
        ['core/heading', { level: 2 }],
        ['core/paragraph', {}],
      ]}
      templateLock="all" // 'all', 'insert', false
    />
  );
}

// Save
export function Save() {
  return <InnerBlocks.Content />;
}
```

---

## 📊 5) BEST PRACTICES

### ✅ DO's (10 Rules)

1. **Always use useBlockProps** - Required for accessibility
2. **Escape output** - Use `wp_kses_post()`, `esc_html()`
3. **Use semantic HTML** - Proper structure for accessibility
4. **Support native features** - align, color, spacing, typography
5. **Add example in block.json** - Help users understand
6. **Use InspectorControls** - Settings in right panel
7. **Lazy load heavy components** - Improve editor performance
8. **Validate attributes** - Check data before use
9. **Use i18n for strings** - `__()`, `_e()`, `_x()`
10. **Document block features** - Help/instructions for users

### ❌ DON'Ts (Avoid)

1. **Don't directly modify attributes** - Use `setAttributes()`
2. **Don't hardcode data** - Use WP REST API
3. **Don't skip sanitization** - Always sanitize output
4. **Don't make CSS too specific** - Cause conflicts
5. **Don't ignore browser support** - Test in multiple browsers
6. **Don't add inline styles** - Use classes instead
7. **Don't fetch data in render** - Use data hooks
8. **Don't forget mobile responsiveness** - Test mobile views
9. **Don't ignore accessibility** - Add aria labels, alt text
10. **Don't skip error handling** - Handle API failures gracefully

---

## 🖱️ API FETCH PATTERNS

```javascript
import apiFetch from '@wordpress/api-fetch';
import { addQueryArgs } from '@wordpress/url';

// GET posts
const posts = await apiFetch({
  path: '/wp/v2/posts?per_page=10'
});

// POST new post
const newPost = await apiFetch({
  path: '/wp/v2/posts',
  method: 'POST',
  data: { title: 'New', status: 'draft' }
});

// With query args
const result = await apiFetch({
  path: addQueryArgs('/wp/v2/posts', { per_page: 5 })
});

// Custom endpoint
const data = await apiFetch({
  path: '/my-plugin/v1/custom',
  method: 'POST',
  data: { key: 'value' }
});
```

---

## 📚 Complete File Structure

| Файл | Кількість | Покриття |
|------|------|-------|
| [COMPONENTS-API.md](./COMPONENTS-API.md) | 40+ | Components, controls, forms |
| [DATA-STORE.md](./DATA-STORE.md) | 50+ | Stores, hooks, API fetch |
| [SKILL-GUTENBERG.md](./SKILL-GUTENBERG.md) | Full | This file |

---

## 🎓 Learning Path (Beginner to Expert)

### 🤜 Level 1: Beginner (30 min)
- Read block.json structure
- Create simple text block
- Use useBlockProps
- Use RichText component

### 🙋 Level 2: Intermediate (1-2 hours)
- Add InspectorControls with TextControl
- Use SelectControl & RangeControl
- Add MediaUpload
- Create container block with InnerBlocks

### 😩 Level 3: Advanced (2-4 hours)
- Use useSelect & useDispatch
- Fetch post data from API
- Create custom data store
- Dynamic block with server-side render

### 🦵 Level 4: Expert (4+ hours)
- Complex nested blocks
- Performance optimization
- Advanced state management
- Block variations & patterns

---

## 📄 Common Questions

**"Як надати кнопки до toolbar?"**  
→ [BlockControls](./COMPONENTS-API.md#blockcontrols-) + ToolbarButton

**"Як скористати apiFetch?"**  
→ [API Fetch Patterns](#-api-fetch-patterns) above

**"Чим useSelect відничається від useQuery?"**  
→ useSelect для WordPress stores, useQuery для REST API

**"Як обробити InnerBlocks?"**  
→ [INNERBLOCKS](#-4-innerblocks-container-blocks) section

**"Коли мні потрібна render.php?"**  
→ Для dynamic blocks (данні з API, яко зберегаються динамічно)

---

## 📈 Statistics

| Метрика | Значення |
|---------|----------|
| **Components** | 40+ ✅ |
| **Stores** | 5 |
| **Hooks** | 3 (useSelect, useDispatch, useEntityProp) |
| **Code Examples** | 20+ |
| **Best Practices** | 20 (10 DO, 10 DON'T) |
| **Total Documentation** | ~30 KB |

---

## 🚛 Status

✅ 40+ components documented  
✅ 5 core stores covered  
✅ 3 data hooks explained  
✅ 20+ code examples  
✅ Complete block development guide  
✅ Best practices & patterns  
✅ Learning path (beginner to expert)  
✅ 100% ready for production  

---

**Версія:** 1.0 Gutenberg Master Router  
**Дата:** 11 січня 2026  
**Статус:** ✅ 100% готово до використання
