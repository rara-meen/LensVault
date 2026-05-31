# LensVault

A professional Android gallery application built with Kotlin, following a clean Xiaomi-inspired design language. Features adaptive grid layouts, a smooth fullscreen image slider with pinch-to-zoom, light/dark theme support, runtime permission handling, and camera integration.

---

## Screenshots

> Run the app on a device or emulator to see the UI in action. The app adapts to light and dark system themes automatically.

---

## Features

- **Permission-first onboarding** — Dedicated permission screen at launch with rationale dialogs, graceful handling for permanent denial, and a direct link to app settings
- **Photo grid** — Responsive grid (2, 3, or 4 columns) with smooth thumbnail loading via Glide
- **Albums view** — Grouped by media bucket (Camera, WhatsApp, Screenshots, etc.) in a two-column card grid
- **Fullscreen image slider** — ViewPager2-based swipe viewer with pinch-to-zoom (up to 5x), double-tap zoom, and drag-to-pan
- **Immersive viewer UI** — Tap to hide/show top toolbar and bottom action bar with fade animation; full edge-to-edge rendering
- **Camera integration** — FAB launches the device camera app; captured photos are saved via FileProvider and the gallery refreshes automatically
- **Share** — Native Android share sheet for sharing any photo
- **Light and dark mode** — Full Material You theme support using `values/themes.xml` and `values-night/themes.xml`
- **All screen sizes** — ConstraintLayout and CoordinatorLayout ensure correct rendering on phones, foldables, and tablets

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| UI | XML Layouts, Material 3 Components |
| Architecture | MVVM (ViewModel + LiveData) |
| Image loading | Glide 4.16 |
| Async | Kotlin Coroutines |
| Navigation | Manual (Activity/Intent-based, no nav graph needed) |
| Min SDK | 26 (Android 8.0) |
| Target SDK | 34 (Android 14) |

---

## Project Structure

```
LensVault/
├── app/
│   ├── build.gradle
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/lensvault/app/
│       │   ├── adapters/
│       │   │   ├── AlbumAdapter.kt          # Album grid RecyclerView adapter
│       │   │   ├── ImagePagerAdapter.kt     # ViewPager2 adapter with pinch-zoom
│       │   │   └── PhotoGridAdapter.kt      # Photo grid RecyclerView adapter
│       │   ├── models/
│       │   │   └── MediaItem.kt             # MediaItem and MediaBucket data classes
│       │   ├── ui/
│       │   │   ├── gallery/
│       │   │   │   └── GalleryActivity.kt   # Main gallery screen
│       │   │   ├── permissions/
│       │   │   │   └── PermissionActivity.kt # Launcher / permission screen
│       │   │   └── viewer/
│       │   │       └── ImageViewerActivity.kt # Fullscreen swipeable viewer
│       │   ├── utils/
│       │   │   └── MediaRepository.kt       # ContentResolver media queries
│       │   └── viewmodels/
│       │       └── GalleryViewModel.kt      # State management, ViewModel
│       └── res/
│           ├── drawable/                    # Vector icons + gradient drawables
│           ├── font/                        # Inter font files (add manually)
│           ├── layout/                      # XML layout files
│           ├── menu/                        # gallery_menu.xml
│           ├── values/                      # strings, colors, themes (light)
│           ├── values-night/               # themes (dark)
│           └── xml/
│               └── file_paths.xml          # FileProvider paths
├── build.gradle
├── settings.gradle
└── gradle.properties
```

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/your-username/LensVault.git
cd LensVault
```

### 2. Open in Android Studio

Open Android Studio Hedgehog (2023.1.1) or later and select `Open` → choose the `LensVault` folder.

### 3. Add Inter fonts

Download the **Inter** font from [Google Fonts](https://fonts.google.com/specimen/Inter) and place these three files in `app/src/main/res/font/`:

```
inter_regular.ttf    (Regular 400)
inter_medium.ttf     (Medium 500)
inter_bold.ttf       (Bold 700)
```

If you prefer a different font, update the `@font/` references in the layout XML files and `themes.xml`.

### 4. Sync and Run

Click **Sync Now** when Android Studio prompts for Gradle sync, then run on a physical device or emulator (API 26+).

---

## Permissions

| Permission | When requested | Purpose |
|---|---|---|
| `READ_MEDIA_IMAGES` | App launch (API 33+) | Read photos from device storage |
| `READ_EXTERNAL_STORAGE` | App launch (API 26–32) | Read photos from device storage |
| `CAMERA` | On camera FAB tap | Launch camera to take a new photo |

The app uses a dedicated `PermissionActivity` as the launcher. It checks permissions at startup and routes to `GalleryActivity` only when access is granted. If permission is permanently denied, the user is directed to system App Settings.

### Permission flow

```
Launch → PermissionActivity
    ├── Permission already granted → GalleryActivity
    ├── Not yet asked → Show rationale → Request → GalleryActivity
    ├── Denied (soft) → Show Try Again button
    └── Denied (permanent) → Show Open Settings dialog
```

---

## Camera Integration

Tapping the FAB launches the system camera app via `MediaStore.ACTION_IMAGE_CAPTURE`. The photo is saved to a temporary file in the app's external files directory using `FileProvider` (so the camera app can write to it without needing broad storage permission). On return, `GalleryViewModel.refreshMedia()` reloads the gallery.

```kotlin
// GalleryActivity.kt
private fun launchCamera() {
    val photoFile = createImageFile()
    currentCameraImageUri = FileProvider.getUriForFile(
        this, "${packageName}.fileprovider", photoFile
    )
    val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE).apply {
        putExtra(MediaStore.EXTRA_OUTPUT, currentCameraImageUri)
    }
    if (intent.resolveActivity(packageManager) != null) {
        cameraLauncher.launch(intent)
    }
}
```

---

## Image Slider

The viewer uses `ViewPager2` with a custom `ImagePagerAdapter`. Each page handles its own touch events:

- **Single tap** — toggle toolbar/bottom bar visibility (fade in/out)
- **Double tap** — zoom in to 2.5x / reset to 1x
- **Pinch gesture** — smooth zoom from 1x to 5x via `ScaleGestureDetector`
- **Drag (when zoomed)** — pan the image while zoomed in

```kotlin
// ImagePagerAdapter.kt — pinch zoom
val scaleDetector = ScaleGestureDetector(context,
    object : ScaleGestureDetector.SimpleOnScaleGestureListener() {
        override fun onScale(detector: ScaleGestureDetector): Boolean {
            scaleFactor = (scaleFactor * detector.scaleFactor).coerceIn(1f, 5f)
            applyTransform()
            return true
        }
    }
)
```

---

## Light and Dark Mode

Themes are defined in `res/values/themes.xml` (light) and `res/values-night/themes.xml` (dark). The app follows the system setting automatically — no manual toggle is needed.

The viewer activity uses `Theme.LensVault.Immersive`, which forces a black background and hides the status bar for a distraction-free experience regardless of system theme.

---

## Grid Columns

Use the overflow menu in the gallery toolbar to switch between 2, 3, or 4 column grids. The change is immediate and persists for the current session.

---

## Dependencies

```groovy
// Image loading
implementation 'com.github.bumptech.glide:glide:4.16.0'

// Material 3 components
implementation 'com.google.android.material:material:1.11.0'

// ViewPager2 for image slider
implementation 'androidx.viewpager2:viewpager2:1.0.0'

// ViewModel + LiveData
implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0'
implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.7.0'

// Activity result APIs
implementation 'androidx.activity:activity-ktx:1.8.2'
```

---

## Known Limitations

- Video files in media buckets are displayed as thumbnails but tapping them opens the image viewer (video playback is not implemented)
- The pinch-to-zoom implementation uses `ImageView` scale transforms; for production, consider a library like [PhotoView](https://github.com/Baseflow/PhotoView) for boundary-clamped zoom with better fling physics
- Photos taken by the camera are saved to the app's private external directory; they will not appear in other gallery apps unless you add `MediaScannerConnection.scanFile()`

---

## License

```
MIT License

Copyright (c) 2024

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
