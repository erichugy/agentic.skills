# UI & Frontend Review Checklist

Applies to React components, CSS, and frontend changes.

## Event Listeners & Cleanup
- [ ] All `document.addEventListener` calls have matching `removeEventListener` in cleanup
- [ ] Drag handlers (`mousedown`→`mousemove`→`mouseup`) clean up on unmount via ref + useEffect
- [ ] Outside-click listeners for dropdowns/menus have cleanup in `useEffect` return
- [ ] Cursor/userSelect overrides on `document.body` are restored in ALL exit paths (mouseup AND unmount)

## Dropdowns & Menus
- [ ] Dropdowns close on outside click
- [ ] Dropdown positioning handles viewport overflow (use `getBoundingClientRect()` to detect, flip left/right)
- [ ] Sub-menus don't overflow viewport edges
- [ ] Nested menus don't interfere with parent menus

## State Wiring
- [ ] Features are fully wired — if you can create a group, you can add items to it
- [ ] IDs are passed through to handlers (e.g., `addingToGroupId` is actually set)
- [ ] Return values from state-creating functions are used (don't discard new IDs)
- [ ] Toggle/click handlers are connected to the right state
- [ ] UI labels and toggle descriptions warn about non-obvious side effects (e.g., disabling cloud files also strips KB source payloads)

## localStorage
- [ ] All reads wrapped in try/catch (private browsing can throw)
- [ ] Parsed data validated before use — deep validation, not just top-level fields
- [ ] Missing/corrupt values fall back to defaults gracefully
- [ ] Serialized data includes all required fields (don't forget new fields added to types)

## Accessibility
- [ ] `aria-expanded` on collapsible/toggle elements
- [ ] `type="button"` on all non-submit buttons
- [ ] `aria-label` on icon-only buttons
- [ ] Keyboard navigable (Enter/Escape on menus)

## CSS Consistency
- [ ] All new CSS classes follow the component's prefix convention (e.g., `rb-`)
- [ ] Active states use consistent color pattern across all toggles/tabs/buttons
- [ ] Transitions/animations are smooth and consistent duration
- [ ] No `!important` unless absolutely necessary

## React Patterns
- [ ] `useCallback` dependencies are complete
- [ ] `useMemo` dependencies are complete
- [ ] `useEffect` cleanup returns handle all side effects
- [ ] No state updates in render (causes infinite loops)
- [ ] Keys are stable and unique (not array indices when items can reorder)
- [ ] No `use`-prefixed variable names for non-hooks (e.g., `useSelectiveExport` for a plain boolean — use `isSelectiveExportEnabled` instead)
- [ ] Symmetric state management — if one selection set has a pruning effect, parallel selection sets should too
