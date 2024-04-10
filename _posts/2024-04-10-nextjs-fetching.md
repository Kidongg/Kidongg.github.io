---
title: Next.js App Router에서 데이터 가져오기(Feat. 클라이언트 컴포넌트, 서버 컴포넌트)
date: 2024-04-10 11:30:00 +0900
categories: [Frontend, Experience]
tags: [Next.js, Next.js App router, Client Component, Server Component, Fetch API]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 최근에 서버 컴포넌트를 클라이언트 컴포넌트로 변경해야할 일이 있었습니다. 그 과정에서 클라이언트 컴포넌트와 서버 컴포넌트가 데이터를 가져오는 방법이 다르다는 점을 알게 되었습니다. 본 포스팅에서는 두 컴포넌트의 특징과 차이점을 데이터 가져오는 방법을 중심으로 살펴보고자 합니다.
{: .prompt-info }

## 클라이언트 컴포넌트와 서버 컴포넌트
Next.js의 SSR(Server Side Rendering)은 전통적인 의미의 SSR이 아니라, SSR과 CSR(Client Side Rendering)의 장점을 동시에 취한 형태입니다. 초기 로딩 시에는 HTML을 서버에서 빠르게 받아오고, JS번들을 병렬적으로 받아와 HTML과 병합하는 Hydration의 과정을 거칩니다. 즉, 빠른 로딩이 장점인 SSR과 인터렉션이 장점인 CSR을 동시에 사용할 수 있는 것입니다. 
<br />

클라이언트 컴포넌트와 서버 컴포넌트의 가장 큰 차이점은 렌더링되는 장소입니다. 클라이언트 컴포넌트는 클라이언트가 JS번들을 다운로드 받은 후 데이터를 가져와 렌더링하며, 서버 컴포넌트는 서버에서 데이터를 가져와 렌더링된 HTML을 전달합니다. 
<br />

다음은 [Next.js 공식문서](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns){:target="\_blank"}에서 안내하는 서버 컴포넌트 및 클라이언트 컴포넌트가 필요한 상황입니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-fetching%2Fimage_1.png?alt=media&token=ce99bff2-e6aa-4b82-bb64-0f69de07bb43)
_출처 : Next.js Docs_

저는 인터렉티브한 상태를 관리해야했기 때문에 클라이언트 컴포넌트로 리팩토링이 필수적이었습니다. 그럼에도 불구하고 상태 관리가 필요하지 않는 화면은 서버 컴포넌트로 구현하고 싶었습니다. 서버 컴포넌트가 다음과 같은 장점이 있기 때문입니다.

1. 클라이언트 컴포넌트로 전달되는 JS번들 사이즈를 줄여준다.
2. 자동으로 코드를 분할해준다.
3. 스트림 방식의 렌더링이 가능하다.
4. 컴포넌트 단위의 refetching이 가능하다.
5. 민감한 정보를 보호할 수 있다. 클라이언트에서 네트워크로 전달되는 데이터를 확인할 수 없다.

## 결과

### 클라이언트 컴포넌트
새로고침 시 서버에서 데이터를 불러오고 있습니다. 따라서 데이터를 불러오는 중에 화면이 변경됩니다. 

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-fetching%2Fimage_2.gif?alt=media&token=faea298b-6184-4121-a08f-0d5aeeaec909)

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-fetching%2Fimage_5.png?alt=media&token=820a70d9-d945-4b15-ba18-f62dcb363f49)

클라이언트 컴포넌트에서는 다음과 같은 과정이 일어납니다.
1. 렌더링되지 않은 HTML을 서버로부터 받아온다.
2. 클라이언트에서 데이터를 불러온다.
3. 클라이언트에서 Hydration 이후 렌더링을 한다.

### 서버 컴포넌트
새로고침 시 캐시에서 데이터를 불러오고 있습니다. 따라서 데이터를 불러오는 중에 화면이 변경되지 않습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-fetching%2Fimage_3.gif?alt=media&token=26677e30-a8f9-47b2-a2a1-7ca757e34311)

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-fetching%2Fimage_4.png?alt=media&token=40771ca9-f4a0-4a70-a6c0-9262bc949731)

서버 컴포넌트에서는 다음과 같은 과정이 일어납니다.
1. 서버에서 데이터를 불러온다.
2. 서버에서 렌더링을 한다.
3. 렌더링된 HTML을 서버로부터 받아온다.
4. 렌더링된 HTML을 캐시에 저장한다.
5. 저장한 캐시에서 HTML을 받아온다.

## 구현

### 1. json-server 설치 및 실행하기
json-server를 설치한 이유는 네트워크에서 데이터를 주고 받고 싶었기 때문입니다. 전역으로 json-server를 설치하고 3001번 포트에서 서버를 실행시켰습니다.

```shell
npm i -g json-server
json-server --watch db.json --port 3001
```

연습용 데이터 구조는 다음과 같습니다.
```json
// db.json

{
  "payments": [
    {
      "id": "728ed52f",
      "price": 100,
      "status": "pending",
      "email": "m@example.com",
      "name": "마늘"
    },
    ...
  ]
}
```

### 2. 테이블 구현하기
UI는 shadcn/ui를 사용했습니다. Next.js와 호환성이 좋고 커스터 마이징을 할 수 있기 때문입니다.

```shell
npx shadcn-ui@latest init
```

shadcn/ui에서는 `/components/ui`에 폴더를 생성해 컴포넌트를 다운로드 받습니다. 파일 구조를 깔끔하게 하기 위해 경로를 `/app/components/ui`로 변경했습니다.

```json
// components.json

{
  ...
  "aliases": {
    "components": "@/app/components",
    "utils": "@/app/lib/utils"
  }
}
```

컴포넌트를 다운받으면 설정한 경로에 다운로드가 됩니다.

```shell
npx shadcn-ui@latest add table
npm install @tanstack/react-table
```

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-fetching%2Fimage_6.png?alt=media&token=e7965a2e-70de-472c-bf12-d05b24f2156c)

사용할 테이블을 컴포넌트화했습니다. 테이블에 필요한 데이터는 테이블 데이터와 컴럼 데이터입니다. 두 데이터를 `useReactTable` 함수의 인자로 넣어줍니다.

```react
// app/components/table/columns.ts

"use client";

...

// 결제 정보 컬럼
export const paymentColumns: ColumnDef<Payment>[] = [
  {
    accessorKey: "name",
    header: "결제명",
  },
  {
    accessorKey: "price",
    header: "금액",
  },
  {
    accessorKey: "email",
    header: "이메일",
  },
  {
    accessorKey: "status",
    header: "상태",
    cell: ({ row }) => {
      return <span>{TransactionType[row.original.status]}</span>;
    },
  },
  ...
];

//! 타입 정의
interface Payment {
  id: string;
  price: number;
  status: "pending" | "processing" | "success" | "failed";
  email: string;
}
```

```react
// app/components/table/DataTable.tsx

"use client";

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
}

const DataTable = <TData, TValue>({
  data,
  columns,
}: DataTableProps<TData, TValue>) => {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
  });

  return (
    <div className="rounded-md border">
      <Table>
        {/* 테이블 헤더 */}
        <TableHeader>
          {table.getHeaderGroups().map((headerGroup) => (
            <TableRow key={headerGroup.id}>
              {headerGroup.headers.map((header) => {
                return (
                  <TableHead key={header.id}>
                    {header.isPlaceholder
                      ? null
                      : flexRender(
                          header.column.columnDef.header,
                          header.getContext()
                        )}
                  </TableHead>
                );
              })}
            </TableRow>
          ))}
        </TableHeader>
        {/* 테이블 바디 */}
        <TableBody>
          {table.getRowModel().rows?.length ? (
            table.getRowModel().rows.map((row) => (
              <TableRow
                key={row.id}
                data-state={row.getIsSelected() && "selected"}
              >
                {row.getVisibleCells().map((cell) => (
                  <TableCell key={cell.id}>
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </TableCell>
                ))}
              </TableRow>
            ))
          ) : (
            <TableRow>
              <TableCell colSpan={columns.length} className="h-24 text-center">
                No results.
              </TableCell>
            </TableRow>
          )}
        </TableBody>
      </Table>
    </div>
  );
};

export default DataTable;
```

### 3. 조회하는 API 함수 만들기
`payment-apis.ts` 폴더에서 결제와 관련된 API를 관리했습니다.

```typescript
// app/apis/payment-api.ts

// 결제 조회 API 함수
export const getPayments = async () => {
  const res = await fetch("http://localhost:3001/payments", {
    method: "GET",
    headers: {
      "Content-Type": "application/json",
    },
  });

  if (!res.ok) {
    throw new Error(`HTTP error! status: ${res.status}`);
  }

  const data = await res.json();
  return data;
};
```

### 4. 클라이언트 컴포넌트에서 데이터를 불러오고 렌더링하기
네트워크 탭에서 불러오는 데이터를 확인할 수 있습니다. 서버로부터 데이터가 없는 HTML을 받고 있으며, 클라이언트에서 불러오는 데이터를 볼 수 있습니다. 로그도 클라이언트(브라우저)에서 찍히는걸 볼 수 있습니다.

```react
// app/page.tsx

const Page = () => {
  return (
    <div className="py-10 pl-10 pr-10 flex flex-col gap-4 w-full">
      <h1 className="text-2xl font-bold">클라이언트 컴포넌트</h1>
      <Button />
      <Table />
    </div>
  );
};

export default Page;
```

```react
// app/components/(client)/button/Button.tsx

"use client";

const Button = () => {
  const { isAddPaymentsModal, setIsAddPaymentsModal } = useModal();

  // 모달 열기 함수
  const handleModalOpen = () => {
    setIsAddPaymentsModal(true);
  };

  return (
    <>
      <div
        className="bg-blue-500 border-none text-white px-8 py-4 text-center no-underline inline-block text-lg mx-1 my-1 cursor-pointer rounded-sm w-full mt-10"
        onClick={handleModalOpen}
      >
        추가하기
      </div>
      {isAddPaymentsModal && <PaymentAddModal />}
    </>
  );
};

export default Button;
```

```react
// app/components/(client)/table/Table.tsx

"use client";

const Table = () => {
  const [payments, setPayments] = useState([]); // 결제 정보 상태

  const { isEditPaymentsModal } = useModal(); // 모달 상태

  // 데이터 패칭 함수 실행
  useEffect(() => {
    const fetchPayments = async () => {
      try {
        const payments = await getPayments();
        setPayments(payments);
      } catch (error) {
        console.error(error);
      }
    };

    fetchPayments();
  }, []);

  // console.log("payments: ", payments);

  return (
    <>
      <DataTable data={payments} columns={paymentColumns} />
      {isEditPaymentsModal && <PaymentEditModal />}
    </>
  );
};

export default Table;
```

### 5. 서버 컴포넌트에서 데이터를 불러오고 렌더링하기
네트워크 탭에서 불러오는 데이터를 확인할 수 없습니다. 서버에서 데이터를 불러오기 때문입니다. 서버로부터 렌더링된 HTML을 받아오고 있습니다. 로그도 서버(터미널)에서 찍히는걸 볼 수 있습니다. 
<br />

`<Page />` 컴포넌트에서 데이터를 불러와 props로 내려준 이유는 `<Table />` 컴포넌트에서 모달 상태를 사용하고 있기 때문입니다.

```react
// app/(example)/server/page.tsx

const Page = async () => {
  const payments = await getPayments();

  return (
    <div className="py-10 pl-10 pr-10 flex flex-col gap-4 w-full">
      <h1 className="text-2xl font-bold">서버 컴포넌트</h1>
      <Button />
      <Table payments={payments} />
    </div>
  );
};

export default Page;
```

```react
// app/component/(server)/button/Button.tsx

"use client";

const Button = () => {
  const { isAddPaymentsModal, setIsAddPaymentsModal } = useModal();

  // 모달 열기 함수
  const handleModalOpen = () => {
    setIsAddPaymentsModal(true);
  };

  return (
    <>
      <div
        className="bg-blue-500 border-none text-white px-8 py-4 text-center no-underline inline-block text-lg mx-1 my-1 cursor-pointer rounded-sm w-full mt-10"
        onClick={handleModalOpen}
      >
        추가하기
      </div>
      {isAddPaymentsModal && <PaymentAddModal />}
    </>
  );
};

export default Button;
```

```react
// app/component/(server)/table/Table.tsx

"use client";

const Table: React.FC<TableProps> = ({ payments }) => {
  const { isEditPaymentsModal } = useModal(); // 모달 상태

  return (
    <>
      <DataTable data={payments} columns={paymentColumns} />
      {isEditPaymentsModal && <PaymentEditModal />}
    </>
  );
};

export default Table;

//! 타입 정의
interface TableProps {
  payments: Payment[];
}

interface Payment {
  id: string;
  price: number;
  status: PaymentStatus;
  email: string;
  name: string;
}

type PaymentStatus = "pending" | "processing" | "success" | "failed";
```

## 결론

- 서버 컴포넌트는 최초 1회를 제외하고 캐시에서 데이터를 불러오고, 클라이언트 컴포넌트는 항상 서버에서 데이터를 불러온다.
- 서버 컴포넌트는 JS번들 사이즈가 적고, 클라이언트 컴포넌트는 JS번들 사이즈가 크다. 따라서 성능 측면에서 서버 컴포넌트가 효율적이다.
- 클라이언트 컴포넌트에서도 리액트 쿼리를 사용하면 최초 1회를 제외하고 캐시에서 데이터를 불러올 수 있다. 물론 초반 JS번들 사이즈는 크지만 API 호출을 줄일 수 있다는 점에서 성능 측면에서 어느 정도 커버할 수 있다.

서버 컴포넌트와 클라이언트 컴포넌트는 트레이드 오프가 존재합니다. 따라서 필요와 상황에 따라 적절히 사용 해야겠습니다. 개인적으로는 정적인 페이지, CRUD 정도의 간단한 동적인 페이지에서는 서버 컴포넌트를, 상태 관리가 필요한 완전히 동적인 페이지에서는 클라이언트 컴포넌트를 활용하는 것이 좋을 것 같습니다. 이를 위해서 개발을 시작하기 전 필요한 기능을 정리하고 컴포넌트를 설계하는 과정이 필요할 것 같습니다.

## 참고자료
- [Next.js 서버 컴포넌트에 대한 고찰](https://velog.io/@2ast/React-%EC%84%9C%EB%B2%84-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8React-Server-Component%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0#rscreact-server-component-vs-rccreact-client-component){:target="\_blank"}
- [Rendering: Composition Patterns](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns){:target="\_blank"}
- [Data Table](https://ui.shadcn.com/docs/components/data-table){:target="\_blank"}