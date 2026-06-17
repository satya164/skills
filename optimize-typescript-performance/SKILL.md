---
name: optimize-typescript-performance
description: Optimize TypeScript type-checking and editor performance in libraries or apps with complex generic types. Use when diagnosing slow `tsc`, sluggish language-service/editor behavior, high type instantiations, TS2589-style deep instantiation errors, expensive overloads, recursive conditional or mapped types, route-tree-style inference, or public APIs whose type surface needs to scale.
---

# Optimize TypeScript Performance

Use this skill to reduce TypeScript checker and language-service work while preserving public type safety and inference.

## Workflow

1. Measure first.
   - Run the project typecheck with diagnostics:
     ```sh
     npx tsc --noEmit --extendedDiagnostics
     ```
   - Track `Types`, `Instantiations`, `Memory used`, `Check time`, and `Total time`.
   - Treat `Instantiations` as the most stable proxy for checker work. Wall time is useful but noisier.
   - For libraries, measure declaration emit (`tsc --emitDeclarationOnly`) separately. A large share of cost can be the library type-checking itself, which declaration emit exposes.
   - Caveat: `Instantiations` maps reliably to wall time under `tsc`. Under the parallel Go compiler (`tsgo`/TypeScript 7), a widely-referenced type can be instantiated once per worker, so the total maps less directly to wall-clock time.
   - If the reported problem is editor latency, also measure `tsserver` operations: first semantic diagnostics after opening a file, hover/quickinfo, completion, and semantic diagnostics after a small edit.

2. Locate the expensive shape.
   - Generate a trace for real regressions:
     ```sh
     npx tsc --noEmit --generateTrace trace
     ```
   - Inspect `trace/trace.json` in Perfetto or another trace viewer.
   - Inspect `trace/types.json` for large instantiated types, recursive conditionals, hot overload resolution, and repeated union or mapped-type construction.

3. Build a focused benchmark.
   - Isolate representative API calls or expressions when possible.
   - Use `@ark/attest` when available to assert expression-level instantiation counts.
   - Include common paths, rare paths, happy paths, error paths, and a baseline expression unrelated to the tested type.
   - For library APIs that scale with app size, generate a realistic stress fixture: deep nesting, wide unions/maps, common callbacks, and representative API consumers.
   - Run multiple samples and handle outliers consistently.

4. Iterate with the BAM loop.
   - **Branch**: isolate one hypothesis per branch.
   - **Adjust**: make one conceptual type-system change.
   - **Measure**: compare diagnostics and editor-latency metrics against the baseline.
   - Keep benchmark changes separate from type implementation changes where possible.

5. Validate type behavior.
   - Run type tests or fixture builds for public API behavior.
   - Verify negative tests still fail for the intended reason.
   - Reject changes that widen types, lose useful inference, hide errors, or rely on `any`/assertions at public boundaries.

## Techniques

### Reduce Overload Search

Use this when many overloads share the same parameter and return shape.

Prefer fewer overloads with literal unions:

```ts
function op<T extends StringExpr>(
  left: T,
  operator: "=" | "!=" | ">" | "<" | "ilike",
  right: T,
): BooleanExpr;
```

Avoid many one-literal overloads that return the same shape. Overload resolution may walk candidates repeatedly, especially for late-match and error cases.

Verify common and rare calls. Consolidating overloads can improve later overloads while making early overloads slightly more expensive.

### Order Branches By Cost And Frequency

Use this when a type has ordered overloads or conditional branches.

Prefer cheap and common success cases before rare or expensive branches. Put an expensive branch first only when that branch is common enough to justify the extra checks for other cases.

Prefer cheap structural checks before template-literal or path-pattern checks when either branch proves the same high-level result:

```ts
type ShouldInfer<T> = T extends { parse: Record<string, unknown> }
  ? true
  : T extends `:${string}` | { path: `:${string}` }
    ? true
    : false;
```

Verify error-path behavior. A branch order that speeds happy paths can move cost into diagnostics users see frequently.

### Guard Expensive Work

Use this when a mapped, recursive, or union-building branch is only needed for a subset of inputs.

Prefer cheap discriminants before expensive branches:

```ts
type ExtraParams<Params, Parse> = [Exclude<keyof Parse, keyof Params>] extends [
  never,
]
  ? {}
  : BuildQueryParamTypes<Params, Parse>;
```

Other useful guards:

- Detect broad names with `string extends Name` before building a full lookup union.
- Check that a config, group, screen map, schema object, or nested branch exists before mapping its keys.
- Add fast paths for config-free cases so simple inputs skip parse-aware, schema-aware, or option-aware helpers.
- Return `never` for impossible union members so later intersections combine less empty work.

### Reuse Heavy Intermediate Types

Use this when the same expensive type expression appears in constraints, parameter types, and return types.

Prefer binding the result once with a helper alias, `infer`, or an extra generic parameter:

```ts
type PickFromList<
  Source,
  List = BuildLargeList<Source>,
  Key extends keyof List = keyof List,
> = List[Key];
```

Avoid repeating `BuildLargeList<Source>` in multiple positions. Repetition can push otherwise-correct types over TypeScript's instantiation-depth budget.

This is reliable when the bound result is read from several positions. Binding a single-position normalization the same way is an experiment, not a guaranteed win, so measure before keeping it:

```ts
type Result<T, Normalized = Normalize<T>> = BuildResult<Normalized>;
```

### Avoid Redundant Constraint Capture

Use this when a conditional type captures a value only to re-assert the same constraint required by a helper.

Prefer direct distributive constraints when TypeScript can narrow the checked type enough:

```ts
type VisitNested<T> = T extends {} ? Visit<T> : never;
```

Avoid extra `infer` wrappers unless they are needed for inference or constraint satisfaction:

```ts
type VisitNested<T> = T extends infer U extends {} ? Visit<U> : never;
```

Measure this in recursive helpers. Redundant captures can add substantial instantiations even when the result type is identical.

### Name Expensive Conditional Types

Use this when a large conditional return type is repeated inline.

Prefer named aliases:

```ts
type FooResult<T, U> =
  U extends TypeA<T> ? ProcessA<U, T> : U extends TypeB<T> ? ProcessB<U, T> : U;

type Api<T> = {
  foo<U>(value: U): FooResult<T, U>;
};
```

Avoid anonymous conditional types embedded in several public members. Named aliases give the checker a reusable target.

A named type is a cache point only when it is used from multiple places with the *same* arguments. Wrapping a conditional that still varies per call site in another named layer adds instantiations instead of removing them. Measure before keeping such a split: prefer removing the computation over merely renaming it.

### Hoist Parameter-Independent Types

Use this when a generic type repeatedly inlines an object fragment that does not depend on its type parameters.

Prefer top-level fragments:

```ts
type NavigateOptions = {
  merge?: boolean;
  pop?: boolean;
};
```

Avoid recreating the same object type inside every route, field, schema, or overload instantiation.

### Prefer Interfaces For Extensible Object Shapes

Use this when a large object shape is repeatedly related to other types.

Prefer:

```ts
interface Foo extends Bar, Baz {
  value: string;
}
```

Avoid equivalent large intersections when an interface can express the same shape:

```ts
type Foo = Bar &
  Baz & {
    value: string;
  };
```

Interface relationships can be cached more effectively. Intersections often require checking each constituent.

### Use Self-Typed Bags For Reusable Families

Use this when a library exposes a family of related generic APIs: navigators, routers, schemas, clients, stores, builders, or adapters.

Prefer one self-typed interface plus shared factories:

```ts
interface BuilderTypeBag extends BuilderTypeBagBase {
  State: BuilderState<this["Input"]>;
  Helpers: BuilderHelpers<this["Input"]>;
}
```

Avoid rebuilding a full generic object type for each concrete input. A self-typed bag keeps the custom surface small while letting a factory supply the varying type parameter.

### Use Variance Annotations When Sound

Use this when an exported generic object type is repeatedly related, and in particular when a type parameter flows into conditional types.

When a type parameter flows into a conditional type, the compiler's variance probing is marked unreliable and it falls back to full structural comparison on every relation. Annotating the parameter `in out` (invariant) makes the compiler relate instantiations by their type arguments directly, skipping the probe and the structural fallback:

```ts
interface Api_Core<in out T, in out U> {
  // ...
}
```

`in out` is the most restrictive annotation, so it always passes the compiler's variance check soundly: narrowing a parameter to invariant only removes assignments that were already allowed, it never introduces an unsound one. The check does not catch over-restriction, though. Forcing a genuinely covariant parameter to invariant still compiles but rejects valid assignments — for example, making a covariant value parameter invariant breaks the widening from `Box<string>` to `Box<unknown>` that callers rely on. Only annotate parameters that are invariant in practice, and treat a failing build as the real check.

### Hand-Write Intersections For Statically-Known Members

Use this when a type intersects a fixed, known-ahead-of-time set of members but builds that intersection with `UnionToIntersection`.

`UnionToIntersection` is the standard user-defined helper for turning a union into an intersection:

```ts
type UnionToIntersection<T> = (
  T extends any ? (x: T) => void : never
) extends (x: infer R) => void
  ? R
  : never;
```

It distributes the union into one function type per member and infers their intersection from a contravariant position. It is expensive, and none of it caches when the union's identity varies with a type parameter. Where the members are statically known, write the intersection directly:

```ts
type Options_All<T, U> = Options_Core<T, U> &
  Partial<
    Options_ModuleA<T, U> &
      Options_ModuleB &
      // ...the other known modules...
      Options_Plugins<T, U>
  >;
```

Keep `UnionToIntersection` only for members that genuinely vary with a type parameter, such as user-registered plugins, and put it behind a guard so the expensive path is skipped when none exist:

```ts
type Options_Plugins<T, U> = [
  Exclude<keyof Options_Map<T, U>, BuiltInKeys>,
] extends [never]
  ? unknown
  : UnionToIntersection</* plugin entries only */>;
```

### Use Broad Internal Types For Hot Internal Paths

Use this when internal library code threads a public type assembled from a conditional selection of members through many generic functions.

Internal code rarely needs the conditional view. Define broad `*_All` types whose slots are all present regardless of registration, and relate those internally instead of the public conditional type:

```ts
interface Api_Internal<in out T, in out U>
  extends Api_PartA<T, U>,
    Api_PartB<T, U> {
  options: Options_All<T, U>;
  initialState: State_All;
}
```

The internal type is then an interface with statically-known members and a stable identity, so the compiler relates two instantiations without re-expanding the conditional on every internal call. The public type is untouched, so inference at call sites is unchanged.

Pair this with variance annotations. Converting a conditional-alias internal type to an interface can regress on its own — a fully materialized interface whose parameters flow into conditionals hits the structural-fallback path above — and only becomes a win once those parameters are annotated `in out`.

### Use The Weakest Internal Guard That Proves The Branch

Use this inside conditional helper types where exact validation already happened at a public boundary.

Prefer checking the minimum structure needed:

```ts
T extends { config: unknown } ? ... : ...
```

Avoid forcing the checker to prove a full precise shape in every internal branch:

```ts
T extends { config: StaticConfig<FullTypeBag> } ? ... : ...
```

Keep exact constraints at public boundaries. Use cheaper structural guards in internal routing, schema, or builder helpers.

### Avoid Hot-Path Mapped Utility Types

Use this when a type is instantiated for every route, field, schema member, prop, or overload.

Prefer purpose-built base types over repeated mapped utilities:

- Split a reusable `Base` type that already excludes unwanted members instead of `Omit`-ing them off a larger type.
- Apply `Readonly` once at the outer object level instead of wrapping nested branches redundantly.
- Use `K extends keyof Parent` instead of `Parent extends Record<K, unknown>` when only key existence matters.
- Avoid `Pick`, `Omit`, `Record`, `Readonly`, and flattening helpers in hot recursive paths unless measurement proves they are acceptable.

### Prefer Direct Lookup Over Union Scans

Use this when an API looks up one item by key or name inside a large tree/map.

Prefer matching the key at each level and recursing only into nested maps:

```ts
type ItemForName<Map, Name extends string> =
  | (Name extends keyof Map ? Item<Map, Name> : never)
  | ItemForNameInChildren<NestedMaps<Map>, Name>;
```

Avoid building every possible item and then filtering:

```ts
type ItemForName<Map, Name extends string> = Extract<
  AllItems<Map>,
  { name: Name }
>;
```

The direct lookup scales with nesting depth. The union scan scales with total item count.

### Materialize Repeated Global Derivations

Use this when a large union or lookup map is derived repeatedly from a stable tree, schema, or registry.

Prefer generated or declared lookup maps:

```ts
type RoutesByPath = {
  "/": typeof IndexRoute;
  "/posts": typeof PostsRoute;
  "/posts/$postId": typeof PostRoute;
};
```

Avoid deriving the same global map at every API boundary:

```ts
type RoutesByPath<Tree> = {
  [R in ParseRoute<Tree> as R["fullPath"]]: R;
};
```

This applies to routes, schemas, generated clients, query builders, event maps, feature registries, and form/path APIs.

### Gather Cheap Metadata Separately

Use this when validation needs only names, keys, tags, or IDs but the existing type builds full objects.

Prefer a dedicated lightweight metadata walk:

```ts
type AllNames<Tree> = keyof Tree | AllChildNames<ChildTrees<Tree>>;
```

Avoid building full item/route/schema object types just to read their `name` or `kind` property.

### Store Type Metadata For Later Reads

Use this when a public type is repeatedly decomposed to recover the same pieces.

Prefer storing private metadata in a brand or hidden slot:

```ts
class TypeStore<T extends unknown[]> {
  protected ""?: T;
}
```

Then read frequently needed pieces from the slot before falling back to structural matching. This is cheaper than repeatedly matching a full public shape or walking `A & B` with mapped utilities.

### Thread Accumulators Through Recursive Types

Use this when recursive transforms repeatedly wrap or recompute all descendants.

Prefer threading accumulated context through recursion:

```ts
type Visit<Node, Parent = undefined> = Compose<Node, Parent> &
  VisitChildren<Node, Compose<Node, Parent>>;
```

Avoid "build all descendants, then wrap the whole descendant list at every level" patterns. Those often become quadratic in nesting depth.

### Give APIs Narrowing Hints

Use this when an API otherwise infers from the whole application graph.

Prefer explicit discriminants such as `from`, `to`, `kind`, `operator`, `schema`, `feature`, or a concrete source registry. For route-like APIs, pass concrete source and destination literals so the checker resolves one path instead of the whole tree.

Avoid loose modes that intentionally collapse everything into one broad union unless the call site really needs them.

### Pass Explicit Type Arguments To Construction Helpers

Use this when a generic helper is called with an anonymous object literal, often a spread, and must infer its type parameters from that object's shape.

Inference from an anonymous object is far more expensive than a plain assignability check: the compiler must solve for the type parameters before it can check anything. Pass the type arguments explicitly so it only has to check assignability:

```ts
const instance = build<T, U>({
  ...options,
  // same object as below
});
```

Avoid relying on inference from the literal:

```ts
const instance = build({ /* same object */ });
```

Passing an already-typed variable instead of a literal has the same effect. Audit construction helpers and hooks for this pattern. A single such call can dominate a package's check time.

### Avoid Recursive Path Types On Recursive Data

Use this when path/key helpers traverse recursive data.

Prefer flattening the input type before passing it to path helpers:

```ts
type FlatNode = Omit<TreeNode, "children">;
```

Avoid feeding recursive fields such as `children` or `parent` into deep key/path utilities. They can cause TS2589 or very high instantiation counts.

## What To Avoid

- Do not optimize based on intuition alone. Type checker performance is often counterintuitive.
- Do not replace precise public types with `any`, broad `unknown`, or assertions just to reduce instantiations.
- Do not assume fewer lines of types means faster checking.
- Do not assume naming or splitting a type always helps. A named type is a cache point only when reused with identical arguments; otherwise it just adds a layer.
- Do not trust one benchmark expression. Measure a representative matrix.
- Do not optimize only full-program `tsc` time when the reported pain is editor latency.

## Reporting Results

When presenting a change, include:

- Baseline and final `Types`, `Instantiations`, `Memory used`, `Check time`, and `Total time`.
- Editor-latency metrics when relevant.
- The benchmarked expressions, fixtures, or files.
- The refactoring hypothesis.
- Tradeoffs, such as common paths improving while rare paths regress.
- Verification that public type behavior is unchanged.

## Sources

- Gel: "An approach to optimizing TypeScript type checking performance" - diagnostics, traces, `@ark/attest`, BAM, overload consolidation, conditional ordering, interfaces over intersections, named conditional types. https://www.geldata.com/blog/an-approach-to-optimizing-typescript-type-checking-performance
- TanStack Router: "A milestone for TypeScript Performance in TanStack Router" - generated route maps, avoiding whole-tree inference, language-service tracing, explicit `from`/`to` narrowing. https://tanstack.com/blog/tanstack-router-typescript-performance
- TanStack Table V9: "Taking Form" - feature registries, type-safe opt-in APIs, stable registries, and performance work that preserves type safety. https://tanstack.com/blog/tanstack-table-v9-taking-form
- TanStack Table V9: "TypeScript Performance in TanStack Table V9" - named feature-map interfaces over hand-written conditional unions, broad `*_All` internal types, hand-written intersections for statically-known member sets, `in out` variance annotations when parameters flow into conditionals, explicit type arguments at construction sites, and the limit of the "name everything" rule. https://tanstack.com/blog/tanstack-table-v9-typescript-performance
- TanStack Intent: "Ship Agent Skills with your npm Packages" - keep agent guidance versioned with the package so type-performance rules match the actual API version. https://tanstack.com/blog/from-docs-to-agents
- React Navigation PR #13080 - reusable type-bag interfaces and shared factories. https://github.com/react-navigation/react-navigation/pull/13080
- React Navigation PR #13081 - variance annotations, cached intermediate types, and weak internal guards. https://github.com/react-navigation/react-navigation/pull/13081
- React Navigation PR #13134 - cached navigation-list types, impossible-branch pruning, single-pass recursive composition, and private metadata slots. https://github.com/react-navigation/react-navigation/pull/13134
- React Navigation PR #13140 - hot-path mapped utility removal and guarded query-param mapped types. https://github.com/react-navigation/react-navigation/pull/13140
- React Navigation PR #13149 - direct keyed lookup, distributive nested lookup, top-level object fragments, and lightweight metadata gathering. https://github.com/react-navigation/react-navigation/pull/13149
