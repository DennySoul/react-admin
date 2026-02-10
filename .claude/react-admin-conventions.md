# React-Admin Project Conventions

## Framework Patterns

When working in react-admin projects:

### Forms
- Use `<SimpleForm>` for edit/create views, never raw HTML forms
- Use `<TabbedForm>` when form has 4+ logical sections
- Always wrap form inputs in react-admin input components (`<TextInput>`, `<DateInput>`, etc.)
- Form validation uses react-hook-form under the hood — use `validate` prop on inputs

### Data Display
- Use `<ReferenceField>` for foreign key relationships, never manual lookups
- Use `<ReferenceManyField>` for one-to-many relationships
- Always include `<Pagination>` in list views
- Prefer `<Datagrid>` for desktop, `<SimpleList>` for mobile

### State Management
- Use `useRecordContext()` to access current record in custom components
- Use `useDataProvider()` for custom data operations
- Never call fetch/axios directly — always go through dataProvider

### Styling
- Use MUI `sx` prop for one-off styles
- Use theme overrides in `src/theme.ts` for global styles
- Never use inline `style` prop or CSS-in-JS libraries

### TypeScript
- All components must have explicit prop types
- Use react-admin's built-in types: `RaRecord`, `Identifier`, `DataProvider`
- Export interfaces for custom record types in `src/types.ts`

## File Structure

```
src/
├── components/      # Reusable UI components
├── resources/       # One folder per resource (posts/, comments/, users/)
│   └── posts/
│       ├── PostList.tsx
│       ├── PostEdit.tsx
│       ├── PostCreate.tsx
│       └── index.ts
├── providers/       # dataProvider, authProvider
├── theme.ts         # MUI theme customization
└── types.ts         # Shared TypeScript interfaces
```

## Code Examples

### Correct List Component
```tsx
export const PostList = () => (
  <List>
    <Datagrid rowClick="edit">
      <TextField source="title" />
      <ReferenceField source="author_id" reference="users" />
      <DateField source="created_at" />
      <EditButton />
    </Datagrid>
  </List>
);
```

### Correct Edit Component
```tsx
export const PostEdit = () => (
  <Edit>
    <SimpleForm>
      <TextInput source="title" validate={required()} />
      <RichTextInput source="body" />
      <ReferenceInput source="author_id" reference="users" />
    </SimpleForm>
  </Edit>
);
```
