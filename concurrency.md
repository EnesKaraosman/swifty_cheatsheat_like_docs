# Concurrency

## Perform asynchronous operation

Asenkron çağrımlarımızı deneyebilmek için yardımcı bir fonksiyon oluşturalım;

```swift
var dateFormatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS "
    return formatter
}()
​
func fetchData(_ seconds: Int) async -> Int {
    try! await Task.sleep(for: .seconds(seconds))
    let timestamp = await dateFormatter.string(from: Date())
    debugPrint("\(timestamp): Task completed in \(seconds) seconds")
    
    return seconds
}
```

### Call in serial

```swift
await fetchData(1)
await fetchData(2)
await fetchData(3)

// Output;
// "2024-01-24 15:18:23.211 : Task completed in 1 seconds"
// "2024-01-24 15:18:25.247 : Task completed in 2 seconds"
// "2024-01-24 15:18:28.320 : Task completed in 3 seconds"
```

### Call in Parallel

```swift
async let first = fetchData(1)
async let second = fetchData(1)
async let third = fetchData(1)

let seconds = await [first, second, third]

// Output;
// "2024-01-24 15:19:40.256 : Task completed in 1 seconds"
// "2024-01-24 15:19:40.256 : Task completed in 1 seconds"
// "2024-01-24 15:19:40.256 : Task completed in 1 seconds"
```
