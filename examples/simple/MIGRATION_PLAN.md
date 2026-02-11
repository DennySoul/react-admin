# Posts Page UI Migration Plan

## Context

The react-admin simple example app needs a visual redesign to match new Figma designs at `https://www.figma.com/make/j59a6zWiCyi0d5TpFG7BcA/Workshop`. The goal is to migrate the Posts page -- including the surrounding layout (theme, sidebar, header) -- to the new branded UI. All existing features must be preserved but restyled.

**Design key changes**: Blue primary color (#2563eb), Inter font, rounded corners (12px), white AppBar with search/notifications, branded sidebar ("Blog Admin") with user profile, redesigned data table with Status icons and explicit action buttons, centered MUI Pagination, responsive card layout on mobile.

---

## Step 1: Create Custom MUI Theme

**Create** `src/theme.ts`

Following the pattern from `packages/ra-ui-materialui/src/theme/radiantTheme.ts`, create a theme that extends `defaultLightTheme` via `deepmerge`:

- **Palette**: primary `#2563eb`, secondary `#64748b`, background default `#f8fafc` / paper `#ffffff`
- **Typography**: fontFamily `"Inter", "Roboto", "Helvetica", "Arial", sans-serif`
- **Shape**: borderRadius `12`
- **Sidebar**: width `260`
- **Component overrides**:
  - `MuiButton` → `textTransform: 'none'`
  - `RaLayout` → `.RaLayout-appFrame` marginTop adjusted for AppBar height
  - `MuiAppBar` → colorSecondary matches white background design

**Modify** `src/index.tsx` -- add `theme={lightTheme}` and `darkTheme={null}` to `<Admin>`

**Modify** `index.html`:
- Add Inter font to WebFontConfig: `['Inter:400,500,600,700:latin', 'Roboto:300,400,500,700:latin']`
- Update body background-color to `#f8fafc`

**Verification**: App loads with blue primary, Inter font, rounded corners. All pages still functional.

---

## Step 2: Create Custom AppBar (Header)

**Create** `src/Header.tsx`

Build a custom AppBar component matching the Figma header:
- White background (`bgcolor: 'background.paper'`), subtle box-shadow, `zIndex: drawer + 1`
- Search bar using styled `InputBase` (hidden on xs, shown on sm+)
- `LocalesMenuButton` from react-admin (reuses existing i18nProvider for EN/FR)
- Dark mode toggle icon (cosmetic for now)
- Notification bell with `<Badge badgeContent={3}>`
- Settings gear icon
- `SidebarToggleButton` on mobile for hamburger menu
- `LoadingIndicator` from react-admin

**Reusable imports from react-admin**: `LocalesMenuButton`, `SidebarToggleButton`, `LoadingIndicator`

---

## Step 3: Create Custom Sidebar Menu

**Create** `src/MySidebar.tsx`

Custom Menu component rendered inside react-admin's Sidebar wrapper:
- Brand section: "Blog Admin" (h6, fontWeight 600) + "Content Management" (caption)
- "Create Post" button (full-width, outlined/contained based on current route) linked to `/posts/create`
- Nav items using `<Menu.Item>` from react-admin: Posts (ArticleIcon), Comments (CommentIcon), Tags (LocalOfferIcon)
- Keep Users resource registered but hidden from nav (matching Figma)
- User profile at bottom: Avatar with gradient `linear-gradient(135deg, #667eea, #764ba2)`, name, role
- Use `useGetIdentity()` from react-admin for user data (fallback to "John Doe" / "Admin")

**Reusable from react-admin**: `Menu.Item`, `useGetIdentity`, `useCreatePath`

---

## Step 4: Update Layout Integration

**Modify** `src/Layout.tsx`

Wire together the new components:
```
<Layout appBar={Header} menu={MySidebar}>
    {children}
</Layout>
```

Keep `ReactQueryDevtools` as-is.

**Verification**: Full layout renders -- white AppBar at top, 260px branded sidebar on left, content area. Mobile shows hamburger menu with temporary drawer.

---

## Step 5: Redesign PostList -- Desktop Table

**Modify** `src/posts/PostList.tsx`

### 5a. Custom Page Header (replaces TopToolbar actions)

Create `PostListActions` component as custom `actions` prop for `<List>`:
- Left: "Posts" (h4, fontWeight 600) + "Manage and organize your blog posts" (body2)
- Right: `<ExportButton variant="outlined" />` + `<CreateButton variant="contained" icon={<AddIcon />} label="Create Post" />`
- Responsive: row on sm+, column on xs
- Also include `<FilterButton />` and `<ColumnsButton />` to preserve existing features
- Set `title={false}` on `<List>` to suppress default title

### 5b. Restyle DataTable Columns

Keep all existing columns but restyle to match Figma:

| Column | Change |
|--------|--------|
| `id` | Same |
| `title` | Add `fontWeight: 500` |
| `published_at` | Remove italic style |
| `nb_comments` | Same (ReferenceManyCount) |
| `commentable` | Rename label to "Status", add custom `StatusField` (CheckCircleIcon green / CancelIcon disabled) replacing BooleanField |
| `views` | Same |
| `tags` | Same (ReferenceArrayField + ChipField) |
| `average_note` | Keep but in `hiddenColumns` (preserved, not visible by default, accessible via ColumnsButton) |
| Actions col | Replace `EditButton`+`ShowButton` with icon buttons: View (VisibilityIcon), Edit (EditIcon), Delete (DeleteButton icon-only), MoreVert (IconButton) |

### 5c. Preserve Features (restyled)

- **Expand panel** (`PostPanel`): Keep -- still shows post body HTML on row expand
- **rowClick**: Keep existing logic (commentable → edit, else → show)
- **Bulk actions**: Keep all three (`ResetViewsButton`, `BulkDeleteButton`, `BulkExportButton`)
- **Filters**: Keep `postFilter` array (SearchInput, TextInput, QuickFilter)
- **Exporter**: Keep existing `exporter` function
- **hiddenColumns**: Move `average_note` here (already was hidden)

### 5d. Custom Pagination

Create `PostPagination` component:
- Uses `useListPaginationContext()` from react-admin for `page`, `perPage`, `total`, `setPage`
- Renders MUI `<Pagination>` (centered, `color="primary"`, `shape="rounded"`)
- Pass as `pagination={<PostPagination />}` to `<List>`

---

## Step 6: Redesign PostList -- Mobile Card Layout

**Modify** `src/posts/PostList.tsx` (same file)

Replace `InfiniteList` + `SimpleList` with `List` + custom card renderer:
- Use standard `<List>` (not InfiniteList) with same `PostPagination` for consistency
- Create `PostCardList` using `useListContext()` to iterate records
- Each card: `<Card variant="outlined">` with:
  - Title (subtitle2, fontWeight 600), date + ID, views + comments count, status icon, tag chips
  - `<CardActions>`: View, Edit, Delete buttons
- Wrap each record in `<RecordContextProvider>` so react-admin fields work

**Preserve**: filters, exporter, sort, bulk actions (via checkboxes in cards)

---

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `src/theme.ts` | CREATE | Custom MUI theme |
| `src/Header.tsx` | CREATE | Custom AppBar with search/notifications |
| `src/MySidebar.tsx` | CREATE | Branded sidebar with nav + user profile |
| `src/Layout.tsx` | MODIFY | Wire new AppBar + Sidebar |
| `src/index.tsx` | MODIFY | Add theme prop to Admin |
| `src/posts/PostList.tsx` | MODIFY | Redesign table, actions, pagination, mobile |
| `index.html` | MODIFY | Add Inter font, update background color |

**Untouched**: PostCreate, PostEdit, PostShow, PostTitle, TagReferenceInput, ResetViewsButton, dataProvider, authProvider, i18nProvider, all comment/tag/user files.

---

## Key References

- Theme pattern: `packages/ra-ui-materialui/src/theme/radiantTheme.ts`
- Layout architecture: `packages/ra-ui-materialui/src/layout/Layout.tsx`
- Figma source code: `file://figma/make/source/j59a6zWiCyi0d5TpFG7BcA/src/app/components/` (PostsView.tsx, Sidebar.tsx, Header.tsx, App.tsx)

---

## Verification

1. `make run-simple` -- app starts without errors
2. Visual check: Posts page matches Figma screenshot (white AppBar, branded sidebar, blue primary, table with Status icons + action buttons, centered pagination)
3. Feature check: all existing features still work:
   - Click row → navigates to edit/show based on commentable
   - Expand arrow → shows post body HTML
   - Search filter → filters by text
   - QuickFilter chip → filters by commentable
   - ColumnsButton → toggle columns including average_note
   - Bulk select → ResetViews, Delete, Export buttons appear
   - Export button → downloads CSV
   - Create Post → navigates to create form
   - Pagination → pages through results
   - Mobile breakpoint → shows card layout
4. Login/logout still works (authProvider untouched)
5. Language switch (EN/FR) works via AppBar LocalesMenuButton
