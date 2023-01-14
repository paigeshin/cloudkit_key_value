# cloudkit_key_value

[https://www.youtube.com/watch?v=wmxwWkVWO1Y](https://www.youtube.com/watch?v=wmxwWkVWO1Y)

### Recommended Usage

- When app starts
- When app goes from background to foreground

### code

```swift
import SwiftUI
import CloudKit

// Basic Usage
struct ContentView: View {
    
    @State var text: String = ""
    
    var body: some View {
        VStack {
            TextField("hehe", text: self.$text)
            Button("Save") {
                NSUbiquitousKeyValueStore().set(text, forKey: "value")
                NSUbiquitousKeyValueStore().synchronize()
            }
            
            Button("Load") {
                if let string = NSUbiquitousKeyValueStore().string(forKey: "value") {
                    self.text = string
                }
            }
            
        }
        .onReceive(NotificationCenter.default.publisher(for: NSUbiquitousKeyValueStore.didChangeExternallyNotification)) { _ in
            
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

@propertyWrapper
struct UbiquitousStore<T: Codable> {
    
    private let key: String
    private let defaultValue: T
    
    init(key: String, defaultValue: T) {
        self.key = key
        self.defaultValue = defaultValue
    }
    
    var wrappedValue: T {
        get {
            guard let data = NSUbiquitousKeyValueStore().object(forKey: self.key) as? Data else {
                return self.defaultValue
            }
            let value = try? JSONDecoder().decode(T.self, from: data)
            return value ?? self.defaultValue
        }
        set{
            let data = try? JSONEncoder().encode(newValue)
            NSUbiquitousKeyValueStore().set(data, forKey: self.key)
            NSUbiquitousKeyValueStore().synchronize()
        }
    }
    
}

extension NSUbiquitousKeyValueStore {
    
    @UbiquitousStore(key: "value", defaultValue: "") static var value: String
//    @UbiquitousStore(key: "setting", defaultValue: "") static var settings: Settings
    
}
```
