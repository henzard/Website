---
description: "Admin dashboard patterns for content management and data administration"
alwaysApply: false
globs: ["**/admin/**", "**/dashboard/**"]
---

# Admin Dashboard Standards

## 🎯 Core Principles

- ✅ Role-based access (admin only)
- ✅ Data tables with sorting/filtering
- ✅ CRUD operations
- ✅ Bulk actions
- ✅ Activity logging

---

## 🔒 Admin Route Protection

```typescript
// components/AdminRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';

export const AdminRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user, userProfile, loading } = useAuth();

  if (loading) return <LoadingSpinner />;
  if (!user) return <Navigate to="/login" />;
  if (!userProfile?.roles.includes('admin')) return <Navigate to="/forbidden" />;

  return <>{children}</>;
};
```

---

## 📊 Data Table Component

```typescript
// components/admin/DataTable.tsx
interface Column<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
  render?: (value: any, item: T) => React.ReactNode;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  onEdit?: (item: T) => void;
  onDelete?: (item: T) => void;
  loading?: boolean;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  onEdit,
  onDelete,
  loading,
}: DataTableProps<T>) {
  const [sortKey, setSortKey] = useState<keyof T | null>(null);
  const [sortDir, setSortDir] = useState<'asc' | 'desc'>('asc');
  const [selected, setSelected] = useState<Set<string>>(new Set());

  const sortedData = useMemo(() => {
    if (!sortKey) return data;
    return [...data].sort((a, b) => {
      const aVal = a[sortKey];
      const bVal = b[sortKey];
      const cmp = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
      return sortDir === 'asc' ? cmp : -cmp;
    });
  }, [data, sortKey, sortDir]);

  return (
    <div className="overflow-x-auto">
      <table className="w-full border-collapse">
        <thead className="bg-gray-50">
          <tr>
            <th className="p-3 text-left">
              <input
                type="checkbox"
                checked={selected.size === data.length}
                onChange={(e) => {
                  setSelected(e.target.checked ? new Set(data.map(d => d.id)) : new Set());
                }}
              />
            </th>
            {columns.map((col) => (
              <th
                key={String(col.key)}
                onClick={() => col.sortable && handleSort(col.key)}
                className={`p-3 text-left ${col.sortable ? 'cursor-pointer hover:bg-gray-100' : ''}`}
              >
                {col.label}
                {sortKey === col.key && (sortDir === 'asc' ? ' ↑' : ' ↓')}
              </th>
            ))}
            <th className="p-3 text-left">Actions</th>
          </tr>
        </thead>
        <tbody>
          {loading ? (
            <tr><td colSpan={columns.length + 2} className="p-8 text-center">Loading...</td></tr>
          ) : sortedData.length === 0 ? (
            <tr><td colSpan={columns.length + 2} className="p-8 text-center text-gray-500">No data</td></tr>
          ) : (
            sortedData.map((item) => (
              <tr key={item.id} className="border-b hover:bg-gray-50">
                <td className="p-3">
                  <input
                    type="checkbox"
                    checked={selected.has(item.id)}
                    onChange={(e) => {
                      const next = new Set(selected);
                      e.target.checked ? next.add(item.id) : next.delete(item.id);
                      setSelected(next);
                    }}
                  />
                </td>
                {columns.map((col) => (
                  <td key={String(col.key)} className="p-3">
                    {col.render ? col.render(item[col.key], item) : String(item[col.key])}
                  </td>
                ))}
                <td className="p-3">
                  <div className="flex gap-2">
                    {onEdit && (
                      <button onClick={() => onEdit(item)} className="text-blue-600 hover:underline">
                        Edit
                      </button>
                    )}
                    {onDelete && (
                      <button onClick={() => onDelete(item)} className="text-red-600 hover:underline">
                        Delete
                      </button>
                    )}
                  </div>
                </td>
              </tr>
            ))
          )}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 📈 Dashboard Stats

```typescript
// components/admin/StatsCards.tsx
interface Stat {
  label: string;
  value: string | number;
  change?: number;
  icon?: React.ReactNode;
}

export const StatsCards: React.FC<{ stats: Stat[] }> = ({ stats }) => {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
      {stats.map((stat, i) => (
        <div key={i} className="bg-white p-6 rounded-lg shadow">
          <div className="flex justify-between items-start">
            <div>
              <p className="text-gray-500 text-sm">{stat.label}</p>
              <p className="text-3xl font-bold mt-1">{stat.value}</p>
              {stat.change !== undefined && (
                <p className={`text-sm mt-1 ${stat.change >= 0 ? 'text-green-600' : 'text-red-600'}`}>
                  {stat.change >= 0 ? '↑' : '↓'} {Math.abs(stat.change)}%
                </p>
              )}
            </div>
            {stat.icon && <div className="text-3xl text-gray-400">{stat.icon}</div>}
          </div>
        </div>
      ))}
    </div>
  );
};
```

---

## 📋 Admin Layout

```typescript
// layouts/AdminLayout.tsx
export const AdminLayout: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <div className="flex min-h-screen">
      {/* Sidebar */}
      <aside className="w-64 bg-gray-900 text-white">
        <div className="p-4 text-xl font-bold">Admin</div>
        <nav className="mt-4">
          <NavLink to="/admin" className="block px-4 py-2 hover:bg-gray-800">Dashboard</NavLink>
          <NavLink to="/admin/users" className="block px-4 py-2 hover:bg-gray-800">Users</NavLink>
          <NavLink to="/admin/orders" className="block px-4 py-2 hover:bg-gray-800">Orders</NavLink>
          <NavLink to="/admin/products" className="block px-4 py-2 hover:bg-gray-800">Products</NavLink>
          <NavLink to="/admin/settings" className="block px-4 py-2 hover:bg-gray-800">Settings</NavLink>
        </nav>
      </aside>

      {/* Main Content */}
      <main className="flex-1 bg-gray-100">
        <header className="bg-white shadow px-6 py-4">
          <div className="flex justify-between items-center">
            <h1 className="text-xl font-semibold">Dashboard</h1>
            <UserMenu />
          </div>
        </header>
        <div className="p-6">{children}</div>
      </main>
    </div>
  );
};
```

---

## ✅ Checklist

```
Access Control
☐ Admin routes protected
☐ Role check on server too
☐ Audit logging enabled

Data Management
☐ CRUD operations work
☐ Confirmation for deletes
☐ Bulk actions available
☐ Search and filter

User Experience
☐ Loading states shown
☐ Error handling
☐ Success feedback
☐ Responsive design

IF ANY ☐ UNCHECKED → Fix before launching
```

---

**Remember: "Admin panels need the same care as public pages."** 📊✨

