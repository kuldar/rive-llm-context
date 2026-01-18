## State Machines

### State Machine Overview

> Add intelligence to your animations.

#### Overview

State Machines are a visual way to connect animations together and define the logic that drives the transitions. They allow you to build interactive motion graphics that are ready to be implemented in your product, app, game, or website.

State machines create a new level of collaboration between designers and developers, allowing both teams to iterate deep in the development process without the need for a complicated handoff.

Using the State Machine requires designers and animators to think more like a developer but in a straightforward, visual way. Every artboard has at least one State Machine by default, but you can create as many as you’d like. To create a new state machine, hit the plus button in the Animations List and select the State Machine option.

#### Anatomy of a State Machine

A basic state machine will consist of a Graph, States, Transitions, Inputs and Layers. We’ll explore each of these pieces and more throughout this section.

* **The Graph:** The space in which you’ll be adding States and connecting Transitions. It appears in place of the Timeline when a state machine is selected in the animations list.
* **States:** Simply timeline animations that can play in your state machine. Typically, these will represent some state that your animated content is in. For example, a button will typically have an Idle state (the button is stationary), a Hovered state (what the button looks like when it is hovered), and a Clicked state (what the button looks like when it’s been clicked).
* **Creating Transitions:** Once we have defined the States of our content, we can tie them together with transitions to create a logical path that our State Machine can take through these different timelines. We’re creating a map that our State Machine can use to get from one animation to the next.

> **DEPRECATION NOTICE:** This section is about the legacy Inputs system.
> * **For new projects:** Use Data Binding instead.
> * **For existing projects:** Plan to migrate from Inputs to Data Binding as soon as possible.
> 
> 
> This content is provided for legacy support only.

**Inputs** are a legacy tool to control transitions in our state machine. While Inputs can still be used to control transitions, Data Binding is considered best practice since View Models are both more powerful and easier to control at runtime. The best use for Inputs is quick, prototype interactions that you don’t plan to migrate to runtime.

Inputs are the contract between designers and developers. As designers, we use them as rules for our transitions to occur. For example, we could have a boolean called `isHovered`. That boolean controls the transition between our idle and hovered state. When the boolean is true, the state machine is in the hovered state, and when it is false, the state machine is in the Idle state. Developers tie into these inputs at runtime and define actions that control the state machines inputs I.E. defining hit areas that can change the `isHovered` boolean.

**Layers:** Lastly, all state machines will have at least one Layer. Because only a single animation can play on a given layer, we have the ability to add multiple layers if we want to mix different animations, or add additional interactions. For example, this state machine has multiple layers, each one with the logic to control one of the buttons in this menu.

---

### States

States are simply timeline animations that can play at any point in your state machine. A state could be as simple as changing the color and position of an object, or as complex as blending multiple timelines together.

There are a few types of states that you’ll end up using as you work with the State Machine, including Default States, Single animations, and Blend States. We’ll explore each of these below.

#### Default States

The Default States are the states that, by default, are added to every State Machine.

* **Entry State:** The state that your State Machine will start from. You’ll notice that by default, your state machine will already have an animation attached to the Entry State, but you can change this animation at any time. Note that you can connect multiple animations to the Entry State if you need I.E. you want to build a switch that can start in either the on or off state.
* **Exit State:** The Exit State tells the State Machine layer to stop playing. This niche state has uses when multiple layers are being used.
* **Any State:** Unlike normal states, states connected to the Any State can be played at any time, regardless of which state your state machine is in. Any States are great to use when you want to create an array of states that can be activated at any time, such as changing the skin of a character.

#### Animation States

Animation states include all states other than the default states added to a State Machine. These states will control the look and motion of your interactive content. There are three types of animation states; Single Animation, 1D blend, and Direct blend states.

To add a State to the Graph, you can drag and drop an animation from the Animations List directly onto the Graph. Notice that this will create a Single Animation state. You can change the state type using the inspector.

Additionally, you can right-click on the graph and create a blank state of any type with no associated timelines. To assign a timeline to a state, use the timeline dropdown in the inspector.

* **Single animation state:** Any timeline that we create can be used as a single animation state. Depending on the type of animation we are using, the single animation state could be a one-shot, looping, or ping-pong state. In most cases, you’ll be using single animation states to create most of your state machines.
* **Blend states:** A Blend State is any state that blends together two or more timeline animations. We use these states for content like loading bars, health systems, scrolling interactions, and dynamic face rigs. There are two types of blend states; 1D and Direct Blend states.

**1D Blend state**
A 1D Blend State allows us to mix multiple timelines together with a single numerical input. This state works by ramping up one animation and ramping down the other while you increase or decrease a number input. Note that this mixing is not linear, but is additive and could give you unexpected results.

**Configuring a 1D Blend State:**

1. You’ll want to start by creating a few timelines for your Blend state. Keep in mind that it’s often best to use timelines with only a few properties keyed. In this health bar example, only the X scale is keyed.
2. After adding a 1D Blend State to the graph, use the Inspector to configure the state.
3. First, add the number input you want to drive the blend using the dropdown. If you haven’t created one yet, you’ll notice that nothing appears here.
4. The plus button that appears below the number input allows you to add timelines to your blend state. Use the dropdown to assign a specific timeline. Note that you can add as many timelines as you’d like.
5. Next, you need to define a numerical range that your blend state will work between. This particular blend works between 0 and 100.

Notice that once you define the range, a graphic appears above the input dropdown, visually representing how your animations will mix. When the state machine is active, as you increase or decrease your input within the defined range, you’ll see a visual representation move across that graph, showing you the mix of your timelines.

**Additive Blend state**
An Additive Blend state allows you to blend together multiple timelines using multiple number inputs. This allows us to create unique poses and facial positions by mixing multiple animations together. While working with an Additive Blend, you’ll either be mixing an animation by value or input. Read more below.

**Value vs Input blend**
When adding animations to an Additive blend state, you’ll be prompted to either add a Blend by Value animation or a Blend by Input animation.

* **Blend by Value:** Can be thought of as the baseline animation, or default pose. This value is not tied to an input, so it can’t be used to control the state machine. Instead, this value describes its mix weighting.
* **Input blend:** An animation that is mixed with the default pose or motion via a number input. Each of your different Input blends should have their own number input.

#### Additional State Options

When you select a state on the State Machine Graph, you’ll have a number of options that you can change.

* **Change state type:** The top three icons allow you to change the type of state. You can select from single animation, 1D blend, and Additive blend.
* **Change animation:** You can use the dropdown to change which animation is assigned to the current state.
* **Speed:** You can alter the playback speed of a state by changing this value. Note that you can play animations forward with a positive value, and backward with a negative value.
* **Transitions:** You can see any transitions that leave from the selected state. You also have the option to ignore specific transitions by turning off the eye icon.

---

### Inputs

> **⚠️ DEPRECATED: Use Data Binding instead of Inputs for controlling Rive graphics**
> **DEPRECATION NOTICE:** This entire page documents the legacy Inputs system.
> * **For new projects:** Use Data Binding instead.
> * **For existing projects:** Plan to migrate from Inputs to Data Binding as soon as possible.
> 
> 
> This content is provided for legacy support only.
> Inputs are a legacy tool to control transitions in our state machine. While Inputs can still be used to control transitions, Data Binding is considered best practice since View Models are both more powerful and easier to control at runtime.
> The best use for Inputs is quick, prototype interactions that you don’t plan to migrate to runtime.

#### Creating a new Input

To create a new Input, use the plus button in the input panel. After hitting the plus button, you’ll be prompted to select the type of input you want to create. There are three types of inputs; booleans, triggers, and numbers.

#### Input Types

We can use three types of inputs depending on the situation and type of interactive content: booleans, triggers, and numbers. We’ll discuss each of these inputs below.

* **Boolean:** A boolean can hold either a true or false value.
* **Trigger:** Triggers are similar to booleans, but can only become true for a short time.
* **Number:** A number input give you a number box that can be any integer.

---

### Transitions

Transitions supply the logical map for the state machine to follow. There are a number of considerations and configurable properties for transitions that we will cover below. Note that we’ll briefly discuss Inputs as well, so be sure to read more about those as well here.

#### Creating a new Transition

To create a transition, place your mouse near the state you want to leave until you notice the ellipse appear. Click and drag the ellipse to the state you want to transition to. Once you’ve connected two states, you’ll notice an ellipse with an arrow icon indicating the transition’s direction.

Note that you can create multiple transitions from one state to another. Each of these transitions can require a different condition to be met, which will fire the transition, thus giving you the ability to make “or” conditions.

#### Configuring a Transition

Once you’ve added a transition, selecting the direction indicator will allow you to configure the transition. There are three different sections to the transition panel, the transition properties, conditions, and interpolation.

**Transition properties**
The transition properties allow you to customize how a transition occurs.

* **Duration:** The duration property describes how long it takes for a transition to occur.
* The duration is set to zero by default, meaning the transition happens immediately. So, when we transition between these two animations, it appears as though the object snaps from one side of the artboard to the other.
* If we increase the duration, you’ll notice that the higher the number, the longer the transition takes.
* In a way, transitions act as their own animation. The starting properties (coming from the state your state machine is leaving from) will be interpolated to the ending properties (the starting properties of the state your state machine is going to). If the starting properties are the first key on a timeline, and the ending properties are the second key, the duration is the timing between those two keys. Transitions are much more complex than this, but thinking about transitions this way will help you diagnose issues with your state machine.


* **Exit Time:** Exit Time tells the state machine how much of the state must play before transitioning.
* By default, Exit Time is unchecked. If you want to enable the Exit Time, use the check box. Once the setting is enabled, you can use either a time value or percent.
* For example, if you want the state machine to play the entire animation before transitioning, you can either enter the duration of the animation, or use 100%.


* **Pause when exiting:** The Pause When Exiting option pauses the animation you are leaving from during the transition. As we discussed in the duration section, when a transition happens, properties from the first state are mixed with the first key of the next state. In reality, the animation your state machine leaves from continues to play as the transition happens.

#### Conditions

Conditions are the rules for our transitions. Without conditions, our transitions would continuously fire and our state machine would likely look either glitchy, or only play a single animation. Conditions require us to define some inputs, which you can read more about here.

**Adding a new condition**
To add a new condition to a transition, hit the plus button next to the Conditions section.

Each new condition provides a dropdown showing all of the inputs you’ve added to the State Machine. The configuration options will be different depending on the input type you select.
Note that you can add multiple conditions to a single transition to create an “and” transition.

* **Configuring a Boolean:** When you configure a boolean, you can decide if the transition happens when said boolean is either true or false.
* **Configuring a Number:** When you configure a number input, you can tell the transition to happen when a numerical condition happens such as equalling a specific number, being greater than or less than a specific number.
* **Configuring a Trigger:** When you add a trigger input to a transition, you are telling the transition to fire when that trigger occurs.

#### Interpolation

You can add interpolation to your transition at the bottom of the Transitions Panel. By default, the interpolation is set to linear, but you can use the cubic and hold interpolations.
Note that the interpolation between states is most effective when your transition duration is longer.

---

### Listeners

> Listeners let designers create interactive behavior without the use of code.

Listeners let designers create interactive behaviors—like clicks, hovers, and drags—directly within Rive, without needing code. For example, you can attach Pointer Enter, Pointer Exit, and Click listeners to a button. When triggered, these listeners can update data bindings, set inputs, fire events, and more—enabling dynamic, interactive experiences at runtime.

#### Creating a new Listener

1. In the Animations tab, select your State Machine.
2. In the Listeners tab, click the plus icon.
3. If you have an object selected when creating a listener, it will automatically be designated as the target.

With the new listener selected, you’ll see its options displayed in a new panel at the bottom of the State Machine Graph and to the right of the Graph.

#### Elements of a Listener

A listener consists of three parts: a Target, a User Action, and a Listener Action.

#### Target

The Target determines where to listen for the user action.

* **Hit Areas:** In most cases the Target defines the interactive area that responds to user actions—similar to a hitbox in game development. When a user interacts with this area (e.g., by clicking or hovering), the associated listener is triggered.
It’s usually best to use shapes as targets—for example, an ellipse or rectangle with 0% opacity. If you use a Group as a target, the shapes within the group will serve as the interactive area.
To select a target, click the Target icon and choose an object from the artboard or the Hierarchy panel.
Note that having an object selected when you create a listener will automatically assign the selected object as the target of the listener.
* **Listening to Events on Components:** We strongly recommend using data binding to communicate between artboards, rather than relying on nested Events. Setting an Artboard or Component as the target allows you to listen for Events fired from that Artboard.
* **Opaque Target:** The Opaque Target option determines whether or not pointer events will pass through the hit area, potentially triggering multiple Listeners at once.

#### User Action

User Actions are the interactions the listener is listening for. The drop-down menu below the Target button allows you to change which User Action the Listener checks for.

Available actions include:

* **Pointer Down:** Mouse down or finger press.
* **Pointer Up:** Releasing a mouse click or finger press.
* **Pointer Enter:** A mouse or finger entering the target area.
* **Pointer Exit:** A mouse or finger exiting the target area.
* **Pointer Move:** Any mouse or finger movement within the target area.
* **Click:** A combination of pointer down and pointer up within the same target area.
* **Listen for Event:** Only visible if the target is an Artboard or Component. If multiple events exist, use the dropdown to select the specific one.

#### Listener Action

A Listener Action defines what happens when the user interaction occurs. To add a Listener Action, click the plus icon in the panel below the State Machine Graph. You can add multiple actions to a single listener.

**View Model Change**
Updates values within your View Model Instance. This is the preferred way to communicate from your Rive file to your runtime code. By default, listeners are set to View Model Change, unless an artboard or component instance is the target of the Listener.

* **View Model Drop Down:** The View Model Dropdown lets you select which View Model Property you want the Listener to change. Note that listeners can change the properties of any View Model in the file, even if it isn’t assigned to the Artboard.
* **Value vs Property:** Once you’ve selected which property you’d like the Listener to modify, you can set it to a specific Value or to equal a different view model property.
* **Value:** If you select Value, you can use the input field to change the specific value you’d like the property set to. The value type changes depending on the property.
* **View Model Property:** Selecting a property will set the View Model Property in the listener equal to another. Note that we can set the View Model Property to be equal to itself.


* **Adding a Converter:** If we choose to set a View Model Property equal to another View Model Property, the converter icon appears to the right of the View Model Property. This lets us apply a converter to a property. For example, we can set Number to Number, but attach an add one converter. Every time this listener fires, we can increase our Number property by one.

**Report Event**
Fires an event each time the user action is triggered. This is the default option when an artboard or component instance is the target of a listener.

**Align Target**
The Align Target action positions a target object to follow the pointer when the specified user action occurs within the listener area.

* Use the Target Picker to select the object you want to align.
* Enable **Preserve Offset** to maintain the original distance between the object and the pointer when the action was triggered. When unchecked, the object will align directly to the pointer’s center.

**Input Change**
Allows the listener to change a defined input—such as toggling a boolean, firing a trigger, or setting a number input to a specific value. This is useful for creating interactive behaviors like hover states or click effects directly on the Artboard.

---

### Layers

> Layers let you build more complex logic and animation with the state machine.

A layer on the state machine allows you to play a single animation at a time. For this reason, you can create multiple layers if you wish to mix multiple animations or add additional interactions to a state machine. This example uses layers to mix different background animations, and add multiple interactions onto a single artboard.

* **Creating a new layer:** To create a new layer, use the plus button on the Layers Tab. Notice that each new tab that you create comes with the Default States.
* **Layer Order:** It may not be obvious, but the order of your Layers matter, with Layers to the right taking priority over the Layers to the left. In most cases, this won’t matter, but if your Layers have States that control the same object properties, the animations in the right most layer will take priority over the layers to the left as they mix. You can change the layer order by dragging and dropping your layers around on the Layers tab.
* **Delete layer:** You can delete a layer with right click over the name and selecting the option “Delete Layer”.
* **Duplicate layer:** You can delete a layer with right click over the name and selecting the option “Delete Layer”.
* **Disable and Enable layer:** You can delete a layer with right click over the name and selecting the option “Delete Layer”.
