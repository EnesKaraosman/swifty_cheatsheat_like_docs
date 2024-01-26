# Concurrency

## Perform asynchronous operation

Lets create some utility functions to play with async operations;

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

### Call sequentially

```swift
await fetchData(1)
await fetchData(2)
await fetchData(3)

// Output;
// "2024-01-24 15:18:23.211 : Task completed in 1 seconds"
// "2024-01-24 15:18:25.247 : Task completed in 2 seconds"
// "2024-01-24 15:18:28.320 : Task completed in 3 seconds"
```

### Call in parallel

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

## Tasks and Task Groups

The async-let syntax described in the previous section implicitly creates a child task — this syntax works well when you already know what tasks your program needs to run. You can also create a task group (an instance of TaskGroup) and explicitly add child tasks to that group, which gives you more control over **priority and cancellation**, and lets you create a dynamic number of tasks.

```swift
// Define a struct that represents a slow divide operation
struct SlowDivideOperation {
    let name: String
    let a: Double
    let b: Double
    let sleepDuration: UInt64
    
    func execute() async -> Double {
        // Sleep for x seconds
        try! await Task.sleep(nanoseconds: sleepDuration * 1_000_000_000)
        let value = a / b
        return value
    }
}

// Define an array of slow divide operations
let operations = [
    SlowDivideOperation(name: "Operation 1", a: 60, b: 2, sleepDuration: 3),
    SlowDivideOperation(name: "Operation 2", a: 60, b: 3, sleepDuration: 2),
    SlowDivideOperation(name: "Operation 3", a: 60, b: 4, sleepDuration: 1)
]

// Create a task group that returns Double values
await withTaskGroup(of: Double.self) { group in
    // Add each operation as a child task to the group
    for operation in operations {
        print("Adding \(operation.name) to the group")
        group.addTask {
            // Execute the operation and return its value
            let value = await operation.execute()
            print("\(operation.name) finished with value \(value)")
            return value
        }
    }
    
    // Iterate over the group as an async sequence and print each result
    for await result in group {
        print("Result from the group: \(result)")
    }
}

// Output;
// Adding Operation 1 to the group
// Adding Operation 2 to the group
// Adding Operation 3 to the group
// Operation 3 finished with value 15.0
// Result from the group: 15.0
// Operation 2 finished with value 20.0
// Result from the group: 20.0
// Operation 1 finished with value 30.0
// Result from the group: 30.0
```
