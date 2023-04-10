# AppValue

A package that provides an alternative to `@AppStorage`, using compiler-aware key paths instead of strings for key names and a central declaration of app-wide defaults.

## Example usage

Traditional `@AppStorage` usage in SwiftUI uses a string-based key for a particular value. Each time it is used, you must also specify a default value. For example:

```swift
struct ContentView: View {
  @AppStorage("appName") private var appName: String = "My App"
  
  var body: some View {
    // ...
  }
}

struct OtherView: View {
  @AppStorage("appName") private var appName: String = "My App"
  
  var body: some View {
    // ...
  }
}
```

If you mistype a key name, or specify different default values in different places in your code, you may end up with unexpected behiour. 

With `AppValue`, you start by declaring all your app-level defaults in one place as a single struct that conforms to `DefaultsConfig`. Individual properties are
defined using the existing `@AppStorage` property wrapper from SwiftUI:

```swift
import AppValue

struct AppConfig: DefaultsConfig {
  static var shared = AppConfig()
  
  @AppStorage("appName") var appName: String = "My App"
}
```

You then tell `AppValue` to look for its defaults within your config object:

```swift
extension AppValue where Config == AppConfig {
    init(_ keyPath: ReferenceWritableKeyPath<AppConfig, Value>) {
        self.init(keyPath)
    }
} 
```

The, use the `@AppValue` property wrapper where you would previously use `@AppStorage`:

```swift
import AppValue

struct ContentView: View {
  @AppValue(\.appName) private var appName
  
  var body: some View {
    // ...
  }
}
``` 

### Custom object storage

`AppValue` also provides support for custom `struct` objects, as long as they conform to the `Codable` protocol.

Within your config, use the `@CodableAppStorage` property wrapper:

```swift
import AppValue

struct AppConfig: DefaultsConfig {
  static var shared = AppConfig()
  
  @AppStorage("appName") var appName: String = "My App"
  @CodableAppStorage("customType") var customType: CustomType = CustomType(name: "Example", value: 42)
}
```

Within your SwiftUI views, `@AppValue(\.customType)` will give you access to the persisted copy of your value:

```swift
import AppValue

struct ContentView: View {
  @AppValue(\.appName) private var appName
  @AppValue(\.customType) private var customType
  
  var body: some View {
    // ...
  }
}
``` 
