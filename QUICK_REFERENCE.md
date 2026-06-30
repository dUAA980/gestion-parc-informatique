# Quick Reference: Component Patterns & API Integration

## Import Statements

```tsx
// Components
import { Layout } from '@/components/layout';
import {
  Button,
  Card,
  Badge,
  Input,
  Select,
  Table,
  Modal,
  MonogramAvatar,
  useToast,
} from '@/components/ui';

// Icons
import { Plus, Trash2, Edit, Search, Download, Upload } from 'lucide-react';

// Motion
import { motion, AnimatePresence } from 'framer-motion';

// React
import { useState, useEffect } from 'react';
```

## Common Page Structure

```tsx
import React, { useState, useEffect } from 'react';
import { motion } from 'framer-motion';
import { Layout } from '../layout/Layout';
import { Button, Card, Input, Table, useToast, Select } from '../ui';

export default function PageName() {
  const { show: showToast } = useToast();
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedRows, setSelectedRows] = useState<string[]>([]);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    setLoading(true);
    try {
      // const result = await API.getAll();
      // setData(result);
    } catch (err) {
      showToast({
        variant: 'error',
        title: 'Failed to load data',
        description: err.message,
      });
    } finally {
      setLoading(false);
    }
  };

  return (
    <Layout>
      {/* Page Title */}
      <motion.div initial={{ opacity: 0, y: -8 }} animate={{ opacity: 1, y: 0 }} className="mb-8">
        <h1 className="display">Page Title</h1>
        <p className="body text-[var(--text-muted)] mt-2">Description</p>
      </motion.div>

      {/* Content */}
      {loading ? <LoadingSkeleton /> : <Content data={data} />}
    </Layout>
  );
}

function Content({ data }) {
  return (
    <Card>
      <Table columns={columns} rows={data} selectable />
    </Card>
  );
}

function LoadingSkeleton() {
  return (
    <Card>
      <div className="space-y-4">
        {[1, 2, 3].map((i) => (
          <motion.div
            key={i}
            className="h-8 bg-[var(--raised)] rounded"
            animate={{ opacity: [0.5, 1, 0.5] }}
            transition={{ duration: 1.4, repeat: Infinity }}
          />
        ))}
      </div>
    </Card>
  );
}
```

## Forms - Add/Edit Modal

```tsx
// State
const [isModalOpen, setIsModalOpen] = useState(false);
const [editingItem, setEditingItem] = useState(null);
const [formData, setFormData] = useState({
  name: '',
  type: 'Type1',
  status: 'active',
});

// Handlers
const handleNew = () => {
  setEditingItem(null);
  setFormData({ name: '', type: 'Type1', status: 'active' });
  setIsModalOpen(true);
};

const handleEdit = (item) => {
  setEditingItem(item);
  setFormData(item);
  setIsModalOpen(true);
};

const handleSave = async () => {
  if (!formData.name) {
    showToast({
      variant: 'error',
      title: 'Validation Error',
      description: 'Name is required',
    });
    return;
  }

  try {
    if (editingItem) {
      // await API.update(editingItem.id, formData);
      showToast({ variant: 'success', title: 'Item updated' });
    } else {
      // await API.create(formData);
      showToast({ variant: 'success', title: 'Item created' });
    }
    setIsModalOpen(false);
    // Refresh data or update state
  } catch (err) {
    showToast({ variant: 'error', title: 'Error saving', description: err.message });
  }
};

// JSX
<Button onClick={handleNew} variant="primary">Add New</Button>

<Modal
  isOpen={isModalOpen}
  onClose={() => setIsModalOpen(false)}
  title={editingItem ? 'Edit Item' : 'Add New Item'}
  actions={[
    { label: 'Cancel', onClick: () => setIsModalOpen(false), variant: 'secondary' },
    { label: 'Save', onClick: handleSave, variant: 'primary' },
  ]}
>
  <div className="space-y-4">
    <Input
      label="Name"
      placeholder="Item name"
      value={formData.name}
      onChange={(e) => setFormData({ ...formData, name: e.target.value })}
      error={!formData.name && 'Required'}
    />

    <Select
      label="Type"
      options={[
        { value: 'Type1', label: 'Type 1' },
        { value: 'Type2', label: 'Type 2' },
      ]}
      value={formData.type}
      onChange={(e) => setFormData({ ...formData, type: e.target.value })}
    />

    <Select
      label="Status"
      options={[
        { value: 'active', label: 'Active' },
        { value: 'inactive', label: 'Inactive' },
      ]}
      value={formData.status}
      onChange={(e) => setFormData({ ...formData, status: e.target.value })}
    />
  </div>
</Modal>
```

## Search & Filter

```tsx
const [searchQuery, setSearchQuery] = useState('');
const [selectedFilter, setSelectedFilter] = useState('all');

const filteredData = data.filter((item) => {
  const matchesSearch =
    item.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
    item.type.toLowerCase().includes(searchQuery.toLowerCase());
  const matchesFilter = selectedFilter === 'all' || item.status === selectedFilter;
  return matchesSearch && matchesFilter;
});

// JSX
<div className="grid grid-cols-3 gap-4 mb-6">
  <Input
    placeholder="Search..."
    icon={<Search size={16} />}
    value={searchQuery}
    onChange={(e) => setSearchQuery(e.target.value)}
  />

  <Select
    options={[
      { value: 'all', label: 'All' },
      { value: 'active', label: 'Active' },
      { value: 'inactive', label: 'Inactive' },
    ]}
    value={selectedFilter}
    onChange={(e) => setSelectedFilter(e.target.value)}
  />
</div>

<Table
  rows={filteredData}
  // ... other props
/>
```

## Bulk Actions

```tsx
const [selectedRows, setSelectedRows] = useState<string[]>([]);

const handleDelete = async () => {
  if (selectedRows.length === 0) return;

  try {
    // await Promise.all(selectedRows.map(id => API.delete(id)));
    
    showToast({
      variant: 'undo',
      title: `${selectedRows.length} item(s) deleted`,
      action: {
        label: 'Undo',
        callback: async () => {
          // Restore items
          showToast({ variant: 'info', title: 'Action undone' });
        },
      },
    });
    
    setSelectedRows([]);
    // Refresh data
  } catch (err) {
    showToast({ variant: 'error', title: 'Error deleting items' });
  }
};

// JSX
<div className="flex items-center gap-4">
  <Button onClick={handleNew} variant="primary">New</Button>
  
  {selectedRows.length > 0 && (
    <>
      <div className="ml-auto text-[14px] text-[var(--text-muted)]">
        {selectedRows.length} selected
      </div>
      <Button onClick={handleDelete} variant="destructive">Delete</Button>
    </>
  )}
</div>

<Table
  selectable
  onRowSelect={setSelectedRows}
  // ... other props
/>
```

## Table Rendering Custom Cells

```tsx
const columns = [
  { key: 'name', label: 'Name', width: '200px' },
  { key: 'type', label: 'Type' },
  { key: 'status', label: 'Status' },
  { key: 'actions', label: '' },
];

const renderCell = (value, row, column) => {
  if (column.key === 'name') {
    return (
      <div className="flex items-center gap-3">
        <MonogramAvatar equipmentType={row.type} size="sm" />
        <div className="body text-[var(--text-primary)] font-500">{value}</div>
      </div>
    );
  }

  if (column.key === 'status') {
    const variants = {
      active: 'ok',
      inactive: 'inactive',
      warning: 'warning',
      critical: 'critical',
    };
    return <Badge variant={variants[value] || 'ok'}>{value}</Badge>;
  }

  if (column.key === 'actions') {
    return (
      <div className="flex items-center gap-2">
        <button
          onClick={() => handleEdit(row)}
          className="p-2 hover:bg-[var(--raised)] rounded transition-colors"
        >
          <Edit size={16} className="text-[var(--text-muted)]" />
        </button>
        <button
          onClick={() => handleDelete(row.id)}
          className="p-2 hover:bg-[var(--raised)] rounded transition-colors"
        >
          <Trash2 size={16} className="text-[var(--red-crit)]" />
        </button>
      </div>
    );
  }

  return <span className="body text-[var(--text-primary)]">{value}</span>;
};

// JSX
<Table columns={columns} rows={data} renderCell={renderCell} selectable />
```

## Toast Notifications

```tsx
const { show } = useToast();

// Success
show({
  variant: 'success',
  title: 'Equipment added',
  description: 'PC Portable Dell XPS has been added to inventory',
});

// Error
show({
  variant: 'error',
  title: 'Error adding equipment',
  description: 'Please check the form and try again',
});

// Warning
show({
  variant: 'warning',
  title: 'Low stock warning',
  description: 'Only 3 units remaining in stock',
});

// Info
show({
  variant: 'info',
  title: 'Equipment imported',
  description: '47 items successfully imported from file',
});

// Undo
show({
  variant: 'undo',
  title: 'Equipment deleted',
  action: {
    label: 'Undo',
    callback: async () => {
      // Restore operation
      show({ variant: 'success', title: 'Restoration completed' });
    },
  },
});
```

## Status Badges by Type

```tsx
// Stock level
const getStockBadge = (quantity: number, min: number) => {
  if (quantity < min) return 'critical';
  if (quantity < min * 1.5) return 'warning';
  return 'ok';
};

<Badge variant={getStockBadge(quantity, minThreshold)}>
  {quantity} units
</Badge>

// Equipment status
const statusBadges = {
  active: 'ok',
  maintenance: 'warning',
  disposed: 'critical',
  stock: 'info',
};

<Badge variant={statusBadges[status]}>{status}</Badge>

// Movement type
const movementBadges = {
  entry: 'ok',
  exit: 'warning',
  transfer: 'info',
  return: 'critical',
};

<Badge variant={movementBadges[type]}>{type}</Badge>
```

## Inline Stock Bar

```tsx
<div className="flex items-center gap-3">
  <div className="flex-1">
    <div className="h-1 w-full bg-[rgba(255,255,255,0.06)] rounded-full overflow-hidden">
      <motion.div
        className="h-full"
        initial={{ width: 0 }}
        animate={{ width: `${(quantity / maxQuantity) * 100}%` }}
        transition={{ duration: 0.5 }}
        style={{
          backgroundColor: quantity < min ? 'var(--red-crit)' : 'var(--teal-ok)',
        }}
      />
    </div>
  </div>
  <span className="mono text-[var(--text-muted)]">{quantity}/{maxQuantity}</span>
</div>
```

## Loading Skeleton

```tsx
const Skeleton = () => (
  <motion.div
    className="h-8 bg-[var(--raised)] rounded"
    animate={{ opacity: [0.5, 1, 0.5] }}
    transition={{ duration: 1.4, repeat: Infinity }}
  />
);

// Usage
{loading ? (
  <div className="space-y-4">
    {[1, 2, 3].map((i) => <Skeleton key={i} />)}
  </div>
) : (
  <Table columns={columns} rows={data} />
)}
```

## Empty State

```tsx
{data.length === 0 ? (
  <div className="flex flex-col items-center justify-center py-24 text-center">
    <Package size={80} className="text-[var(--text-muted)] mb-6" />
    <h2 className="h2 text-[var(--text-primary)] mb-2">No equipment found</h2>
    <p className="body text-[var(--text-muted)] mb-6">
      Create your first equipment to get started
    </p>
    <Button onClick={handleNew} variant="primary">
      Create Equipment
    </Button>
  </div>
) : (
  <Table columns={columns} rows={data} />
)}
```

## Error State

```tsx
const [error, setError] = useState(null);

{error && (
  <Card className="border-l-4" style={{ borderLeftColor: 'var(--red-crit)' }}>
    <div className="flex items-start justify-between">
      <div>
        <h3 className="h2 text-[var(--red-crit)]">Error Loading Data</h3>
        <p className="body text-[var(--text-secondary)] mt-2">{error.message}</p>
      </div>
      <Button onClick={loadData} variant="secondary">
        Retry
      </Button>
    </div>
  </Card>
)}
```

## Animated List (Stagger)

```tsx
<motion.div
  animate="visible"
  initial="hidden"
  variants={{
    visible: {
      transition: {
        staggerChildren: 0.04,
      },
    },
  }}
>
  {items.map((item) => (
    <motion.div
      key={item.id}
      variants={{
        hidden: { opacity: 0, y: 4 },
        visible: { opacity: 1, y: 0 },
      }}
    >
      {/* Item content */}
    </motion.div>
  ))}
</motion.div>
```

## Responsive Grid

```tsx
// 4 columns on desktop, 2 on tablet, 1 on mobile
<div className="grid grid-cols-4 md:grid-cols-2 sm:grid-cols-1 gap-6">
  {items.map((item) => (
    <Card key={item.id}>{/* content */}</Card>
  ))}
</div>

// Note: Use Tailwind breakpoints
// sm: 640px, md: 768px, lg: 1024px, xl: 1280px, 2xl: 1536px
```

## Color Overrides (When Needed)

```tsx
// Use CSS custom properties
<div style={{ color: 'var(--teal-ok)' }}>Text</div>
<div style={{ backgroundColor: 'var(--red-crit)' }}>Background</div>

// Don't hardcode colors
<div style={{ color: '#10B981' }}>❌ Bad</div>
```

## Keyboard Shortcuts (Future)

```tsx
useEffect(() => {
  const handleKeyDown = (e: KeyboardEvent) => {
    if (e.key === '/' && e.ctrlKey) {
      e.preventDefault();
      setSearchFocused(true);
    }
    if (e.key === 'n' && e.ctrlKey) {
      e.preventDefault();
      handleNew();
    }
    if (e.key === 'Escape') {
      handleCloseModal();
    }
  };

  window.addEventListener('keydown', handleKeyDown);
  return () => window.removeEventListener('keydown', handleKeyDown);
}, []);
```

## Type Definitions (For Reference)

```typescript
// Equipment
interface Equipment {
  id: string;
  name: string;
  type: 'PC Portable' | 'PC Fixe' | 'Imprimante' | 'Périphérique' | 'Accessoire' | 'Autre';
  status: 'active' | 'stock' | 'maintenance' | 'disposed';
  location: string;
  assignedUser: string | null;
  purchaseDate: string;
  serialNumber: string;
  macAddress?: string;
  notes?: string;
}

// Stock
interface StockLevel {
  equipmentId: string;
  quantity: number;
  minThreshold: number;
  location: string;
  lastRestocked: string;
}

// Movement
interface Movement {
  id: string;
  equipmentId: string;
  type: 'entry' | 'exit' | 'transfer' | 'return';
  date: string;
  from: string;
  to: string;
  user: string;
  notes?: string;
}

// Disposal
interface DisposedEquipment {
  id: string;
  equipment: Equipment;
  disposalDate: string;
  disposalReason: string;
  disposalMethod: 'recycle' | 'donate' | 'destroy';
  certificate?: string;
}

// Location
interface Location {
  id: string;
  name: string;
  floor: string;
  capacity: number;
  equipment: Equipment[];
  manager: string;
}
```

---

These patterns cover ~95% of the use cases. For unique requirements, refer to the specific page implementations in `DashboardNew.tsx` and `ParcNew.tsx`.
