---
name: typescript
description: TypeScript 寫法規範。讀取或編輯 .ts、.tsx 檔案時套用。
disable-model-invocation: true
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

# TypeScript

寫或改 .ts、.tsx 時逐條套用。每條寫明規則、何時適用、最短範例。

## 型別建模

**Discriminated union。** 用 `kind` 字面值判別欄位建模各個變體，讓不可能的狀態無法被表示。不要用一包可選欄位。選定一個判別欄位名稱（`kind`、`type`、`tag`）並一路沿用。

```ts
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: string }
  | { kind: "error"; error: string };
```

**Branded type。** 用 `& { readonly __brand: "X" }` 為原始型別加 brand，避免互相混用。在邊界只驗證一次，下游信任型別。brand 來自會實際驗證的 type guard，絕不來自 `as`。沿用 `readonly __brand: "X"` 的形狀，不發明新慣例。

```ts
type AgentId = string & { readonly __brand: "AgentId" };
const isAgentId = (input: string): input is AgentId => /^[0-9a-f-]{36}$/.test(input);
```

**建構式建模。** 把形狀建構成讓不合法的值造不出來，不用執行期檢查去限制寬鬆的型別。非空用 `[T, ...T[]]`，偶數長度用 `[T, T][]`，範圍用 `start` 加 `durationMs`。收到一般的 `T[]` 時，用 guard 縮窄一次。`durationMs` 維持一般 number，除非原始數字可能被傳到預期是 duration 的地方，才依 branded type 加 brand。

```ts
type NonEmpty<T> = [T, ...T[]];
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
type TimeRange = { start: Date; durationMs: number };
```

**最簡單的 total type。** 對每個操作都仍是 total，就保留 `T[]`。只有寬鬆型別逼你用 `!`、`as` 或「不應該發生」的 throw 時，才在那個地方強化成 `NonEmpty<T>`。把結果弱化成 `T | undefined` 也是一種 total 的簽章。

```ts
type NonEmpty<T> = [T, ...T[]];
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0); // [] 得 0，不必強化
const newest = (sessions: NonEmpty<{ id: string }>) => sessions[0]; // 不需要 `!`
```

**由既有型別衍生。** 當 `.proto`、OpenAPI、GraphQL schema 或資料庫 migration 已定義某個形狀，就從產生出來的型別衍生，不要複製。宣告新 interface 前，先用 `Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof`。

```ts
type Row = { id: string; name: string; status: string };
type Summary = Pick<Row, "id" | "status">;
```

## 外部資料與轉型

**`unknown` 優於 `any`。** 外部資料一律是 `unknown`，使用前先縮窄。外部來源包括 RPC payload、`JSON.parse`、`postMessage`、IPC、檔案內容、環境變數、資料庫結果。

```ts
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    return input.foo;
  }
}
```

**先用 schema，再考慮 guard。** 為外部資料手寫逐屬性檢查的 type guard 之前，先找專案既有的執行期 schema 函式庫與 schema，從 schema 推導型別，不要同時維護 schema、重複的 interface 與 guard。失敗是預期分支時用 `safeParse`。專案用別的 schema 函式庫就用它的推導 helper。不為一個 guard 新增 schema 相依套件。

```ts
import { z } from "zod";

const UserSchema = z.object({ id: z.string().uuid(), role: z.enum(["admin", "member"]) });
type User = z.infer<typeof UserSchema>;
const parseUser = (input: unknown): User => UserSchema.parse(input);
```

**不用 `as` 轉型。** 每個 `as` 都是等著發生的執行期崩潰，只在驗證之後才轉型。型別先存在時，用它所證明的型別標註 validator，編譯器就會拒絕證明得比型別少的 validator。

```ts
import { z } from "zod";

type User = { id: string; name: string };
const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

重構掉既有的 `as` 時，先找出 TypeScript 為什麼推斷不出來：

- 缺 discriminant：補上，改成 discriminated union。
- 來源型別過寬（例如 `Record<string, unknown>`）：縮窄它。
- 邊界沒有型別：用擁有該形狀的 schema 解析，沒有 schema 的地方才新增。
- 確實無法表達：用 branded type 或 `satisfies`。

**`satisfies` 優於 `as`。** 驗證數值，又不放寬字面值型別。

```ts
type Config = { theme: "dark" | "light"; cols: number };
const config = { theme: "dark", cols: 3 } satisfies Config; // config.theme 是 "dark"
```

**邊界驗證。** 在資料進入的地方解析一次，轉成具名的領域型別。`Record<string, unknown>`（不論怎麼寫）在這次解析就到此為止。內部信任型別，不在呼叫鏈深處重複驗證。

- 傳輸格式（proto、JSON-RPC）：用 `ignoreUnknownFields` 解析，讓向前相容的變更不弄壞舊用戶端。
- 持久化的 JSON：帶版本的 blob，解析外面包 try/catch。

## 縮窄

**縮窄層級。** 由好到最後手段：discriminant switch > `in` 運算子 > `typeof`/`instanceof` > 使用者定義的 type guard > `as`。

```ts
type S = { radius: number } | { w: number; h: number };
const area = (s: S) => ("radius" in s ? s.radius ** 2 * Math.PI : s.w * s.h);
```

**Type guard。** 必須驗證它所宣稱的事。說謊的 guard 比 `as` 更糟，因為 bug 藏在宣稱安全的名字後面。命名為 `isX` 或 `hasX`。可以用 discriminant 縮窄時，優先用它。

```ts
type Shape = { kind: "circle"; radius: number } | { kind: "rect"; width: number; height: number };
const isCircle = (s: Shape): s is Extract<Shape, { kind: "circle" }> => s.kind === "circle";
```

**窮盡檢查。** 在 default 分支內嵌 `const _exhaustive: never = x;`，新增變體時編譯器就會報錯。回傳值的 switch 用 return 風格，陳述式的 switch 用 `void _exhaustive`。

```ts
type Shape = { kind: "circle"; radius: number } | { kind: "rect"; width: number; height: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}
```

## 函式與程式風格

**物件參數。** 傳物件，不傳位置參數，讓參數順序自帶說明。熱路徑除外：每影格渲染、tokenizer、parser，以及配置成本重要的緊密迴圈。

```ts
declare function openFile(opts: { uri: string; line: number }): void;
openFile({ uri: "a.ts", line: 10 });
```

**真實測試。** 能執行的就不要 mock。優先用框架真正的測試原語，搭配洩漏與 disposable 檢查，並在執行中的 build 驗證 UI。只 mock 無法在本機執行的東西。

**結構化遙測。** 診斷訊息優先用結構化 logger，附上足夠的 context，讓人能憑 id 除錯。交付的程式裡不留 `console.log`。
