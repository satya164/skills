---
name: code-quality-review
description: Review pull requests, local code changes, and existing code for correctness, regressions, API design, accessibility, and meaningful simplifications. Use when asked to review code, a diff, or an existing codebase.
---

# Code quality review

Review the code or changes against the criteria below.

## Requirements and behavior

1. **Requirements:** The approach fits the stated intent. The code implements all requested behavior and covers all intended cases without affecting unrelated cases. Edits stay within the requested scope. Bug fixes address the reported version and root cause.

2. **Edge cases:** Empty, missing, and invalid inputs produce the intended results. Values at, just below, and just above a limit produce the intended results. Loops and indexes do not skip items or go past the end.

3. **Error handling:** The code handles exceptions, rejected promises, failed responses, and rate limits with defined outcomes. Broad catches and silent fallbacks do not hide failures or invalid state, and do not report one kind of failure as another. Each retry, timeout, and other defensive measure addresses a specific risk, including in new code. Each wait on a peer, a process, or a user has an end condition. Each failure that users see has a message that matches its cause and tells users how to recover. Failures that need different user actions have different messages, unless the difference exposes protected information. Each message stays reachable for its cause after changes to control flow. Users see text written for them, not raw system or exception messages.

4. **State modeling:** The data model allows only valid states. A new state type is added only when it simplifies transitions or enforces a demonstrated rule. Each value has one source of truth. Related values derive from that source instead of separate copies that could disagree. State includes fields, local variables, flags, markers, and resource settings that together describe one thing that changes over time, such as a connection, a permission, or an edit mode. The code rejects operations in states where they are not valid. Each transition keeps related values consistent and includes its required follow-up, so no write can skip it.

5. **Mutation and concurrency:** Code does not mutate data it does not own, such as React props or shared Python defaults. Concurrent operations avoid races. Related writes cannot leave invalid partial state. Timing checks stay correct when work waits in a queue, responses arrive immediately, events overlap, or work completes in a different order.

6. **Cancellation:** Stale work and interrupted animations cannot leave state, focus, or callbacks inconsistent. Work started for one session, connection, or request does not apply its result after a new one replaces it.

7. **Compatibility:** Backward compatibility, including compatibility code and migrations, requires an explicit request. Documented support commitments alone do not count as a request.

8. **API design:** Public APIs describe what consumers can do. The API prevents contradictory states. Visibility, defaults, and extensibility fit the intended use. Existing style or native props take priority when they meet the requirements. Customization does not expose implementation details or allow invalid behavior. Defaults are not used for values that shouldn't have duplicates across instances such as `id`, `testID` etc. Subclasses meet the requirements of their parent classes.

9. **Counterparts:** Code that has a counterpart, such as the other side of a protocol, the same feature on another platform, or a parallel code path, follows the same contract as its counterpart. The contract covers accepted input, order of operations, limits, failure detection, and user feedback. Each side accepts all data that the other side can send. Differences come from requirements or platform constraints, and a comment states the reason. A change to one side includes the matching change to the other side.

## User interface

10. **Design specifications:** States, tokens, dimensions, icons, shapes, press effects, and disabled appearance match the design specification. Styles use theme defaults and named tokens. Spacing follows the design scale. Without a design specification, the interface follows the platform design guidelines for components, spacing, menus, and destructive actions. Custom values and components have a stated reason.

11. **Interface text:** Each concept has one name across screens, platforms, messages, and documentation. Equivalent messages in the same context use the same words. Text follows project and platform conventions for casing, punctuation, and terms. Status text shows a state, a button shows an action, and a hint gives a cause or a next step. Text is correct in every state where it appears. Surfaces that show the same state agree. A summary shows the overall state, and the details appear in one place. Instructions tell users where to find the UI they mention. Changed or removed text keeps the information that users need to act. User-facing text follows the project's localization approach and uses complete messages, not joined fragments.

12. **Platforms and layout:** Behavior works on each affected platform, including iOS, Android, and web. Scrolling and safe areas do not break positioning or interaction. Text and controls do not overlap or overflow at supported screen sizes.

13. **Accessibility:** Keyboard, mouse, and touch input work where applicable. Accessible labels, roles, focus order, and loading states fit the interaction. Disabled states suit the control's purpose and platform under the relevant guidelines. Disabled controls expose their state to assistive technology and prevent activation. A control that becomes unavailable after activation keeps focus. Closing temporary UI restores focus to an appropriate element.

14. **Animation and layout transitions:** Layout changes do not cause unnecessary shifts. Animations use only transform and opacity unless another property is necessary. Layout transitions use the First, Last, Invert, Play (FLIP) technique where relevant. Duration and easing fit the interaction and follow platform conventions. Dimensions are measured near animation start when this avoids extra state and gives current values.

## Component behavior

15. **Rendering:** Rendering derives values when possible instead of copying them into state through effects. Rendering does not run side effects or assign refs.

16. **Hook dependencies:** React Hooks follow the rules of hooks. Dependency arrays include the values hooks use. Dependencies do not include values recreated on each render. Assumed stability does not justify omitted dependencies, including consumer styles.

17. **Controlled components:** Controlled values have the callbacks needed to update them. Components support uncontrolled use when controlled rendering causes a specific performance or reliability problem. Components forward relevant native props and events without changing their meaning.

18. **Composition:** Composition does not depend on `React.Children`, element inspection, or `cloneElement` in React. Components still work when wrapped in other components. Composition uses context, component parts, render props, or data objects as appropriate.

19. **Platform code:** React Native platform branches use direct `Platform.OS` checks or `Platform.select`. React Native uses `.native` files for native/web splits.

20. **CSS features:** Code written for Web doesn't re-implement CSS features in JavaScript.

21. **Style overrides:** Consumer styles merge after defaults for supported overrides. Computed styles do not silently replace supplied values.

22. **Server rendering:** Components don't use browser-only APIs, layout measurements etc. during render. If they are necessary, their usage doesn't produce mismatches between server and client render.

## Type system

23. **Type design:** Types describe domain concepts. Related fields have consistent types. They require fields that must be present and reject values the code cannot handle. Generic arguments use inference where possible. Related types derive from one source. Custom types do not re-implement types that can be imported or derived from the standard library, dependencies.

24. **Type safety:** Type fixes correct source types instead of adding casts. The code does not use non-null assertions. TypeScript code does not use `@ts-ignore`. Each `@ts-expect-error` has a comment explaining why the code produces the error and why the error cannot be fixed.

25. **Type syntax:** Runtime enums require a specific reason to use them instead of unions. Interfaces require a specific reason to use them instead of type aliases. Overloads and complex type logic appear only when necessary. JSDoc does not repeat existing type annotations.

## Readability and documentation

26. **Project conventions:** Code, comments, and tests follow documented project conventions and architectural decisions. Code that claims to follow an external standard or convention matches the official guidance.

27. **Naming:** Variable, function, constant, type, and prop names match their purpose and use established project and platform terms. Numbers and strings that have meaning outside their own line have names. A constant derived from other values shows its parts.

28. **Comments:** Code with a reason or behavior that is not clear, such as a specific requirement, a workaround, a timing assumption, or a step that looks redundant, has a comment next to it that explains it. Comments appear only on such code. They are accurate and up to date. Comment rewrites correct inaccurate or misleading text only, follow existing language and style and don't change already correct text.

29. **Documentation content:** Documentation and help text explain purpose and current behavior in simple language. The text omits unnecessary counts and implementation details. Behavior changes include related documentation. Mentions of other APIs, components, or features link to the relevant content.

30. **Examples and structure:** Examples match the actual API, are self-contained, and focus on the relevant behavior. Migration examples show before/after code or a diff where useful. Heading levels follow the document structure. Links lead to the intended content.

## Structure and dependencies

31. **Module responsibilities:** Related logic stays near its data in the layer that owns it. Unrelated responsibilities stay separate. General utilities contain no feature-specific details. Dependencies are necessary and do not form cycles.

32. **Abstractions:** Abstractions simplify existing code and represent clear concepts on their own. Straightforward duplication is acceptable unless the copies must change together, such as keys, magic numbers, and rules (validity checks, preconditions, priority orders). Each of those has one definition where ownership and dependency boundaries allow it, and all call sites that depend on it use that definition.

33. **Complexity:** Branches, flags, nesting, and special cases are easy to follow. Refactoring adds indirection or spreads related logic only when that makes the code easier to follow.

34. **Unused code:** Changes contain no unused or unreachable code, obsolete features, or no-op normalization. Internal code does not check again conditions that types, callers, or earlier validation already guarantee. Checks at trust boundaries stay. A change to a constraint, such as a minimum version or a removed feature, also removes the code, dependencies, and workarounds that existed only for the old constraint.

35. **Package selection:** New packages provide enough benefit to justify their maintenance burden, license requirements, security risks, and added size. Existing or smaller alternatives take priority when they meet the requirements.

36. **Dependency updates:** Upgrades to direct and indirect dependencies preserve required behavior. The deprecation status of new or changed API usage is verified against the installed dependency version. Supported replacements take priority over deprecated APIs. Dependency manifests and lockfiles specify consistent versions. Install scripts follow project policy.

## Security

37. **External input:** External data passes validation before use. Queries use parameters. Rendered content uses the required encoding. Dynamic execution does not accept unsafe input. Uploads have size, type, and content restrictions.

38. **Authorization:** Protected operations check identity, ownership, roles, and required permissions. Authentication confirms the intended peer and role before protected operations. A message that is valid in one direction or phase cannot give authority in another. Peers cannot use unlimited connections, memory, or time before they authenticate.

39. **Sessions and account recovery:** Token validation covers signatures, expiry, and issuer. Sessions expire, and the system can revoke them. Account recovery verifies ownership. Recovery tokens expire and work only once.

40. **Sensitive data:** Source, logs, responses, and model context contain no secrets. Passwords, transmitted data, stored data, and backups have the required protection. Error messages do not expose internal details.

41. **Request security:** State-changing requests have protection against cross-site request forgery. The service permits only intended origins and redirect destinations. Outbound request validation prevents access to restricted networks. Changed endpoints have suitable security headers and request limits.

42. **File operations:** Destructive operations stay within authorized directories, including when paths contain symbolic links. Ownership checks use trusted evidence. Path changes between a check and an operation cannot redirect the operation.

## Performance and resources

43. **Work limits:** Loops, requests, allocations, and model calls have limits or pagination that fit real input. Independent work runs concurrently when sequential execution adds unnecessary delay. Blocking work does not prevent timely responses.

44. **Database queries:** Queries retrieve records together instead of making a separate query for each record. Indexes support filters and sorting without unnecessary write cost. Connection pools stay within capacity.

45. **Resource cleanup:** Resource cleanup releases connections, listeners, timers, and retained objects on success, failure, cancellation, and unmounting, as applicable. Repeated setup does not leave extra resources allocated. Resources and permissions stay valid until all work that depends on them finishes. Completion callbacks run on all final paths, including cancellation and shutdown.

46. **Caching:** Cache keys include all inputs and access context that affect a response. The cache behaves correctly during expiry, invalidation, eviction, concurrent misses, missing results, and pending writes. The cache provides enough benefit to justify its cost. Code uses cached state only while the state is known to be valid. Changes outside the cache owner update or invalidate the cache before decisions depend on it.

47. **Rendering performance:** Assets, fonts, bundles, and third-party scripts fit load budgets. Resources needed for the initial view take priority. Other resources load when needed. The code avoids unnecessary renders and forced layout work. Long lists and gestures stay responsive. Continuous gesture work runs off the JavaScript thread when needed.

48. **Memoization:** Memoization is limited to expensive calculations or values that must keep the same identity. Ordinary object creation or access alone does not justify it.

## Tests

49. **Coverage:** Tests cover requirements, boundaries, errors, regressions, meaningful integrations, and critical user paths. Tests assert public output and interaction instead of mock setup, private properties, context internals, or internal styles. Internal helpers have direct tests only when their complexity justifies them.

50. **Test quality:** Tests are readable, independent, and repeatable. Test names describe user-facing behavior. Test logic is simple. Additional test cases check behavior that existing tests do not cover.

51. **Mocking:** Mocks are only used for non-deterministic, external, slow dependencies or when the tests cannot run in the test environment.

52. **Querying:** Tests use accessibility attributes for queries when possible, and test IDs otherwise. Test IDs only identify public interaction or content and do not access internals.

## Review practices

### Scope and context

For pull requests, review the diff. For local changes, review staged, unstaged, and untracked changes. For existing code, review the requested feature and its counterparts across files. For a diff, also read the unchanged definitions, call sites, consumers, and counterparts of the values, rules, text, and messages that it changes. Report a problem in unchanged code only when the change causes it, depends on it, or makes it inconsistent. Label a problem that existed before the change as an existing problem.

Use available requirements, the pull request description, and project conventions to understand the intended behavior. Ask for clarification when unclear intent could change whether something is a problem. Continue the review even if there is no formal specification.

### Sub-agents

Use sub-agents for review tasks when possible unless the current context is fresh. Each sub-agent sees all files that its comparisons need, and counterparts stay with one sub-agent. Briefs describe the change and its intent but do not say that the change is correct. One reviewer checks how the delegated areas interact and resolves conflicting findings with evidence.

### Working copy and checks

Review local changes in their existing working copy. For pull requests, use the current working copy if the branch containing the changes is already checked out. Otherwise, check out that branch in a separate working copy.

Run the project's build, lint, type-check, and tests only when reviewing local changes. For pull requests, use the existing results without rerunning them, even when the branch is checked out locally.

### Evidence

Verify findings against code and available evidence. State what is uncertain. Passing checks alone do not prove that the code is correct. For visual changes, use before/after evidence for affected platforms and a nearby unaffected case.

For criteria that compare items across files, list the items and compare them together. These items include user-visible text and the checks of each condition. For state, trace entry paths, writes, transitions, and consumers. For errors, trace each cause to its message and recovery.

A finding about platform, framework, or library behavior cites documentation, source code, or a test for the supported versions. Otherwise, the finding states that the behavior is assumed. A finding that asks for extraction or a new type shows the repetition or the contradictory state that the change removes. A finding about a redundant check or unused code shows that the guarantee it relies on holds on all entry paths up to the point of use, and names what the removed code protected. Claims that code is unused account for use outside the repository and for build and configuration effects.

Exclude findings already caught by the project's formatter or linter. Report readability and convention issues those tools cannot detect.

### Handling feedback

Verify suggestions from the requester against the code, requirements, and prior decisions. Explain tradeoffs in correctness, scope, and maintenance.

Prior decisions and accepted tradeoffs are part of the review context. Do not report a rejected suggestion or an accepted risk again unless there is new evidence or a changed assumption.

### Findings

Present issues and regressions first, then simplifications and code quality improvements. Order each group by importance. Number findings continuously across groups.

Highlight changes to public APIs. For each finding, include its exact file and line, the problem, the conditions that cause it, its impact, the size of the fix, and a blocking or optional label.

Missed simplifications and meaningful structural regressions are blocking. Line counts and file size are not blocking. Wording findings are blocking when they break an explicit requirement, mislead users, contradict another surface, or remove information that users need. Other wording findings are optional.
