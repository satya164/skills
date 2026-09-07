---
name: code-quality-review
description: Review pull requests and local code changes for correctness, regressions, API design, accessibility, and meaningful simplifications. Use when asked to review code or a diff.
---

# Code quality review

Review the changes against the criteria below.

## Requirements and behavior

1. **Requirements:** The approach fits the stated intent. The code implements all requested behavior and covers all intended cases without affecting unrelated cases. Edits stay within the requested scope. Bug fixes address the reported version and root cause.

2. **Edge cases:** Empty, missing, and invalid inputs produce the intended results. Values at, just below, and just above a limit produce the intended results. Loops and indexes do not skip items or go past the end.

3. **Error handling:** The code handles exceptions, rejected promises, failed responses, and rate limits with defined outcomes. Broad catches and silent fallbacks do not hide failures or invalid state. Retries, timeouts, and other defensive measures address specific risks, including in new code.

4. **State modeling:** The data model allows only valid states. Explicit states and their required data replace flags and optional fields that allow contradictory combinations. Each value has one source of truth. Related values derive from that source instead of separate copies that could disagree.

5. **Mutation and concurrency:** Code does not mutate data it does not own, such as React props or shared Python defaults. Concurrent operations avoid races. Related writes cannot leave invalid partial state.

6. **Cancellation:** Stale work and interrupted animations cannot leave state, focus, or callbacks inconsistent.

7. **Compatibility:** Backward compatibility, including compatibility code and migrations, requires an explicit request. Documented support commitments alone do not count as a request.

8. **API design:** Public APIs describe what consumers can do. The API prevents contradictory states. Visibility, defaults, and extensibility fit the intended use. Existing style or native props take priority when they meet the requirements. Customization does not expose implementation details or allow invalid behavior. Defaults are not used for values that shouldn't have duplicates across instances such as `id`, `testID` etc.

## User interface

11. **Design specifications:** States, tokens, dimensions, icons, shapes, press effects, and disabled appearance match the design specification. Styles use theme defaults and named tokens. Spacing follows the design scale.

12. **Platforms and layout:** Behavior works on each affected platform, including iOS, Android, and web, and during server rendering. Scrolling and safe areas do not break positioning or interaction. Text and controls do not overlap or overflow at supported screen sizes.

13. **Accessibility:** Keyboard, mouse, and touch input work where applicable. Accessible labels, roles, focus order, and loading states fit the interaction. Disabled states suit the control's purpose and platform under the relevant guidelines. Disabled controls expose their state to assistive technology and prevent activation. A control that becomes unavailable after activation keeps focus. Closing temporary UI restores focus to an appropriate element.

14. **Animation and layout transitions:** Layout changes do not cause unnecessary shifts. Animations use only transform and opacity unless another property is necessary. Layout transitions use the First, Last, Invert, Play (FLIP) technique where relevant. Duration and easing fit the interaction and follow platform conventions. Dimensions are measured near animation start when this avoids extra state and gives current values.

## Component behavior

15. **Rendering:** Rendering derives values when possible instead of copying them into state through effects. Rendering does not run side effects or assign refs.

16. **Hook dependencies:** React Hooks follow the rules of hooks. Dependency arrays include the values hooks use. Dependencies do not include values recreated on each render. Assumed stability does not justify omitted dependencies, including consumer styles.

17. **Controlled components:** Controlled values have the callbacks needed to update them. Components support uncontrolled use when controlled rendering causes a specific performance or reliability problem. Components forward relevant native props and events without changing their meaning.

18. **Composition:** Composition does not depend on `React.Children`, element inspection, or `cloneElement` in React. Components still work when wrapped in other components. Composition uses context, component parts, render props, or data objects as appropriate.

19. **Platform code:** React Native platform branches use direct `Platform.OS` checks or `Platform.select`. React Native uses `.native` files for native/web splits.

20. **CSS features:** Code written for Web doesn't re-implement CSS features in JavaScript. Components avoid unnecessary layout work and measurements, re-renders etc. if the same result can be achieved with CSS.

21. **Style overrides:** Consumer styles merge after defaults for supported overrides. Computed styles do not silently replace supplied values.

22. **Serve rendering:** Components don't use browser-only APIs, layout measurements etc. during render. If they are necessary, their usage doesn't produce mismatches between server and client render.

## Type system

30. **Type design:** Types describe domain concepts. Related fields have consistent types. They require fields that must be present and reject values the code cannot handle. Generic arguments use inference where possible. Related types derive from one source. Custom types do not re-implement types that can be imported or derived from the standard library, dependencies.

31. **Type safety:** Type fixes correct source types instead of adding casts. The code does not use non-null assertions. TypeScript code does not use `@ts-ignore`. Each `@ts-expect-error` has a comment explaining why the code produces the error and why the error cannot be fixed. Runtime checks do not repeat conditions already guaranteed by types.

32. **Type syntax:** Runtime enums require a specific reason to use them instead of unions. Interfaces require a specific reason to use them instead of type aliases. Overloads and complex type logic appear only when necessary. JSDoc does not repeat existing type annotations.

## Readability and documentation

18. **Project conventions:** Code, comments, and tests follow documented project conventions and architectural decisions. Code that claims to follow an external standard or convention matches the official guidance.

19. **Naming:** Variable, function, constant, type, and prop names match their purpose and use established project and platform terms.

20. **Comments:** Comments explain only specific requirements, workarounds, reasoning, or behavior that is not self-explanatory. Comments are next to the relevant code. They are accurate and up to date. Comment rewrites correct inaccurate or misleading text only, follow existing language and style and don't change already correct text.

21. **Documentation content:** Documentation and help text explain purpose and current behavior in simple language. The text omits unnecessary counts and implementation details. Behavior changes include related documentation. Mentions of other APIs, components, or features link to the relevant content.

22. **Examples and structure:** Examples match the actual API, are self-contained, and focus on the relevant behavior. Migration examples show before/after code or a diff where useful. Heading levels follow the document structure. Links lead to the intended content.

## Structure and dependencies

23. **Module responsibilities:** Related logic stays near its data in the layer that owns it. Unrelated responsibilities stay separate. General utilities contain no feature-specific details. Dependencies are necessary and do not form cycles.

24. **Abstractions:** Abstractions simplify existing code and represent clear concepts on their own. Simple logic stays inline when that is clearer than an abstraction. Straightforward duplication is acceptable unless it repeats magic numbers or values that must change together. Subclasses meet the requirements of their parent classes.

25. **Complexity:** Branches, flags, nesting, and special cases are easy to follow. Refactoring adds indirection or spreads related logic only when that makes the code easier to follow. Missed simplifications and meaningful structural regressions are blockers. Line counts and file size are not blockers.

26. **Unused code:** Changes contain no unused or unreachable code, obsolete features, no-op normalization, or redundant conditions. Claims that code is unused account for use outside the repository.

27. **Package selection:** New packages provide enough benefit to justify their maintenance burden, license requirements, security risks, and added size. Existing or smaller alternatives take priority when they meet the requirements.

28. **Dependency updates:** Upgrades to direct and indirect dependencies preserve required behavior. The deprecation status of new or changed API usage is verified against the installed dependency version. Supported replacements take priority over deprecated APIs. Dependency manifests and lockfiles specify consistent versions. Install scripts follow project policy.

## Security

33. **External input:** External data passes validation before use. Queries use parameters. Rendered content uses the required encoding. Dynamic execution does not accept unsafe input. Uploads have size, type, and content restrictions.

34. **Authorization:** Protected operations check identity, ownership, roles, and required permissions.

35. **Sessions and account recovery:** Token validation covers signatures, expiry, and issuer. Sessions expire, and the system can revoke them. Account recovery verifies ownership. Recovery tokens expire. Account recovery rejects tokens that were already used.

36. **Sensitive data:** Source, logs, responses, and model context contain no secrets. Passwords, transmitted data, stored data, and backups have the required protection. Error messages do not expose internal details.

37. **Request security:** State-changing requests have protection against cross-site request forgery. The service permits only intended origins and redirect destinations. Outbound request validation prevents access to restricted networks. Changed endpoints have suitable security headers and request limits.

38. **File operations:** Destructive operations stay within authorized directories, including when paths contain symbolic links. Ownership checks use trusted evidence. Path changes between a check and an operation cannot redirect the operation.

## Performance and resources

39. **Work limits:** Loops, requests, allocations, and model calls have bounds that fit real input. Work that could grow too large uses limits or pagination. Independent work runs concurrently when sequential execution adds unnecessary delay. Blocking work does not prevent timely responses.

40. **Database queries:** Queries retrieve records together instead of making a separate query for each record. Indexes support filters and sorting without unnecessary write cost. Connection pools stay within capacity.

41. **Resource cleanup:** Resource cleanup releases connections, listeners, timers, and retained objects on success, failure, cancellation, and unmounting, as applicable. Repeated setup does not leave extra resources allocated.

42. **Caching:** Cache keys include all inputs and access context that affect a response. The cache behaves correctly during expiry, invalidation, eviction, concurrent misses, missing results, and pending writes. The cache provides enough benefit to justify its cost.

43. **Rendering performance:** Assets, fonts, bundles, and third-party scripts fit load budgets. Resources needed for the initial view take priority. Other resources load when needed. The code avoids unnecessary renders and forced layout work. Long lists and gestures stay responsive. Continuous gesture work runs off the JavaScript thread when needed.

44. **Memoization:** Memoization is limited to expensive calculations or values that must keep the same identity. Ordinary object creation or access alone does not justify it.

## Tests

45. **Coverage:** Tests cover requirements, boundaries, errors, regressions, meaningful integrations, and critical user paths. Tests assert public output and interaction instead of mock setup, private properties, context internals, or internal styles. Internal helpers have direct tests only when their complexity justifies them.

46. **Test quality:** Tests are readable, independent, and repeatable. Tests cover both happy-path and edge cases. Test names describe user-facing behavior. Test logic is simple. Additional test cases check behavior that existing tests do not cover.

47. **Mocking:** Mocks are not used for code that can be tested with real data. Mocks are only used for non-deterministic, external, slow dependencies or when the tests cannot run in the test environment.

48. **Querying:** Tests use accessibility attributes for queries when possible, and test IDs otherwise. Test IDs only identify public interaction or content and do not access internals.

## Review practices

### Scope and context

For pull requests, review the diff. For local changes, review staged, unstaged, and untracked changes. Read surrounding code as needed. Keep findings tied to the reviewed changes.

Use available requirements, the pull request description, and project conventions to understand the intended behavior. Ask for clarification when unclear intent could change whether something is a problem. Continue the review even if there is no formal specification.

### Sub-agents

Use sub-agents for review tasks when possible unless the current context is fresh.

### Working copy and checks

Review local changes in their existing working copy. For pull requests, use the current working copy if the branch containing the changes is already checked out. Otherwise, check out that branch in a separate working copy.

Run project checks only when reviewing local changes. For pull requests, use existing build, lint, type-check, and test results without rerunning them, even when the branch is checked out locally.

### Evidence

Verify findings against code and available evidence. State what is uncertain. Passing checks alone do not prove that the code is correct. For visual changes, use before/after evidence for affected platforms and a nearby unaffected case.

Exclude findings already caught by the project's formatter or linter. Report readability and convention issues those tools cannot detect.

### Handling feedback

Verify suggestions from the requester. Push back only when a suggestion is incorrect.

### Findings

Present issues and regressions first, then simplifications and code quality improvements. Order each group by importance. Number findings continuously across groups.

Highlight changes to public APIs. For each finding, include its exact file and line, the problem, its impact, and a blocking or optional label. Either group can contain blocking or optional findings.
