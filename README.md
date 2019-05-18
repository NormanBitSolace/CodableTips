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
Additionally, `JSONDecoder.decode()` allows us to just bind the properties we're interested in. For example, if we're not interested in the `id` property we can bind to:

```swift
struct QuoteModel: Codable {
    let author: String
    let quote: String
}
```
Conversly, an exception is thrown if the model contains a new property e.g. `anotherProperty` that is not in the JSON:

```swift
struct QuoteModel: Codable {
    let author: String
    let quote: String
    let anotherProperty: String
}
```
The thrown error "`The data couldn’t be read because it is missing.`" doesn't provide much information about what caused the exception.

Similarly, if a property type doesn't match the server's type the error "`The data couldn’t be read because it isn’t in the correct format.`" is thrown.

API you depend on can change without warning. This is particuarly true when the API is in development. An app that ran used to run may break even though no code changes were made. They can be hard to track down, especially given that the localized error messages do not provide information to help.

Here are some defensive steps you can take to avoid these mysterious breakings and to help you know when the server's data structure has changed.

### Tip 1
Make all model fields optional e.g.:
```swift
struct QuoteModel: Codable {
    let id: Int?
    let author: String?
    let quote: String?
    let anotherProperty: String?
}
```
This will prevent `JSONDecoder.decode()` from throwing an exception when it encounters missing properties in the JSON.

### Tip 2
Make view model intializers failable e.g.:
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
If you're interested in knowing what data is missing from an API add a `valideate()` method to the view model's failable initializer e.g.:
```swift
extension QuoteViewModel {

    private func validate(quoteModel: QuoteModel) {
        assert(quoteModel.author != nil, "Data missing 'author'.")
        assert(quoteModel.author?.count == 0 , "Data 'author' property is empty.")
        assert(quoteModel.quote != nil, "Data missing 'quote'.")
        assert(quoteModel.quote?.count == 0 , "Data 'quote' property is empty.")
    }

    init?(quoteModel: QuoteModel) {
        validate(quoteModel: quoteModel)
        guard let author = quoteModel.author, let quote = quoteModel.quote else { return nil }
        guard author.count > 0, quote.count > 0 else { return nil }
        attributedQuote = "\(quote) –\(author)"
    }
}
```
