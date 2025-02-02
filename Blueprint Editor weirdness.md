2021-06-21_19:13:03

# Blueprint Editor weirdness

Editing components inside the Blueprint Editor is weird.

Read `FComponentVisualizer::NotifyPropertiesModified` in `ComponentVisualizer.cpp` line 237 in Unreal Engine 4.25.

When editing a Blueprint Actor class in the Blueprint Editor there are multiple representations of the class simultaneously.
When selecting one of the Components in there are multiple representations of that as well.
The Details Panel owns one of them. That Component instance is named `MyVariable_GEN_VARIABLE` and does not have an owning Actor.
The Viewport owns the other. That Compnent instance is named `MyVariable` and is owned by an Actor named `MyBlueprint_C_0`.
I guess that in addition to these two there are also Blueprint-specific types representing the same thing, such as `UBlueprint` and `USCSNode`.
- Details Panel: `MyVariable_GEN_VARIABLE` in owner `nullptr`.
- Visualizer: `MyVariable` in owner `MyBlueprint_C_0`.

If you have a Component Visualizer with Hit Proxies then the Component owned by the `_C_0` Actor will be provided.
If you ask ask Detail Layout Builder for the customized object then it will return the `GEN_VARIABLE` one without an owning Actor.

There can be multiple Details Panel open at the same time, but only a single Component Visualizer.
Each frame all Details Panels have their `Tick` function called.
If they get the Component Visualizer for their Component then they will all get the same instance even though they may be the Details Panel for different Component instances.
You can test this with the Spline Component. Create a Blueprint Actor with a Spline in it and an Empty Actor in the level also with a Spline. Select the Spline in the Empty Actor and keep an eye on the Selected Points category in the Details Panel. Then open the Blueprint in a separate window next to the main Unreal Editor window, select one of the Spline nodes and move it around. You will see the Blueprint's Spline's data in the Selected Points category of the Details Panel in the main Unreal Editor window. All other categories will display the state of the Empty Actor Spline, but that one Category will display the state of the Blueprint-owned Spline. That's gonna lead to confusion.

If using the click-to-track approach in the Component Visualizer, like the Spline does, then when the Details Panel is first opened nothing will have been clicked yet so the Component Visualizer will have a null Component.

It may be possible to track multiple selected Components in the Component Visualizer and do some kind of look-up in the Detail Panel's `Tick` function so that the correct Component is always used.
Not sure how one would know that it's safe to remove stuff from that map/cache.

[[2020-08-06_18:48:41]] [Component visualizer](./Component%20visualizer.md)  