# Photo Booth

A C++20/OpenCV photo booth application for IMGS.361 Image Processing.

The project keeps the original semester-project architecture: camera acquisition and configuration remain separate from the reusable image-processing functions, while `apps/photo_booth.cpp` owns the interactive runtime state and processing order.

## Final processing capabilities

The project includes the following grouped capabilities. A group counts as one substantive capability; its required algorithms are implemented together.

1. **Contrast enhancement:** percentile-based dynamic stretch, linear contrast stretch, luminance histogram equalization, and per-channel histogram matching against a captured reference frame.
2. **Geometric transformations:** arbitrary rotation, affine transformation, perspective transformation, and log-polar mapping.
3. **Spatial filtering:** box/mean, Gaussian, median, bilateral, and sharpening presets, plus a user-entered 3x3 kernel.
4. **Frequency-domain filtering:** ideal, Butterworth, and Gaussian lowpass/highpass filtering.
5. **Data embedding:** embeds and recovers a built-in camera icon using Fourier phase dispersion.
6. **Corner detection:** Harris and FAST corner detection.
7. **Classification:** HSV histogram feature vectors, minimum-distance-to-mean classification, and cosine-similarity classification. Class 1 and class 2 reference samples are captured interactively.

Existing channel swapping, grayscale, inversion, quantization, Canny edge detection, and capture behavior are retained.

## Additional controls

| Key | Function |
| --- | --- |
| `V` | Cycle contrast modes: percentile stretch, linear, equalization, histogram match |
| `M` | Capture current processed frame as histogram-matching reference |
| `J` | Cycle geometry: rotation, affine, perspective, log-polar |
| `T` | Cycle five spatial presets |
| `6` | Enter nine values for a custom 3x3 spatial kernel in the console |
| `Y` | Toggle frequency filtering |
| `U` | Cycle ideal, Butterworth, Gaussian frequency shapes |
| `O` | Toggle lowpass/highpass |
| `1` / `2` | Save current frame as a labeled classifier training sample |
| `K` | Run minimum-distance and cosine classifiers and print their scores |

For geometric transforms, enable `R` as well as selecting the mode with `J`. The histogram-matching reference is captured with `M`. Classifier predictions require labeled samples from both classes.

## Image representation

Successful camera frames use OpenCV's `CV_8UC3` BGR representation.

Processing functions preserve that representation. In particular, grayscale and edge detection internally use a single-channel image but convert their results back to BGR before returning. This allows the processing operations to be composed without changing the pipeline's image type.

## Controls

Run the application and use the following keys while the preview window has focus:

| Key | Function |
| --- | --- |
| `C` | Toggle dynamic contrast |
| `I` | Toggle inversion |
| `N` | Toggle quantization |
| `G` | Toggle grayscale |
| `B` | Toggle Gaussian blur |
| `E` | Toggle Canny edge detection |
| `H` | Enable / disable Harris corner detection (switches off FAST) |
| `F` | Enable / disable FAST corner detection (switches off Harris) |
| `X` | Turn off corner detection |
| `D` | Toggle Fourier phase-dispersion icon embedding |
| `R` | Toggle rotation |
| `[` / `]` | Decrease / increase quantization levels |
| `-` / `+` | Decrease / increase blur kernel size |
| `,` / `.` | Decrease / increase rotation angle |
| `SPACE` | Start a 3-second capture countdown |
| `Q` or `Esc` | Quit |

Quantization levels range from 2 through 256. Blur kernel sizes range from 3 through 31 and remain odd. Rotation ranges from -180 through +180 degrees in five-degree steps.

## Capture

Press `SPACE` to capture the currently processed image.

The application:

1. Freezes the current processed frame.
2. Displays a three-second countdown.
3. Saves the processed image.
4. Automatically chooses the next unused filename.

With the default configuration, captures are saved as:

```text
captured_image_001.png
captured_image_002.png
captured_image_003.png
```

If a directory is included in `capture.output_filename`, the directory is created automatically when needed.

## Project organization

```text
Photo-Booth-Final/
├── CMakeLists.txt
├── config.toml
├── LICENSE
├── README.md
├── apps/
│   ├── capture_single_image.cpp
│   ├── live_preview.cpp
│   └── photo_booth.cpp
├── include/
│   └── photo_booth/
│       ├── AppConfig.hpp
│       ├── ImageCapture.hpp
│       └── ImageProcessing.hpp
├── src/
│   ├── AppConfig.cpp
│   ├── ImageCapture.cpp
│   └── ImageProcessing.cpp
└── tools/
```

The Week 15 work is intentionally concentrated in:

```text
apps/photo_booth.cpp
include/photo_booth/ImageProcessing.hpp
src/ImageProcessing.cpp
config.toml
README.md
```

No new framework or processing class hierarchy is required. Processing algorithms remain ordinary functions and `processFrame()` continues to determine the pipeline order.

## Requirements

- CMake 3.30 or later
- C++20 compiler
- OpenCV 4.x or 5.x with `core`, `videoio`, `highgui`, `imgcodecs`, and `imgproc`
- Boost with the `program_options` component
- Eigen3
- toml++ 3.4.0

The existing CMake configuration downloads toml++ with `FetchContent`.

## Build from a clean clone

From the project root:

```bash
cmake -S . -B build
cmake --build build
```

The executables are placed in:

```text
build/bin/
├── capture_single_image
├── live_preview
└── photo_booth
```

For a clean-clone verification, remove the entire `build` directory before running the commands above.

## Run the photo booth

```bash
./build/bin/photo_booth
```

Or provide a different TOML configuration:

```bash
./build/bin/photo_booth webcam.toml
```

The default configuration is `config.toml` in the current working directory.

## Other applications

Minimal live preview:

```bash
./build/bin/live_preview
```

Single-image diagnostic utility:

```bash
./build/bin/capture_single_image
```

All applications also support:

```bash
./build/bin/photo_booth --help
```

## Configuration

The supplied `config.toml` contains camera, preview, capture, and processing settings.

The processing effects added for the final application are intentionally runtime-controlled rather than adding a large configuration system. This keeps the final architecture close to the semester's existing design while allowing the user to combine processing capabilities interactively.

## Design notes

The application uses an explicit processing pipeline rather than a general-purpose processing framework.

`ImageProcessing.cpp` contains reusable image-processing algorithms.

`photo_booth.cpp` contains:

- `ProcessingState` for runtime effect state.
- `processFrame()` for pipeline order.
- `showPreviewFrame()` for the interactive display.
- `handleKey()` for keyboard controls.
- Capture/countdown behavior.

This preserves the architec
