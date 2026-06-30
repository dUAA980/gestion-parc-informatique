# Gestion Parc — Design System v1.0

## Overview

This is a complete redesign of the Gestion Parc Informatique application following a precision instrument aesthetic: "air traffic control meets Swiss design system."

## Design Philosophy

- **Calm, authoritative**: No visual noise. Every surface has a reason to exist.
- **Information-dense**: Rich content presented clearly without chrome.
- **Data speaks; chrome whispers**: Graphics and data take priority. UI is invisible.

## Color System

All colors are defined as CSS custom properties in `styles/tokens.css`.

### Primary Palette

```
--canvas: #0F1117              /* Page background */
--surface: #161B26             /* Card & panel backgrounds */
--raised: #1E2535              /* Inputs, hover rows, elevated elements */
--border: rgba(255,255,255,0.08)  /* Default border */
--border-em: rgba(255,255,255,0.16) /* Hover/focus border */
```

### Text Colors

```
--text-primary: #E2E8F0    /* Headlines, primary text */
--text-secondary: #94A3B8  /* Body, labels */
--text-muted: #4B5563      /* Timestamps, placeholders */
```

### Semantic Colors

```
--accent-blue: #2563EB          /* Primary actions, links */
--accent-blue-hover: #1D4ED8
--teal-ok: #10B981              /* Success, healthy stock */
--amber-warn: #F59E0B           /* Low stock, warnings */
--red-crit: #EF4444             /* Empty stock, critical errors */
--purple-info: #8B5CF6          /* Informational highlights */
```

## Typography

### Font Families

- **DM Sans** (headings) - 400, 500, 600, 700
- **Inter** (UI body) - 400, 500, 600, 700
- **JetBrains Mono** (codes/serials) - 400, 600

### Scales

```
Display (28px, DM Sans 600):    Page titles
H2 (18px, DM Sans 500):         Section headings
Label (12px, Inter 500, UPPERCASE): Card eyebrows, form labels
Body (14px, Inter 400):         Table rows, paragraphs
Mono (12px, JetBrains 400):     IDs, serials, MACs
Micro (11px, Inter 400):        Timestamps, tooltips
```

Use CSS classes: `.display`, `.h2`, `.label`, `.body`, `.mono`, `.micro`

## Motion System

All animations use Framer Motion and honor `prefers-reduced-motion`:

```
--dur-instant: 80ms (button press, toggle click)
--dur-micro: 150ms (hover states, badge color change)
--dur-panel: 220ms (sidebar, modals, drawers)
--dur-page: 300ms (page enter: opacity 0→1, y 8→0)
--dur-data: 400ms (chart bars grow from zero)
--dur-skeleton: 1400ms infinite (shimmer loading placeholders)
```

### Page Transitions

Wrap route children in `motion.div`:

```tsx
<motion.div
  initial={{ opacity: 0, y: 8 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 300, ease: 'easeOut' }}
>
  {children}
</motion.div>
```

### Staggered Lists

Use `staggerChildren: 0.04` for table rows:

```tsx
<motion.div
  animate="visible"
  variants={{
    visible: {
      transition: {
        staggerChildren: 0.04,
      },
    },
  }}
>
  {items.map((item) => (
    <motion.div key={item.id} variants={itemVariants}>
      {/* row content */}
    </motion.div>
  ))}
</motion.div>
```

### Number Counters

Use `useMotionValue` + `useTransform` for KPI animations:

```tsx
const motionValue = useMotionValue(0);
const displayValue = useTransform(motionValue, (v) => Math.round(v).toLocaleString());

useEffect(() => {
  motionValue.set(value);
}, [value, motionValue]);

<motion.div>{displayValue}</motion.div>
```

## Component Library

### Button

```tsx
<Button variant="primary" size="md" icon={<Icon />}>
  Click Me
</Button>
```

**Variants:**
- `primary` - Blue background, white text
- `secondary` - Bordered, transparent background
- `destructive` - Red border, red text
- `ghost` - Minimal styling

**Sizes:** `sm` (32px), `md` (36px), `lg` (40px)

### Card

```tsx
<Card interactive onClick={() => {}}>
  Card content
</Card>
```

**Props:**
- `interactive` - Add hover effects and cursor pointer
- `variant` - `'default'` | `'raised'`

### Badge

```tsx
<Badge variant="ok|warning|critical|inactive">
  12 items
</Badge>
```

### MonogramAvatar

Equipment types get a unique 2-letter monogram with deterministic color:

```tsx
<MonogramAvatar equipmentType="PC Portable" size="md" />
```

**Output:** "PP" in blue (always same color for same type)

### KPICard

Dashboard KPI with animated counter and sparkline:

```tsx
<KPICard
  label="Total Equipment"
  value={520}
  delta={12}
  sparklineData={[12, 19, 3, 5, 2, 3, 9]}
  icon={<Package size={20} />}
  color="blue"
/>
```

## Layout Components

### Layout

Wrapper component providing Sidebar + TopBar + Content area:

```tsx
<Layout>
  <h1>Page content</h1>
</Layout>
```

### Sidebar

- Fixed left, 240px expanded / 64px icon-only
- Smooth width transition (220ms)
- Bottom: user avatar + logout
- Top: app logo + version badge

### TopBar

- Fixed top, 56px height
- Left: breadcrumb navigation
- Right: notification bell + user avatar chip

## Monogram System

Each equipment type gets a unique 2-letter avatar that experienced users internalize:

```tsx
import { getMonogram, getMonogramColor, MonogramAvatar } from '@/utils/monogram';

const type = "PC Portable";
getMonogram(type);        // Returns "PP"
getMonogramColor(type);   // Returns { bg, text, border } colors

// Always same color across all pages/sessions for same type
```

## Building New Pages

### Template: Parc Page

```tsx
import { Layout } from '@/components/layout';
import { Card, Badge, Button, MonogramAvatar } from '@/components/ui';
import { motion } from 'framer-motion';

export default function Parc() {
  return (
    <Layout>
      {/* Page Title */}
      <motion.div initial={{ opacity: 0, y: -8 }} animate={{ opacity: 1, y: 0 }}>
        <h1 className="display">Parc</h1>
        <p className="body text-[var(--text-muted)]">Description</p>
      </motion.div>

      {/* Filter Bar */}
      <div className="flex gap-3 my-6 overflow-x-auto pb-2">
        {/* Filter chips */}
      </div>

      {/* Table or Grid */}
      <Card>
        {/* Content */}
      </Card>
    </Layout>
  );
}
```

### Common Patterns

**Table Row with Monogram:**

```tsx
<motion.tr className="border-b border-[var(--border)] hover:bg-[var(--raised)]">
  <td className="px-4 py-3 flex items-center gap-3">
    <MonogramAvatar equipmentType={type} size="sm" />
    <span className="body">{name}</span>
  </td>
  {/* More columns */}
</motion.tr>
```

**Status Badge:**

```tsx
<Badge variant={quantity < 5 ? 'critical' : quantity < 10 ? 'warning' : 'ok'}>
  {quantity} units
</Badge>
```

**Inline Stock Bar:**

```tsx
<div
  className="h-1 w-20 bg-[rgba(255,255,255,0.06)] rounded-full overflow-hidden"
>
  <div
    className="h-full"
    style={{
      width: `${(quantity / 20) * 100}%`,
      backgroundColor: quantity < 5 ? 'var(--red-crit)' : 'var(--teal-ok)',
    }}
  />
</div>
```

## Accessibility

- **Focus rings:** Automatic via `:focus-visible` (2px blue outline, 2px offset)
- **Color contrast:** All text meets WCAG AA (4.5:1 minimum)
- **Keyboard navigation:** All interactive elements accessible via Tab
- **Skip links:** Todo - add skip-to-content link to Layout
- **ARIA labels:** Add where needed on icons/buttons

### Keyboard Shortcuts (Future)

```
/ = Search
n = New item
? = Help
Esc = Close modal/dialog
```

## Responsive Design

### Breakpoints

```
--breakpoint-mobile: 480px
--breakpoint-tablet: 768px
--breakpoint-desktop: 1024px
--breakpoint-wide: 1440px
```

Currently, only desktop (1024px+) is implemented. Responsive refactor coming in Phase 2.

## Empty States

Every page with empty data should show:

1. Large icon (SVG from lucide-react, 80px)
2. Heading in primary text color
3. Description in muted text
4. CTA button (primary or secondary)

```tsx
<div className="flex flex-col items-center justify-center py-24 text-center">
  <Package size={80} className="text-[var(--text-muted)] mb-6" />
  <h2 className="h2 text-[var(--text-primary)] mb-2">No Equipment Found</h2>
  <p className="body text-[var(--text-muted)] mb-6">
    Import or create your first equipment to get started.
  </p>
  <Button onClick={onAction}>Create Equipment</Button>
</div>
```

## Error States

Failed API calls show inline error cards:

```tsx
{error && (
  <Card className="border-l-4" style={{ borderLeftColor: 'var(--red-crit)' }}>
    <div className="flex items-start justify-between">
      <div>
        <h3 className="h2 text-[var(--red-crit)]">Error Loading Data</h3>
        <p className="body text-[var(--text-secondary)] mt-2">{error.message}</p>
      </div>
      <Button onClick={onRetry}>Retry</Button>
    </div>
  </Card>
)}
```

## Loading States

Use skeleton shimmer (not spinners):

```tsx
const skeletonVariants = {
  animate: {
    opacity: [0.5, 1, 0.5],
    transition: {
      duration: 1.4,
      repeat: Infinity,
      ease: 'ease-in-out',
    },
  },
};

<motion.div
  className="h-4 w-32 bg-[var(--raised)] rounded"
  variants={skeletonVariants}
  animate="animate"
/>
```

## Color Reference

### Equipment Categories

```
PC Portable  → Blue (#2563EB)
PC Fixe      → Teal (#10B981)
Imprimante   → Amber (#F59E0B)
Autre        → Purple (#8B5CF6)
```

(Colors assigned deterministically via monogram hash)

## File Structure

```
src/
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Badge.tsx
│   │   ├── MonogramAvatar.tsx
│   │   └── index.ts
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   ├── TopBar.tsx
│   │   ├── Layout.tsx
│   │   └── index.ts
│   ├── dashboard/
│   │   ├── KPICard.tsx
│   │   └── (other dashboard components)
│   └── ...
├── pages/
│   ├── DashboardNew.tsx         ← Reference implementation
│   ├── Parc.tsx
│   ├── Stock.tsx
│   ├── Mouvements.tsx
│   ├── Dechets.tsx
│   ├── LocalsIT.tsx
│   └── ...
├── styles/
│   ├── tokens.css              ← All CSS variables
│   └── index.css
├── utils/
│   ├── monogram.ts             ← Color/avatar system
│   └── ...
└── App.tsx
```

## Next Steps

1. ✅ Token system (colors, typography, motion)
2. ✅ UI component library (Button, Card, Badge, Monogram)
3. ✅ Layout shell (Sidebar, TopBar, Layout wrapper)
4. ✅ Dashboard reference implementation
5. □ Parc page (table + grid views, bulk actions)
6. □ Stock page (critical alerts, inline stock bars)
7. □ Mouvements page (timeline + table views)
8. □ Dechets page (disposal records, restore workflow)
9. □ Locaux IT page (card grid, expandable equipment)
10. □ Mobile responsiveness (bottom tab bar, full-width forms)
11. □ Advanced features (search, filters, exports)

## Support

For questions or issues with the design system, refer to:
- `styles/tokens.css` — All design tokens
- `components/ui/` — Component implementations
- `pages/DashboardNew.tsx` — Reference implementation showing patterns
