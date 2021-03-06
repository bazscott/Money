# Money

A precise, type-safe representation of monetary amounts in a given currency.

This functionality is discussed in Chapter 3 of
[Flight School Guide to Swift Numbers](https://gumroad.com/l/swift-numbers).

## Requirements

- Swift 4.0+

## Installation

### Swift Package Manager

Add the Money package to your target dependencies in `Package.swift`:

```swift
import PackageDescription

let package = Package(
  name: "YourProject",
  dependencies: [
    .package(
        url: "https://github.com/Flight-School/Money",
        from: "1.0.0"
    ),
  ]
)
```

Then run the `swift build` command to build your project.

### Carthage

To use `Money` in your Xcode project using Carthage,
specify it in `Cartfile`:

```
github "Flight-School/Money" ~> 1.0.0
```

Then run the `carthage update` command to build the framework,
and drag the built Money.framework into your Xcode project.

## Usage

### Creating Monetary Amounts

The `Money` type has a required associated `Currency` type.
These currency types are named according to their
three letter [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) currency code.
You can initialize a monetary using a `Decimal` value:

```swift
let amount = Decimal(12)
let monetaryAmount = Money<USD>(amount)
```

You can also create monetary amounts using
integer, floating-point, and string literals.

```swift
12 as Money<USD>
12.00 as Money<USD>
"12.00" as Money<USD>
```

This package provides all of the currencies
defined by the ISO 4217 standard as of June 2018,
with the exception of special codes, such as USN (US Dollar, Next day) and
XBC (Bond Markets Unit European Unit of Account 9).

**Important**:
Swift floating-point literals are currently initialized
using binary floating-point number type,
which cannot precisely express certain values.
As a workaround, monetary amounts initialized
from a floating-point literal are rounded
to the number of places of the minor currency unit.
If you want to express a smaller fractional monetary amount,
initialize from a string literal or `Decimal` value instead.

```swift
let preciseAmount: Money<USD> = "123.4567"
let roundedAmount: Money<USD> = 123.4567

preciseAmount.amount // 123.4567
roundedAmount.amount // 123.46
```

For more information, see https://bugs.swift.org/browse/SR-920.

### Comparing Monetary Amounts

You can compare two monetary amounts with the same currency:

```swift
let amountInWallet: Money<USD> = 60.00
let price: Money<USD> = 19.99

amountInWallet >= price // true
```

Attempting to compare monetary amounts with different currencies
results in a compiler error:

```swift
let dollarAmount: Money<USD> = 123.45
let euroAmount: Money<EUR> = 4567.89

dollarAmount == euroAmount // Error: Binary operator '==' cannot be applied
```

### Adding, Subtracting, and Multiplying Monetary Amounts

Monetary amounts can be added, subtracted, and multiplied
using the standard binary arithmetic operators (`+`, `-`, `*`):

```swift
let prices: [Money<USD>] = [2.19, 5.39, 20.99, 2.99, 1.99, 1.99, 0.99]
let subtotal = prices.reduce(0.00, +) // "$36.53"
let tax = 0.08 * subtotal // "$2.92"
let total = subtotal + tax // "$39.45"
```

**Important**: Multiplying a monetary amount by a floating-point number
results in an amount rounded to the number of places
of the minor currency unit.
If you want to produce a smaller fractional monetary amount,
multiply by a `Decimal` value instead.

### Supporting Multiple Currencies

Consider a `Product` structure with a `price` property.
If you only support a single currency, such as US Dollars,
you would define `price` to be of type `Money<USD>`:

```swift
struct Product {
    var price: Money<USD>
}
```

If you want to support multiple currencies, however,
you can't specify an explicit currency type in the property declaration.
Instead, the `Product` would have to be defined as a generic type:

```swift
struct Product<Currency: CurrencyType> {
    var price: Money<Currency>
}
```

Unfortunately, this approach is unwieldy,
as each type that interacts with `Product` would also need to be generic,
and so on, until the entire code base is generic over the currency type.

```swift
class ViewController<Currency: CurrencyType> : UIViewController { ... } // 😭
```

A better solution would be to define a new `Price` protocol
with requirements that match the `Money` type:

```swift
protocol Price {
    var amount: Decimal { get }
    var currency: CurrencyType.Type { get }
}

extension Money: Price {}
```

Doing this allows prices to be defined in multiple currencies
without making `Product` generic over the currency type:

```swift
struct Product {
    var price: Price
}

let product = Product(price: 12.00 as Money<USD>)
product.price // "$12.00"
```

If you want to support only certain currencies, such as US Dollars and Euros,
you can define a `SupportedCurrency` protocol
and add conformance to each currency type through an extension:

```swift
protocol SupportedCurrency: CurrencyType {}
extension USD: SupportedCurrency {}
extension EUR: SupportedCurrency {}

extension Money: Price where Currency: SupportedCurrency {}
```

Now, attempting to create a `Product` with a price in an unsupported currency
results in a compiler error:

```swift
Product(price: 100.00 as Money<EUR>)
Product(price: 100.00 as Money<GBP>) // Error
```

### Formatting Monetary Amounts

You can create a localized representation of a monetary amount
using `NumberFormatter`.
Set the `currencyCode` property of the formatter
to the `currency.code` property of the `Money` value
and pass the `amount` property to the formatter `string(for:)` method.

```swift
let allowance: Money<USD> = 10.00
let formatter = NumberFormatter()
formatter.numberStyle = .currency
formatter.locale = Locale(identifier: "fr-FR")
formatter.currencyCode = allowance.currency.code
formatter.string(for: allowance.amount) // "10,00 $US"
```

### Adding Custom Currencies

You can create your own custom currency types by defining an enumeration
that conforms to the `CurrencyType` protocol.
For example, here's how you might represent Bitcoin (BTC):

```swift
enum BTC: CurrencyType {
    static var name: String { return "Bitcoin" }
    static var code: String { return "BTC" }
    static var minorUnit: Int { return 8 }
}

let satoshi: Money<BTC> = 0.00000001
```

`NumberFormatter` only supports currencies defined by ISO 4217,
so you'll have to configure the symbol, currency code,
and any other necessary parameters:

```swift
let formatter = NumberFormatter()
formatter.numberStyle = .currency
formatter.currencySymbol = "₿"
formatter.currencyCode = "BTC"
formatter.maximumFractionDigits = 8

formatter.string(for: satoshi.amount) // ₿0.00000001
```

### Showing Off with Emoji

If you're the type of person who enjoys putting clip art in your source code,
here's a trick that'll _really_ impress your teammates:

```swift
typealias 💵 = Money<USD>
typealias 💴 = Money<JPY>
typealias 💶 = Money<EUR>
typealias 💷 = Money<GBP>

let tubeFare: 💷 = 2.40 // "£2.40"
```

## License

MIT

## Contact

Mattt ([@mattt](https://twitter.com/mattt))
