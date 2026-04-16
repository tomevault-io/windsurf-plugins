---
trigger: always_on
description: - React 18 + TypeScript 5
---

# ForwaIQ 專案開發規範

## 技術棧
- React 18 + TypeScript 5
- Vite 6 (使用 SWC)
- Tailwind CSS + Radix UI
- Supabase (後端 API)
- React Hook Form (表單管理)

---

## 一、TypeScript 規範

### 基本原則
- 嚴格模式啟用：使用 `strict: true`
- 所有函數參數和返回值必須明確定義型別
- 使用 `interface` 定義物件結構，使用 `type` 定義聯合型別或複雜型別
- 避免使用 `any`，必要時使用 `unknown` 並進行型別守衛
- 使用 `Record<string, T>` 而非索引簽名 `{ [key: string]: T }`

### 型別定義規範
```typescript
// ✅ Props 介面命名：ComponentNameProps
interface QuoteListProps {
  quotes: Quote[];
  onEdit: (quote: Quote) => void;
  onDelete: (id: string) => void;
}

// ✅ 使用 interface 定義物件
interface Quote {
  id: string;
  vendorName: string;
  price: number;
}

// ✅ 使用 type 定義聯合型別
type VendorType = 'shipping' | 'trucking' | 'customs';

// ✅ 使用 Record 而非索引簽名
type CustomFieldValues = Record<string, any>;

// ❌ 避免使用 any
const processData = (data: any) => { };

// ✅ 使用具體型別
const processData = (data: Quote[]) => { };
```

---

## 二、React 組件規範

### 組件結構
- 使用函數式組件和 Hooks
- 組件檔案使用 PascalCase：`QuoteList.tsx`
- 一個檔案一個主要組件
- 使用具名導出 `export function ComponentName()` 而非預設導出（除了 App.tsx）
- Props 解構應在函數參數中完成

### 組件內部結構順序
```typescript
export function MyComponent({ prop1, prop2 }: MyComponentProps) {
  // 1. Hooks (useState, useRef, useContext)
  const [state, setState] = useState<string>('');
  const ref = useRef<HTMLDivElement>(null);
  
  // 2. useEffect
  useEffect(() => {
    loadData();
  }, []);
  
  // 3. 數據載入函數
  const loadData = async () => {
    try {
      // ...
    } catch (error) {
      console.error('載入數據失敗:', error);
    }
  };
  
  // 4. 事件處理函數
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // ...
  };
  
  const handleDelete = (id: string) => {
    // ...
  };
  
  // 5. 工具/輔助函數
  const formatData = (data: any) => {
    // ...
  };
  
  // 6. 渲染輔助函數
  const renderItem = (item: Item) => {
    // ...
  };
  
  // 7. 條件提早返回
  if (loading) {
    return <div>載入中...</div>;
  }
  
  if (error) {
    return <div>發生錯誤</div>;
  }
  
  // 8. 主要渲染
  return (
    <div>
      {/* JSX 內容 */}
    </div>
  );
}
```

### 組件範例
```typescript
// ✅ 好的組件結構
import { useState, useEffect } from 'react';
import type { Quote } from '../App';
import { Button } from './ui/button';

interface QuoteListProps {
  quotes: Quote[];
  onEdit: (quote: Quote) => void;
  onDelete: (id: string) => void;
}

export function QuoteList({ quotes, onEdit, onDelete }: QuoteListProps) {
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      setLoading(true);
      // API call
    } catch (error) {
      console.error('載入資料時發生錯誤:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleDelete = (id: string) => {
    if (confirm('確定要刪除嗎？')) {
      onDelete(id);
    }
  };

  if (loading) {
    return <div>載入中...</div>;
  }

  return (
    <div className="space-y-4">
      {quotes.map((quote) => (
        <div key={quote.id}>
          {/* Content */}
        </div>
      ))}
    </div>
  );
}
```

---

## 三、Hooks 使用規範

### 基本規則
- 自定義 Hook 必須以 `use` 開頭
- `useEffect` 依賴陣列必須完整且正確
- 避免在 `useEffect` 中直接定義 async 函數，應在外部定義後調用
- 使用 `useState` 時提供明確的初始值型別
- 複雜狀態邏輯考慮使用 `useReducer`

### 範例
```typescript
// ✅ useState 提供型別
const [quotes, setQuotes] = useState<Quote[]>([]);
const [loading, setLoading] = useState<boolean>(false);
const [formData, setFormData] = useState<FormData>({
  name: '',
  email: '',
});

// ✅ useEffect 正確使用
useEffect(() => {
  loadData();
}, []); // 依賴陣列完整

const loadData = async () => {
  // async 函數在外部定義
};

// ✅ 自定義 Hook
function useQuotes() {
  const [quotes, setQuotes] = useState<Quote[]>([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    // ...
  }, []);
  
  return { quotes, loading };
}

// ❌ 不要在 useEffect 中直接定義 async
useEffect(async () => {
  // 錯誤！
  const data = await fetchData();
}, []);
```

---

## 四、函數撰寫與命名規範

### 4.1 命名規則

#### 基本規則
- **camelCase**：所有函數使用駝峰式命名
- **動詞開頭**：函數名應以動詞開頭，清楚描述其行為
- **具體明確**：避免模糊命名

#### 常見動詞前綴

**事件處理函數**
```typescript
// 組件內部定義：使用 handle*
const handleSubmit = (e: React.FormEvent) => { };
const handleDelete = (id: string) => { };
const handleClickOutside = (event: MouseEvent) => { };

// 作為 props 傳遞：使用 on*
interface Props {
  onSubmit: (data: any) => void;
  onDelete: (id: string) => void;
  onOpenChange: (open: boolean) => void;
}
```

**數據操作函數**
```typescript
// load* - 從 API 或存儲載入數據
const loadQuotes = async () => { };
const loadCustomFields = async (vendorType: string) => { };

// save* - 保存數據
const saveVendor = async (vendor: Vendor) => { };

// update* - 更新數據
const updateContact = (id: string, field: string, value: any) => { };

// delete* / remove* - 刪除數據
const removeContact = (id: string) => { };

// create* / add* - 創建/新增數據
const addContact = () => { };
```

**狀態判斷函數（返回布林值）**
```typescript
// is* - 判斷狀態
const isExpired = (validUntil: string): boolean => {
  return new Date(validUntil) < new Date();
};

// has* - 判斷是否擁有
const hasValidContacts = (contacts: VendorContact[]): boolean => {
  return contacts.some(c => c.name?.trim());
};

// should* - 判斷是否應該
const shouldShowCustomField = (field: CustomField): boolean => {
  return !field.isRequired && activeOptionalFields.has(field.id);
};

// can* - 判斷是否能夠
const canSubmitForm = (): boolean => {
  return formData.name && formData.email;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kurt-liao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
