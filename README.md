# Composable Architecture Extras
A companion library to Point-Free's [swift-composable-architecture](https://github.com/pointfreeco/swift-composable-architecture).

* [TaskResult+VoidSuccess](#taskresult-and-voidsuccess)
* [Installation](#installation)

## TaskResult And VoidSuccess
In the current TCA, TaskResult<Void> was not Equatable compliant, so it must be written like this
```Swift
public struct HogeFeature: Reducer {
  public struct State: Equatable {}
  public enum Action: Equatable {
    case onAppear
    case response(TaskResult<EquatableVoid>)
  }

  public func reduce(into state: inout State, action: Action) -> Effect<Action> {
    switch action {
    case .onAppear:
      return .run { send in
        await send(
          .response(
            TaskResult {
              try await asyncThrowsVoidFunc()
              return VoidSuccess()
            }
          )
        )
      }
    case .response(.success):
      // do something
      return .none

    case .response(.failure(let error)):
      // handle error
      return .none
    }
  }
}
```
it is not kind to create an `VoidSuccess` on user just to conform `TaskResult` to `Equatable` even though `Void` is already provided in Swift.
So we extended TaskResult so that it could be written like this.
```Swift
public struct HogeFeature: Reducer {
  public struct State: Equatable {}
  public enum Action: Equatable {
    case onAppear
    case response(TaskResult<VoidSuccess>)
  }

  public func reduce(into state: inout State, action: Action) -> Effect<Action> {
    switch action {
    case .onAppear:
      return .run { send in
        await send(
          .response(
            TaskResult {
              try await asyncThrowsVoidFunc()
            }
          )
        )
      }
    case .response(.success):
      // do something
      return .none

    case .response(.failure(let error)):
      // handle error
      return .none
    }
  }
}
```

## Installation
In the dependencies section, add:
```Swift
.package(url: "https://github.com/Ryu0118/swift-composable-architecture-extras", from: "1.0.0")
```
In each module, add:
```Swift
.target(
  name: "MyModule",
  dependencies: [
    .product(name: "ComposableArchitectureExtras", package: "swift-composable-architecture-extras")
  ]
),
```
