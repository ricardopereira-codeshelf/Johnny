![Logo](/Johnny/johnny-logo.png?raw=true)

![pod](https://cdn.rawgit.com/zolomatok/Johnny/master/pod.svg)
![platform](https://cdn.rawgit.com/zolomatok/Johnny/master/platform.svg)
![license](https://cdn.rawgit.com/zolomatok/Johnny/master/license.svg)

Johnny is a caching library written in Swift 3.

## Features
- [x] Generic cache with out-of-the-box support for
  - String, Bool, Int, UInt, Int64, UInt64, Float, Double
  - URL, Data, Date
  - UIImage, UIColor
  - Arrays, Dictionaries and Sets of the above
  - **Optionals** of the above
  - **Any** type that conforms to the `Storable` protocol
- [x] Multiplatform, supporting iOS, macOS, tvOS & watchOS
- [x] First-level memory cache using `NSCache`
- [x] Second-level LRU disk cache
- [x] Disk access in background thread (always when saving, optionally when fetching)
- [x] Syncronous & Asynchronous API support
- [x] Automatic cache eviction on memory warnings & disk capacity reached
- [x] Unit tested
- [x] Concise, well-structured code to encourage contributions

Extra ❤️ for images:
- [x] `func setImageWithURL` extension on UIImageView, UIButton & NSImageView, optimized for cell reuse

## Usage

###Caching###
```swift
Johnny.cache(Date(), key: "FirstStart")
```

###Retrieving###

```swift
// The type of the retrived value must be explicitly stated for the compiler.
let date: Date? = Johnny.pull("FirstStart")
```

###Async fetch###

If you know you are retrieving a large object (> 1.5 MB):

```swift
Johnny.pull("4KImage") { (value: UIImage?) in
     
}
```

###Prefix###

You can set a global prefix that gets attached to the key parameter in every call to enable for example caching of data on a per userID basis. 

```swift
Johnny.prefix = localUser.userID
```

## Examples

### Storable ###

*Note, that since the protocol uses a static constructor function instead of an initializer, you can extend any class without errors, even if you don't have access to its source (like Standard library classes)*

```swift
public protocol Storable {
    associatedtype Result
    static func fromData(data: NSData) -> Result?
    func toData() -> NSData
}
```

###Collections ###

You can cache any collection of items conforming to the Storable protocol (most Stdlib data types already do)

```swift
let array: [String] = ["Folsom", "Prison", "Blues"]
let stringSet: Set<String> = ["I've", "been", "everywhere"]
// In case of dictionaries, the value must explicitly conform to Storable (so [String: AnyObject] does not work, while [String: Double] does)
let dictionary: [String: String] = ["first": "Solitary", "second": "man"]

Johnny.cache(array, key: "folsom")
Johnny.cache(stringSet, key: "everywhere")
Johnny.cache(dictionary, key: "solitary")
```

### Custom types ###

Due to current Swift limitations, since the Storable protocol has an `associatedType`, conformance must be added through an extension.
`class User: Storable` will not work.


```swift
class User {

    enum Badassery: String { case Total }

    var name: String? = "Johnny"
    var uid: Int = 84823682
    var badassery = Badassery.Total
}


extension User: Storable {
typealias Result = User

static func fromData(data: NSData) -> User.Result? {
    let dict = try! NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions()) as! [NSObject: AnyObject]

    let user = User()
    user.uid = dict["identification"] as! Int
    user.name = dict["name"] as? String
    user.badassery = Badassery(rawValue: dict["badassery"] as! String)!
    return user
}

func toData() -> NSData {
    let json = ["identification": uid, "name": name, "badasery": badassery.rawValue]
    return try! NSJSONSerialization.dataWithJSONObject(json, options: NSJSONWritingOptions())
    }
}
```

**Using it with Johnny:**


```swift
let johnny: User = User()
Johnny.cache(johnny, key: "John")
let cachedJohn: User = Johnny.pull("John")
```



## Requirements
- iOS 8.0+
- macOS 10.10+
- tvOS 9.0+
- watchOS 2.0+
- Swift 2.0+ (there's also a swift3 branch)

## Install

**CocoaPods**

```swift
pod 'Johnny'
```

## Attribution
I'd like to thank the creators of [Pantry](https://github.com/nickoneill/Pantry) and [Haneke](https://github.com/Haneke/HanekeSwift) as those projects provided much of the inspiration and some code. Johnny was dreamed up to be the best of both worlds.

## License
Johnny is released under the MIT license. See LICENSE for details.
