---
layout: post
title: "Using .sheet to produce different screens"
date: 2021-01-17 23:35:36 GMT
tags: SwiftUI swift iOS swift 
description: "Associating modals sheets in SwiftUI to optional types, instead of a boolean"
---

Frequently, we use `.sheet` to deterministically produce a given view, and we link the process to a [binding boolean](https://developer.apple.com/documentation/swiftui/view/sheet(ispresented:ondismiss:content:)). But, we can bind an optional source of truth for the sheet. That way, when the binding optional is set to a non-nil value, the system calls the responsible view, passing the value, and asking for a view to populate the modal. And if the value is set to nil, then the system dismisses the modal. 

Let's say we have a Home Screen, and we want to give our users access to a settings screen with a bar button. That would be easy, we can bind such control with a boolean, and link our settings sheet to that same boolean. 

```
struct ContentView: View {
    @State var showingSettings = false

    var body: some View {
        Button(action: {
            self.showingSettings.toggle()
        }) {
            Text("Show Settings")
        }.sheet(isPresented: $showingSettings) {
            SettingsView()
        }
    }
}
```

That's fine. But now, we want to show a list of, let's say, news stories, and we want to give our users the chance to annotate them. In each cell of our list, we have a button that should trigger a popup linked to the story. 

We can pass the selected story using environmental variables, and toggle a bool to produce the popup, and that should be fine, a little messy, but fine. 

But, what if we want to be able to produce more than one type of popups from the same root view? Well, we can let `.sheet` do the heavy lifting for us. 

First, we need a type that can adopt different values and convey different meanings.

```
enum ActiveSheet: Identifiable {
    case settings
    case selectedStory(Story)
    
    var id: Int {
        switch self {
        case .settings:
            return 1
        case .selectedStory(_):
            return 2
        }
    }
}
```

That's going to be the type of our source of truth. 

```
@State var activeSheet: ActiveSheet?
```

If `activeSheet` is nil, no popup is shown, if we have a concrete value, the system will call the closure defined as content in `sheet(item:onDismiss:content:)`. The system will call the closure with the corresponding value:

```
.sheet(item: $activeSheet, content: { (item: ActiveSheet) -> AnyView in
		switch item {
				case .settings:
					return AnyView(SettingsView())
        case .selectedStory(let story):
	        return AnyView(AnnotateStory(story))
    }
})
```

We can use the same technique to pass the story, even if we don't need to differentiate popup types, but just to have the story linked without environmental variables. 

One more thing, if the binding optional is changed while the system is showing a modal corresponding to a given value (we have on-screen the modal associated with type A, and the source of truth is changed to B), then the system will dismiss the first modal, and produce a new modal corresponding to the appropriate value. How cools is that?