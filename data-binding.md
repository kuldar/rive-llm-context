## Data Binding

### Data Binding Overview

> Connect editor elements to data and code using View Models.

### What is Data Binding?

Data binding is a powerful way to create reactive connections between editor elements, data, and code. For instance, you might:

* Bind the color of an icon to a color data property that can be adjusted by a developer at runtime.
* Bind the X-position of an animated object so that it can be followed with an offset by another object.
* Listen for a click event on two buttons to increment and decrement a counter.

### Why Use Data Binding?

Data binding decouples design and code by introducing intermediate data that both sides can bind to. This forms the “contract” between designers and developers. Once this contract is in place, both sides can iterate independently, speeding your ability to deliver new features and experiment with what’s possible.

Within the editor, data binding allows for more reactivity in your designs. You can establish relationships between objects and ensure that certain invariants hold true, no matter the state of the artboard. The data binding system will ensure that these relationships are always up to date as animations and calls from code change the values.

It also offers the opportunity to shift more logic into the Rive file and out of code. You will need to decide whether a piece of logic lives in code or data binding for your given use case, but one consideration is that any data binding logic will be universal across runtimes, rather than needing separate re-implementations.

---

### Intro Videos

The following video series introduces data binding through a guided walkthrough of the editor, which will help solidify the concepts below.

* Introduction
* View Models
* Number Properties
* String Properties
* Color Properties
* Binding State Machine Conditions

---

### Glossary

Data binding introduces a number of concepts that you will need to familiarize yourself with. The names of these concepts are loosely derived from the Model, View, Viewmodel (MVVM) pattern in software development.

#### Editor Element

For the purposes of data binding, an “editor element” simply refers to an editable UI element in the editor with a value that can have a binding attached to it.

#### View Model

A view model is a blueprint for a collection of data. Developers might think of this as similar to a class in object-oriented programming. View models typically describe all of the associated data with a given use case—commonly one per artboard. View models themselves don’t have concrete values. For that, you must have an instance.

#### View Model Property

A view model property is one piece of data within a view model. Developers might think of this as similar to a field in object-oriented programming. Properties have a data type which is selected when they are created and a name which can be referenced in code. Each property can be bound to different editor elements of the same type.

#### View Model Instance

A view model instance is the living version of a view model with actual values. Developers might think of this as similar to a class instance in object-oriented programming. Instances have the same properties as the view model they are derived from, except now each of these properties has a living value that can change over time.

You may create as many instances as you’d like from a given view model. Each can be given a unique name associated with what those values represent. Each can have different initial values for its properties, representing a design-time configuration.

> **Example:** If you had a menu with three buttons with icons: 🏠 Home, 👤 Profile, and ❓ About, you might have a single artboard representing the menu item, but three view model instances, each with the menu item’s label and associated icon, that can be applied to that artboard to configure the buttons.

Artboards are assigned an instance to populate the data bindings. Changing which instance is applied will change the initial state of the properties and all associated bound elements.

In order for an instance to be visible to developers, it must be marked as **Exported**. Otherwise, it is considered internal to the file. One reason you may want to keep it internal is if you only use the instance to test your design when it is configured with a given set of values, including edge cases.

These exported instances can then be assigned to an artboard at runtime by developers. Alternatively, developers can create empty instances which have default values, such as zero for numbers and empty strings. Once the instance is assigned, its values will begin updating according to the bindings.

#### Binding

A binding is an association between a property and an editor element. For instance, you might have a property named “Name” bound to a text run’s text value.

Bindings can be **source to target**, **target to source**, or **bidirectional**. In this case, “source” means the property, and “target” means the editor element.

* **Source to Target (Default):** Changes to the property update the value of the element. For example, an XPos property updates the X position of an object.
* **Target to Source:** Changes to the element’s value update the property. For example, the X position of an object updates the XPos property.
* **Bidirectional:** Changes are applied in both directions, meaning either the element or the property can update the other.

Additionally, a binding may be marked as **“Bind Once”**. This means that the initial value will apply and thereafter the binding will not apply any updates.

#### View Model Nesting

View models can have another view model as one of their properties. This is referred to as “nesting”. This is useful when a parent instance wants to associate with a particular child instance, similar to components.

#### Enumeration (Enum)

An enum represents a fixed set of options, similar to a drop-down. Use this property type to constrain the available values to a known, unchanging set.

Enum properties can be either a **“system” enum**, in which case they represent a fixed set of options in the editor, such as the “Horizontal Align” options, or a **“user defined” enum**, in which case they can represent any fixed set of options applicable to your use case.

#### Converter

A converter is a general purpose way of transforming a binding’s value when it is applied. These transformations might involve changing its type. For instance, the “Convert to String” converter can be used to convert a numerical binding to text, so that an object’s X position could be applied to a text run.

To apply a converter on a value that already has a binding, right click on the bound property, click **Update Bind**, and select your converter from the **Convert** dropdown.

---

### Comparing to Existing Features

Data binding fills some of the same roles as existing features in Rive. In general, it is considered a more powerful alternative to both inputs and events, and we recommend you adopt it for most use cases going forward. However, this does not mean that you need to retrofit existing files as they will continue to work as expected.

#### Type Support

View model properties can represent more types of data compared to inputs and events. See below for a comparison.

| Feature | Inputs | Events | View Model Properties |
| --- | --- | --- | --- |
| Floating point numbers | ✅ | ✅ | ✅ |
| Booleans | ✅ | ✅ | ✅ |
| Triggers | ✅ | ❌ | ✅ |
| Strings | ❌ | ✅ | ✅ |
| Enumerations (Enums) | ❌ | ❌ | ✅ |
| Colors | ❌ | ❌ | ✅ |
| View Model Nesting | ❌ | ❌ | ✅ |
| Lists | ❌ | ❌ | ✅ |
| Images | ❌ | ❌ | ✅ |
| Artboards | ❌ | ❌ | ✅ |

#### State Machine Inputs

Before data binding, state machine inputs were the primary way for developers to affect designs. They formed the “input” side of the contract with design. View model properties are a more flexible system.

* Inputs can only be used to drive state machine transitions, whereas data binding can be used to drive most editor elements in Rive and state machine transitions.
* Inputs must be used as-is where data-bound properties can be converted, either before being used by developers or before being applied to editor elements.
* View model properties also support both polling and listening APIs for developers, whereas inputs only support polling. This means that developers can more naturally react to changes in data.
* View model properties can also be used in two features currently used by inputs, that being blended states (both Blend 1D and Blend Additive) as the mix parameter and as the receiver for listeners, e.g. setting a value on a mouse click or tap.

#### Events

The counterpart to inputs, events were the primary way for developers to receive “outputs” from designs. Data binding is a much richer channel for developers to observe values from the Rive design. Additionally, events were used internally in Rive files to add reactivity using listeners. Both of these use cases are addressed by properties. For developers, the runtime APIs allow you to subscribe to changes to their values. For designers, you can bind reacting elements directly to the property.

* Events can only be triggered by timelines, state machine transitions, or listeners. By comparison, data bound properties can be changed from any number of sources.
* Events can have keyable properties with values that are passed when triggered. This is limited to being updated by animation keys and can be tricky to “bubble” when the animation exists on a component. By comparison, view model properties carry the most recent data each time they change, from any level of the hierarchy, triggering a developer’s listener with the new value.
* One use case which events offer functionality not yet supported by data binding is in their ability to play audio.

#### Constraints

Constraints allow for a specific kind of binding between two objects, such as Translation for position. This constraint is optimized for that use case, with built in options for local/world space conversion, a strength parameter, minimum and maximum values, etc. For use cases where this is all you need, this is likely to be the more concise option. By comparison, for example, data binding the X and Y positions can be used for a broader range of output behavior, though it may require some setup with converters to achieve.

#### Nesting

There are a few use cases related to nesting where you may want to consider updating to use data binding, as it offers a much more straightforward approach:

* Setting nested inputs
* Setting nested text runs
* “Bubbling” nested events

These three use cases are unified by view model instances, where components can pull from top-level viewmodels. This simplifies the developer interop, as the structure, naming, and nesting of the file’s artboards can change without breaking the code’s reference to the data.

---

## Lists

> Use Data Binding to generate lists at runtime.

A List is a way to display a set of items that are dynamically generated based on bound data values that you set up in your View Models. This allows you to build Rive files that can change in real time based on updates to those values. You can use Lists to create:

* Menus with a dynamic amount of options
* Product listings
* Notifications or activity feeds
* Chat messages
* Dropdown menus
* Contacts, friends, or followers lists
* High scores, tables, and more

### Artboard Lists

Artboard Lists enable you to generate a number of list items using an Artboard to represent each item in your List. Artboard Lists must be added as children of an Artboard or Layout.

To add an Artboard List to the stage, first select either an Artboard or a Layout. In the inspector on the right side of the editor, you will see a button to add an Artboard List. Click it to add the List to your hierarchy. It will appear as a child of the Artboard or Layout had previously selected.

Once the Artboard List is added to your hierarchy, you can select it and see its inspector.

The next step is to bind data to it using Data Binding. This will determine the content and number of items in your List. There are 2 ways to generate List content:

1. Using the **View Model List property**
2. Using a **View Model Number property** together with a **Number to List converter**

---

### View Model List Property

Before continuing, it’s important to understand the fundamentals of Data Binding, in particular, what a View Model is, how to create one and how to bind it to an object’s properties. Learn more in the Data Binding Overview.

A View Model List is a property that can contain a dynamic number of items which each represent a View Model instance. In order to be used in a List, the View Model must be bound to an Artboard.

**To Create a View Model List and bind it to an Artboard List:**

1. **Create a List Item View Model:** Navigate to the Data tab in the Editor. Click the `+` button beside View Models to create a View Model (this will represent your List item).
2. **Bind the List Item View Model to your Item Artboard:** Bind that View Model to an Artboard that you want to use to render your List item. This is where you may also want to create additional View Model properties and bind those to object properties on that Artboard.
3. **Create your Main View Model:** Click the `+` button beside View Models to create another View Model (this will be the View Model that contains your List and should be bound to the Artboard where you want your List to render).
4. **Add a List property:** Select the newly created View Model and click the `+` button next to it. From the popout select **List**. This adds a List property to the View Model.
5. **Add List Items:** Select the List property and in the inspector on the right, you can add items by clicking **Add List item**.
* *Note:* Once a List item is added, you can click the settings button to the left of its name in order to set the View Model and View Model instance for that item.


6. **Bind the List property to your Artboard List:** After adding your List items, go back to the Hierarchy tab, select the Artboard List, and from the Artboard List property dropdown, select the ViewModel List property you created above.

Run the state machine and you should see your list items. Remember that the layout will be determined by the Artboard List’s parent, so modify the parent’s settings in order to tweak your List’s layout.

> Rive’s runtimes provide APIs to modify the List and List items at runtime (for example, adding or removing items).

---

### View Model Number with Converter

The second way to populate your list is by specifying the number of items you want in your List along with the ViewModel (Artboard) you want to instance. This can be done using a View Model Number property in combination with the new Number to List converter.

**To Create a View Model Number to List converter and bind it to an Artboard List:**

1. **Create a List Item View Model:** Navigate to the Data tab in the Editor. Click the `+` button beside View Models to create a View Model (this will represent your List item).
2. **Bind the List Item View Model to your Item Artboard:** Bind that View Model to an Artboard that you want to use to render your List item. This is where you may also want to create additional View Model properties and bind those to object properties on that Artboard.
3. **Create your Main View Model:** Click the `+` button beside View Models to create another View Model (this will be the View Model that contains your Number property and should be bound to the Artboard where you want your List to render).
4. **Add a Number property:** Select the newly created View Model and click the `+` button next to it. From the popout select **Number**. Select the newly created Number and in the inspector, set the number of items you want in your List.
5. **Add a Number to List converter:** Click the `+` button and choose **Converters > List > Number to List**.
6. **Set the View Model to use in the converter:** Select the created converter and in the inspector, choose the View Model that you created earlier that represents your List item. The converter will convert the Number of items to actual Artboard items.
7. **Bind the Number property and converter to your Artboard List:** After adding your Number property and converter, go back to the Hierarchy tab, select the Artboard List, and from the Artboard List property dropdown, select the ViewModel Number property you created above. The Combobox will show a yellow outline.
* Right click the Combobox and choose **Update Bind**.
* In the converter field, select the Number to List converter you created. The yellow outline should change to green.



Run the state machine and you should see your list items. Remember that the layout will be determined by the Artboard List’s parent, so modify the parent’s settings in order to tweak your List’s layout.

---

### View Model List Item Index

There may be times where it is useful for the Artboard to know at which index it exists within its parent List. This is available using the **View Model list item index property**.

This can be added to your item’s View Model by clicking the `+` button and selecting **List Attributes > Index**.

This property can then be bound to an object’s properties and used directly (for example, to change the position of an object based on the index), used together with a converter (to display the index value as a string) or used as a condition in a state machine (to provide different behavior depending on the item’s index).

> **Note:** When using item index with the List property and list items, be aware that if more than 1 list item is bound to the same View Model instance, they will share the same index value.

---

### Lists & Scrolling

If you add your Artboard List as a child of a Layout, your List items will behave as children of its parent’s Layout. This means things like direction, wrapping, padding, gap, alignment, etc. are all dictated by the parent Layout’s properties.

In addition, if you have applied a Scroll Constraint to the parent Layout, the List items will have the ability to scroll out of the box without any additional setup.

There are additional Scroll properties that can be applied when scrolling Lists. See **List Scrolling** for more details.

---

Would you like me to also format the "List Scrolling" section mentioned at the end if you provide that text?
