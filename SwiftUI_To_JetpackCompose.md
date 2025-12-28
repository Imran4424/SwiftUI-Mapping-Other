# SwiftUI ↔ Jetpack Compose “A→Z” Mapping Cheat Sheet 

This is a practical mapping reference for people fluent in SwiftUI who are learning Jetpack Compose.
- SwiftUI: `View` + modifiers
- Compose: `@Composable` functions + `Modifier`

> Notes:
> - Compose examples assume Material3 (`androidx.compose.material3.*`) unless stated.
> - Many “SwiftUI 1-liner” features in iOS map to “patterns” (not a single API) in Android/Compose.

---

## Table of Contents
1. Core Mental Model & Syntax
2. Layout & Containers
3. Scroll, Lists, Grids
4. Text, Images, Icons
5. Buttons, Inputs, Selection
6. Modifiers & Styling
7. State & Data Flow
8. Side Effects & Lifecycle Hooks
9. Navigation & Presentation
10. Gestures, Focus, Keyboard
11. Animation
12. Theming & Design System
13. Accessibility & Semantics
14. Safe Area & Window Insets
15. Canvas / Drawing / Custom Rendering
16. Testing (Unit + UI)
17. Performance & Recomposition Rules
18. Concurrency & Async (Swift Concurrency ↔ Coroutines)
19. Reactive Streams (Combine ↔ Flow)
20. Dependency Injection & Environment
21. Persistence (SwiftData/CoreData ↔ Room/DataStore)
22. Networking (URLSession ↔ Retrofit/Ktor)
23. Background Work (BGTask ↔ WorkManager)
24. Interop (UIKit/View ↔ Android View)
25. Tooling: Previews, Debugging, Logging
26. Architecture Patterns (MVVM, UDF)
27. Quick “Anchor” Snippets

---

## 1) Core Mental Model & Syntax

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `struct MyView: View { var body: some View { ... } }` | `@Composable fun MyUI(...) { ... }` | Compose uses functions instead of structs: `@Composable fun MyUI() { Text("Hi") }` |
| `ViewBuilder` | Composable lambdas | `content: @Composable () -> Unit` is common in APIs like `Scaffold { ... }` |
| `body` recomputes | recomposition | Compose recomposes based on state reads; keep composables pure + stable inputs |
| `some View` type erasure | no direct equivalent | Compose prefers typed composables; pass `@Composable () -> Unit` if needed |
| `AnyView` | avoid / use lambdas | Prefer: `val slot: @Composable () -> Unit = { ... }` |

---

## 2) Layout & Containers

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `VStack` | `Column` | `Column { Text("A"); Text("B") }` |
| `HStack` | `Row` | `Row { Text("A"); Text("B") }` |
| `ZStack` | `Box` | `Box { Image(...); Text("Overlay") }` |
| `Spacer()` | `Spacer` | Flexible: `Spacer(Modifier.weight(1f))` (in Row/Column) |
| `Divider()` | `Divider()` | `Divider()` |
| `alignment:` | `horizontalAlignment` / `verticalAlignment` / `contentAlignment` | `Column(horizontalAlignment = Alignment.CenterHorizontally)`; `Box(contentAlignment = Alignment.Center)` |
| `spacing:` | `Arrangement.spacedBy(...)` | `Row(horizontalArrangement = Arrangement.spacedBy(8.dp))` |
| `.frame(width:,height:)` | `Modifier.size(...)` | `Modifier.size(48.dp)` |
| `.frame(maxWidth: .infinity)` | `fillMaxWidth()` | `Modifier.fillMaxWidth()` |
| `.frame(maxHeight: .infinity)` | `fillMaxHeight()` | `Modifier.fillMaxHeight()` |
| `.layoutPriority(1)` | weights | `Modifier.weight(1f)` (Row/Column children) |
| `GeometryReader` | `BoxWithConstraints` | `BoxWithConstraints { val w = maxWidth }` |
| `alignmentGuide` | custom layout | Compose: custom `Layout { ... }` or alignment via modifiers and arrangement |

---

## 3) Scroll, Lists, Grids

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `ScrollView(.vertical)` | `Column(verticalScroll(...))` | `val s=rememberScrollState(); Column(Modifier.verticalScroll(s)) { ... }` |
| `ScrollView(.horizontal)` | `Row(horizontalScroll(...))` | `val s=rememberScrollState(); Row(Modifier.horizontalScroll(s)) { ... }` |
| `List` | `LazyColumn` | `LazyColumn { items(data) { ItemRow(it) } }` |
| `ForEach(data)` | `items(data)` | `items(data, key={it.id}) { ... }` |
| `LazyVStack` | `LazyColumn` | Virtualized vertical list |
| `LazyHStack` | `LazyRow` | Virtualized horizontal list |
| `Section(header:)` | `stickyHeader` | `LazyColumn { stickyHeader { Text("Header") }; items(...) {...} }` |
| `Grid` / `LazyVGrid` | `LazyVerticalGrid` | `LazyVerticalGrid(columns = GridCells.Fixed(2)) { items(data) { ... } }` |
| `ScrollViewReader` | `LazyListState` | `val st=rememberLazyListState(); LaunchedEffect(...) { st.animateScrollToItem(i) }` |
| `.refreshable {}` | pull-to-refresh pattern | Material3 has pull-to-refresh APIs (or libs). Pattern: `rememberPullToRefreshState()` + container |
| `.listStyle(...)` | Compose styling/theme | Lists are built from primitives; style via Material components + modifiers |

---

## 4) Text, Images, Icons

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `Text("Hi")` | `Text("Hi")` | `Text("Hi")` |
| `.font(.title)` | `style = MaterialTheme.typography.*` | `Text("T", style = MaterialTheme.typography.titleLarge)` |
| `.fontWeight(.bold)` | `fontWeight = FontWeight.Bold` | `Text("T", fontWeight = FontWeight.Bold)` |
| `.foregroundColor(.red)` | `color = Color.Red` | `Text("T", color = Color.Red)` |
| `.lineLimit(2)` | `maxLines = 2` | `Text("...", maxLines=2, overflow=TextOverflow.Ellipsis)` |
| `.multilineTextAlignment(.center)` | `textAlign = TextAlign.Center` | `Text("...", textAlign=TextAlign.Center)` |
| `Image("asset")` | `Image(painterResource(...))` | `Image(painterResource(R.drawable.pic), contentDescription=null)` |
| `Image(systemName:)` | `Icon(Icons.*)` | `Icon(Icons.Default.Home, contentDescription=null)` |
| `AsyncImage(url:)` | Coil `AsyncImage` | `coil.compose.AsyncImage(model=url, contentDescription=null)` |
| `.resizable()` | sizing + `contentScale` | `Image(..., contentScale=ContentScale.Crop, modifier=Modifier.size(80.dp))` |
| `.aspectRatio(...)` | `Modifier.aspectRatio(...)` | `Modifier.aspectRatio(16f/9f)` |

---

## 5) Buttons, Inputs, Selection

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `Button("Tap") {}` | `Button(onClick={}) {}` | `Button(onClick={}) { Text("Tap") }` |
| `Button(role:.destructive)` | `Button` + colors | Usually `ButtonColors` via theme or `FilledTonalButton` / `OutlinedButton` |
| `TextField("Hint", text:)` | `TextField(value, onValueChange)` | `var t by remember{mutableStateOf("")}; TextField(t,{t=it}, label={Text("Hint")})` |
| `SecureField` | password transform | `TextField(..., visualTransformation = PasswordVisualTransformation())` |
| `TextEditor` | multiline `TextField` | `TextField(value, onValueChange, minLines=5)` |
| `Toggle(isOn:)` | `Switch` | `Switch(checked=on, onCheckedChange={on=it})` |
| `Picker` | dropdown menus | `ExposedDropdownMenuBox { ... DropdownMenu { ... } }` |
| `DatePicker` | Material pickers | `DatePickerDialog(...)` (Material3) |
| `Stepper` | custom controls | `Row { IconButton(-); Text("$n"); IconButton(+) }` |
| `Slider(value:)` | `Slider` | `Slider(value=v, onValueChange={v=it})` |
| `ProgressView()` | `CircularProgressIndicator()` | `CircularProgressIndicator()` |
| `ProgressView(value:)` | `LinearProgressIndicator(progress)` | `LinearProgressIndicator(progress = 0.6f)` |
| `Alert` | `AlertDialog` | `AlertDialog(onDismissRequest={}, confirmButton={...}, text={...})` |
| `ActionSheet` | `ModalBottomSheet` | `if(show) ModalBottomSheet(onDismissRequest={show=false}) { ... }` |
| `ContextMenu` | long-press menu | Often `DropdownMenu(expanded=...)` triggered by `combinedClickable` |

---

## 6) Modifiers & Styling (SwiftUI modifiers ↔ Compose Modifier)

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `.padding(16)` | `Modifier.padding(16.dp)` | `Modifier.padding(16.dp)` |
| `.frame(width:, height:)` | `Modifier.size(...)` | `Modifier.size(80.dp)` |
| `.frame(minWidth:, maxWidth:)` | `Modifier.widthIn(...)` | `Modifier.widthIn(min=80.dp, max=200.dp)` |
| `.background(Color.red)` | `Modifier.background(Color.Red)` | Often with shape: `background(Color.Red, RoundedCornerShape(12.dp))` |
| `.cornerRadius(12)` | `clip(RoundedCornerShape)` | `Modifier.clip(RoundedCornerShape(12.dp))` |
| `.border(width:, color:)` | `Modifier.border(...)` | `Modifier.border(1.dp, Color.Gray, RoundedCornerShape(12.dp))` |
| `.shadow(radius:)` | `Modifier.shadow(...)` | `Modifier.shadow(8.dp, RoundedCornerShape(12.dp))` |
| `.opacity(0.5)` | `Modifier.alpha(0.5f)` | `Modifier.alpha(0.5f)` |
| `.offset(x:y:)` | `Modifier.offset(x.dp,y.dp)` | `Modifier.offset(10.dp, 4.dp)` |
| `.scaleEffect(...)` | `Modifier.scale(...)` | `Modifier.scale(1.05f)` |
| `.rotationEffect(...)` | `Modifier.rotate(...)` | `Modifier.rotate(15f)` |
| `.blur(radius:)` | `Modifier.blur(...)` | `Modifier.blur(16.dp)` |
| `.clipShape(Circle())` | `Modifier.clip(CircleShape)` | `Modifier.clip(CircleShape)` |
| `.overlay(...)` | `Box` layering / draw | `Box { Base(); Overlay() }` |
| `.onTapGesture {}` | `Modifier.clickable {}` | `Modifier.clickable { ... }` |
| `.disabled(true)` | `enabled=false` | `Button(enabled=false, onClick={}) { ... }` |
| `.contentShape(...)` | pointer input hit area | Compose: `Modifier.pointerInput` / `Modifier.clickable` + `indication` tweaks |
| `.animation(..., value:)` | `animate*AsState` | tie animation to state: `val a by animateFloatAsState(target)` |

**Compose modifier order matters** (often more than SwiftUI). Example: `clip(...).background(...)` vs `background(...).clip(...)` yields different results.

---

## 7) State & Data Flow

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `@State var x` | `remember { mutableStateOf }` | `var x by remember { mutableStateOf(0) }` |
| `@Binding` | state hoisting | Pass `value` + `onValueChange`. `MyUI(value=x, onChange={x=it})` |
| `@StateObject` | `viewModel()` | `val vm: MyVM = viewModel()` (or `hiltViewModel()`) |
| `@ObservedObject` | `collectAsState()` / `observeAsState()` | `val s by vm.stateFlow.collectAsState()` |
| `@EnvironmentObject` | DI / CompositionLocal | Hilt DI, or `CompositionLocalProvider(LocalX provides x)` |
| `@AppStorage` | `DataStore` | Preferences DataStore -> `Flow` -> `collectAsState()` |
| `@SceneStorage` | `rememberSaveable` | `var t by rememberSaveable { mutableStateOf("") }` |
| `UserDefaults` | `DataStore` / `SharedPreferences` | Prefer DataStore for modern apps |
| `ObservableObject + @Published` | `StateFlow` / `MutableStateFlow` | Common Compose state container pattern |
| `@Published` stream | `Flow` | Compose integrates via `collectAsState()` |

---

## 8) Side Effects & Lifecycle Hooks

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `.onAppear {}` | `LaunchedEffect(Unit)` | `LaunchedEffect(Unit) { /* run once */ }` |
| `.onDisappear {}` | `DisposableEffect(Unit)` | `DisposableEffect(Unit) { onDispose { /* cleanup */ } }` |
| `.task { await ... }` | `LaunchedEffect(key)` | Runs coroutine when key changes |
| `.onChange(of:)` | `LaunchedEffect(key)` / derived state | `LaunchedEffect(value) { /* react */ }` |
| `NotificationCenter` observers | `DisposableEffect` + callbacks | Register/unregister in `DisposableEffect` |
| timers | coroutine loop | `LaunchedEffect(Unit) { while(true){ delay(1000); ... } }` |

---

## 9) Navigation & Presentation

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `NavigationStack {}` | `NavHost` | `NavHost(nav, "home") { composable("home"){...} }` |
| `NavigationLink(...)` | `navController.navigate("route")` | Trigger by click/button |
| `.navigationTitle("T")` | `TopAppBar` title | `Scaffold(topBar={ TopAppBar(title={Text("T")}) }) { ... }` |
| `.toolbar {}` | `TopAppBar` / `Scaffold` | Compose uses `Scaffold` slots |
| `TabView` | `NavigationBar` / `NavigationRail` | Bottom tabs often built with `Scaffold(bottomBar={ ... })` |
| `.sheet` | `ModalBottomSheet` / `Dialog` | `if(show) ModalBottomSheet(...) { ... }` |
| `.fullScreenCover` | navigation route | Typically just navigate to a full-screen destination |
| `Alert` | `AlertDialog` | Built-in dialog composable |

---

## 10) Gestures, Focus, Keyboard

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `.onTapGesture` | `Modifier.clickable` | `Modifier.clickable { ... }` |
| `LongPressGesture` | `combinedClickable` | `Modifier.combinedClickable(onLongClick={...}, onClick={...})` |
| `DragGesture` | `pointerInput { detectDragGestures }` | `Modifier.pointerInput(Unit){ detectDragGestures { _, drag -> ... } }` |
| `MagnificationGesture` | transformable | `Modifier.transformable(state)` + `rememberTransformableState` |
| `@FocusState` | `FocusRequester` | `val fr=remember{FocusRequester()}; Modifier.focusRequester(fr)` |
| dismiss keyboard | focus manager | `LocalFocusManager.current.clearFocus()` |
| keyboard avoid (safe area) | `imePadding()` | `Modifier.imePadding()` |

---

## 11) Animation

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `withAnimation {}` | `animate*AsState` | `val alpha by animateFloatAsState(targetValue=...)` |
| `.animation(..., value:)` | state-driven animation | Use `animate*AsState` or `updateTransition` |
| `.transition(.opacity)` | `AnimatedVisibility` | `AnimatedVisibility(visible=show) { ... }` |
| `matchedGeometryEffect` | shared element (pattern) | Usually via navigation + animation libs; not a simple core 1-liner |
| `Animation.spring()` | `spring()` | `animateDpAsState(target, animationSpec=spring())` |

---

## 12) Theming & Design System

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `Color` | `Color` | Compose uses `androidx.compose.ui.graphics.Color` |
| `Font` | `TextStyle` / `Typography` | Prefer `MaterialTheme.typography.*` |
| `accentColor` | `MaterialTheme.colorScheme.primary` | Theme-driven |
| `@Environment(\.colorScheme)` | `isSystemInDarkTheme()` | `val dark = isSystemInDarkTheme()` |
| `.tint(...)` | colors via theme or params | Buttons/Icons/Indicators often use theme colors |
| `EnvironmentValues` | `CompositionLocal` | `LocalContext`, `LocalDensity`, custom locals |

---

## 13) Accessibility & Semantics (Very Important)

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `.accessibilityLabel("...")` | `semantics { contentDescription = "..." }` | `Modifier.semantics { contentDescription = "Play" }` |
| `.accessibilityHint("...")` | `semantics { }` + state | Compose semantics can include role/state; hints are usually conveyed by content + role |
| `.accessibilityValue(...)` | semantics state | `Modifier.semantics { stateDescription = "On" }` |
| `.accessibilityHidden(true)` | `clearAndSetSemantics {}` | `Modifier.clearAndSetSemantics { }` or `invisibleToUser()` patterns |
| `.accessibilityAddTraits(.isButton)` | `role = Role.Button` | `Modifier.semantics { role = Role.Button }` |
| dynamic type | `sp` text sizes + Material | Compose uses `sp`; honor system font scaling automatically |

---

## 14) Safe Area & Window Insets

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `.ignoresSafeArea()` | window inset handling | Common: rely on `Scaffold` defaults or use `WindowInsets` APIs |
| `.safeAreaInset(edge:)` | `WindowInsets` padding | Use: `Modifier.windowInsetsPadding(WindowInsets.systemBars)` |
| keyboard safe area | IME insets | `Modifier.imePadding()` or `windowInsetsPadding(WindowInsets.ime)` |
| notch/status bar | system bars insets | `WindowInsets.statusBars` / `systemBars` + padding helpers |
| navigation bar | system bars | Often `Scaffold` manages basic padding; customize when doing edge-to-edge |

---

## 15) Canvas / Drawing / Custom Rendering

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `Canvas { context, size in ... }` | `Canvas(modifier) { ... }` | `Canvas(Modifier.size(100.dp)) { drawCircle(...) }` |
| `Path` | `Path` | Compose has `androidx.compose.ui.graphics.Path` |
| `Shape` | `Shape` / `DrawModifier` | Custom shape via `Shape` interface or drawing in `drawBehind` |
| `.drawingGroup()` | layer caching (pattern) | Use `graphicsLayer` / caching strategies; measure performance |
| gradients | `Brush.linearGradient(...)` | `Modifier.background(Brush.linearGradient(...))` |

Example (Compose):
- `Modifier.drawBehind { drawRoundRect(...) }`
- `Canvas { drawLine(...); drawPath(...); drawArc(...) }`

---

## 16) Testing (Unit + UI)

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| Xcode UI Tests | Compose UI test rule | `createComposeRule()` + `onNodeWithText("...").performClick()` |
| `accessibilityIdentifier` | `testTag` | `Modifier.testTag("login_button")` then `onNodeWithTag("login_button")` |
| snapshot tests | screenshot tests (pattern) | Use Android screenshot testing tools; Compose has ecosystem options |
| view inspection | Semantics tree | Compose tests query semantics tree |

---

## 17) Performance & Recomposition Rules (Key differences vs SwiftUI)

| SwiftUI concept | Compose equivalent | Notes |
|---|---|---|
| “View updates” | recomposition + state reads | Compose tracks state reads and recomposes affected nodes |
| identity in lists | stable keys | Always use keys for `Lazy*` when items can move: `items(data, key={it.id})` |
| expensive work in body | `remember` caching | `val x = remember(input) { expensive(input) }` |
| computed properties | `derivedStateOf` | `val filtered by remember { derivedStateOf { ... } }` |
| minimizing diff work | stable params | Prefer immutable/stable data classes; avoid recreating lambdas/objects per frame |
| “equatable view” pattern | stability + `remember` | Compose optimizes when inputs are stable; also use `@Stable` / `@Immutable` (advanced) |
| animations cost | prefer targeted anim APIs | Use `animate*AsState` rather than re-animating entire trees |

Common Compose performance tools/patterns:
- `rememberLazyListState()`
- `remember` / `rememberSaveable`
- `key(...) { ... }`
- `derivedStateOf`
- avoid passing unstable collections recreated each recomposition

---

## 18) Concurrency & Async (Swift Concurrency ↔ Coroutines)

| SwiftUI / Swift | Jetpack Compose / Kotlin | Notes |
|---|---|---|
| `async/await` | `suspend` + coroutines | `suspend fun fetch()` + `withContext(Dispatchers.IO)` |
| `Task { ... }` | `LaunchedEffect { ... }` / `rememberCoroutineScope()` | `val scope=rememberCoroutineScope(); scope.launch { ... }` |
| `@MainActor` | `Dispatchers.Main` | UI work on main thread |
| `Task.sleep` | `delay(...)` | `delay(500)` |
| cancellation | coroutine cancellation | coroutines cancel cooperatively; `LaunchedEffect` cancels when leaving composition |

---

## 19) Reactive Streams (Combine ↔ Flow)

| SwiftUI / Combine | Jetpack Compose / Flow | Notes |
|---|---|---|
| `Publisher` | `Flow` | Both represent streams |
| `@Published` | `MutableStateFlow` | `val state = MutableStateFlow(initial)` |
| `sink` | `collect` | `flow.collect { ... }` |
| `map/filter/debounce` | `map/filter/debounce` | Flow operators exist; debouncing often from `kotlinx.coroutines.flow` |
| `receive(on:)` | `flowOn` / context | `flowOn(Dispatchers.IO)` |
| bind to UI | `collectAsState()` | `val s by vm.state.collectAsState()` |

---

## 20) Dependency Injection & Environment

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `@EnvironmentObject` | Hilt DI / CompositionLocal | Common: `val vm: MyVM = hiltViewModel()` |
| `@Environment(\.xxx)` | `Local*` | Example: `LocalContext.current`, `LocalDensity.current` |
| custom environment values | custom CompositionLocal | `val LocalFoo = compositionLocalOf<Foo> { error("...") }` + `CompositionLocalProvider` |

---

## 21) Persistence (SwiftData/CoreData ↔ Room/DataStore)

| SwiftUI / iOS | Android / Compose | Notes |
|---|---|---|
| SwiftData / CoreData | Room | Room: entities + DAO + database; expose `Flow<List<T>>` for UI |
| `@Query` | Flow from DAO | `@Query("SELECT * FROM note") fun notes(): Flow<List<Note>>` |
| UserDefaults | DataStore (Preferences) | DataStore is async and Flow-based |
| Keychain | Encrypted storage | Android often uses EncryptedSharedPreferences / Keystore patterns |

Room + Compose pattern:
- DAO returns `Flow`
- ViewModel collects/transforms
- UI uses `collectAsState()`

---

## 22) Networking (URLSession ↔ Retrofit/Ktor)

| SwiftUI / iOS | Android / Kotlin | Notes |
|---|---|---|
| `URLSession` | Retrofit / Ktor | Retrofit common with suspend APIs |
| Codable models | Kotlin data classes + serialization | `kotlinx.serialization` or Moshi/Gson |
| `async let` | parallel coroutines | `coroutineScope { val a=async{...}; val b=async{...}; }` |
| caching (URLCache) | OkHttp cache | Usually via OkHttp configuration |

---

## 23) Background Work (BGTask ↔ WorkManager)

| iOS | Android | Notes |
|---|---|---|
| BGTaskScheduler | WorkManager | Deferrable background jobs with constraints (network, charging, etc.) |
| push-triggered updates | FCM + workers | Many background patterns go through services + WorkManager |

---

## 24) Interop (UIKit/View ↔ Android View)

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `UIViewRepresentable` | `AndroidView` | `AndroidView(factory={ ctx -> SomeLegacyView(ctx) })` |
| SwiftUI inside UIKit | Compose inside Activity/Fragment | `setContent { MyCompose() }` |

---

## 25) Tooling: Previews, Debugging, Logging

| SwiftUI | Jetpack Compose | Notes |
|---|---|---|
| `#Preview` / `PreviewProvider` | `@Preview` | `@Preview @Composable fun Preview() { MyUI() }` |
| live preview | interactive previews | Android Studio Compose previews support interactions (limits apply) |
| `print()` | `Log.d(...)` | Use Android logcat + tags |
| view hierarchy inspector | Layout Inspector | Works well with Compose + semantics tree |

---

## 26) Architecture Patterns (MVVM, UDF)

| SwiftUI approach | Compose approach | Notes |
|---|---|---|
| MVVM with `ObservableObject` | MVVM with ViewModel + StateFlow | UI observes `StateFlow` via `collectAsState()` |
| unidirectional flow | same | `UI -> events -> VM -> state -> UI` |
| `@Binding` heavy | state hoisting | Keep state at the highest reasonable owner and pass callbacks down |
| global state via env | DI + locals | Hilt, CompositionLocal for cross-cutting concerns |

---

## 27) Quick “Anchor” Snippets (Memorize these)

### A) `VStack` → `Column`
```kotlin
@Composable
fun ExampleColumn() {
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        Text("Title", style = MaterialTheme.typography.titleLarge)
        Button(onClick = {}) { Text("Continue") }
    }
}
```

### B) `@State` → `remember { mutableStateOf }`
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) { Text("Count: $count") }
}
```

### C) `List`/`LazyVStack` → `LazyColumn`
```kotlin
@Composable
fun Names(names: List<String>) {
    LazyColumn {
        items(names) { name ->
            Text(name, Modifier.padding(16.dp))
        }
    }
}
```

### D) `onAppear` / `task` → `LaunchedEffect`
```kotlin
@Composable
fun LoadOnce(vm: MyVM) {
    LaunchedEffect(Unit) {
        vm.load()
    }
    // UI...
}
```

### E) Binding pattern → state hoisting
```kotlin
@Composable
fun Parent() {
    var text by remember { mutableStateOf("") }
    Child(value = text, onChange = { text = it })
}

@Composable
fun Child(value: String, onChange: (String) -> Unit) {
    TextField(value = value, onValueChange = onChange, label = { Text("Name") })
}
```

# Android Jetpack Compose Own Components

### Card

[Card Container](https://developer.android.com/develop/ui/compose/components/card)
