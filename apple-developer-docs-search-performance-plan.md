# Apple Developer Docs Search Performance Plan

## Context

This plan focuses on improving the normal Apple Developer Docs search path without considering archive-document search overhead.

The current command sends a remote Apple search request whenever the search text changes. Raycast throttling helps, but fast typing can still trigger unnecessary requests, loading-state churn, and list updates. The goal is to keep typing responsive while reducing low-value network work and visual movement.

## Goals

- Reduce unnecessary Apple Search API requests during typing.
- Avoid remote searches for very short, low-signal queries.
- Keep existing results visible while a newer request is loading.
- Reduce list rendering work for common searches.
- Preserve existing type filters and search history behavior where useful.

## Proposed Changes

1. Add a debounced query

   Introduce a `debouncedQuery` derived from `searchText`, with a delay around `300ms`. The search input should update immediately, while remote requests only use the debounced value.

2. Skip low-value short queries

   Do not call the Apple Search API for one-character queries. For empty search text, keep the existing default query behavior or show recent searches, depending on the desired product behavior.

3. Reduce returned result count

   Lower `config.maxResults` from `50` to `30`. Raycast users usually interact with the first screen of results, so this should reduce response parsing and list rendering without hurting normal usage.

4. Improve loading behavior

   Only show global list loading when there are no usable results yet. If previous results are available, keep them visible while the new request is in flight.

5. Limit search history noise

   Show the searched-history section only when the search text is empty or very short. For meaningful queries, prioritize live API results and avoid extra section movement.

6. Explicitly disable native filtering

   Set `enableFiltering={false}` on the `List`. The command already performs custom remote filtering, and making this explicit avoids future double-filtering regressions.

## Expected Impact

- API requests during fast typing should drop substantially, roughly `50%-80%` depending on typing speed.
- Rendered result volume should drop by about `40%` after lowering `maxResults` from `50` to `30`.
- The list should feel steadier because old results remain visible while new results load.
- Short queries should avoid slow, low-quality broad searches.

## Validation Plan

Run:

```bash
npm run lint
npm run build
```

Manual checks:

- Empty search opens cleanly.
- One-character search does not fire a remote request.
- Fast typing, such as `swiftui navigation`, does not cause visible result flicker.
- Results update after the debounce delay.
- Type filters still work for `all`, `general`, `documentation`, `sample_code`, and `video`.
- Search history is useful when the query is empty, but does not compete with live results for normal queries.
