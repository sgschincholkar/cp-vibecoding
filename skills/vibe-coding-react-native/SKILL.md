---
name: vibe-coding-react-native
description: |
  React Native and Expo best practices — list performance, animations, navigation, UI patterns, state management, rendering, monorepo setup, and configuration.
  Use when: (1) vibe-coding-code-review detects React Native or Expo project and delegates here,
  (2) User says "review my React Native code", "check my Expo app", "optimize RN performance",
  (3) User says "fix scroll performance", "optimize animations", "review navigation setup".
  Returns findings with severity ratings and exact file:line references.
  Source: vercel-labs/agent-skills react-native-skills.
---

# Vibe Coding — React Native & Expo Review

Called by `vibe-coding-code-review` when React Native/Expo detected.

## Entry Router

```
Verify React Native project first:
  Check package.json for "react-native" or "expo" — if NOT found → stop: "Not a React Native project. Use vibe-coding-code-review or vibe-coding-review-react instead."
  If .kt/.swift/.dart files detected without package.json → stop: "This is a native/Flutter project. This skill is for React Native/Expo only."

Run in priority order. Report CRITICAL issues immediately.

Cat 1 (CRITICAL): List Performance → READ: ## Cat 1
Cat 2 (HIGH): Animations → READ: ## Cat 2
Cat 3 (HIGH): Navigation → READ: ## Cat 3
Cat 4 (HIGH): UI Patterns → READ: ## Cat 4
Cat 5 (MEDIUM): State Management → READ: ## Cat 5
Cat 6 (MEDIUM): Rendering → READ: ## Cat 6
Cat 7 (MEDIUM): Monorepo Setup → READ: ## Cat 7 (only if monorepo)
Cat 8 (LOW): Configuration → READ: ## Cat 8
```

**Jump directly to each ## Cat N section in order. Do not read ahead to later categories before completing the current one. Skip ## Cat 7 entirely if not a monorepo.**

---

## Cat 1: List Performance (CRITICAL)

FAIL if: `ScrollView` for lists with >20 items, missing `keyExtractor`, `renderItem` not in `useCallback`, `getItemLayout` missing on fixed-height lists, `initialNumToRender` too high.

```jsx
// FAIL: <ScrollView>{items.map(...)}</ScrollView>
// PASS: <FlashList data={items} renderItem={...} estimatedItemSize={80} keyExtractor={i => i.id} />
// Also: <FlatList removeClippedSubviews={true} windowSize={5} />
```

Recommend FlashList over FlatList: `npm install @shopify/flash-list`

---

## Cat 2: Animations (HIGH)

FAIL if: `Animated.Value`/`Animated.View` used (JS thread), `useNativeDriver: true` missing on legacy Animated, heavy calculations in animation callbacks.

```jsx
// PASS (Reanimated 2+):
import Animated, { useSharedValue, useAnimatedStyle, withTiming } from 'react-native-reanimated'
const opacity = useSharedValue(0)
const animStyle = useAnimatedStyle(() => ({ opacity: opacity.value }))
opacity.value = withTiming(1, { duration: 300 })
<Animated.View style={[styles.box, animStyle]} />
```

---

## Cat 3: Navigation (HIGH)

FAIL if: non-standard navigation library, deep linking not configured, navigation params pass full objects (pass ID only, fetch in destination), lazy loading missing on heavy tabs.

Recommended: Expo Router (file-based, new projects) or React Navigation v6+.

```tsx
// FAIL: navigation.navigate('Detail', { user: fullObject })
// PASS: navigation.navigate('Detail', { userId: user.id })
```

---

## Cat 4: UI Patterns (HIGH)

FAIL if: `TouchableOpacity` instead of `Pressable`, hardcoded colors (use theme constants), missing `Platform.select` for OS-specific shadows, `<Image>` from react-native (use `expo-image`), mixed Nativewind `className` + `StyleSheet`.

```jsx
// PASS: <Pressable style={({ pressed }) => pressed && styles.pressed} />
// PASS: import { Image } from 'expo-image'  (blurhash, caching, better perf)
```

---

## Cat 5: State Management (MEDIUM)

FAIL if: Redux for local UI state (use useState/Zustand), Context re-rendering whole tree (split by update frequency), persistent state missing AsyncStorage/MMKV, frequent reads/writes not using MMKV (10x faster than AsyncStorage).

---

## Cat 6: Rendering (MEDIUM)

FAIL if: list item components missing `React.memo`, inline style objects in components (new object every render — use `StyleSheet.create`), heavy work not deferred via `InteractionManager.runAfterInteractions`, Hermes not enabled.

```jsx
// FAIL: <View style={{ flex: 1, padding: 16 }}>
// PASS: <View style={styles.container}>  (StyleSheet.create outside component)
```

---

## Cat 7: Monorepo Setup (MEDIUM)

Only check if project is a monorepo.

FAIL if: `metro.config.js` missing `watchFolders` for monorepo packages, TypeScript paths not configured in both `tsconfig.json` and `babel.config.js`, native modules not rebuilt after config changes (`npx expo prebuild --clean`).

---

## Cat 8: Configuration (LOW)

Check `app.json` / `app.config.js` for: missing splash screen, missing icons (iOS + Android + web), no `eas.json` if building for distribution, missing runtime permissions (camera, location, notifications), `expo-updates` not configured.

**New Architecture check (Expo SDK 51+):**
- Read `app.json` — if Expo SDK version is 51 or higher:
  - WARN if `expo.newArchEnabled` is `false` (New Architecture is default-on in SDK 52+, opt-in in SDK 51)
  - FAIL if using packages known to be incompatible with New Architecture (check against package.json)
  - PASS if `"newArchEnabled": true` or SDK >= 52 (enabled by default)
- For SDK < 51: skip New Architecture check entirely.

---

## Output Format

```
REACT NATIVE CODE REVIEW
=========================
File: [path]
Platform: [React Native | Expo]

CRITICAL FINDINGS:
[1] [file:line] — [description]
    Found: [code]
    Fix: [change]
    Impact: [effect]

HIGH / MEDIUM / LOW FINDINGS: [same pattern]

SUMMARY: [N] critical, [N] high, [N] medium, [N] low
```
