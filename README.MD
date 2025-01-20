# Liveness Library

The **Liveness Library** provides an easy-to-use solution for integrating liveness detection into Android applications. Users perform interactive challenges like smiling, turning their head, and blinking, ensuring the presence of a live person. The library supports video recording, extensive customization, and event handling.

1. [For Users of Jetpack Compose](#for-users-of-jetpack-compose)
2. [For Users of Fragments](#for-users-of-fragments)
---

## Features

- Perform liveness challenges (e.g., smile, turn head, blink).
- Optional video recording of the liveness process.
- Support for emulators without cameras through mock data.
- Customizable UI colors and texts.
- Event callbacks for handling completion, permissions, and user actions.

---

## Installation

To add the library to your project, include the following dependencies in your `build.gradle` file (replace `<version>` with the latest version):

```kotlin
// Compose Permisisons
val composeBom = platform("androidx.compose:compose-bom:2024.11.00")
implementation(composeBom)
implementation("androidx.compose.material3:material3")
implementation("androidx.compose.ui:ui-tooling-preview")
implementation("androidx.navigation:navigation-compose:2.8.4")
implementation("androidx.constraintlayout:constraintlayout-compose:1.1.0"
debugImplementation("androidx.compose.ui:ui-tooling")

// MLKit
implementation("com.google.android.gms:play-services-mlkit-face-detection:17.1.0"

// CameraX 
val camerax_version = "1.4.0"
implementation("androidx.camera:camera-core:${camerax_version}")
implementation("androidx.camera:camera-camera2:${camerax_version}")
implementation("androidx.camera:camera-lifecycle:${camerax_version}")
implementation("androidx.camera:camera-view:${camerax_version}")

// Liveness
implementation("kz.rbk.liveness:liveness:<version>")
```

## For Users of Jetpack Compose
### Basic Setup
Include the CoreLivenessScreen composable in your UI and configure its parameters as needed:

```kotlin
CoreLivenessScreen(
  isVideoRequired = true,
  isMockData = false,
  isTorchEnabled = true,
  challenges = listOf(
    Challenge.SMILE,
    Challenge.TURN_LEFT
  ),
  onComplete = { images, video ->
    // Handle completion
  },
  onMockData = {
    // Handle mock data
  },
  onShowRationale = {
    // Show rationale dialog
  },
  onNavigateBack = {
    // Handle back navigation
  },
  onTorchToggle = { isTorchOn ->
    // Handle torch toggle
  },
  isIndicatorLoading = false,
  textDataSource = MyTextDataSource(),
  livenessColors = MyLivenessColors()
)
```
### Parameters
CoreLivenessScreen Parameters
- `isVideoRequired (Boolean)`: Determines if video recording is required during the liveness process.
- `isMockData (Boolean)`: Simulates the liveness flow in emulators without a camera. If true, triggers the onMockData callback immediately.
- `isTorchEnabled (Boolean)`: Enables a torch button in dark mode. The user must update livenessColors for torch-related UI changes.
- `challenges (List<Challenge>)`: A list of challenges for the user to perform. Available challenges: `SMILE`, `TURN_LEFT`, `TURN_RIGHT`, `TILT_LEFT`, `TILT_RIGHT`, `BLINK`, `OPEN_MOUTH`, `SAY_WORD`, `UNKNOWN`
- `onComplete ((List<File>, File?) -> Unit)`: Callback triggered upon completing all challenges. Parameters:
  List<File>: Images captured during the challenges.
  File?: Video file (if isVideoRequired is true).
- `onMockData (() -> Unit)`: Callback triggered if isMockData is true.
- `onShowRationale (() -> Unit)`: Callback to handle showing rationale when the user denies camera permission.
- `onNavigateBack (() -> Unit)`: Callback triggered when the user clicks the back button.
- `onTorchToggle ((Boolean) -> Unit)`: Callback triggered when the torch button is toggled. Receives true if the torch is turned on.
- `isIndicatorLoading (Boolean)`: Shows a loading indicator after completing challenges.
- `textDataSource (TextDataSource)`: Provides customizable texts for the UI.
- `livenessColors (LivenessColors)`: Defines custom colors for screen elements.

### Text Customization
You can customize the texts displayed in the UI by implementing the TextDataSource interface.

```kotlin
interface TextDataSource {
  fun textForAlert(alert: LivenessAlert): String?
  fun textForChallenge(challenge: Challenge): String?
  fun descriptionTextForChallenge(challenge: Challenge): String?
  fun textForPassedChallenge(challenge: Challenge): String?
  fun textForLoading(): String?
}
```

Example:
```kotlin
class MyTextDataSource : TextDataSource {
  override fun textForAlert(alert: LivenessAlert) = "Please allow camera access."
  override fun textForChallenge(challenge: Challenge) = "Perform the ${challenge.name} challenge."
  override fun descriptionTextForChallenge(challenge: Challenge) = "Detailed instructions for ${challenge.name}."
  override fun textForPassedChallenge(challenge: Challenge) = "${challenge.name} completed!"
  override fun textForLoading() = "Processing, please wait..."
}
```
### Color Customization
Customize the appearance of the liveness screen using the LivenessColors data class.

```kotlin
data class LivenessColors(
  @ColorInt val bgColor: Int,
  @ColorInt val titleColor: Int,
  @ColorInt val descriptionColor: Int,
  @ColorInt val iconColor: Int,
  @ColorInt val torchColor: Int,
  @ColorInt val torchBackgroundColor: Int,
  @ColorInt val progressColor: Int,
  @ColorInt val progressTrackColor: Int
)
```
Example:

```kotlin
val lightLivenessColors = LivenessColors(
  bgColor = Color.Black,
  titleColor = Color.White,
  descriptionColor = Color.Gray,
  iconColor = Color.White,
  torchColor = Color.Yellow,
  torchBackgroundColor = Color.DarkGray,
  progressColor = Color.Green,
  progressTrackColor = Color.LightGray
)
```
## Example Usage
Here is a complete example setup:

```kotlin
val lightLivenessColors = LightLivenessColors
val darkLivenessColors = DarkLivenessColors
val isTorchEnabled = isInDarkMode()
var isIndicatorLoading by remember { mutableStateOf(false) }
var livenessColors by remember { mutableStateOf(livenessColors) }
CoreLivenessScreen(
  isVideoRequired = true,
  isMockData = false,
  isTorchEnabled = isTorchEnabled,
  challenges = listOf(Challenge.SMILE, Challenge.TURN_LEFT),
  onComplete = { images, video ->
    isIndicatorLoading = true
    // Process images and video
  },
  onMockData = {
    // Handle mock data scenario
  },
  onShowRationale = {
    // Show a custom dialog explaining why camera permission is needed
  },
  onNavigateBack = {
    // Navigate back to the previous screen
  },
  onTorchToggle = { isTorchOn ->
    // Handle torch toggle
    livenessColors = if (isTorchOn) lightLivenessColors else darkLivenessColors
  },
  isIndicatorLoading = isIndicatorLoading,
  textDataSource = MyTextDataSource(),
  livenessColors = livenessColors
)
```

## For Users of Fragments
For users working with Fragments, the library provides a LivenessFragment abstract class. This class internally uses CoreLivenessScreen and offers an easy-to-extend structure.

### Basic Setup
1. Extend the LivenessFragment class.
2. Override the necessary methods to customize behavior.

### Parameters
- `isVideoRequired (Boolean)`: Determines if video recording is required.
- `isMockData (Boolean)`: Simulates the liveness flow without a camera. Calls processMockData immediately if true.
- `isTorchEnabled (Boolean)`: Enables a torch button in dark mode.
- `isIndicatorLoading (StateFlow<Boolean>)`: StateFlow controlling the loading indicator.
- `livenessColors (LivenessColors)`: Colors for normal mode.
- `livenessDarkColors (LivenessColors)`: Colors for dark mode (applied if isTorchEnabled is true).

### Methods to Override
- `onCameraPermissionDenied()`: Called when camera permission is denied. Use this to show a rationale dialog or handle permissions.
- `goBack()`: Called when the back button is clicked.
- `getTextDataSource()`: Return a TextDataSource instance for custom UI texts.
- `generateChallenges()`: Provide a list of challenges for the user.
- `processResults(frames: List<File>, video: File?)`: Handle results after completing the challenges.
- `processMockData()`: Handle mock data scenarios.

### Example Implementation
```kotlin
class MyLivenessFragment : LivenessFragment() {

  protected val _isIndicatorLoading = MutableStateFlow(false)

  override val isVideoRequired = true
  override val isMockData = false
  override val isIndicatorLoading: StateFlow<Boolean> = _isIndicatorLoading.asStateFlow()

  override fun onCameraPermissionDenied() {
    // Show a custom rationale dialog
  }

  override fun goBack() {
    // Navigate to the previous screen
  }

  override val livenessColors by lazy {
    LivenessColors(
      bgColor = Color.WHITE,
      titleColor = Color.BLACK,
      descriptionColor = Color.GRAY,
      progressColor = Color.GREEN,
      progressTrackColor = Color.CYAN,
      torchColor = Color.GRAY,
      torchBackgroundColor = Color.DKGRAY,
      iconColor = Color.BLACK
    )
  }

  override val livenessDarkColors by lazy {
    LivenessColors(
      bgColor = Color.BLACK,
      titleColor = Color.WHITE,
      descriptionColor = Color.GRAY,
      progressColor = Color.GREEN,
      progressTrackColor = Color.CYAN,
      torchColor = Color.GRAY,
      torchBackgroundColor = Color.DKGRAY,
      iconColor = Color.WHITE
    )
  }

  override fun getTextDataSource(): TextDataSource = object : TextDataSource {
    override fun textForAlert(alert: LivenessAlert): String? = "Permission required for camera access."
    override fun textForChallenge(challenge: Challenge): String? = "Perform the ${challenge.name} challenge."
    override fun descriptionTextForChallenge(challenge: Challenge): String? = "Instructions for ${challenge.name}."
    override fun textForPassedChallenge(challenge: Challenge): String? = "${challenge.name} passed."
    override fun textForLoading(): String? = "Processing, please wait..."
  }

  override fun generateChallenges(): List<Challenge> = listOf(
    Challenge.SMILE,
    Challenge.TURN_LEFT,
    Challenge.BLINK
  )

  override fun processResults(frames: List<File>, video: File?) {
    // Handle captured images and video
  }

  override fun processMockData() {
    // Handle mock data scenario
  }
}
```