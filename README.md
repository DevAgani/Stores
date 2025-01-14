# 🗂 Stores

[![Stores](https://github.com/omaralbeik/Stores/actions/workflows/CI.yml/badge.svg)](https://github.com/omaralbeik/Stores/actions/workflows/CI.yml)
[![codecov](https://codecov.io/gh/omaralbeik/Stores/branch/main/graph/badge.svg?token=iga0JA6Mwo)](https://codecov.io/gh/omaralbeik/Stores)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fomaralbeik%2FStores%2Fbadge%3Ftype%3Dplatforms)](https://swiftpackageindex.com/omaralbeik/Stores)
[![](https://img.shields.io/endpoint?url=https%3A%2F%2Fswiftpackageindex.com%2Fapi%2Fpackages%2Fomaralbeik%2FStores%2Fbadge%3Ftype%3Dswift-versions)](https://swiftpackageindex.com/omaralbeik/Stores)
[![License](https://img.shields.io/badge/License-MIT-red.svg)](https://opensource.org/licenses/MIT)

A typed key-value storage solution to store `Codable` types in various persistence layers like User Defaults, File System, Core Data, Keychain, and more with a few lines of code!

## Features

- [x] macOS Catalina+, iOS 13+, tvOS 13+, watchOS 6+, any Linux supporting Swift 5.4+.
- [x] Store any `Codable` type.
- [x] Single API with implementations using User Default, file system, Core Data, Keychain, and fakes for testing.
- [x] Thread-safe implementations.
- [x] Swappable implementations with a type-erased store.
- [x] [Modular library](#installation), add all, or only what you need.

---

## Motivation

When working on an app that uses the [composable architecture](https://github.com/pointfreeco/swift-composable-architecture), I fell in love with how reducers use an environment type that holds any dependencies the feature needs, such as API clients, analytics clients, and more.

**Stores** tries to abstract the concept of a store and provide various implementations that can be injected in such environment and swapped easily when running tests or based on a remote flag.

It all boils down to the two protocols [`SingleObjectStore`](https://github.com/omaralbeik/Stores/blob/main/Sources/Blueprints/SingleObjectStore.swift) and [`MultiObjectStore`](https://github.com/omaralbeik/Stores/blob/main/Sources/Blueprints/MultiObjectStore.swift) defined in the [**Blueprints**](https://github.com/omaralbeik/Stores/tree/main/Sources/Blueprints) layer, which provide the abstract concepts of stores that can store a single or multiple objects of a generic `Codable` type.

The two protocols are then implemented in the different modules as explained in the chart below:

![Modules chart](https://raw.githubusercontent.com/omaralbeik/Stores/main/Assets/stores-light.png#gh-light-mode-only)
![Modules chart](https://raw.githubusercontent.com/omaralbeik/Stores/main/Assets/stores-dark.png#gh-dark-mode-only)

---

## Usage

Let's say you have a `User` struct defined as below:

```swift
struct User: Codable {
    let id: Int
    let name: String
}
```

Here's how you store it using Stores:

<details>
<summary>1. Conform to <code>Identifiable</code></summary>

This is required to make the store associate an object with its id.

```swift
extension User: Identifiable {}
```

The property `id` can be on any `Hashable` type. [Read more](https://developer.apple.com/documentation/swift/identifiable).

</details>

<details>
<summary>2. Create a store</summary>

Stores comes pre-equipped with the following stores:

<ul>

<li>
<details>
<summary>UserDefaults</summary>

```swift
// Store for multiple objects
let store = MultiUserDefaultsStore<User>(identifier: "users")

// Store for a single object
let store = SingleUserDefaultsStore<User>(identifier: "users")
```
</details>
</li>

<li>
<details>
<summary>FileSystem</summary>

```swift
// Store for multiple objects
let store = MultiFileSystemStore<User>(identifier: "users")

// Store for a single object
let store = SingleFileSystemStore<User>(identifier: "users")
```
</details>
</li>

<li>
<details>
<summary>CoreData</summary>

```swift
// Store for multiple objects
let store = MultiCoreDataStore<User>(identifier: "users")

// Store for a single object
let store = SingleCoreDataStore<User>(identifier: "users")
```
</details>
</li>

<li>
<details>
<summary>Keychain</summary>

```swift
// Store for multiple objects
let store = MultiKeychainStore<User>(identifier: "users")

// Store for a single object
let store = SingleKeychainStore<User>(identifier: "users")
```
</details>
</li>

<li>
<details>
<summary>Fakes (for testing)</summary>

```swift
// Store for multiple objects
let store = MultiObjectStoreFake<User>()

// Store for a single object
let store = SingleObjectStoreFake<User>()
```
</details>
</li>

</ul>

You can create a custom store by implementing the protocols in [`Blueprints`](https://github.com/omaralbeik/Stores/tree/main/Sources/Blueprints)

<ul>
<li>
<details>
<summary>Realm</summary>

```swift
// Store for multiple objects
final class MultiRealmStore<Object: Codable & Identifiable>: MultiObjectStore {
    // ...
}

// Store for a single object
final class SingleRealmStore<Object: Codable>: SingleObjectStore {
    // ...
}
```
</details>
</li>

<li>
<details>
<summary>SQLite</summary>

```swift
// Store for multiple objects
final class MultiSQLiteStore<Object: Codable & Identifiable>: MultiObjectStore {
    // ...
}

// Store for a single object
final class SingleSQLiteStore<Object: Codable>: SingleObjectStore {
    // ...
}
```
</details>
</li>

</ul>
</details>


<details>
<summary>3. Inject the store</summary>

Assuming we have a view model that uses a store to fetch data:

```swift
struct UsersViewModel {
    let store: AnyMultiObjectStore<User>
}
```

Inject the appropriate store implementation:

```swift
let coreDataStore = MultiCoreDataStore<User>(databaseName: "users")
let prodViewModel = UsersViewModel(store: coreDataStore.eraseToAnyStore())
```

or:

```swift
let fakeStore = MultiObjectStoreFake<User>()
let testViewModel = UsersViewModel(store: fakeStore.eraseToAnyStore())
```

</details>

<details>
<summary>4. Save, retrieve, update, or remove objects</summary>

```swift
let john = User(id: 1, name: "John Appleseed")

// Save an object to a store
try store.save(john)

// Save an array of objects to a store
try store.save([jane, steve, jessica])

// Get an object from store
let user = store.object(withId: 1)

// Get an array of object in store
let users = store.objects(withIds: [1, 2, 3])

// Get an array of all objects in store
let allUsers = store.allObjects()

// Check if store has an object
print(store.containsObject(withId: 10)) // false

// Remove an object from a store
try store.remove(withId: 1)

// Remove multiple objects from a store
try store.remove(withIds: [1, 2, 3])

// Remove all objects in a store
try store.removeAll()
```

</details>
    
---
    
## Documentation

Read the full documentation at [Swift Package Index](https://swiftpackageindex.com/omaralbeik/Stores).

---

## Installation

You can add Stores to an Xcode project by adding it as a package dependency.

1. From the **File** menu, select **Add Packages...**
2. Enter "https://github.com/omaralbeik/Stores" into the package repository URL text field
3. Depending on what you want to use Stores for, add the following target(s) to your app:
    - `Stores`: the entire library with all stores.
    - `UserDefaultsStore`: use User Defaults to persist data.
    - `FileSystemStore`: persist data by saving it to the file system.
    - `CoreDataStore`: use a Core Data database to persist data.
    - `KeychainStore`: persist data securely in the Keychain.
    - `Blueprints`: protocols only, this is a good option if you do not want to use any of the provided stores and build yours.
    - `StoresTestUtils` to use the fakes in your tests target.

---

## Credits and thanks

- [Tom Harrington](https://twitter.com/atomicbird) for writing ["Core Data Using Only Code"](https://www.atomicbird.com/blog/core-data-code-only/).
- [Keith Harrison](https://twitter.com/kharrison) for writing ["Testing Core Data In A Swift Package"](https://useyourloaf.com/blog/testing-core-data-in-a-swift-package/).
- [Riccardo Cipolleschi](https://twitter.com/cipolleschir) for writing [Retrieve multiple values from Keychain](https://medium.com/macoclock/retrieve-multiple-values-from-keychain-77641248f4a1).
---

## License

Stores is released under the MIT license. See [LICENSE](https://github.com/omaralbeik/Stores/blob/main/LICENSE) for more information.
