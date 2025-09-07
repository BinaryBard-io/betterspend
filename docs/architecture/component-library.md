# BetterSpend Component Library

## Overview
This document describes the BetterSpend component library, including design system, reusable components, and development guidelines.

## Design System

### Design Principles
- **Consistency**: Uniform look and feel across the application
- **Accessibility**: WCAG 2.1 AA compliance
- **Responsiveness**: Mobile-first design approach
- **Performance**: Optimized for fast loading and interaction
- **Maintainability**: Easy to update and extend

### Color Palette
```css
/* Primary Colors */
--primary-50: #eff6ff;
--primary-100: #dbeafe;
--primary-500: #3b82f6;
--primary-600: #2563eb;
--primary-700: #1d4ed8;
--primary-900: #1e3a8a;

/* Secondary Colors */
--secondary-50: #f8fafc;
--secondary-100: #f1f5f9;
--secondary-500: #64748b;
--secondary-600: #475569;
--secondary-700: #334155;
--secondary-900: #0f172a;

/* Status Colors */
--success-500: #10b981;
--warning-500: #f59e0b;
--error-500: #ef4444;
--info-500: #06b6d4;
```

### Typography
```css
/* Font Families */
--font-sans: 'Inter', system-ui, sans-serif;
--font-mono: 'JetBrains Mono', monospace;

/* Font Sizes */
--text-xs: 0.75rem;    /* 12px */
--text-sm: 0.875rem;   /* 14px */
--text-base: 1rem;     /* 16px */
--text-lg: 1.125rem;   /* 18px */
--text-xl: 1.25rem;    /* 20px */
--text-2xl: 1.5rem;    /* 24px */
--text-3xl: 1.875rem;  /* 30px */
--text-4xl: 2.25rem;   /* 36px */

/* Font Weights */
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;
--font-bold: 700;
```

### Spacing System
```css
/* Spacing Scale */
--space-1: 0.25rem;   /* 4px */
--space-2: 0.5rem;    /* 8px */
--space-3: 0.75rem;   /* 12px */
--space-4: 1rem;      /* 16px */
--space-5: 1.25rem;   /* 20px */
--space-6: 1.5rem;    /* 24px */
--space-8: 2rem;      /* 32px */
--space-10: 2.5rem;   /* 40px */
--space-12: 3rem;     /* 48px */
--space-16: 4rem;     /* 64px */
--space-20: 5rem;     /* 80px */
```

### Border Radius
```css
--radius-sm: 0.125rem;   /* 2px */
--radius-md: 0.375rem;   /* 6px */
--radius-lg: 0.5rem;     /* 8px */
--radius-xl: 0.75rem;    /* 12px */
--radius-2xl: 1rem;      /* 16px */
--radius-full: 9999px;
```

## Component Categories

### Layout Components

#### Container
```typescript
interface ContainerProps {
  children: React.ReactNode;
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl' | '2xl';
  padding?: 'sm' | 'md' | 'lg';
  className?: string;
}

const Container: React.FC<ContainerProps> = ({
  children,
  maxWidth = 'xl',
  padding = 'md',
  className = ''
}) => {
  return (
    <div className={cn(
      'mx-auto',
      maxWidthClasses[maxWidth],
      paddingClasses[padding],
      className
    )}>
      {children}
    </div>
  );
};
```

#### Grid
```typescript
interface GridProps {
  children: React.ReactNode;
  cols?: 1 | 2 | 3 | 4 | 6 | 12;
  gap?: 'sm' | 'md' | 'lg';
  className?: string;
}

const Grid: React.FC<GridProps> = ({
  children,
  cols = 1,
  gap = 'md',
  className = ''
}) => {
  return (
    <div className={cn(
      'grid',
      gridColsClasses[cols],
      gapClasses[gap],
      className
    )}>
      {children}
    </div>
  );
};
```

#### Stack
```typescript
interface StackProps {
  children: React.ReactNode;
  direction?: 'row' | 'column';
  spacing?: 'sm' | 'md' | 'lg';
  align?: 'start' | 'center' | 'end' | 'stretch';
  justify?: 'start' | 'center' | 'end' | 'between' | 'around';
  className?: string;
}

const Stack: React.FC<StackProps> = ({
  children,
  direction = 'column',
  spacing = 'md',
  align = 'stretch',
  justify = 'start',
  className = ''
}) => {
  return (
    <div className={cn(
      'flex',
      directionClasses[direction],
      spacingClasses[spacing],
      alignClasses[align],
      justifyClasses[justify],
      className
    )}>
      {children}
    </div>
  );
};
```

### Form Components

#### Input
```typescript
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  helperText?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  variant?: 'default' | 'filled' | 'outlined';
}

const Input: React.FC<InputProps> = ({
  label,
  error,
  helperText,
  leftIcon,
  rightIcon,
  variant = 'default',
  className = '',
  ...props
}) => {
  return (
    <div className="w-full">
      {label && (
        <label className="block text-sm font-medium text-gray-700 mb-1">
          {label}
        </label>
      )}
      <div className="relative">
        {leftIcon && (
          <div className="absolute left-3 top-1/2 transform -translate-y-1/2">
            {leftIcon}
          </div>
        )}
        <input
          className={cn(
            'w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500',
            variantClasses[variant],
            error && 'border-error-500 focus:ring-error-500',
            leftIcon && 'pl-10',
            rightIcon && 'pr-10',
            className
          )}
          {...props}
        />
        {rightIcon && (
          <div className="absolute right-3 top-1/2 transform -translate-y-1/2">
            {rightIcon}
          </div>
        )}
      </div>
      {error && (
        <p className="mt-1 text-sm text-error-500">{error}</p>
      )}
      {helperText && !error && (
        <p className="mt-1 text-sm text-gray-500">{helperText}</p>
      )}
    </div>
  );
};
```

#### Select
```typescript
interface SelectProps {
  label?: string;
  error?: string;
  helperText?: string;
  options: Array<{ value: string; label: string; disabled?: boolean }>;
  placeholder?: string;
  value?: string;
  onChange?: (value: string) => void;
  className?: string;
}

const Select: React.FC<SelectProps> = ({
  label,
  error,
  helperText,
  options,
  placeholder,
  value,
  onChange,
  className = ''
}) => {
  return (
    <div className="w-full">
      {label && (
        <label className="block text-sm font-medium text-gray-700 mb-1">
          {label}
        </label>
      )}
      <select
        className={cn(
          'w-full px-3 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-primary-500',
          error && 'border-error-500 focus:ring-error-500',
          className
        )}
        value={value}
        onChange={(e) => onChange?.(e.target.value)}
      >
        {placeholder && (
          <option value="" disabled>
            {placeholder}
          </option>
        )}
        {options.map((option) => (
          <option
            key={option.value}
            value={option.value}
            disabled={option.disabled}
          >
            {option.label}
          </option>
        ))}
      </select>
      {error && (
        <p className="mt-1 text-sm text-error-500">{error}</p>
      )}
      {helperText && !error && (
        <p className="mt-1 text-sm text-gray-500">{helperText}</p>
      )}
    </div>
  );
};
```

#### Button
```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  fullWidth?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  loading = false,
  leftIcon,
  rightIcon,
  fullWidth = false,
  children,
  className = '',
  disabled,
  ...props
}) => {
  return (
    <button
      className={cn(
        'inline-flex items-center justify-center font-medium rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2',
        variantClasses[variant],
        sizeClasses[size],
        fullWidth && 'w-full',
        (disabled || loading) && 'opacity-50 cursor-not-allowed',
        className
      )}
      disabled={disabled || loading}
      {...props}
    >
      {loading && (
        <div className="mr-2 h-4 w-4 animate-spin rounded-full border-2 border-current border-t-transparent" />
      )}
      {leftIcon && !loading && (
        <div className="mr-2">{leftIcon}</div>
      )}
      {children}
      {rightIcon && !loading && (
        <div className="ml-2">{rightIcon}</div>
      )}
    </button>
  );
};
```

### Data Display Components

#### Table
```typescript
interface TableProps {
  data: Array<Record<string, any>>;
  columns: Array<{
    key: string;
    label: string;
    render?: (value: any, row: any) => React.ReactNode;
    sortable?: boolean;
    width?: string;
  }>;
  onSort?: (key: string, direction: 'asc' | 'desc') => void;
  loading?: boolean;
  emptyMessage?: string;
  className?: string;
}

const Table: React.FC<TableProps> = ({
  data,
  columns,
  onSort,
  loading = false,
  emptyMessage = 'No data available',
  className = ''
}) => {
  if (loading) {
    return (
      <div className="animate-pulse">
        <div className="h-4 bg-gray-200 rounded w-full mb-2"></div>
        <div className="h-4 bg-gray-200 rounded w-3/4 mb-2"></div>
        <div className="h-4 bg-gray-200 rounded w-1/2"></div>
      </div>
    );
  }

  if (data.length === 0) {
    return (
      <div className="text-center py-8 text-gray-500">
        {emptyMessage}
      </div>
    );
  }

  return (
    <div className={cn('overflow-x-auto', className)}>
      <table className="min-w-full divide-y divide-gray-200">
        <thead className="bg-gray-50">
          <tr>
            {columns.map((column) => (
              <th
                key={column.key}
                className={cn(
                  'px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider',
                  column.sortable && 'cursor-pointer hover:bg-gray-100'
                )}
                onClick={() => column.sortable && onSort?.(column.key, 'asc')}
                style={{ width: column.width }}
              >
                {column.label}
                {column.sortable && (
                  <div className="inline-block ml-1">
                    <svg className="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path d="M5 12l5-5 5 5H5z" />
                    </svg>
                  </div>
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody className="bg-white divide-y divide-gray-200">
          {data.map((row, index) => (
            <tr key={index} className="hover:bg-gray-50">
              {columns.map((column) => (
                <td key={column.key} className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                  {column.render ? column.render(row[column.key], row) : row[column.key]}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};
```

#### Card
```typescript
interface CardProps {
  children: React.ReactNode;
  title?: string;
  subtitle?: string;
  actions?: React.ReactNode;
  variant?: 'default' | 'elevated' | 'outlined';
  className?: string;
}

const Card: React.FC<CardProps> = ({
  children,
  title,
  subtitle,
  actions,
  variant = 'default',
  className = ''
}) => {
  return (
    <div className={cn(
      'bg-white rounded-lg',
      variantClasses[variant],
      className
    )}>
      {(title || subtitle || actions) && (
        <div className="px-6 py-4 border-b border-gray-200">
          <div className="flex items-center justify-between">
            <div>
              {title && (
                <h3 className="text-lg font-medium text-gray-900">{title}</h3>
              )}
              {subtitle && (
                <p className="text-sm text-gray-500">{subtitle}</p>
              )}
            </div>
            {actions && (
              <div className="flex items-center space-x-2">
                {actions}
              </div>
            )}
          </div>
        </div>
      )}
      <div className="px-6 py-4">
        {children}
      </div>
    </div>
  );
};
```

### Feedback Components

#### Alert
```typescript
interface AlertProps {
  children: React.ReactNode;
  variant?: 'success' | 'warning' | 'error' | 'info';
  title?: string;
  dismissible?: boolean;
  onDismiss?: () => void;
  className?: string;
}

const Alert: React.FC<AlertProps> = ({
  children,
  variant = 'info',
  title,
  dismissible = false,
  onDismiss,
  className = ''
}) => {
  return (
    <div className={cn(
      'rounded-md p-4',
      variantClasses[variant],
      className
    )}>
      <div className="flex">
        <div className="flex-shrink-0">
          {variantIcons[variant]}
        </div>
        <div className="ml-3 flex-1">
          {title && (
            <h3 className={cn(
              'text-sm font-medium',
              variantTitleClasses[variant]
            )}>
              {title}
            </h3>
          )}
          <div className={cn(
            'text-sm',
            variantTextClasses[variant]
          )}>
            {children}
          </div>
        </div>
        {dismissible && (
          <div className="ml-auto pl-3">
            <button
              className={cn(
                'inline-flex rounded-md p-1.5 focus:outline-none focus:ring-2 focus:ring-offset-2',
                variantDismissClasses[variant]
              )}
              onClick={onDismiss}
            >
              <svg className="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
                <path fillRule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clipRule="evenodd" />
              </svg>
            </button>
          </div>
        )}
      </div>
    </div>
  );
};
```

#### Loading Spinner
```typescript
interface LoadingSpinnerProps {
  size?: 'sm' | 'md' | 'lg';
  color?: 'primary' | 'secondary' | 'white';
  className?: string;
}

const LoadingSpinner: React.FC<LoadingSpinnerProps> = ({
  size = 'md',
  color = 'primary',
  className = ''
}) => {
  return (
    <div className={cn(
      'animate-spin rounded-full border-2 border-t-transparent',
      sizeClasses[size],
      colorClasses[color],
      className
    )} />
  );
};
```

## Component Development Guidelines

### Naming Conventions
- Use PascalCase for component names
- Use descriptive, action-oriented names
- Include component type in name (Button, Input, Card)
- Use consistent prefixes for related components

### Props Interface
- Define clear, typed interfaces for all props
- Use optional props with sensible defaults
- Include className prop for styling flexibility
- Document all props with JSDoc comments

### Styling Approach
- Use Tailwind CSS utility classes
- Create reusable style variants
- Support custom className overrides
- Maintain consistent spacing and sizing

### Accessibility
- Include proper ARIA attributes
- Support keyboard navigation
- Provide screen reader support
- Ensure color contrast compliance

### Testing
- Write unit tests for all components
- Test different prop combinations
- Include accessibility tests
- Test responsive behavior

### Documentation
- Document component purpose and usage
- Provide code examples
- Include prop descriptions
- Show different variants and states

## Component Library Structure

```
src/components/
├── ui/                    # Basic UI components
│   ├── Button/
│   ├── Input/
│   ├── Select/
│   ├── Card/
│   └── Table/
├── layout/               # Layout components
│   ├── Container/
│   ├── Grid/
│   ├── Stack/
│   └── Sidebar/
├── forms/                # Form-specific components
│   ├── FormField/
│   ├── FormGroup/
│   └── ValidationMessage/
├── feedback/             # Feedback components
│   ├── Alert/
│   ├── Toast/
│   └── LoadingSpinner/
├── navigation/           # Navigation components
│   ├── Navbar/
│   ├── Sidebar/
│   └── Breadcrumb/
└── business/             # Business-specific components
    ├── RequisitionCard/
    ├── VendorCard/
    └── ApprovalStatus/
```

## Usage Examples

### Basic Form
```typescript
import { Button, Input, Select, Card } from '@/components';

const RequisitionForm = () => {
  return (
    <Card title="Create Requisition" className="max-w-2xl">
      <div className="space-y-4">
        <Input
          label="Title"
          placeholder="Enter requisition title"
          required
        />
        <Input
          label="Description"
          placeholder="Enter description"
          multiline
          rows={3}
        />
        <Select
          label="Vendor"
          options={vendorOptions}
          placeholder="Select vendor"
        />
        <div className="flex justify-end space-x-2">
          <Button variant="outline">Cancel</Button>
          <Button variant="primary">Create</Button>
        </div>
      </div>
    </Card>
  );
};
```

### Data Table
```typescript
import { Table, Button, Badge } from '@/components';

const RequisitionsTable = ({ requisitions }) => {
  const columns = [
    {
      key: 'title',
      label: 'Title',
      sortable: true
    },
    {
      key: 'status',
      label: 'Status',
      render: (value) => <Badge variant={value}>{value}</Badge>
    },
    {
      key: 'totalAmount',
      label: 'Amount',
      render: (value) => formatCurrency(value)
    },
    {
      key: 'actions',
      label: 'Actions',
      render: (_, row) => (
        <div className="flex space-x-2">
          <Button size="sm" variant="outline">View</Button>
          <Button size="sm" variant="primary">Edit</Button>
        </div>
      )
    }
  ];

  return (
    <Table
      data={requisitions}
      columns={columns}
      onSort={handleSort}
      loading={loading}
    />
  );
};
```

## Maintenance

### Regular Tasks
- Review component usage across application
- Update component documentation
- Refactor components for better performance
- Add new components based on needs

### Version Control
- Use semantic versioning for component library
- Maintain changelog for breaking changes
- Provide migration guides for updates
- Test components across different environments

### Performance Optimization
- Lazy load components when possible
- Optimize bundle size
- Use React.memo for expensive components
- Implement virtual scrolling for large lists
