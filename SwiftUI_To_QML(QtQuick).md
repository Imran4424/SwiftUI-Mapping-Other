# SwiftUI ⇄ QML(Qt Quick) Cheatsheet

> Imports you’ll see in snippets  
> **SwiftUI:** `import SwiftUI`  
> **QML:**  
> ```qml
> import QtQuick 2.15
> import QtQuick.Controls 2.15
> import QtQuick.Layouts 1.15
> // optional styles: import QtQuick.Controls.Material 2.15
> ```

## 1) Layout & Containers

| SwiftUI                       | QML / Qt Quick                                                              | Notes                                                                                     |
| ----------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `VStack`                      | `ColumnLayout` (preferred), or `Column`                                     | `ColumnLayout` supports per-child sizing/alignment; `Column` is a lightweight positioner. |
| `HStack`                      | `RowLayout`, or `Row`                                                       | Same story as above.                                                                      |
| `ZStack`                      | `Item` with stacked children                                                | Children render in order; use `z` property to adjust stacking.                            |
| `Spacer()`                    | `Item { Layout.fillWidth/Height: true }`                                    | In a `*Layout`, this acts like a stretch.                                                 |
| `.frame(maxWidth: .infinity)` | `Layout.fillWidth: true`                                                    | In Layouts. Outside Layouts, use anchors (e.g., `anchors.fill: parent`).                  |
| `.padding()`                  | `Control { padding: X }` or `anchors.margins: X` on parent `Item/Rectangle` | Controls have `padding`; plain `Item` does not.                                           |
| `LazyVStack/LazyHStack`       | `ListView`, `GridView`, `PathView`                                          | Qt’s views are virtualized (lazy) by default.                                             |
| `LazyVGrid/LazyHGrid`         | `GridView`, `Flow`                                                          | `GridView` virtualizes; `Flow` is simple wrap layout (no virtualization).                 |
| `ScrollView`                  | `Flickable` or `ScrollView` (Controls)                                      | `Flickable` is the core; `ScrollView` wraps it with scroll bars.                          |
| `NavigationStack`             | `StackView`                                                                 | For push/pop navigation (not layout).                                                     |
| `TabView`                     | `TabView` (Controls)                                                        | Similar API; uses `TabBar`/`TabButton` internally.                                        |

### Tiny equivalence
```qml
// QML VStack-like
ColumnLayout {
  spacing: 8
  Text { text: "Title"; Layout.alignment: Qt.AlignLeft }
  Rectangle { Layout.fillWidth: true; height: 40 }
  Item { Layout.fillHeight: true } // Spacer
}
```

## 2) Text, Images & Labels

| SwiftUI                      | QML                                                                  | Notes                                                                        |
| ---------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `Text`                       | `Text` (QtQuick), **or** `Label` (Controls)                          | `Label` inherits styling from Controls themes; `Text` is low-level and fast. |
| `Label` (title + icon)       | `Label` + `icon` property (Controls), or `Row` with `Image` + `Text` | Yes, QML **does** have a `Label`.                                            |
| `Image`                      | `Image`, `IconImage` (Controls)                                      | `IconImage` respects the active style’s icon theme.                          |
| `.font(.title)`              | `font.pointSize / pixelSize / bold`                                  | No named scales; set numbers or use style logic.                             |
| `.foregroundStyle` / `.tint` | `color`                                                              | For Controls, theme palettes apply automatically.                            |

## 3) Controls & Forms

| SwiftUI                    | QML (Controls)                                       | Notes                                      |
| -------------------------- | ---------------------------------------------------- | ------------------------------------------ |
| `Button`                   | `Button`, `ToolButton`                               | `onClicked: { ... }`.                      |
| `Toggle`                   | `Switch`                                             | Boolean.                                   |
| `Slider`                   | `Slider`                                             | Same.                                      |
| `Stepper`                  | `SpinBox`                                            | Integer spinner.                           |
| `Picker`                   | `ComboBox`                                           | For lists.                                 |
| `TextField`                | `TextField`                                          | `echoMode: TextInput.Password` for secure. |
| `TextEditor`               | `TextArea`                                           | Multiline.                                 |
| `DatePicker`               | `DatePicker`                                         | In Controls 2 (Qt 6+).                     |
| `ProgressView`             | `ProgressBar`, `BusyIndicator`                       | Determinate/indeterminate.                 |
| `ToggleStyle/ControlStyle` | Styles: `Material`, `Universal`, `Fusion`, `Imagine` | Choose once; palette follows.              |

## 4) Lists & Data

| SwiftUI                | QML                                                  | Notes                                                         |
| ---------------------- | ---------------------------------------------------- | ------------------------------------------------------------- |
| `List`                 | `ListView` + `model` + `delegate`                    | Delegate is your row view.                                    |
| `ForEach`              | `Repeater` / `ListView`                              | `Repeater` creates real items (no recycling).                 |
| `Section`              | `ListView` + `section.property` + `section.delegate` | Built-in sectioning.                                          |
| Identifiable / `id`    | `id` (object id) vs `model` keys                     | `id` is local object identifier; list data comes via `model`. |
| Observable collections | `ListModel` / JS arrays / C++ models                 | Use roles to expose fields.                                   |

**Simple list**
```qml
ListView {
  model: 100
  delegate: Item {
    width: ListView.view.width; height: 44
    Text { anchors.verticalCenter: parent.verticalCenter; text: "Row " + index }
  }
}
```

## 5) Navigation, Sheets & Overlays

| SwiftUI          | QML                         | Notes                                    |
| ---------------- | --------------------------- | ---------------------------------------- |
| `.sheet`         | `Dialog`, `Popup`, `Drawer` | `modal: true` for blocking dialogs.      |
| `.alert`         | `Dialog` with buttons       |                                          |
| `NavigationLink` | `StackView.push(item)`      | Use `StackView` API.                     |
| `popover`        | `Popup`                     | Position with `x/y`, `anchors.centerIn`. |
| `toolbar`        | `ToolBar`                   | With `ToolButton`s.                      |

## 6) State, Binding, Env

| SwiftUI            | QML                                                | Notes                                           |
| ------------------ | -------------------------------------------------- | ----------------------------------------------- |
| `@State`           | `property <type> name`                             | QML properties auto-bind and notify.            |
| `@Binding`         | `property alias`                                   | Alias to child’s property for two-way.          |
| `@ObservedObject`  | QML `QtObject` with `property` + `signal`          | Or C++/Python backend with `Q_PROPERTY`.        |
| `@Environment`     | Singletons / `Qt.application` / context properties | Expose via `QtObject` singleton or C++ context. |
| Combine publishers | QML **signals** + `Connections`                    | Reactive-by-default property binding.           |

**Two-way binding-ish**
```qml
// Parent.qml
property alias name: field.text
TextField { id: field }
```

## 7) Gestures, Input & Focus

| SwiftUI                          | QML                                  | Notes                                |
| -------------------------------- | ------------------------------------ | ------------------------------------ |
| `TapGesture`, `LongPressGesture` | `TapHandler`, `LongPressHandler`     | In `QtQuick` Pointer Handlers.       |
| `DragGesture`                    | `DragHandler`, `MouseArea` (classic) | Use Handlers (Qt 6+) when possible.  |
| `MagnificationGesture`           | `PinchHandler`                       |                                      |
| Keyboard focus                   | `focus: true`, `Keys.onPressed`      | `KeyNavigation` for focus traversal. |

**Long press (≈ 1s)**
```qml
Rectangle {
  width: 120; height: 44; color: "#eee"
  LongPressHandler { minimumPressDuration: 1000; onLongPressed: console.log("long press") }
}
```

## 8) Animation & Transitions

| SwiftUI                 | QML                                             | Notes                                                         |
| ----------------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| `withAnimation {}`      | `NumberAnimation`, `ColorAnimation`, `Behavior` | Attach `Behavior on x { NumberAnimation { duration: 200 } }`. |
| `matchedGeometryEffect` | `States` + `Transitions` (manual)               | You orchestrate start/end states.                             |
| Springs                 | `SpringAnimation`                               | Also `SmoothedAnimation`.                                     |

**Hover animation**
```qml
Rectangle {
  id: card; width: 200; height: 120; radius: 12; color: "#fafafa"
  property real target: 0
  Behavior on scale { NumberAnimation { duration: 120 } }
  HoverHandler { onHoveredChanged: card.scale = hovered ? 1.03 : 1.0 }
}
```

## 9) Styling & Theming

| SwiftUI                   | QML                                                                 | Notes                                 |
| ------------------------- | ------------------------------------------------------------------- | ------------------------------------- |
| `.preferredColorScheme`   | Control style modules: `Material`, `Universal`, `Fusion`, `Imagine` | Set once app-wide.                    |
| Accent/tint color         | `palette` on `ApplicationWindow` or `Control`                       | `Material.theme`, `Material.accent`.  |
| `.environment(\.font, …)` | Inherit via `font` on parent Controls                               | Palettes/fonts cascade down the tree. |

**Material style**
```qml
import QtQuick.Controls.Material
ApplicationWindow {
  Material.theme: Material.Dark
  Material.accent: Material.Teal
}
```

## 10) Networking & Storage

| SwiftUI (URLSession)          | QML                                                          | Notes                                                |
| ----------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| `URLSession`                  | `XmlHttpRequest` (JS in QML), or C++ `QNetworkAccessManager` | For serious apps, prefer C++/Python backend service. |
| `AppStorage` / `UserDefaults` | `Settings` (Qt.labs.settings)                                | Simple key-value.                                    |
| File I/O                      | `File` via JS, or C++                                        | Sandboxing depends on platform.                      |

**Quick fetch**
```qml
Item {
  Component.onCompleted: {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
      if (xhr.readyState === XMLHttpRequest.DONE) console.log(xhr.responseText);
    }
    xhr.open("GET", "https://api.example.com/data");
    xhr.send();
  }
}
```

## 11) Graphics, Effects & Canvas

| SwiftUI Shapes                   | QML                                              | Notes                                                      |
| -------------------------------- | ------------------------------------------------ | ---------------------------------------------------------- |
| `Rectangle`, `Circle`, `Capsule` | `Rectangle`, `Image`, `Shape` (Qt Quick Shapes)  | Rounded corners via `radius`.                              |
| Blur/Shadow                      | `Qt Quick Effects` (`DropShadow`, `MultiEffect`) | Use `Qt5Compat.GraphicalEffects` if needed for older Qt 6. |
| Custom drawing                   | `Canvas` (HTML5-like)                            | Or ShaderEffect for GPU work.                              |

## 12) Common Migrations & “Gotchas”

- **`ColumnLayout` vs `Column`:** use `ColumnLayout` when you’d normally rely on `.frame/Spacer/alignment` in SwiftUI. Avoid mixing **anchors** and **Layouts** on the same item.
- **Alignment:** `VStack(alignment: .leading)` → set `Layout.alignment` on each child (e.g., `Qt.AlignLeft`). `Column` alone has no per-child alignment.
- **Lazy lists:** QML’s `ListView` is virtualized by default—great performance, but you design cell reuse logic via the `delegate`.
- **Bindings are live:** `text: someProp` auto-updates when `someProp` changes (no explicit `@Published`). Don’t mutate behind QML’s back.
- **Models:** Keep models simple (roles/keys). For complex logic, back with C++/Python models and expose to QML.
- **Accessibility:** use `Accessible` attached properties (Qt Quick). Not automatic—add roles/labels as needed.
- **Threading:** UI is single-threaded like SwiftUI. Do heavy work in workers/C++ threads; signal results back.

---

## Mini Side-by-Side Example

**SwiftUI**
```swift
struct ProfileCard: View {
  @State private var name = "Mahi"
  var body: some View {
    VStack(alignment: .leading, spacing: 12) {
      HStack {
        Image(systemName: "person.circle.fill")
          .font(.system(size: 48))
        VStack(alignment: .leading) {
          Text(name).font(.title2).bold()
          Text("Lead iOS • SmartThings").foregroundStyle(.secondary)
        }
        Spacer()
        Button("Follow") { /* ... */ }
      }
      ProgressView(value: 0.6)
    }
    .padding()
  }
}
```

**QML**
```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
import QtQuick.Layouts 1.15
// import QtQuick.Controls.Material 2.15

Item {
  width: 360; height: 140

  Rectangle {
    anchors.fill: parent
    radius: 14
    color: "#ffffff"
    border.color: "#e6e6e6"

    ColumnLayout {
      anchors.fill: parent
      anchors.margins: 16
      spacing: 12

      RowLayout {
        spacing: 12
        Label { text: "\u{f2bd}"; font.pixelSize: 48 } // or IconImage
        ColumnLayout {
          Label { text: name; font.pixelSize: 20; font.bold: true }
          Label { text: "Lead iOS • SmartThings"; opacity: 0.7 }
        }
        Item { Layout.fillWidth: true } // Spacer
        Button { text: "Follow"; onClicked: {/* ... */} }
      }

      ProgressBar { value: 0.6 }
    }
  }

  property string name: "Mahi"
}
```

---

## What to reach for first (mental map)

- **Vertical/Horizontal stacks:** `ColumnLayout` / `RowLayout`
- **Sections & long lists:** `ListView` (+ `section.delegate`)
- **Forms:** `TextField`, `ComboBox`, `Switch`, `Slider`, `SpinBox`
- **Navigation:** `StackView` (push/pop), `TabView`
- **Modals/Sheets:** `Dialog` / `Popup` / `Drawer`
- **Gestures:** `TapHandler`, `LongPressHandler`, `DragHandler`, `PinchHandler`
- **Animations:** `Behavior`, `NumberAnimation`, `Transitions`
- **Theming:** pick `Material` or `Universal` early
