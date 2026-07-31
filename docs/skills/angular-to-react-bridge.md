# Angular → React Bridge

## What it is

A conceptual map for senior Angular developers learning React and Next.js. React is a UI library (not a full framework); Next.js adds routing, SSR, and build tooling. There is no built-in DI, modules, or RxJS — patterns you rely on in Angular must be re-learned as composition, hooks, and explicit state.

## Why it matters for this role

The job requires React/Next.js at expert level. Your 15 years of component architecture, services, and routing transfer directly — but only if you stop reaching for Angular equivalents. This bridge shortens the ramp from weeks to days for everything except streaming-specific code.

## Core concepts checklist

- [ ] Function components replace class components + decorators
- [ ] Props are read-only (like `@Input()`); no two-way binding
- [ ] `useState` for local state; `useReducer` for complex state machines (player state)
- [ ] `useEffect` replaces `ngOnInit`, `ngOnDestroy`, and many RxJS subscriptions
- [ ] Custom hooks replace injectable services (`useAuth`, `useShakaPlayer`)
- [ ] Context API or Zustand replaces NgRx for global state
- [ ] Next.js `'use client'` vs Server Components (no Angular equivalent — new concept)
- [ ] File-based routing in `app/` directory replaces Angular Router config
- [ ] No `async` pipe — fetch with TanStack Query or `useEffect` + `fetch`
- [ ] JSX replaces templates; `className` replaces `class`; events are `onClick` not `(click)`

## Angular → React mapping

| Angular | React / Next.js | Notes |
|---------|-----------------|-------|
| `@Component` | `function MyComponent()` | One component per file is convention |
| `@Input()` | `props.title` | Destructure in function signature |
| `@Output() EventEmitter` | `props.onSave(data)` | Pass callbacks as props |
| `*ngIf="show"` | `{show && <Child />}` | |
| `*ngFor="let x of items"` | `{items.map(x => <Item key={x.id} />)}` | Always set `key` |
| `[ngClass]` | `className={cn('base', isActive && 'active')}` | Use `clsx` or Tailwind |
| `(click)="handler()"` | `onClick={handler}` | Pass function ref, not invocation |
| `[(ngModel)]` | `value={x} onChange={e => setX(e.target.value)}` | Controlled component pattern |
| `@accessToken` | `useContext(AuthContext)` | Or Zustand store |
| `HttpClient` | `fetch()` or axios in hooks / server actions | |
| `Router.navigate()` | `useRouter().push('/path')` from `next/navigation` | |
| `ActivatedRoute.params` | `useParams()` or page `params` prop | App Router passes params to pages |
| Guards | Middleware (`middleware.ts`) or layout checks | |
| Pipes | Plain functions or small utils | `formatDuration(secs)` |
| `@ViewChild` | `useRef<HTMLVideoElement>(null)` | Essential for player DOM access |

## Learning resources

| Resource | Type | URL |
|----------|------|-----|
| React docs — Thinking in React | Official | https://react.dev/learn/thinking-in-react |
| Next.js App Router migration | Official | https://nextjs.org/docs/app |
| React TypeScript Cheatsheet | Reference | https://react-typescript-cheatsheet.netlify.app |
| Angular vs React (Scrimba) | Course | https://scrimba.com/learn/angularreact |

## Hands-on exercises

1. **Translate a component:** Take an Angular component with `@Input`, `@Output`, and `*ngFor`. Rewrite as a React function component with typed props.
2. **Service → hook:** Convert an Angular `ContentService` with `HttpClient.get()` into a `useContent(id)` custom hook.
3. **Route migration:** Map 3 Angular routes to Next.js App Router file structure.
4. **State machine:** Model player states (`idle`, `loading`, `playing`, `buffering`, `error`) with `useReducer` — compare to NgRx reducer you know.
5. **Side-by-side:** Build the same catalog list in Angular (1 hr refresh) then React (2 hr) to feel the differences.

## Interview / on-the-job topics

- Why React uses one-way data flow vs Angular two-way binding
- When to use Server Components vs `'use client'` in Next.js (player = always client)
- How you migrated mental models from Angular services to React hooks
- Player state management: Context vs Zustand vs useReducer trade-offs
- How RxJS patterns (debounce, switchMap) map to React (useDebouncedCallback, TanStack Query)

## Related skills

- [typescript-react-nextjs.md](typescript-react-nextjs.md) — Deep dive on Next.js App Router
- [video-player-sdks.md](video-player-sdks.md) — `useShakaPlayer` hook pattern
- [css-styling-ui.md](css-styling-ui.md) — Tailwind vs Angular Material
