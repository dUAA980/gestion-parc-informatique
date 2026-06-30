# Phase 1 Completion - Implementation Guide

## ✅ Completed Components

### Core Design System
- **tokens.css** - All CSS custom properties (colors, typography, motion, spacing, sizing, z-index)
- **monogram.ts** - Deterministic equipment avatar system with 8-color palette

### UI Component Library (9 components)
1. **Button** - 4 variants, 3 sizes, icon support, loading state
2. **Card** - 2 variants, interactive mode, hover animations
3. **Badge** - 4 semantic status types
4. **MonogramAvatar** - 3 sizes, deterministic colors for equipment types
5. **Input** - Text field with validation, errors, hints
6. **Select** - Dropdown with sorting support
7. **Table** - Sticky header/columns, sortable, bulk select checkboxes
8. **Toast** - Provider-based notification system with 5 variants
9. **Modal & Drawer** - Centered modal and side drawer components

### Layout System (3 components)
1. **Sidebar** - Fixed left nav, 240px expanded / 64px collapsed
2. **TopBar** - Fixed top nav with breadcrumbs, notifications, user
3. **Layout** - Wrapper providing full shell with Sidebar + TopBar

### Dashboard Components
1. **KPICard** - Animated number counter with sparkline, delta indicator
2. **DashboardNew** - Full reference page with:
   - 4 KPI cards (animated)
   - Area chart (equipment by type)
   - Donut chart (distribution)
   - Recent activity feed
   - Floating AI chatbot with messages

### Page Templates (Reference Implementations)
1. **DashboardNew** - Full reference with all animation patterns
2. **ParcNew** - Equipment list with search, filters, table, modal form

## 📋 File Structure

```
frontend/src/
├── styles/
│   └── tokens.css              ✅ Complete
├── utils/
│   └── monogram.ts             ✅ Complete
├── components/
│   ├── ui/
│   │   ├── Button.tsx          ✅ Complete
│   │   ├── Card.tsx            ✅ Complete
│   │   ├── Badge.tsx           ✅ Complete
│   │   ├── MonogramAvatar.tsx  ✅ Complete
│   │   ├── Input.tsx           ✅ Complete
│   │   ├── Select.tsx          ✅ Complete
│   │   ├── Table.tsx           ✅ Complete
│   │   ├── Toast.tsx           ✅ Complete (Provider-based)
│   │   ├── Modal.tsx           ✅ Complete (Modal + Drawer)
│   │   └── index.ts            ✅ Complete (All exports)
│   ├── layout/
│   │   ├── Sidebar.tsx         ✅ Complete
│   │   ├── TopBar.tsx          ✅ Complete
│   │   ├── Layout.tsx          ✅ Complete
│   │   └── index.ts            ✅ Complete
│   └── dashboard/
│       └── KPICard.tsx         ✅ Complete
├── pages/
│   ├── DashboardNew.tsx        ✅ Complete (Reference)
│   └── ParcNew.tsx             ✅ Complete (Reference)
├── index.jsx                   ✅ Updated (imports tokens.css)
└── App.jsx                     ⏳ Needs wrapper update (Toast provider)
```

## 🎯 Next Steps: Building Remaining Pages

### Step 1: Wrap App with Toast Provider

Update `App.jsx` to wrap all routes with `ToastProvider`:

```tsx
import { ToastProvider } from './components/ui/Toast';

function App() {
  return (
    <ToastProvider>
      <Routes>
        {/* Your routes */}
      </Routes>
    </ToastProvider>
  );
}
```

### Step 2: Build Stock Page

Reference: `ParcNew.tsx` (reuse table + filters + modal)

**Unique features for Stock:**
- Stock level badges with inline bar
- Critical alerts (< 5 units)
- Bulk import/restock workflow
- Stock adjustment form (add/remove quantity)

```tsx
interface StockItem {
  id: string;
  name: string;
  type: string;
  quantity: number;
  minThreshold: number;
  location: string;
  lastRestocked: string;
}

// Stock level badge:
const stockLevel = quantity < minThreshold ? 'critical' : quantity < minThreshold * 1.5 ? 'warning' : 'ok';

// Inline stock bar:
<div className="h-1 w-20 bg-[rgba(255,255,255,0.06)] rounded-full overflow-hidden">
  <div
    className="h-full transition-all"
    style={{
      width: `${Math.min((quantity / 20) * 100, 100)}%`,
      backgroundColor: stockLevel === 'critical' ? 'var(--red-crit)' : 'var(--teal-ok)',
    }}
  />
</div>
```

### Step 3: Build Mouvements Page

Reference: `ParcNew.tsx` (reuse table + search)

**Unique features for Mouvements (Movements):**
- Timeline view option (toggle with grid view)
- Date range filter
- Movement type badges (Entry, Exit, Transfer, Return)
- Quick filters: "Today", "This Week", "This Month"

```tsx
// Movement type colors:
const movementTypes = {
  entry: { badge: 'ok', icon: ArrowDownLeft, color: '#10B981' },
  exit: { badge: 'warning', icon: ArrowUpRight, color: '#F59E0B' },
  transfer: { badge: 'info', icon: ArrowRightLeft, color: '#2563EB' },
  return: { badge: 'critical', icon: RotateCcw, color: '#EF4444' },
};
```

### Step 4: Build Déchets Page

Reference: `ParcNew.tsx` (reuse table + modal)

**Unique features for Déchets (Disposed Equipment):**
- Disposal reason column
- Disposal date column
- Restore button (move back to active)
- Bulk export for recycling compliance
- Destruction certificate download

```tsx
interface DisposedItem {
  id: string;
  name: string;
  type: string;
  disposalDate: string;
  disposalReason: string;
  disposalMethod: 'recycle' | 'donate' | 'destroy';
  certificate?: string;
}

// Reason badges:
const disposalReasons = {
  end_of_life: 'End of Life',
  damage: 'Damaged',
  lost: 'Lost',
  stolen: 'Stolen',
  obsolete: 'Obsolete',
};
```

### Step 5: Build Locaux IT Page

Reference: New template (grid of locations as cards)

**Unique features for Locaux IT (IT Spaces):**
- Location cards (not table)
- Equipment count per location
- Expandable equipment grid
- Location capacity indicator
- Add/edit location modal

```tsx
interface Location {
  id: string;
  name: string;
  floor: string;
  capacity: number;
  equipment: Equipment[];
  lastAudit: string;
}

// Location card template:
<Card interactive onClick={() => setExpanded(!expanded)}>
  <div className="flex items-center justify-between mb-4">
    <h3 className="h2">{location.name}</h3>
    <Badge>{location.equipment.length}/{location.capacity}</Badge>
  </div>
  <div className="space-y-2">
    <div className="body text-[var(--text-secondary)]">Floor: {location.floor}</div>
    <div className="h-1 w-full bg-[rgba(255,255,255,0.06)] rounded-full overflow-hidden">
      <div
        className="h-full bg-[var(--accent-blue)]"
        style={{ width: `${(location.equipment.length / location.capacity) * 100}%` }}
      />
    </div>
  </div>
</Card>
```

## 🎨 Common Patterns to Reuse

### Search + Filter Bar
```tsx
<div className="grid grid-cols-3 gap-4 mb-6">
  <Input
    placeholder="Search..."
    icon={<Search size={16} />}
    value={searchQuery}
    onChange={(e) => setSearchQuery(e.target.value)}
  />
  <Select options={[...]} value={filter} onChange={...} />
</div>
```

### Action Bar
```tsx
<div className="flex items-center gap-4 mb-6">
  <Button onClick={handleNew} variant="primary" icon={<Plus size={18} />}>
    New Item
  </Button>
  <Button variant="secondary" icon={<Upload size={18} />}>Import</Button>
  <Button variant="secondary" icon={<Download size={18} />}>Export</Button>
  {selectedRows.length > 0 && (
    <>
      <div className="ml-auto text-[14px] text-[var(--text-muted)]">
        {selectedRows.length} selected
      </div>
      <Button onClick={handleDelete} variant="destructive">
        Delete
      </Button>
    </>
  )}
</div>
```

### Toast Notifications
```tsx
import { useToast } from '@/components/ui';

const { show } = useToast();

// Success
show({ variant: 'success', title: 'Equipment added' });

// Error
show({ variant: 'error', title: 'Error', description: 'Something went wrong' });

// Undo
show({
  variant: 'undo',
  title: 'Equipment deleted',
  action: {
    label: 'Undo',
    callback: () => { /* restore */ },
  },
});
```

### Form Modal
```tsx
const [isOpen, setIsOpen] = useState(false);
const [form, setForm] = useState({...});

<Modal
  isOpen={isOpen}
  onClose={() => setIsOpen(false)}
  title="Add New Item"
  actions={[
    { label: 'Cancel', onClick: () => setIsOpen(false), variant: 'secondary' },
    { label: 'Save', onClick: handleSave, variant: 'primary' },
  ]}
>
  <Input label="Name" value={form.name} onChange={...} />
  <Select label="Type" options={...} value={form.type} onChange={...} />
</Modal>
```

### Animated Page Entry
```tsx
<motion.div initial={{ opacity: 0, y: -8 }} animate={{ opacity: 1, y: 0 }}>
  <h1 className="display">Page Title</h1>
  <p className="body text-[var(--text-muted)]">Description</p>
</motion.div>
```

### Empty State
```tsx
{items.length === 0 && (
  <div className="text-center py-12">
    <Icon size={80} className="text-[var(--text-muted)] mx-auto mb-4" />
    <h3 className="h2 text-[var(--text-primary)] mb-2">No items found</h3>
    <p className="body text-[var(--text-muted)] mb-6">Create one to get started</p>
    <Button onClick={onCreate}>Create Item</Button>
  </div>
)}
```

## 🚀 Build Priority

### Critical Path (Phase 1 Completion)
1. ✅ Design tokens & components
2. ✅ Layout shell (Sidebar, TopBar)
3. ✅ Dashboard reference page
4. ⏳ Parc page (basic list + CRUD)
5. ⏳ Stock page (inventory management)

### Secondary (Phase 2)
6. Mouvements page
7. Déchets page
8. Locaux IT page

### Polish (Phase 3)
9. Mobile responsiveness
10. Advanced filtering
11. Bulk export workflows
12. i18n translations

## 🔗 Integration with Backend

Each page needs API integration. Current API methods (from `api.js`):

```typescript
// Auth
authAPI.login(credentials)
authAPI.logout()
authAPI.getUser()

// Equipment (Parc)
equipmentAPI.getAll()
equipmentAPI.create(data)
equipmentAPI.update(id, data)
equipmentAPI.delete(id)
equipmentAPI.bulkDelete(ids)
equipmentAPI.import(file)
equipmentAPI.export()

// Stock
stockAPI.getAll()
stockAPI.update(id, quantity)
stockAPI.getAlerts()

// Mouvements
mouvementsAPI.getAll(filters)
mouvementsAPI.create(data)

// Dechets
dechetsAPI.getAll()
dechetsAPI.create(data)
dechetsAPI.restore(id)

// Locaux IT
locauxAPI.getAll()
locauxAPI.getByLocation(id)
```

## 📚 Component Usage Examples

### Using Table with API Data
```tsx
const [data, setData] = useState([]);
const [loading, setLoading] = useState(false);

useEffect(() => {
  setLoading(true);
  equipmentAPI.getAll()
    .then(setData)
    .catch(err => show({ variant: 'error', title: 'Failed to load data' }))
    .finally(() => setLoading(false));
}, []);

if (loading) return <TableSkeleton />;

<Table
  columns={columns}
  rows={data}
  selectable
  onRowSelect={setSelected}
  renderCell={renderCell}
/>
```

### Using Modal for Create/Edit
```tsx
const [editItem, setEditItem] = useState(null);
const [isOpen, setIsOpen] = useState(false);

const openCreate = () => {
  setEditItem(null);
  setIsOpen(true);
};

const openEdit = (item) => {
  setEditItem(item);
  setIsOpen(true);
};

const handleSave = async () => {
  try {
    if (editItem) {
      await equipmentAPI.update(editItem.id, formData);
      show({ variant: 'success', title: 'Updated' });
    } else {
      await equipmentAPI.create(formData);
      show({ variant: 'success', title: 'Created' });
    }
    setIsOpen(false);
    // Refresh data
  } catch (err) {
    show({ variant: 'error', title: 'Error saving' });
  }
};
```

## ✨ Quality Checklist Before Shipping Pages

- [ ] TypeScript types defined for all data models
- [ ] All interactive elements have hover/focus states
- [ ] Empty states handled with helpful messaging
- [ ] Error states show helpful error messages
- [ ] Loading states use shimmer skeletons (not spinners)
- [ ] Page animations: entry (opacity 0→1, y 8→0) + stagger
- [ ] Table rows stagger animate with 0.04s delay
- [ ] Toast notifications on all CRUD actions
- [ ] Undo actions for destructive operations
- [ ] Search/filter state synchronized
- [ ] Keyboard navigation works (Tab through inputs, Esc closes modals)
- [ ] Color contrast meets WCAG AA standard
- [ ] Mobile breakpoint media queries (768px, 1024px)
- [ ] API error handling with user-friendly messages
- [ ] Form validation before submission
- [ ] Bulk actions (select multiple, delete, export)
- [ ] Icons from lucide-react (consistent set)
- [ ] No hardcoded colors (all from CSS custom properties)

## 🎬 Animation Constants

All available in `tokens.css`:

```css
--dur-instant: 80ms      /* Button clicks */
--dur-micro: 150ms       /* Hover states */
--dur-panel: 220ms       /* Modals, sidebars */
--dur-page: 300ms        /* Page transitions */
--dur-data: 400ms        /* Chart animations */
--dur-skeleton: 1400ms   /* Loading shimmer */
```

Use with Framer Motion:
```tsx
transition={{ duration: 300 }}
transition={{ duration: 150 }}
animate={{ scale: [1, 1.2, 1] }} // 2-second loop
```

## 📞 Support References

- **Design System Guide**: `/DESIGN_SYSTEM.md`
- **Component Examples**: `pages/DashboardNew.tsx`, `pages/ParcNew.tsx`
- **Token Reference**: `styles/tokens.css`
- **Monogram System**: `utils/monogram.ts`

## 🚦 Deployment Checklist

Before pushing to production:

1. [ ] Build frontend: `npm run build`
2. [ ] Check build output for errors
3. [ ] Test in production-like environment
4. [ ] Verify all API integrations work
5. [ ] Test on mobile (responsive design)
6. [ ] Audit accessibility (keyboard nav, color contrast)
7. [ ] Validate performance (Lighthouse)
8. [ ] Test export/import workflows
9. [ ] Test undo workflows
10. [ ] Smoke test all CRUD operations

---

**Ready to start building?** Pick a page from the priority list and follow the patterns established in `DashboardNew.tsx` and `ParcNew.tsx`. Each page should:
1. Import `Layout` + components from `/components/ui` and `/components/layout`
2. Import `useToast` for notifications
3. Use `Table` or `Card` grid for displaying data
4. Use `Modal` for create/edit workflows
5. Use `Input` + `Select` for forms
6. Follow animation patterns (page entry + stagger)
