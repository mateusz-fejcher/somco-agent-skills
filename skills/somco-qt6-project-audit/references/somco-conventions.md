# Somco Software Qt6 Conventions — Good & Bad Practices

This file defines Somco Software's house-style rules for Qt6 C++ and QML projects.
Each rule has an ID, rationale, and a concrete good/bad example.
The audit skill enforces these during the conventions check phase.

---

## QML Conventions

### QCONV-001 — Use `qt_add_qml_module` for all QML modules

**Bad:** Manually managing `qmldir`, resource files, or using deprecated
`qt_add_resources` / `qt5_add_resources` for QML files.

```cmake
# Bad
qt_add_resources(myapp "qml_resources" PREFIX "/" FILES main.qml Page.qml)
```

**Good:** Declare every QML module with `qt_add_qml_module` so the tooling
(qmllint, qmlls, type compilation) works automatically.

```cmake
# Good
qt_add_qml_module(myapp
    URI MyApp
    VERSION 1.0
    QML_FILES main.qml Page.qml
)
```

**Rationale:** `qt_add_qml_module` generates correct `qmldir`, enables
ahead-of-time type compilation, and integrates with all Qt tooling.

---

### QCONV-002 — No business logic in QML

**Bad:** HTTP calls, database access, file I/O, complex data transforms,
or algorithmic logic directly in QML/JavaScript.

```qml
// Bad
function fetchUsers() {
    var xhr = new XMLHttpRequest()
    xhr.open("GET", "https://api.example.com/users")
    xhr.send()
}
```

**Good:** Expose a C++ model or service object; QML only binds to its
properties and calls thin slots.

```qml
// Good
ListView {
    model: userService.users
}
Button {
    onClicked: userService.refresh()
}
```

**Rationale:** QML's JS engine is single-threaded and untyped. Business
logic in C++ is testable, debuggable, and faster.

---

### QCONV-003 — No imperative property assignments in signal handlers when bindings suffice

**Bad:** Using `onXChanged` handlers to imperatively push values that
could be expressed as declarative bindings.

```qml
// Bad
onWidthChanged: rect.width = parent.width * 0.5
```

**Good:** Use a binding.

```qml
// Good
Rectangle { width: parent.width * 0.5 }
```

**Rationale:** Imperative assignments break bindings and create
one-directional data flow bugs that are hard to diagnose.

---

### QCONV-004 — Prefer `required property` over context properties

**Bad:** Relying on `setContextProperty` or untyped context injection.

```cpp
// Bad
engine.rootContext()->setContextProperty("myModel", &model);
```

**Good:** Use `required property` in QML and set via `setInitialProperties`.

```qml
// Good — QML side
Window {
    required property MyModel myModel
}
```

```cpp
// Good — C++ side
view.setInitialProperties({{"myModel", QVariant::fromValue(&model)}});
```

**Rationale:** `required property` is type-safe, documented in the
component API, and avoids the deprecated context property mechanism.

---

### QCONV-005 — Use unversioned QML imports

**Bad:**
```qml
import QtQuick 2.15
import QtQuick.Controls 2.15
```

**Good:**
```qml
import QtQuick
import QtQuick.Controls
```

**Rationale:** Qt6 uses unversioned imports. Versioned imports are a
Qt5 pattern that is ignored at best and confusing at worst.

---

### QCONV-006 — Avoid `var` for typed properties

**Bad:**
```qml
property var count: 0
property var userName: ""
```

**Good:**
```qml
property int count: 0
property string userName: ""
```

**Rationale:** `var` disables QML type checking and ahead-of-time
compilation. Use concrete types whenever the type is known.

---

### QCONV-007 — No deeply nested Component / Loader chains

**Bad:** More than two levels of inline `Component { Loader { Component { ... } } }`.

**Good:** Extract inner components into separate `.qml` files.

**Rationale:** Deep nesting hurts readability, makes profiling difficult,
and prevents the QML compiler from optimizing the component.

---

## C++ Conventions

### CCONV-001 — Use `QML_ELEMENT` / `QML_NAMED_ELEMENT` for QML-exposed types

**Bad:** Manual `qmlRegisterType` calls scattered through `main.cpp`.

```cpp
// Bad
qmlRegisterType<MyItem>("MyApp", 1, 0, "MyItem");
```

**Good:** Declarative registration via macros + CMake integration.

```cpp
// Good
class MyItem : public QQuickItem {
    Q_OBJECT
    QML_ELEMENT
    // ...
};
```

**Rationale:** Declarative registration is compile-time checked, works
with `qt_add_qml_module`, and is the only supported path in Qt6.

---

### CCONV-002 — Signals must use `Q_EMIT`, not bare `emit`

**Bad:**
```cpp
emit dataChanged();
```

**Good:**
```cpp
Q_EMIT dataChanged();
```

**Rationale:** `Q_EMIT` is the official macro and avoids conflicts with
third-party libraries that define `emit` (e.g., Boost.Signals2).

---

### CCONV-003 — Prefer pointer-to-member `connect` syntax

**Bad:**
```cpp
connect(sender, SIGNAL(clicked()), receiver, SLOT(onClicked()));
```

**Good:**
```cpp
connect(sender, &QPushButton::clicked, receiver, &MyClass::onClicked);
```

**Rationale:** Compile-time checked, refactor-safe, and slightly faster
at runtime (no string comparison).

---

### CCONV-004 — No raw `new` for QObject-derived classes without parent

**Bad:**
```cpp
auto *item = new MyItem();  // no parent, no smart pointer
```

**Good:**
```cpp
auto *item = new MyItem(parentObject);
// or
auto item = std::make_unique<MyItem>();
```

**Rationale:** QObjects without parents and without smart pointers leak.
Always assign a parent or use RAII ownership.

---

### CCONV-005 — Mark QObject subclasses as `Q_DISABLE_COPY_MOVE`

**Bad:** QObject subclass with no copy/move guard (relies on the implicit
one from QObject, which is easy to bypass with templates).

**Good:**
```cpp
class MyService : public QObject {
    Q_OBJECT
    Q_DISABLE_COPY_MOVE(MyService)
public:
    explicit MyService(QObject *parent = nullptr);
};
```

**Rationale:** Explicit guard prevents accidental slicing and makes the
non-copyable contract visible to readers.

---

### CCONV-006 — Use `QStringLiteral` or `u"..."_s` for string literals

**Bad:**
```cpp
QString name = "hello";
```

**Good:**
```cpp
using namespace Qt::StringLiterals;
QString name = u"hello"_s;
```

**Rationale:** Avoids runtime Latin-1 to UTF-16 conversion. `u"..."_s`
is the modern Qt6 idiom; `QStringLiteral` is also acceptable.

---

### CCONV-007 — Prefer scoped enums (`enum class`) exposed via `Q_ENUM`

**Bad:**
```cpp
enum Status { Active, Inactive };
Q_ENUM(Status)
```

**Good:**
```cpp
enum class Status { Active, Inactive };
Q_ENUM(Status)
```

**Rationale:** Scoped enums prevent implicit conversions and namespace
pollution. Qt6 supports `Q_ENUM` with `enum class`.

---

## CMake Conventions

### CMCONV-001 — Minimum CMake version 3.16+

**Bad:**
```cmake
cmake_minimum_required(VERSION 3.10)
```

**Good:**
```cmake
cmake_minimum_required(VERSION 3.16)
```

**Rationale:** Qt6 requires CMake 3.16+. Many Qt6 CMake functions
behave differently or are unavailable on older versions.

---

### CMCONV-002 — Use versionless Qt CMake targets

**Bad:**
```cmake
target_link_libraries(myapp PRIVATE Qt6::Core Qt6::Quick)
```

**Good:**
```cmake
target_link_libraries(myapp PRIVATE Qt::Core Qt::Quick)
```

**Rationale:** Versionless targets (`Qt::`) work with both Qt5
(via `QT_DEFAULT_MAJOR_VERSION`) and Qt6, easing future upgrades.

---

### CMCONV-003 — Set `CMAKE_AUTOMOC`, `CMAKE_AUTORCC` globally

**Bad:** Calling `set_target_properties(... AUTOMOC ON)` per-target
or forgetting entirely.

**Good:**
```cmake
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
```

**Rationale:** Global setting ensures no target is missed. MOC/RCC
are required for virtually all Qt code.

---

### CMCONV-004 — No `file(GLOB)` for source files

**Bad:**
```cmake
file(GLOB SOURCES "src/*.cpp")
```

**Good:**
```cmake
target_sources(myapp PRIVATE
    src/main.cpp
    src/mymodel.cpp
    src/myservice.cpp
)
```

**Rationale:** `GLOB` does not re-run when files are added/removed,
leading to stale builds. Explicit listing is reliable.

---

## Anti-Patterns (Always Flag)

| ID | Pattern | Description |
|---|---|---|
| ANTI-001 | `QQmlApplicationEngine` + `setContextProperty` | Deprecated context property injection |
| ANTI-002 | `qmlRegisterType` / `qmlRegisterSingletonType` calls | Use declarative `QML_ELEMENT` macros instead |
| ANTI-003 | `#include <QtWidgets>` in a QML-only project | Module header pulls in all of QtWidgets unnecessarily |
| ANTI-004 | `QThread::sleep` / `QThread::msleep` in main thread | Blocks the event loop; use QTimer or async patterns |
| ANTI-005 | `.pro` / `.pri` files present alongside CMakeLists.txt | Mixed build systems; remove qmake files |
| ANTI-006 | `QML_FILES` missing from `qt_add_qml_module` | QML files won't be compiled or picked up by tooling |
| ANTI-007 | `Qt.createComponent` / `Qt.createQmlObject` for static content | Use Loader or inline components instead |
