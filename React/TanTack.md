# TanStack Query trong Next.js

## 1. TanStack Query là gì?

TanStack Query (trước đây là React Query) là thư viện quản lý state bất đồng bộ (async state) cho React.

Nó giúp:

- Fetch API dễ hơn
    
- Cache dữ liệu
    
- Đồng bộ dữ liệu server/client
    
- Retry request
    
- Pagination / Infinite Query
    
- Mutation (POST, PUT, DELETE)
    
- Optimistic Update
    
- Background Refetch
    
- Devtools debug
    

Trong Next.js, TanStack Query thường được dùng để:

- Quản lý API state
    
- Giảm việc dùng `useEffect`
    
- Cache dữ liệu giữa các page/component
    
- Tối ưu UX khi loading
    

---

# 2. Cài đặt

## Cài package

```bash
npm install @tanstack/react-query
```

Devtools:

```bash
npm install @tanstack/react-query-devtools
```

---

# 3. Setup QueryClient

## app/providers.tsx

```tsx
'use client';

import {
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query';
import { ReactNode, useState } from 'react';

export default function Providers({
  children,
}: {
  children: ReactNode;
}) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 1000 * 60,
            retry: 1,
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

---

# 4. Inject Provider vào layout

## app/layout.tsx

```tsx
import Providers from './providers';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

# 5. useQuery cơ bản

## API function

```ts
export async function getUsers() {
  const res = await fetch('https://jsonplaceholder.typicode.com/users');

  if (!res.ok) {
    throw new Error('Failed to fetch users');
  }

  return res.json();
}
```

---

## Sử dụng useQuery

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { getUsers } from '@/lib/apis/user';

export default function UserPage() {
  const {
    data,
    isLoading,
    error,
  } = useQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error</div>;
  }

  return (
    <div>
      {data.map((user: any) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

---

# 6. queryKey là gì?

`queryKey` dùng để định danh cache.

Ví dụ:

```ts
['users']
['users', id]
['posts', page]
['profile', userId]
```

TanStack Query sẽ cache theo key.

Ví dụ:

```ts
['users', 1]
```

khác với:

```ts
['users', 2]
```

---

# 7. Cách TanStack Query hoạt động

Flow:

```text
Component Mount
    ↓
Check Cache
    ↓
Có cache?
    ↓
YES → return cache
NO → fetch API
    ↓
Cache Result
    ↓
Update UI
```

---

# 8. staleTime

```ts
staleTime: 1000 * 60
```

Nghĩa là:

- Data được xem là fresh trong 1 phút
    
- Trong thời gian đó sẽ không refetch
    

---

# 9. gcTime (cacheTime cũ)

```ts
gcTime: 1000 * 60 * 5
```

Sau khi không còn component nào dùng query:

- cache vẫn tồn tại 5 phút
    
- sau đó bị garbage collect
    

---

# 10. Refetch

## Manual Refetch

```tsx
const { refetch } = useQuery({
  queryKey: ['users'],
  queryFn: getUsers,
});

<button onClick={() => refetch()}>
  Reload
</button>
```

---

# 11. Dynamic Query

```tsx
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => getUser(userId),
  enabled: !!userId,
});
```

`enabled` giúp query chỉ chạy khi có `userId`.

---

# 12. Mutation

Mutation dùng cho:

- POST
    
- PUT
    
- PATCH
    
- DELETE
    

---

## Ví dụ create user

```ts
export async function createUser(data: any) {
  const res = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify(data),
    headers: {
      'Content-Type': 'application/json',
    },
  });

  if (!res.ok) {
    throw new Error('Create failed');
  }

  return res.json();
}
```

---

## useMutation

```tsx
'use client';

import {
  useMutation,
  useQueryClient,
} from '@tanstack/react-query';

const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: createUser,

  onSuccess: () => {
    queryClient.invalidateQueries({
      queryKey: ['users'],
    });
  },
});
```

---

# 13. invalidateQueries

```ts
queryClient.invalidateQueries({
  queryKey: ['users'],
});
```

Nghĩa là:

- mark query là stale
    
- tự động refetch
    

Rất quan trọng sau mutation.

---

# 14. Optimistic Update

Cho phép update UI trước khi API hoàn thành.

Ví dụ:

- Chat app
    
- Todo app
    
- Social feed
    

---

## Flow

```text
User Action
    ↓
Update UI ngay lập tức
    ↓
Call API
    ↓
Success → giữ nguyên
Fail → rollback
```

---

# 15. Ví dụ Optimistic Update

```tsx
const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: createTodo,

  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({
      queryKey: ['todos'],
    });

    const previousTodos = queryClient.getQueryData([
      'todos',
    ]);

    queryClient.setQueryData(
      ['todos'],
      (old: any) => [...old, newTodo]
    );

    return { previousTodos };
  },

  onError: (_err, _newTodo, context) => {
    queryClient.setQueryData(
      ['todos'],
      context?.previousTodos
    );
  },

  onSettled: () => {
    queryClient.invalidateQueries({
      queryKey: ['todos'],
    });
  },
});
```

---

# 16. Pagination

```tsx
const { data } = useQuery({
  queryKey: ['posts', page],
  queryFn: () => getPosts(page),
});
```

Mỗi page sẽ có cache riêng.

---

# 17. Infinite Query

Dùng cho:

- Infinite Scroll
    
- Feed mạng xã hội
    
- Chat history
    

---

## Ví dụ

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';

const {
  data,
  fetchNextPage,
  hasNextPage,
} = useInfiniteQuery({
  queryKey: ['posts'],

  queryFn: ({ pageParam = 1 }) =>
    getPosts(pageParam),

  initialPageParam: 1,

  getNextPageParam: (lastPage) => {
    return lastPage.nextPage;
  },
});
```

---

# 18. Prefetch

Prefetch giúp load data trước.

Ví dụ:

- Hover link
    
- Preload page
    
- SSR hydration
    

```ts
await queryClient.prefetchQuery({
  queryKey: ['users'],
  queryFn: getUsers,
});
```

---

# 19. SSR với Next.js

TanStack Query hỗ trợ:

- SSR
    
- SSG
    
- Hydration
    

Trong App Router thường dùng:

- dehydrate
    
- HydrationBoundary
    

---

## Ví dụ SSR

### app/users/page.tsx

```tsx
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query';

import Users from './users';
import { getUsers } from '@/lib/apis/user';

export default async function Page() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });

  return (
    <HydrationBoundary
      state={dehydrate(queryClient)}
    >
      <Users />
    </HydrationBoundary>
  );
}
```

---

## users.tsx

```tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { getUsers } from '@/lib/apis/user';

export default function Users() {
  const { data } = useQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });

  return (
    <div>
      {data.map((user: any) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

---

# 20. Cấu trúc thư mục đề xuất

```text
src/
 ├── app/
 ├── components/
 ├── hooks/
 │    ├── queries/
 │    └── mutations/
 ├── lib/
 │    ├── apis/
 │    └── query-client.ts
 └── providers/
```

---

# 21. Pattern thực tế

## Query Hook

```ts
export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });
}
```

---

## Mutation Hook

```ts
export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createUser,

    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: ['users'],
      });
    },
  });
}
```

---

# 22. Axios với TanStack Query

## axios instance

```ts
import axios from 'axios';

export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  withCredentials: true,
});
```

---

## API function

```ts
export async function getProfile() {
  const res = await api.get('/users/profile');
  return res.data;
}
```

---

# 23. Error Handling

```tsx
const { error } = useQuery({
  queryKey: ['profile'],
  queryFn: getProfile,
});

if (error instanceof Error) {
  console.log(error.message);
}
```

---

# 24. Loading State

```tsx
const {
  isLoading,
  isFetching,
} = useQuery({
  queryKey: ['users'],
  queryFn: getUsers,
});
```

Khác nhau:

|State|Ý nghĩa|
|---|---|
|isLoading|loading lần đầu|
|isFetching|đang refetch|

---

# 25. Devtools

## Setup

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  {children}

  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

---

# 26. Khi nào nên dùng TanStack Query?

Nên dùng khi:

- App có nhiều API
    
- Có cache logic
    
- Có realtime/refetch
    
- Infinite scroll
    
- Pagination
    
- Mutation phức tạp
    
- Đồng bộ state server
    

---

# 27. Khi nào KHÔNG cần?

Không cần nếu:

- App rất nhỏ
    
- Chỉ vài API đơn giản
    
- Không cần cache
    
- Static data
    

---

# 28. TanStack Query vs Zustand

|TanStack Query|Zustand|
|---|---|
|Server State|Client State|
|API cache|UI state|
|Async data|Local state|
|Fetching|Modal/theme/auth UI|

---

# 29. Best Practices

## Tách API layer

Không fetch trực tiếp trong component.

---

## Query Key rõ ràng

```ts
['posts', page]
['profile', userId]
```

---

## Dùng custom hook

```ts
useUsers()
useProfile()
useCreatePost()
```

---

## Không lạm dụng invalidateQueries

Chỉ invalidate query cần thiết.

---

## Prefetch route quan trọng

Giúp UX nhanh hơn.

---

# 30. Một flow thực tế

```text
User mở Feed
    ↓
useQuery gọi API
    ↓
Cache data
    ↓
User tạo bài viết
    ↓
useMutation POST
    ↓
onSuccess
    ↓
invalidateQueries(['feed'])
    ↓
Feed refetch
```

---

# 31. Tổng kết

TanStack Query giải quyết:

- Data fetching
    
- Cache
    
- Sync server state
    
- Mutation lifecycle
    
- Optimistic UI
    
- Pagination
    
- Infinite scroll
    
- Retry/refetch
    
- SSR hydration
    

Đây gần như là standard cho React app hiện đại khi làm việc với API phức tạp.