# Gutenberg Data & State Management

**Повний гайд WordPress data stores, useSelect, useDispatch, і custom store рестрації.**

---

## 🚀 Quick Start (20 seconds)

```javascript
import { useSelect, useDispatch } from '@wordpress/data';
import { store as coreStore } from '@wordpress/core-data';

// Get data
const posts = useSelect((select) => 
  select(coreStore).getEntityRecords('postType', 'post')
);

// Modify data
const { saveEntityRecord } = useDispatch(coreStore);
await saveEntityRecord('postType', 'post', { id: 1, title: 'Updated' });
```

---

## 📑 Core Stores

| Store | Purpose | Import |
|-------|---------|--------|
| `core` | Posts, users, taxonomies | `store as coreStore` from `@wordpress/core-data` |
| `core/block-editor` | Block editor state | `store as blockEditorStore` from `@wordpress/block-editor` |
| `core/editor` | Post editor state | `store as editorStore` from `@wordpress/editor` |
| `core/notices` | Admin notices | `store as noticesStore` from `@wordpress/notices` |
| `core/preferences` | User preferences | `'core/preferences'` as string |

---

## 🕍 useSelect Hook (Get Data)

### Get Posts

```javascript
import { useSelect } from '@wordpress/data';
import { store as coreStore } from '@wordpress/core-data';

const posts = useSelect((select) => {
	return select(coreStore).getEntityRecords('postType', 'post', {
		per_page: 10,
		status: 'publish',
		orderby: 'date',
		order: 'desc',
	});
}, []);
```

### Get Current Post

```javascript
const { currentPost, isAutosaving, isSaving } = useSelect((select) => {
	const editor = select('core/editor');
	return {
		currentPost: editor.getCurrentPost(),
		isAutosaving: editor.isAutosavingPost(),
		isSaving: editor.isSavingPost(),
	};
}, []);
```

### Get Selected Block

```javascript
const { selectedBlock, selectedBlockClientId } = useSelect((select) => {
	const blockEditor = select('core/block-editor');
	return {
		selectedBlock: blockEditor.getSelectedBlock(),
		selectedBlockClientId: blockEditor.getSelectedBlockClientId(),
	};
}, []);
```

### Get Categories & Tags

```javascript
const { categories, tags } = useSelect((select) => {
	const { getEntityRecords } = select(coreStore);
	return {
		categories: getEntityRecords('taxonomy', 'category'),
		tags: getEntityRecords('taxonomy', 'post_tag'),
	};
}, []);
```

### Get Users

```javascript
const users = useSelect((select) => {
	return select(coreStore).getEntityRecords('root', 'user', {
		per_page: -1,
	});
}, []);
```

### Get Media

```javascript
const media = useSelect((select) => {
	return select(coreStore).getEntityRecords('postType', 'attachment', {
		per_page: 20,
		media_type: 'image',
	});
}, []);
```

### Check Loading State

```javascript
const { posts, isLoading } = useSelect((select) => {
	const { getEntityRecords, hasFinishedResolution } = select(coreStore);
	return {
		posts: getEntityRecords('postType', 'post'),
		isLoading: !hasFinishedResolution('getEntityRecords', [
			'postType',
			'post',
		]),
	};
}, []);
```

---

## 📄 useDispatch Hook (Modify Data)

### Save Post

```javascript
import { useDispatch } from '@wordpress/data';
import { store as coreStore } from '@wordpress/core-data';

const { saveEntityRecord } = useDispatch(coreStore);

const handleSave = async () => {
	try {
		await saveEntityRecord('postType', 'post', {
			id: postId,
			title: 'New Title',
			content: 'New Content',
		});
		createSuccessNotice('Saved!');
	} catch (error) {
		createErrorNotice('Error: ' + error.message);
	}
};
```

### Delete Post

```javascript
const { deleteEntityRecord } = useDispatch(coreStore);

const handleDelete = async () => {
	await deleteEntityRecord('postType', 'post', postId, {
		force: true, // Skip trash
	});
};
```

### Edit Post (Stage Changes)

```javascript
const { editEntityRecord } = useDispatch(coreStore);

// Stage changes without saving
editEntityRecord('postType', 'post', postId, {
	title: 'Updated Title',
});
```

### Create Notices

```javascript
const { createSuccessNotice, createErrorNotice } = 
  useDispatch('core/notices');

createSuccessNotice('Success!', {
	type: 'snackbar',
	dismissible: true,
});

createErrorNotice('Error!', {
	type: 'snackbar',
});
```

### Insert Block

```javascript
import { createBlock } from '@wordpress/blocks';

const { insertBlock } = useDispatch('core/block-editor');

const newBlock = createBlock('core/paragraph', {
	content: 'New block content',
});

insertBlock(newBlock, index, rootClientId);
```

### Update Block Attributes

```javascript
const { updateBlockAttributes } = 
  useDispatch('core/block-editor');

updateBlockAttributes(clientId, {
	content: 'Updated content',
	alignment: 'center',
});
```

---

## 🎅 Block Editor Selectors

```javascript
const blockEditor = select('core/block-editor');

// Selected blocks
blockEditor.getSelectedBlock();
blockEditor.getSelectedBlockClientId();
blockEditor.getSelectedBlockClientIds();
blockEditor.hasSelectedBlock();

// Block data
blockEditor.getBlock(clientId);
blockEditor.getBlocks();
blockEditor.getBlockCount();
blockEditor.getBlockName(clientId);
blockEditor.getBlockAttributes(clientId);
blockEditor.getBlockIndex(clientId);

// Hierarchy
blockEditor.getBlockParents(clientId);
blockEditor.getBlockRootClientId(clientId);
blockEditor.getBlockOrder(clientId);
blockEditor.getClientIdsOfDescendants([clientId]);

// Insertion
blockEditor.getInserterItems();
blockEditor.canInsertBlockType('core/paragraph', rootClientId);
```

---

## 📐 Editor Selectors

```javascript
const editor = select('core/editor');

// Current post
editor.getCurrentPost();
editor.getCurrentPostId();
editor.getCurrentPostType();
editor.getCurrentPostAttribute('title');
editor.getEditedPostAttribute('title');
editor.getEditedPostContent();

// Status
editor.isEditedPostDirty();
editor.isEditedPostSaveable();
editor.isEditedPostPublishable();
editor.isCurrentPostPublished();
editor.isCurrentPostScheduled();

// Saving
editor.isSavingPost();
editor.isAutosavingPost();
editor.isPublishingPost();

// Meta
editor.getEditedPostAttribute('meta');
```

---

## 🖱️ Core Data Selectors

### Get Entity Records

```javascript
const { getEntityRecords, getEntityRecord } = select(coreStore);

// Multiple records
const posts = getEntityRecords('postType', 'post', {
	per_page: 10,
	page: 1,
	status: 'publish',
	search: 'query',
	orderby: 'date',
	order: 'desc',
	_embed: true,
});

// Single record
const post = getEntityRecord('postType', 'post', postId);

// Check if loaded
const isLoading = !hasFinishedResolution('getEntityRecords', [
	'postType',
	'post',
]);
```

### Taxonomies

```javascript
const categories = getEntityRecords('taxonomy', 'category');
const tags = getEntityRecords('taxonomy', 'post_tag');
const customTax = getEntityRecords('taxonomy', 'my_custom_tax');
```

### Site Data

```javascript
const siteSettings = getEntityRecord('root', 'site');
const users = getEntityRecords('root', 'user', { per_page: -1 });
```

---

## 📚 useEntityProp Hook

### Edit Post Title

```javascript
import { useEntityProp } from '@wordpress/core-data';

function PostTitleEditor() {
	const [title, setTitle] = useEntityProp(
		'postType',
		'post',
		'title',
		postId
	);

	return (
		<input
			value={title}
			onChange={(e) => setTitle(e.target.value)}
		/>
	);
}
```

### Edit Meta Fields

```javascript
function PostMetaEditor() {
	const [meta, setMeta] = useEntityProp('postType', 'post', 'meta');

	const updateMeta = (key, value) => {
		setMeta({ ...meta, [key]: value });
	};

	return (
		<input
			value={meta?.my_field || ''}
			onChange={(e) => updateMeta('my_field', e.target.value)}
		/>
	);
}
```

---

## 🌙 Custom Data Store

```javascript
import { createReduxStore, register } from '@wordpress/data';

const DEFAULT_STATE = {
	items: [],
	isLoading: false,
};

// Actions
const actions = {
	setItems(items) {
		return { type: 'SET_ITEMS', items };
	},
	addItem(item) {
		return { type: 'ADD_ITEM', item };
	},
};

// Selectors
const selectors = {
	getItems(state) {
		return state.items;
	},
	getItemById(state, id) {
		return state.items.find((item) => item.id === id);
	},
};

// Reducer
const reducer = (state = DEFAULT_STATE, action) => {
	switch (action.type) {
		case 'SET_ITEMS':
			return { ...state, items: action.items };
		case 'ADD_ITEM':
			return { ...state, items: [...state.items, action.item] };
		default:
			return state;
	}
};

// Create & register store
const store = createReduxStore('my-plugin/store', {
	reducer,
	actions,
	selectors,
});

register(store);

// Usage
const items = useSelect((select) => 
  select('my-plugin/store').getItems()
);
const { setItems } = useDispatch('my-plugin/store');
```

---

## 🌐 API Fetch

```javascript
import apiFetch from '@wordpress/api-fetch';
import { addQueryArgs } from '@wordpress/url';

// GET
const posts = await apiFetch({
	path: '/wp/v2/posts',
});

// POST
const newPost = await apiFetch({
	path: '/wp/v2/posts',
	method: 'POST',
	data: {
		title: 'New Post',
		content: 'Content',
		status: 'draft',
	},
});

// Query params
const result = await apiFetch({
	path: addQueryArgs('/wp/v2/posts', {
		per_page: 10,
		status: 'publish',
	}),
});

// Custom endpoint
const custom = await apiFetch({
	path: '/my-plugin/v1/endpoint',
	method: 'POST',
	data: { key: 'value' },
});
```

---

## 📊 Stats

| Category | Count |
|----------|-------|
| **Core Stores** | 5 ✅ |
| **Hooks** | 3 (useSelect, useDispatch, useEntityProp) |
| **Selectors** | 20+ |
| **Actions** | 15+ |
| **API Methods** | 5+ |

---

**Версія:** 1.0 WordPress Data Stores  
**Дата:** 11 січня 2026  
**Статус:** ✅ 100% готово
