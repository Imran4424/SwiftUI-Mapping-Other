# SwiftUI to QML cheatsheet

---

# Components (1:1-ish)

| SwiftUI | QML (Qt Quick Controls) | Notes / Gotchas |
|---|---|---|
| `Text` | `Text` / `Label` | `Label` is a styled control; `Text` is lightweight. |
| `Label` (title + icon) | `ToolButton` or `Button` with `icon` + `text`, or custom `Row { Image; Text }` | QML `Label` is *text only*. To mimic SwiftUI `Label`, use a control that supports `icon` + `text`, or roll your own. Examples below. |
| `Image` | `Image` | `source:` file, `qrc:/`, or URL. |
| `Button` | `Button` | `icon.source` available; `onClicked: { … }`. |
| `Toggle` | `Switch` / `CheckBox` | Boolean controls. |
| `TextField` / `SecureField` | `TextField` (and `echoMode: TextInput.Password`), `TextArea` | |
| `Slider` | `Slider` | |
| `Stepper` | `SpinBox` | Integer by default; `DoubleSpinBox` exists in Labs. |
| `Picker` | `ComboBox` / `ListView` + delegate | Simple picker → `ComboBox`. Complex → `ListView`. |
| `DatePicker` | `DatePicker` (Qt Quick Controls) | |
| `ProgressView` | `BusyIndicator` / `ProgressBar` | |
| `List` / `ForEach` | `ListView` / `Repeater` / `Instantiator` | `ListView` is virtualized; `Repeater` is not. |
| `LazyVStack/HStack` | `ListView` (vertical) / `GridView` | Use views, not `Column`/`Row`, for big datasets. |
| `ScrollView` | `Flickable` / `ScrollView` | `Flickable` is the low-level scroller. |
| `VStack/HStack/ZStack` | `Column` / `Row` / `Item`+`anchors` / `StackLayout` | `anchors.centerIn`, `anchors.fill`, etc. |
| `Spacer()` | empty `Item { Layout.fillWidth: true }` or `anchors` | In `Row/Column/…Layout`, use `Layout.*`. |
| `Divider()` | `Rectangle { height: 1 }` / `Separator` | |
| `Form` | `GroupBox`, `Frame`, layouts | No magic “form”; compose with Controls. |
| `NavigationStack` | `StackView` | `stack.push(item)` / `pop()` / `replace()`. |
| `NavigationLink` | `StackView.push(…)` | Often via `onClicked: stack.push("Page.qml")`. |
| `TabView` | `TabBar` + `StackLayout` / `SwipeView` | |
| Sheets/Alerts | `Dialog`, `Popup`, `Drawer`, `MessageDialog` | |
| `ContextMenu` | `Menu` / `MenuItem` | |
| Gestures (`TapGesture`, `DragGesture`, `MagnificationGesture`) | `MouseArea`, `MultiPointTouchArea`, `PinchArea`, `Drag` attached prop | |
| `withAnimation` / implicit anim. | `Behavior` / `NumberAnimation` / `States` + `Transitions` | Attach `Behavior on x { NumberAnimation{} }`. |
| `@State` | `property` | `property int count: 0`. |
| `@Binding` | `property alias` / two-way bindings | `property alias name: textField.text`. |
| `@ObservedObject`/`@StateObject` | `QObject` with `Q_PROPERTY` + `NOTIFY` | Expose from C++ or singletons. |
| `@Environment` | context properties / singletons / `Qt.application` | Inject via `engine.rootContext()->setContextProperty(...)`. |
| Combine (`Publisher`, `sink`) | Signals/slots | QML reacts to `signal`s; bindings update automatically. |
| `Task`, async/await | Signals/callbacks, `QNetworkAccessManager`, `WorkerScript` | C++ side can use `QFuture`/`Qt Concurrent`. |
| App storage (`@AppStorage`) | `Settings` (QML) / `QSettings` (C++) | |
| Theming (`.tint`, `.accentColor`) | Qt Quick Controls styles (Fusion, Imagine, Material, Universal); palette | Set `QtQuick.Controls` style app-wide. |
| Preview canvas | Live re-run in Creator / Qt Design Studio | Not as instant as SwiftUI previews, but close. |

---

# SwiftUI `Label` → QML patterns

SwiftUI variants:
```swift
Label("Downloads", systemImage: "arrow.down.circle")
Label("Only Title", systemImage: "tray").labelStyle(.titleOnly)
Label("Only Icon", systemImage: "star").labelStyle(.iconOnly)
