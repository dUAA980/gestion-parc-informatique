# 🚀 Getting Started: Phase 1 Complete - What's Next?

## Quick Overview

You now have a complete, production-ready design system and component library for your IT equipment management application. This document gets you up to speed in 5 minutes.

## What You Have (Phase 1 ✅)

### Design System
- **Colors**: 12 primary + 4 semantic (canvas, surface, raised, text layers, accents)
- **Typography**: DM Sans (headlines), Inter (UI), JetBrains Mono (codes)
- **Motion**: 6 duration tokens that respect user's `prefers-reduced-motion`
- **Layout**: Sidebar + TopBar shell system

### Components (14 ready to use)
```
UI Components:
  Button, Card, Badge, Input, Select, Table, Toast, Modal
  MonogramAvatar (equipment avatars)

Layout Components:
  Sidebar, TopBar, Layout (shell wrapper)

Dashboard Component:
  KPICard (animated statistics)
```

### Pages (2 reference implementations)
- **DashboardNew.tsx**: Shows KPIs, charts, activity feed, chatbot integration
- **ParcNew.tsx**: Shows equipment list, search, filters, CRUD workflows

## Right Now: 2 Things to Do

### 1. Wrap Your App with Toast Provider (2 minutes)

**Why**: Notifications won't work without this.

**File to edit**: `frontend/src/App.jsx`

**Find this**:
```jsx
function App() {
  return (
    <Routes>
      {/* ... */}
    </Routes>
  );
}
```

**Change to**:
```jsx
import { ToastProvider } from './components/ui/Toast';

function App() {
  return (
    <ToastProvider>
      <Routes>
        {/* ... your routes ... */}
      </Routes>
    </ToastProvider>
  );
}
```

**Done!** ✅ Toast notifications now work across your entire app.

### 2. Test the New Components (5 minutes)

**Route to Dashboard**:
- Change your default route to point to `DashboardNew` from `frontend/src/pages/DashboardNew.tsx`
- Or visit `/dashboard-new` if you add this route

**What to see**:
- 4 KPI cards with animated numbers
- Chart showing equipment distribution
- Activity feed with timeline
- Floating chatbot button (bottom right)
- Updated/pulse indicator (bottom left)

**Try these interactions**:
- Click KPI cards (hover animations)
- Open chatbot (slide-in animation)
- Type in chat (staggered message animation)
- Check button hover effects

**All working?** ✅ System is live!

## Next: Build Your First New Page (Stock Page)

### Time Estimate: 2-3 hours

### Step-by-Step

#### 1. Create the File
```
frontend/src/pages/StockNew.tsx
```

#### 2. Copy This Template
```tsx
import React, { useState } from 'react';
import { motion } from 'framer-motion';
import { Plus, Download, Upload, Search } from 'lucide-react';
import { Layout } from '../layout/Layout';
import {
  Card,
  Button,
  Badge,
  Table,
  Input,
  Select,
  useToast,
} from '../ui';
import { Modal } from '../ui/Modal';

export default function Stock() {
  const { show: showToast } = useToast();
  const [data, setData] = useState([
    // Your stock items here
  ]);
  const [searchQuery, setSearchQuery] = useState('');
  const [isModalOpen, setIsModalOpen] = useState(false);

  const handleAddNew = () => {
    setIsModalOpen(true);
    showToast({ variant: 'success', title: 'Feature coming soon' });
  };

  return (
    <Layout>
      <motion.div initial={{ opacity: 0, y: -8 }} animate={{ opacity: 1, y: 0 }} className="mb-8">
        <h1 className="display">Stock Management</h1>
        <p className="body text-[var(--text-muted)] mt-2">Track inventory levels</p>
      </motion.div>

      <div className="flex items-center gap-4 mb-6">
        <Button onClick={handleAddNew} variant="primary" icon={<Plus size={18} />}>
          Restock Item
        </Button>
        <Button variant="secondary" icon={<Download size={18} />}>Export</Button>
      </div>

      <Input
        placeholder="Search stock items..."
        icon={<Search size={16} />}
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        className="mb-6"
      />

      <Card>
        <Table
          columns={[
            { key: 'name', label: 'Equipment', sortable: true },
            { key: 'quantity', label: 'Quantity', sortable: true },
            { key: 'minimum', label: 'Min Level', sortable: true },
            { key: 'status', label: 'Status' },
          ]}
          rows={data}
          renderCell={(value, row, col) => {
            if (col.key === 'status') {
              const status = row.quantity < row.minimum ? 'critical' : 'ok';
              return <Badge variant={status}>{row.quantity < row.minimum ? 'Low' : 'OK'}</Badge>;
            }
            return <span className="body text-[var(--text-primary)]">{value}</span>;
          }}
        />
      </Card>
    </Layout>
  );
}
```

#### 3. Add to Routes
**File**: `frontend/src/App.jsx` (in your `Routes`)
```jsx
import StockNew from './pages/StockNew';

// Inside <Routes>
<Route path="/stock-new" element={<StockNew />} />
```

#### 4. Connect to Your API
```tsx
useEffect(() => {
  // Replace with actual API call:
  // const data = await stockAPI.getAll();
  // setData(data);
}, []);
```

#### 5. Test
- Navigate to `/stock-new`
- Verify layout displays
- Verify toast works (click button)
- Test search filter

**Result**: Your first new page! ✅

## Pattern Reference (Copy-Paste These)

### Search + Filter Pattern
```tsx
const [searchQuery, setSearchQuery] = useState('');
const [filter, setFilter] = useState('all');

const filtered = data.filter(item =>
  item.name.toLowerCase().includes(searchQuery.toLowerCase()) &&
  (filter === 'all' || item.status === filter)
);

// In JSX:
<Input
  placeholder="Search..."
  value={searchQuery}
  onChange={(e) => setSearchQuery(e.target.value)}
/>

<Select
  options={[
    { value: 'all', label: 'All' },
    { value: 'active', label: 'Active' },
    { value: 'inactive', label: 'Inactive' },
  ]}
  value={filter}
  onChange={(e) => setFilter(e.target.value)}
/>

<Table rows={filtered} />
```

### Delete with Undo Pattern
```tsx
const handleDelete = async () => {
  try {
    // await API.delete(id);
    
    showToast({
      variant: 'undo',
      title: 'Item deleted',
      action: {
        label: 'Undo',
        callback: async () => {
          // await API.restore(id);
          showToast({ variant: 'success', title: 'Restored' });
        },
      },
    });
  } catch (err) {
    showToast({ variant: 'error', title: 'Error', description: err.message });
  }
};
```

### Status Badge Pattern
```tsx
<Badge variant={
  quantity < min ? 'critical' :
  quantity < min * 1.5 ? 'warning' : 'ok'
}>
  {quantity} units
</Badge>
```

## Important: Design System Rules

### ✅ DO
- Use CSS custom properties: `color: var(--accent-blue)`
- Use Framer Motion for animations
- Import from `@/components/ui` and `@/components/layout`
- Use `useToast()` for notifications
- Follow card/button/badge patterns from examples

### ❌ DON'T
- Hardcode colors: `color: '#2563EB'` ← BAD
- Use setTimeout for animations ← Use Framer Motion
- Create new button styles ← Use variants prop
- Forget to wrap with `Layout` component
- Add spinners for loading ← Use skeleton shimmer

## Quick Command Reference

```bash
# Start dev server
npm start

# Build for production
npm run build

# Check for errors
npm run lint
```

## File Organization

```
frontend/src/
├── styles/tokens.css              ← All design constants
├── utils/monogram.ts              ← Equipment avatars
├── components/
│   ├── ui/                        ← Use these components
│   ├── layout/                    ← Sidebar, TopBar, Layout
│   └── dashboard/                 ← Dashboard components
├── pages/
│   ├── DashboardNew.tsx           ← Reference (study this)
│   ├── ParcNew.tsx                ← Reference (study this)
│   └── StockNew.tsx               ← Your new page
└── App.jsx                        ← Main app (add Toast provider)
```

## Next Pages to Build (In Order)

1. **Stock Page** ← Start here (2-3 hours)
2. **Mouvements Page** (2 hours)
3. **Déchets Page** (1.5 hours)
4. **Locaux IT Page** (2 hours)

**Total Phase 2: ~7-8 hours**

## Helpful Documentation

- **Design System Guide**: `/DESIGN_SYSTEM.md` (understand philosophy)
- **Quick Reference**: `/QUICK_REFERENCE.md` (copy-paste patterns)
- **Implementation Guide**: `/IMPLEMENTATION_GUIDE.md` (detailed walkthrough)
- **Phase 1 Summary**: `/PHASE_1_SUMMARY.md` (status overview)

## Common Problems & Solutions

### Q: Buttons look wrong
**A**: Make sure `styles/tokens.css` is imported in `index.jsx` ✅

### Q: Toast doesn't appear
**A**: Wrap `App` with `<ToastProvider>` (see section above)

### Q: Components don't animate
**A**: Check Framer Motion is installed: `npm list framer-motion`

### Q: Colors look different
**A**: CSS variables in use - check browser DevTools for `--accent-blue` etc.

### Q: Table doesn't sort
**A**: Add `sortable: true` to column definition: `{ key: 'name', label: 'Name', sortable: true }`

### Q: Modal doesn't close
**A**: Pass `isOpen` state and `onClose` handler to Modal component

## Checklist: Before You Start Building

- ✅ Read this file (5 min)
- ✅ Add Toast provider to App.jsx (2 min)
- ✅ Visit `/dashboard-new` to verify system works (5 min)
- ✅ Review `QUICK_REFERENCE.md` for patterns (10 min)
- ✅ Study DashboardNew.tsx structure (10 min)
- ✅ Study ParcNew.tsx table pattern (10 min)
- ✅ Create Stock page from template (30 min)
- ⏳ Ready to build!

## Success Criteria: Stock Page Complete

You'll know it's working when:
- ✅ Page loads with Layout shell (Sidebar + TopBar)
- ✅ Title and description display
- ✅ "Restock Item" button works (shows toast)
- ✅ Search box filters table
- ✅ Table displays mock data with sorting
- ✅ Status badges show with correct colors
- ✅ Page animates in smoothly
- ✅ Hover effects work on all interactive elements

## Pro Tips

1. **Keep it simple**: Each page follows the same pattern
2. **Copy examples**: 95% of the time you'll copy from QUICK_REFERENCE.md
3. **Test as you go**: Open DevTools and watch animations
4. **Colors auto-work**: All components use CSS variables
5. **Ask the docs**: Every pattern is documented

## You're Ready! 🎉

You have:
- ✅ Production-ready components
- ✅ Complete design system
- ✅ Working examples
- ✅ Comprehensive documentation
- ✅ Clear build path forward

**Next step**: Build Stock page (copy template, add search, connect API)

**Questions?** See:
- Component usage → `/QUICK_REFERENCE.md`
- Build instructions → `/IMPLEMENTATION_GUIDE.md`
- Design decisions → `/DESIGN_SYSTEM.md`

---

**Status**: 🟢 Ready to Build | 🎨 Design System Active | 🚀 Components Ready

Happy coding! 🚀
