# Zustand

> **"Giống 1 biến cục bộ có phạm vi global"** — nhưng tự động trigger re-render component đang dùng nó.

## Zustand là gì?

Zustand là thư viện state management nhỏ gọn cho React. Về bản chất chỉ là:

- **1 object JS** nằm ngoài React tree
- Kèm cơ chế **subscribe/notify** — component nào đang đọc field đó thì tự re-render khi field thay đổi

```
JavaScript thường:  let count = 0         // global, nhưng KHÔNG trigger re-render
Zustand:            count = 0             // global + TỰ ĐỘNG trigger re-render
```

---

## Cài đặt

```bash
npm install zustand
```

---

## Tạo Store

```typescript
// stores/counterStore.ts
import { create } from 'zustand';

type CounterStore = {
    count: number;
    increment: () => void;
    decrement: () => void;
    reset: () => void;
};

export const useCounterStore = create<CounterStore>((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 })),
    decrement: () => set((state) => ({ count: state.count - 1 })),
    reset: () => set({ count: 0 }),
}));
```

---

## Dùng trong Component

```typescript
function Counter() {
    // ✅ Selector — chỉ re-render khi 'count' thay đổi
    const count = useCounterStore((s) => s.count);
    const increment = useCounterStore((s) => s.increment);

    return <button onClick={increment}>{count}</button>;
}
```

---

## Selector — Quan trọng nhất

Selector quyết định component nào re-render khi nào.

```typescript
// ✅ Đúng — chỉ re-render khi field đó thay đổi
const count = useCounterStore((s) => s.count);
const user  = useStore((s) => s.user);

// ❌ Sai — re-render mỗi khi BẤT KỲ field nào trong store thay đổi
const store = useCounterStore();
```

---

## Pattern thực tế — Realtime Message (Next.js)

Bài toán: socket nhận message mới → nhiều component cần update UI.

```typescript
// stores/messageStore.ts
import { create } from 'zustand';
import { MessageNotify } from '@/lib/types/types';

type MessageStore = {
    latestMessage: MessageNotify | null;
    setLatestMessage: (msg: MessageNotify) => void;
};

export const useMessageStore = create<MessageStore>((set) => ({
    latestMessage: null,
    setLatestMessage: (msg) => set({ latestMessage: msg }),
}));
```

```typescript
// MessageListener.tsx — subscriber socket duy nhất
export default function MessageListener() {
    const { onUserPersonal, connected } = useSocket();
    const setLatestMessage = useMessageStore((s) => s.setLatestMessage);

    useEffect(() => {
        if (!connected) return;
        const unsub = onUserPersonal((data) => {
            if (data.message) setLatestMessage(data.message);
        });
        return () => unsub();
    }, [connected]);

    return null;
}
```

```typescript
// InboxList.tsx — không subscribe socket, chỉ đọc store
function InboxList() {
    const latestMessage = useMessageStore((s) => s.latestMessage);

    useEffect(() => {
        if (!latestMessage) return;
        // update conversation list UI...
    }, [latestMessage]);
}
```

```
Trước:  Socket → Component A ─┐
        Socket → Component B ─┤  2 subscriber, race condition
        Socket → Component C ─┘

Sau:    Socket → MessageListener → Zustand Store
                                        ↓
                          Component A (useStore)
                          Component B (useStore)
                          Component C (useStore)
```

---

## set() — Các cách dùng

```typescript
// Replace một field
set({ count: 0 });

// Dựa vào state cũ
set((state) => ({ count: state.count + 1 }));

// Merge nhiều field
set((state) => ({
    count: state.count + 1,
    lastUpdated: new Date(),
}));

// Mặc định set() MERGE — không replace toàn bộ store
// Muốn replace toàn bộ: set(newState, true)
set(newState, true);
```

---

## get() — Đọc state bên trong action

```typescript
export const useStore = create<Store>((set, get) => ({
    count: 0,
    double: () => {
        const current = get().count;   // đọc state hiện tại
        set({ count: current * 2 });
    },
}));
```

---

## Dùng ngoài Component (không cần hook)

```typescript
// Gọi action từ bất kỳ đâu — không cần ở trong React component
useMessageStore.getState().setLatestMessage(msg);

// Đọc state
const msg = useMessageStore.getState().latestMessage;

// Subscribe thủ công
const unsub = useMessageStore.subscribe(
    (state) => state.latestMessage,
    (latest) => console.log('New message:', latest)
);
unsub(); // cleanup
```

---

## Persist — Lưu vào localStorage

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useThemeStore = create(
    persist(
        (set) => ({
            theme: 'light',
            setTheme: (theme: string) => set({ theme }),
        }),
        { name: 'theme-storage' } // key trong localStorage
    )
);
```

---

## Khi nào nên dùng Zustand?

|Tình huống|Dùng Zustand?|
|---|---|
|State chỉ dùng trong 1 component|❌ Dùng `useState`|
|State cha → con 1-2 cấp|❌ Dùng props / `useContext`|
|State dùng ở nhiều nơi không liên quan|✅|
|Realtime data (socket, SSE) cần share|✅|
|State cần persist qua session|✅|
|Server data (API response)|❌ Dùng React Query / SWR|

---

## Zustand vs Context API

||Zustand|Context API|
|---|---|---|
|Boilerplate|Ít|Nhiều (Provider, wrap component)|
|Re-render|Chỉ component dùng field đó|Toàn bộ component trong Provider|
|Dùng ngoài React|✅ `getState()`|❌|
|DevTools|✅ Redux DevTools|❌|
|Bundle size|~1KB|0 (built-in)|

---

## Zustand vs Redux

||Zustand|Redux|
|---|---|---|
|Setup|5 dòng|Boilerplate nhiều|
|Learning curve|Thấp|Cao|
|DevTools|✅|✅ (tốt hơn)|
|Middleware|Có|Rất nhiều|
|Dùng khi|App vừa-lớn|App enterprise, team lớn|

---

## Lưu ý về performance

```typescript
// ✅ Store ít data — KHÔNG lag
latestMessage: MessageNotify | null    // 1 object nhỏ

// ⚠️ Mới cần cẩn thận
allMessages: MessageNotify[]           // array lớn + re-render không kiểm soát
```

Nguyên tắc: **store delta, không store toàn bộ history**. History để cho React Query / server.

---

## Related

- [[React Query]] — dành cho server state (API data)
- [[Socket Provider]] — nơi khởi tạo socket connection
- [[MessageListener]] — pattern subscriber duy nhất

---

_Tags: #react #nextjs #state-management #zustand_