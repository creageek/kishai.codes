---
title: "Beyond the Straight Line: Crafting Wavy Sliders in Jetpack Compose"
description: Customizing Material3 for Unique Tracks, Thumbs, and Animations
slug: wavy-slider-animation-using-jetpack-compose
date: 2024-12-04 00:00:00+0000
math: true
image: "./cover.png"
# image: wavy-slider-animation-using-jetpack-compose/cover.jpg
---

<p align="center"><img src="./wavy-slider.png" /></p>

## Project Foundations

To begin customizing the Material3 Slider in Jetpack Compose, let's first set up the project and configure the necessary dependencies.

In this section, we’ll configure our project to include the required libraries, starting with the Material3 library for the UI components. We’ll use `libs.versions.toml` for version management, which allows us to keep our dependencies up to date in a central place.

In your `libs.versions.toml` file, add the following:
```toml
[versions]
composeBom = "2024.11.00"

[libraries]
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-material3 = { group = "androidx.compose.material3", name = "material3" }

```
In your `build.gradle.kts`, make sure to include these dependencies:
```toml
dependencies {
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.material3)
    // other dependencies
}
```
This will ensure that you’re using the correct versions of Material3 and Jetpack Compose. With everything set up, you're ready to start customizing the Material3 Slider.

## Exploring the Material3 Slider

In this chapter, we’ll explore the Material3 Slider, a versatile component in Jetpack Compose. This slider is customizable, allowing you to replace the default track and thumb with your own composables.

<p align="center"><img src="./simple-slider.png" /></p>

The basic structure of the `Slider` function looks like this:

```kotlin
fun Slider(
    value: Float,
    onValueChange: (Float) -> Unit,
    onValueChangeFinished: (() -> Unit)? = null,
    ...
    thumb: @Composable (SliderState) -> Unit = {
        SliderDefaults.Thumb(
            interactionSource = interactionSource,
            colors = colors,
            enabled = enabled
        )
    },
    track: @Composable (SliderState) -> Unit = { sliderState ->
        SliderDefaults.Track(
            colors = colors,
            enabled = enabled,
            sliderState = sliderState
        )
    }
)
```
Here, we see that `thumb` and `track` are composable lambdas, which allows us to fully customize the slider’s appearance. You can replace these defaults with any custom composables that fit your design needs.

Let’s begin with a simple modification: using `Canvas` to create a custom track for the slider.

Inside the `Slider` composable, we’ll use the `Canvas` composable to draw a custom track. We will receive a `SliderState` inside the lambda, which contains the current value of the slider (ranging from `0f` to `1f`). Based on this value, we will draw two lines to represent the track.

Here’s how you can draw the first line:
```kotlin
drawLine(
    color = Color.Cyan,
    start = Offset(0f, canvasHeight / 2),
    end = Offset(canvasWidth * state.value, canvasHeight / 2),
    strokeWidth = 4.dp.toPx(),
    cap = StrokeCap.Round
)
```
This will draw the first part of the track up to the current value. Now, for the second part of the track, we’ll do something similar:
```kotlin
drawLine(
    color = Color.Magenta,
    start = Offset(canvasWidth * state.value, canvasHeight / 2),
    end = Offset(canvasWidth, canvasHeight / 2),
    strokeWidth = 4.dp.toPx(),
    cap = StrokeCap.Round
)
```

By combining these two lines, we get a slider with a two-toned track based on the current slider value.

Here’s the full implementation of the custom track:
```kotlin
var sliderPosition by remember { mutableFloatStateOf(0f) }

Slider(
    modifier = Modifier.weight(1f),
    value = sliderPosition,
    onValueChange = {
        sliderPosition = it
    },
    track = { state ->
        Canvas(
            modifier = Modifier
                .fillMaxWidth()
                .height(4.dp)
        ) {
            val canvasWidth = size.width
            val canvasHeight = size.height

            // Draw the first segment of the track (cyan)
            drawLine(
                color = Color.Cyan,
                start = Offset(0f, canvasHeight / 2),
                end = Offset((canvasWidth * state.value), canvasHeight / 2),
                strokeWidth = 4.dp.toPx(),
                cap = StrokeCap.Round
            )

            // Draw the second segment of the track (magenta)
            drawLine(
                color = Color.Magenta,
                start = Offset((canvasWidth * state.value), canvasHeight / 2),
                end = Offset((canvasWidth), canvasHeight / 2),
                strokeWidth = 4.dp.toPx(),
                cap = StrokeCap.Round
            )
        }
    },
)
```

The end result is a slider with a visually distinct track that dynamically updates as the slider value changes.

Here’s how it would look in action:
[Insert image of the custom slider with a cyan and magenta track]

## The Mathematics of Waves

<p align="center"><img src="./wave-mathematics.png" /></p>

Let's break down the mathematics behind the wavy track in our custom slider. The key here is using a sinusoidal function to create a smooth, oscillating path. By understanding the math, we can tweak the wave’s appearance and behavior to our liking.


To create a wavy path, we use a sine wave, which is a periodic oscillation described by the sine function:

$$y(t) = A \cdot \sin(k \cdot t + \phi)$$

Where: 
- $A$ is the amplitude (height) of the wave.
- $k$ is the wave number, which is related to the wavelength.
- $\phi$ is the phase shift, which allows us to offset the wave.
- $t$ is the horizontal position.

In our case, we are using this formula to calculate the vertical position of each point on the track, based on its horizontal position $x$ (in pixels).

Let’s go step by step.

The amplitude controls the vertical height of the wave. In our code, we define it as:
```kotlin 
val amplitude = 3.dp.toPx()  // Wave height
```
This means the wave will oscillate up and down by `3.dp` in pixel units.

The wavelength is the length of one full cycle of the wave, i.e., the distance between two consecutive peaks (or troughs). In our code, it’s defined as:
```kotlin
val wavelength = canvasWidth / 10  // Length of one wave cycle
```
This means the wave will repeat every `canvasWidth / 10` units, which will determine how many waves fit in the slider track.

For each horizontal position $x$, we calculate the corresponding vertical position $y$ using the sine function:
```kotlin
val angle = ((x / waveLength) * 2 * PI)
val y = (canvasHeight / 2) + amplitude * sin(angle).toFloat()
```
- The angle for the sine function is determined by the horizontal position $x$. The term $(x/waveLength)$ normalizes the position along the wave’s cycle, and multiplying by 
$2π$ (the full period of a sine wave) ensures the wave repeats every full cycle.
- The vertical position $y$ is the middle of the canvas (height divided by 2) plus the sine value, scaled by the amplitude. This causes the wave to oscillate above and below the center of the track.

## Crafting a Wavy Track

With the mathematical foundation in place, it's time to craft our wavy slider track.
The core of our wavy track lies in the `Path` object, where we translate the sine wave math from the previous chapter into drawable instructions. Here's how we create the path:
```kotlin
val amplitude = 3.dp.toPx()  // Wave height
val wavelength = canvasWidth / 10  // Length of one wave cycle

val path = Path().apply {
    for (x in 0..(canvasWidth * state.value).toInt()) {
        val angle = ((x / wavelength) * 2 * PI)
        val y = (canvasHeight / 2) + amplitude * sin(angle).toFloat()
        if (x == 0) moveTo(x.toFloat(), y) else lineTo(x.toFloat(), y)
    }
}

```

The amplitude sets the height of the wave, while the wavelength determines how frequently it oscillates across the slider track. As the slider moves, the wave's length dynamically adapts to the slider's current value (`state.value`), ensuring a seamless transition. The path is formed by beginning at an initial point with `moveTo` and smoothly connecting each subsequent point using `lineTo`, resulting in a continuous and flowing wave.

Now that we have the path, let’s integrate it into the slider track using a `Canvas`:
```kotlin
Canvas(
    modifier = Modifier
        .fillMaxWidth()
        .height(8.dp)  // Height of the track
) {
    val canvasWidth = size.width
    val canvasHeight = size.height

    // Draw the wavy segment of the track
    drawPath(
        path = path,
        color = Color.Cyan,
        style = Stroke(width = 4.dp.toPx(), cap = StrokeCap.Round)
    )
    
    // Draw the straight segment of the track
    drawLine(
        color = Color.Magenta,
        start = Offset((canvasWidth * state.value), canvasHeight / 2),
        end = Offset((canvasWidth), canvasHeight / 2),
        strokeWidth = 4.dp.toPx(),
        cap = StrokeCap.Round
    )
}
```

## Customizing the Slider Thumb

While the wavy track adds a dynamic and eye-catching visual to our slider, the thumb—the draggable handle—offers another opportunity for customization.

The Material3 `Slider` provides a `thumb` parameter, which allows you to define a custom composable for the thumb. The composable receives the slider's current state (`SliderState`) and can dynamically adjust its appearance based on the 
slider’s position.

When customizing the slider, sometimes less is more. While there are plenty of creative ways to design a slider thumb, a simple, elegant approach can often be just as effective.

Here’s the implementation:
```kotlin
Slider(
    value = sliderPosition,
    onValueChange = { sliderPosition = it },
    thumb = { state ->
        Box(
            modifier = Modifier
                .background(
                    color = MaterialTheme.colorScheme.primary,
                    shape = CircleShape
                )
                .size(24.dp)
        )
    }
)
```
This design keeps the focus on the wavy track while ensuring the thumb is easy to spot and interact with. The simplicity also makes it adaptable to various themes and use cases without overwhelming the overall slider design.
## Animating the Wave
A wavy slider track is already visually striking, but adding animation can elevate the experience, creating a more dynamic and engaging interaction.

To animate the wave, we’ll use the `rememberInfiniteTransition` API to create a smoothly varying offset. This offset will shift the wave horizontally, simulating the effect of a wave flowing endlessly along the slider track.

Here’s how we can implement it:
```kotlin
val infiniteTransition = rememberInfiniteTransition()
val waveOffset by infiniteTransition.animateFloat(
    initialValue = 0f,
    targetValue = 2 * Math.PI.toFloat(), // One full wave cycle
    animationSpec = infiniteRepeatable(
        animation = tween(
            durationMillis = 2000, // Adjust speed
            easing = LinearEasing
        ),
        repeatMode = RepeatMode.Restart
    )
)
```
The `waveOffset` value will be used in the mathematical calculation of the wave's path to create the illusion of motion.

To integrate the animation into the wave path, we’ll modify the existing path logic to factor in the `waveOffset` value:
```kotlin
val path = Path().apply {
    for (x in 0..(canvasWidth * state.value).toInt()) {
        val angle = ((x / wavelength) * 2 * Math.PI) + waveOffset
        val y = (canvasHeight / 2) + amplitude * sin(angle).toFloat()
        if (x == 0) moveTo(x.toFloat(), y) else lineTo(x.toFloat(), y)
    }
}
```
The `waveOffset` ensures that the wave appears to be moving while maintaining its shape.

Here’s the complete code for the animated wave:
```kotlin
val amplitude = 8.dp.toPx()
val waveLength = size.width / 10
val infiniteTransition = rememberInfiniteTransition()
val waveOffset by infiniteTransition.animateFloat(
    initialValue = 0f,
    targetValue = 2 * Math.PI.toFloat(),
    animationSpec = infiniteRepeatable(
        animation = tween(
            durationMillis = 2000,
            easing = LinearEasing
        ),
        repeatMode = RepeatMode.Restart
    )
)

Slider(
    value = sliderPosition,
    onValueChange = { sliderPosition = it },
    track = { state ->
        Canvas(modifier = Modifier.fillMaxWidth().height(24.dp)) {
            val path = Path().apply {
                for (x in 0..(size.width * state.value).toInt()) {
                    val angle = ((x / waveLength) * 2 * Math.PI) + waveOffset
                    val y = (size.height / 2) + amplitude * sin(angle).toFloat()
                    if (x == 0) moveTo(x.toFloat(), y) else lineTo(x.toFloat(), y)
                }
            }

            drawPath(
                path = path,
                color = Color.Cyan,
                style = Stroke(width = 4.dp.toPx(), cap = StrokeCap.Round)
            )
            
            drawLine(
                color = Color.Magenta,
                start = Offset((canvasWidth * state.value), canvasHeight / 2),
                end = Offset((canvasWidth), canvasHeight / 2),
                strokeWidth = 4.dp.toPx(),
                cap = StrokeCap.Round
            )
        }
    }
)
```
The parameters of the animation can be adjusted to fine-tune the effect:
- `durationMillis`: Controls the speed of the wave. Decrease for faster motion or increase for slower motion.
- `amplitude`: Adjusts the wave’s height for more dramatic or subtle movement.
- `waveLength`: Changes the frequency of the wave cycles.


## Final Thoughts and Next Steps
Congratulations! You’ve built a fully functional, visually engaging slider using Jetpack Compose, complete with a custom wavy track, animated motion, and a minimalist thumb. 

Your slider is already a great starting point, but here are some ways to make it even more unique:
- Expose `amplitude`, `waveLength`, and `durationMillis` as parameters to allow runtime adjustments.
- Add subtle animations, such as scaling or color changes, to the thumb when interacting with it.
- Replace the sine wave with other patterns (e.g., square waves or custom paths) to suit your app’s purpose.

Animating and drawing custom components can impact performance, especially on low-end devices. To optimize:
- Use `Int` for efficient calculations where possible to reduce 
- Ensure the `Canvas` only occupies the required space.
- Avoid unnecessary recompositions by using proper state handling with `remember`.

