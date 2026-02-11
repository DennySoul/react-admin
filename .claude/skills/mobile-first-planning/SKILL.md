---
name: mobile-first-planning
description: Mobile-first responsive design patterns and best practices for Material-UI (MUI) based React applications. Use when building responsive layouts, implementing mobile-first designs, working with MUI breakpoints (xs/sm/md/lg/xl), creating touch-friendly interfaces, ensuring 44px minimum touch targets, designing responsive navigation (bottom nav for mobile, sidebar for desktop), or converting desktop-first designs to mobile-first.
---

# Mobile-First Responsive Design

## Core Philosophy

Design for mobile first, then enhance for larger screens.

## MUI Breakpoints

Always use MUI's breakpoint system:

```tsx
// Breakpoint values
// xs: 0px    (mobile portrait)
// sm: 600px  (mobile landscape / small tablet)
// md: 900px  (tablet)
// lg: 1200px (desktop)
// xl: 1536px (large desktop)
```

## Responsive Patterns

### Conditional Rendering by Breakpoint

```tsx
import { useMediaQuery, useTheme } from '@mui/material';

const PostList = () => {
  const theme = useTheme();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));

  return (
    <List>
      {isMobile ? (
        <MobilePostCards />
      ) : (
        <DesktopDataGrid />
      )}
    </List>
  );
};
```

### Responsive sx Props

```tsx
// ✅ Correct: mobile-first with breakpoint overrides
<Box
  sx={{
    display: 'flex',
    flexDirection: 'column',        // mobile default
    gap: 2,
    p: 2,
    // Override for larger screens
    md: {
      flexDirection: 'row',
      p: 4,
    },
  }}
>

// ❌ Wrong: desktop-first (requires overrides for mobile)
<Box
  sx={{
    display: 'flex',
    flexDirection: 'row',
    p: 4,
    xs: {
      flexDirection: 'column',
      p: 2,
    },
  }}
>
```

### Grid Layouts

```tsx
<Grid container spacing={{ xs: 2, md: 3 }}>
  <Grid item xs={12} sm={6} md={4} lg={3}>
    <PostCard />
  </Grid>
</Grid>
```

## Mobile UI Components

### Cards Instead of Tables

For mobile, replace DataGrid/tables with cards:

```tsx
// Mobile card component
const PostCardMobile: React.FC<{ record: Post }> = ({ record }) => (
  <Card sx={{ mb: 2 }}>
    <CardContent>
      <Typography variant="h6" noWrap>
        {record.title}
      </Typography>
      <Typography variant="body2" color="text.secondary">
        {record.author} • {formatDate(record.created_at)}
      </Typography>
    </CardContent>
    <CardActions>
      <EditButton record={record} />
      <DeleteButton record={record} />
    </CardActions>
  </Card>
);
```

### Touch-Friendly Targets

- Minimum touch target: 44x44px
- Adequate spacing between interactive elements
- Use IconButton with `size="large"` on mobile

```tsx
<IconButton 
  size={isMobile ? 'large' : 'medium'}
  sx={{ minWidth: 44, minHeight: 44 }}
>
  <EditIcon />
</IconButton>
```

### Navigation

- Use bottom navigation on mobile instead of sidebar
- Collapsible drawer for secondary navigation
- Sticky headers for context

```tsx
const Navigation = () => {
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));
  
  return isMobile ? (
    <BottomNavigation />
  ) : (
    <Sidebar />
  );
};
```

## Typography Scaling

```tsx
<Typography
  variant="h4"
  sx={{
    fontSize: { xs: '1.5rem', md: '2rem', lg: '2.5rem' },
  }}
>
  {title}
</Typography>
```

## Testing Responsive Designs

Always test at these widths:
- 320px (small mobile)
- 375px (iPhone)
- 768px (tablet portrait)
- 1024px (tablet landscape)
- 1440px (desktop)

## Forbidden Patterns

- Fixed pixel widths on containers
- Horizontal scrolling on mobile (except for data tables with scroll hint)
- Hover-only interactions without touch alternatives
- Text smaller than 14px on mobile
- Click targets smaller than 44px
