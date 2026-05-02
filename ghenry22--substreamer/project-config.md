---
trigger: always_on
description: React Native component patterns, styling, and UI conventions for Substreamer
---


# Component Patterns

## Structure

- All components are **functional** – no class components.
- Use **`memo()`** for list-rendered items (cards, rows) and frequently re-rendered components.
- Use **named exports** for components. Wrap memo'd components with a named function:

```tsx
export const AlbumCard = memo(function AlbumCard({ album, width }: { album: AlbumID3; width: number }) {
  const { colors } = useTheme();
  // ...
});
```

## Props Typing

Most memo'd components use **inline props** directly in the function signature:

```tsx
export const AlbumRow = memo(function AlbumRow({ album }: { album: AlbumID3 }) {
```

Use a named `ComponentNameProps` interface when the props type is complex, shared, or referenced elsewhere:

```tsx
interface AlphabetScrollerProps {
  letters: string[];
  onLetterChange: (letter: string) => void;
  listRef: RefObject<FlashListRef<unknown>>;
  sectionMap: Map<string, number>;
}

export const AlphabetScroller = memo(function AlphabetScroller(props: AlphabetScrollerProps) {
```

## Entity Component Pairs

Each entity (Album, Artist, Song, Playlist) follows a Card + Row + ListView pattern:

- **Card** (`AlbumCard`) – grid display with cover art and title
- **Row** (`AlbumRow`) – list display with thumbnail, title, and metadata
- **ListView** (`AlbumListView`) – FlashList wrapper supporting list/grid toggle, pull-to-refresh, empty states, and alphabet scrolling

## Styling

- Use `StyleSheet.create()` at module scope for static styles.
- Apply theme colors inline via `useTheme()`:

```tsx
<Text style={[styles.title, { color: colors.textPrimary }]}>{title}</Text>
```

- Use `useMemo` for dynamic `StyleSheet.create` when many styles depend on theme colors.
- Use `Pressable` with function styles for pressed states:

```tsx
<Pressable style={({ pressed }) => [styles.row, { backgroundColor: colors.card }, pressed && styles.pressed]}>
```

## Cover Art

Always use `CachedImage` for Subsonic cover art – never raw `<Image>`:

```tsx
<CachedImage coverArtId={album.coverArt} size={300} style={styles.cover} resizeMode="cover" />
```

Standard sizes: 50 (thumbnails), 150 (small), 300 (cards/lists), 600 (hero/detail).

## FlashList Performance

Uses `@shopify/flash-list` v2 (`FlashList`) instead of React Native's `FlatList`. FlashList v2 handles view recycling, batching, and size estimation automatically.

- Use `keyExtractor={(item) => item.id}`.
- Memoize `renderItem` with `useCallback`.
- Do **not** pass `estimatedItemSize` – FlashList v2 removed this prop and handles size estimation automatically.
- Do **not** pass `windowSize`, `maxToRenderPerBatch`, `initialNumToRender`, `removeClippedSubviews`, or `getItemLayout` – these are FlatList-only concepts that do not exist in FlashList's architecture.
- Use `drawDistance` (pixels) to control off-screen rendering distance when needed (default 250px is usually sufficient; use 300 for lists with alphabet scrollers).
- Grid: use `numColumns={2}`. Handle inter-column gaps via padding on individual grid items (FlashList does not support `columnWrapperStyle`):

```tsx
const renderGridItem = ({ item, index }: { item: AlbumID3; index: number }) => {
  const isLeftColumn = index % GRID_COLUMNS === 0;
  return (
    <View style={{
      flex: 1,
      paddingLeft: isLeftColumn ? 0 : GRID_GAP / 2,
      paddingRight: isLeftColumn ? GRID_GAP / 2 : 0,
    }}>
      <AlbumCard album={item} width={cardWidth} />
    </View>
  );
};
```

- Ref type: `useRef<FlashListRef<T>>(null)` (import `FlashListRef` from `@shopify/flash-list`).

**Exception:** Two screens use `ReorderableList` from `react-native-reorderable-list` instead of FlashList because they require drag-to-reorder functionality:

- `src/screens/download-queue.tsx` – the download queue is inherently small (typically under 20 items) and does not need FlashList's virtualization.
- `src/screens/playlist-detail.tsx` – conditionally renders `ReorderableList` when the user enters edit mode to reorder tracks.

`ReorderableList` is built natively for the New Architecture on Reanimated worklets. Drag is initiated via the `useReorderableDrag()` hook called inside the row component (not threaded as a `drag` prop) and wired to a dedicated drag-handle `Pressable` so the rest of the row body still scrolls normally. Reorder events fire as `{ from, to }` indices; use the exported `reorderItems(data, from, to)` helper to apply the move to local state.

**Exception:** Bounded, static, single-digit-item horizontal carousels may use RN `FlatList` with `getItemLayout`. Example: `src/components/OnboardingGuide.tsx` (4-slide onboarding carousel). FlashList's virtualization adds zero value below ~20 items, and its recycling doesn't pair cleanly with horizontal paging on a fixed list. Always document the carveout with an inline comment.

## Modals and Bottom Sheets

Use RN `Modal` with transparent backdrop for bottom sheets:

```tsx
<Modal visible={visible} transparent animationType="slide" onRequestClose={onClose}>
  <Pressable style={styles.backdrop} onPress={onClose} />
  <View style={[styles.sheet, { backgroundColor: colors.card, paddingBottom: Math.max(insets.bottom, 16) }]}>
    <View style={[styles.handle, { backgroundColor: colors.border }]} />
    {/* content */}
  </View>
</Modal>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghenry22/substreamer](https://github.com/ghenry22/substreamer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
