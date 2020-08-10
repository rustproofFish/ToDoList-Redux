# ToDoList-Redux
Topic : Redux based ToDo / Task application

## Objectives
Lately I have been looking at migrating part of the code from the application I am developing from UIKit to SwiftUI. While the pure transition from UIKit to SwiftUI was actually pretty smooth in my first tribulation, the [architectural framework](https://github.com/DeclarativeHub/TheBinderArchitecture) I am using for UIKit, just [doesn't seem to fit](https://github.com/DeclarativeHub/TheBinderArchitecture/issues/10) very well to SwiftUI. This led me on a wild-goose chase for other, more suited, framework for SwiftUI.

### Enters Redux
While on my search I came across [Redux](https://redux.js.org) (deja vu ?) and more specifically [SwiftRex](https://github.com/SwiftRex/SwiftRex). There are a ton of other Redux frameworks for Swift out there, but SwiftRex seemed to line up pretty well with what I was looking for, and its creator, [@luizmb](https://github.com/luizmb), was super responsive and very helpful in getting me started.

### ToDo / Task application

As I was going through the evaluation of the different Swift Redux frameworks, I saw numerous "Counter" examples. This seems like the "Hello World !" equivalent for Redux. However it's really not nearly enough to get a decent feel for how well it will suit one's needs.

So I used a fairly basic Task application to compare those different Swift Redux framework. I initially started reading about [how to model app state using Store object](https://swiftwithmajid.com/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) using `ObservableObject`. Thanks to some really good reading from [Peter Friese](https://peterfriese.dev/replicating-reminder-swiftui-firebase-part1/) and [Majid Jabrayilov](https://swiftwithmajid.com/2019/07/31/introducing-container-views-in-swiftui/), I settled on a design for the initial Task application (read those links if you want to learn more about the different designs).

## Iterations

### 1. First Redux Concepts [(swiftrex-option1b)](https://github.com/npvisual/ToDoList-Redux/tree/swiftrex-option1b)

The first approach is based on a very light introduction of Redux and SwiftRex concepts : 

* it uses Combine's [`@StateObject`](https://github.com/npvisual/ToDoList-Redux/blob/96d705993b02318989045aa95dfd8534c1ff46ca/ToDoList-Redux/ToDoList_ReduxApp.swift#L18) and [`@ObservedObject`](https://github.com/npvisual/ToDoList-Redux/blob/96d705993b02318989045aa95dfd8534c1ff46ca/ToDoList-Redux/Views/TaskList.swift#L13) to "link" with SwiftRex's [`ObservableViewModel`](https://github.com/SwiftRex/SwiftRex/blob/develop/Sources/CombineRex/ObservableViewModel.swift). This is (roughly) how you [pass down Redux's Store](https://github.com/npvisual/ToDoList-Redux/blob/96d705993b02318989045aa95dfd8534c1ff46ca/ToDoList-Redux/Views/TaskList.swift#L19) (or projections of the Store) to your Views,
* it shows how to [create the Store](https://github.com/npvisual/ToDoList-Redux/blob/96d705993b02318989045aa95dfd8534c1ff46ca/ToDoList-Redux/ToDoList_ReduxApp.swift#L15) early on in your app,
* it demonstrates how to [bind](https://github.com/npvisual/ToDoList-Redux/blob/96d705993b02318989045aa95dfd8534c1ff46ca/ToDoList-Redux/Views/TaskList.swift#L20) the State and Actions to a regular SwiftUI view.
* it gives an [example](https://github.com/npvisual/ToDoList-Redux/blob/96d705993b02318989045aa95dfd8534c1ff46ca/ToDoList-Redux/Views/TaskList.swift#L64) of how Previews can be achieved with SwiftRex.

You can find some great explanations from Luis [here](https://github.com/SwiftRex/SwiftRex/issues/67#issuecomment-669599677) on the different possible approaches I was going after.

However, while this was a good first step, it was nowehre near what Luis described as "Option 1b", and it still wasn't making much use of the Redux framework since it was directly tying the cell view into the ForEach of the TaskList.

### 2. Container / Rendering View [(swiftrex-containerview)](https://github.com/npvisual/ToDoList-Redux/tree/swiftrex-containerview)

This iteration was my attempt to implement what Luis had described as "Option 1b" :

* it creates an abstration level, i.e. the ["Container" concept](https://github.com/npvisual/ToDoList-Redux/blob/14db58edbaf79646845dfc7d3732b511f7163310/ToDoList-Redux/Views/CheckmarkCellView.swift#L28) that Majid described in [his post](https://swiftwithmajid.com/2019/10/02/redux-like-state-container-in-swiftui-part3/), between the actual view and the parent view. But with a little twist... by pushing the notion of State and Actions to that container,
* it still leaves the [implementation](https://github.com/npvisual/ToDoList-Redux/blob/14db58edbaf79646845dfc7d3732b511f7163310/ToDoList-Redux/Views/CheckmarkCellView.swift#L11) of the Rendering View and puts the Container View in charge of doing the [data binding job](https://github.com/npvisual/ToDoList-Redux/blob/14db58edbaf79646845dfc7d3732b511f7163310/ToDoList-Redux/Views/CheckmarkCellView.swift#L42),
* the Container View uses Combine's `@ObservedObject` to receive an `ObservableViewModel` from the calling View,
* that now forces us to use a [projection](https://github.com/npvisual/ToDoList-Redux/blob/14db58edbaf79646845dfc7d3732b511f7163310/ToDoList-Redux/Views/TaskList.swift#L22) of our original view model -- we're a little more Redux-like.

### 3. `ObservableViewModel` in the Rendering View itself [(swiftrex-observableviewmodel)](https://github.com/npvisual/ToDoList-Redux/tree/swiftrex-observableviewmodel)

Another approach suggested by Luis was to integrate the View Model directly in the Rendering View (i.e. bypassing the Container View), while still keeping the View independent of any "higher level" Application Logic.

While I didn't like this approach initially because the presence of the `ObservableViewModel` forces the creator of the Rendering View to know about this particular SwiftRex concept, it somewhat grew on me, because of its simplicity :

* it uses an [extension of the Rendering View](https://github.com/npvisual/ToDoList-Redux/blob/d630c425c1a3cbfb0b58b0b3609b7bf4d464d12f/ToDoList-Redux/Binders/CheckmarkCellViewModel.swift#L12) to define the Redux-specific concepts,
* it still leaves the Rendering View itself [very clean](https://github.com/npvisual/ToDoList-Redux/blob/d630c425c1a3cbfb0b58b0b3609b7bf4d464d12f/ToDoList-Redux/Views/CheckmarkCellView.swift#L13), when doing so...
* the separation of (Redux) concerns in the [extension](https://github.com/npvisual/ToDoList-Redux/blob/d630c425c1a3cbfb0b58b0b3609b7bf4d464d12f/ToDoList-Redux/Binders/TaskListViewModel.swift#L13) (and [here](https://github.com/npvisual/ToDoList-Redux/blob/d630c425c1a3cbfb0b58b0b3609b7bf4d464d12f/ToDoList-Redux/Binders/ReduxFramework.swift#L105)) allows us to define more Redux-like concepts, like `projection`, at the appropriate layer ; see for example how the TaskList extension allows us to define a [projection](https://github.com/npvisual/ToDoList-Redux/blob/d630c425c1a3cbfb0b58b0b3609b7bf4d464d12f/ToDoList-Redux/Binders/TaskListViewModel.swift#L67) specifically for the `CheckmarkCellView` (i.e. abstracting that mapping logic where it makes sense),
* it allows us to have a [fairly clean call](https://github.com/npvisual/ToDoList-Redux/blob/d630c425c1a3cbfb0b58b0b3609b7bf4d464d12f/ToDoList-Redux/Views/TaskList.swift#L23) from the parent View, because we've abstracted most of the binding logic out in the extensions.


Now, a lot of the changes between #2 and #3 were purely cosmetic and a lot of the simplications introduced in #3 could have been made for `CheckmarkCellContainerView` as well. But we're slowly getting to what I consider a more "Redux-like" approach.
