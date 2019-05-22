# Codable Tips

Swift 4 added the `Codable` protocol that automatically handles binding Data to custom model types. A typical example of this is binding JSON returned from a REST call to a model. Swift's `JSONDecoder.decode()` implements all the functionality required, e.g.:

```json
[
    {
        "id" : 1,
        "author" : "Rin Tin Tin",
        "quote" : "Woof"
    }
]
```

```swift
struct QuoteModel: Codable {
    let id: Int
    let author: String
    let quote: String
}
```
Additionally, `JSONDecoder.decode()` allows us to just bind the fields we're interested in. For example, if we're not interested in the `id` field we can bind to:

```swift
struct QuoteModel: Codable {
    let author: String
    let quote: String
}
```
Conversly, an exception is thrown if the model contains a field the data doesn't e.g. `extraField` that is not in the JSON:

```swift
struct QuoteModel: Codable {
    let author: String
    let quote: String
    let extraField: String
}
```
Additionally, the thrown error "`The data couldn’t be read because it is missing.`" doesn't provide much information about what caused the exception.

Similarly, if a model field's type doesn't match the JSON's field type the error "`The data couldn’t be read because it isn’t in the correct format.`" is thrown, without any information about which field has the incorrect type.

API you depend on can change without warning. This is particuarly true when the API is in development. An app that used to run may break even though no code changes were made. Such errors can be hard to track down, especially given that the localized error messages do not provide pertinent info.

Here are some defensive steps you can take to avoid these mysterious breakings and to help you know when the server's JSON structure has changed.

### Tip 1
### Prevent missing fields exceptions.
Make all model fields optional e.g.:

```swift
struct QuoteModel: Codable {
    let id: Int?
    let author: String?
    let quote: String?
    let extraField: String?
}
```
This will prevent `JSONDecoder.decode()` from throwing an exception when it encounters missing fields in the JSON.

### Tip 2
#### Filter out malformed data
Use failable initializers for view model e.g.:

```swift
struct QuoteViewModel {
    let attributedQuote: String
}

extension QuoteViewModel {
    init?(quoteModel: QuoteModel) {
        guard let author = quoteModel.author, let quote = quoteModel.quote else { return nil }
        guard author.count > 0, quote.count > 0 else { return nil }
        attributedQuote = "\(quote) –\(author)"
    }
}
```
This way you can use `compactMap` to filter out mal formed data e.g.:
```swift
let viewModel = models.compactMap { QuoteViewModel(quoteModel: $0) }
```

### Tip 3
#### Use Xcode debugger Quick Look to examine data
Viewing `Data` in the debugger doesn't reveal information about the `Data`'s contents. Xcode's `Data` Quick Look plugin helps visualize `Data` content.
Click the Quick Look icon ![image](https://user-images.githubusercontent.com/2135673/57973249-f969f880-795a-11e9-991b-7c60962070a1.png) to see a visualization of the `Data`'s content:

![image](https://user-images.githubusercontent.com/2135673/57973307-ca07bb80-795b-11e9-830a-fb30dfe9c20d.png)


### Tip 4
#### Mixed CRUD operations
When alternating between GET, POST, PUT and DELETE REST API it is often helpful to log the full URL with parameters and the HTTP method.
```swift
extension URLSession {
    open func dataTaskDebug(with request: URLRequest, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask {
        print("\(request.httpMethod ?? "method unspecified") \(request.url?.absoluteString ?? "nil URL")")
        return URLSession.shared.dataTask(with: request, completionHandler: completionHandler)
    }

}
```

### Miscellaneous

> When showing an issue to the team responsible for the API layer, demonstrate the problem with a REST client like RESTED, it eliminates any discussion of where the issue actually lies.

### Todo
1. Add support for incorrect types
1. Add support for logging missing data to console.
