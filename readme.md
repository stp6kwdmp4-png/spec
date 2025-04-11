# Simple Color Palette Spec v0.1

> A minimal JSON-based file format for defining color palettes

It uses [extended linear sRGB](#what-is-extended-linear-srgb) with optional opacity.

[Learn more ›](https://simplecolorpalette.com)

[**Feedback wanted!**](https://github.com/simple-color-palette/spec/issues)

> [!CAUTION]
> Work in progress.

## Links

- [JSON Schema](schema.json) *(v0.1)*

## JSON Structure

```json
{
	"name": "Favorites",
	"colors": [
		{
			"name": "Hot Pink",
			"components": [1, 0.1274, 0.418]
		},
		{
			"components": [0.592, 0.278, 0.996, 0.9]
		}
	]
}
```

- Palette name is optional.
- Color name is optional.
- Opacity is optional.

## Details

### Top-level Object

- `name`: Optional. String. Name of the palette. If omitted, use the filename without extension.
- `colors`: Required. Array of color entries.

### Color Entry

Each color is an object with the following fields:

- `name`: Optional. String.
- `components`: Required. Array of 3 or 4 floating-point numbers.
	- `[red, green, blue]` or `[red, green, blue, opacity]`
	- The color should use extended linear sRGB color space.
	- The color components can be negative and more than 1.
	- The opacity defaults to 1 if omitted.
	- The opacity should be clamped to `0...1` range when reading and writing the palette. It should not throw if outside the range.

### Numeric Precision

All numeric values in the color components array must be stored with a maximum of 4 decimal places, using banker's rounding (round-half-to-even) when needed. For example, 0.12345 becomes 0.1235, while 0.12350 rounds to 0.1235 (nearest even). This applies to both RGB components and opacity.

#### Why 4 Decimal Places?

- Provides up to 10000 distinct values per component (0 to 10000).
- Exceeds human color perception capabilities.
- Matches or exceeds most display technology precision.
- Sufficient for HDR and professional color work.
- Enables reliable color deduplication and comparison.
- Preserves color identity through format conversions and rounding errors.

#### Implementation Considerations

- Use banker's rounding to prevent statistical bias.
- Perform calculations at full precision.
- Apply rounding only for final storage or display.
- Validate that stored values never exceed 4 decimal places.

#### Example (Swift)

```swift
extension Double {
	/**
	Rounds the number to specified decimal places using banker's rounding.

	- Parameter places: Number of decimal places (must be >= 0).
	- Returns: The rounded number.
	*/
	func rounded(toPlaces places: Int) -> Self {
		guard places >= 0 else {
			return self
		}

		let multiplier = pow(10.0, Self(places))
		return (self * multiplier).rounded(.toNearestOrEven) / multiplier
	}
}

let number = 3.14159265359
number.rounded(toPlaces: 4)
//=> 3.1416
```

## Format

### File Extension

`.color-palette`

### MIME Type

```
application/x.sindresorhus.simple-color-palette+json
```

*(temporary)*

### Uniform Type Identifier

```
com.sindresorhus.simple-color-palette
```

#### Swift example

```swift
extension UTType {
	static var simpleColorPalette: Self { .init(importedAs: "com.sindresorhus.simple-color-palette", conformingTo: .json) }
}
```

#### `Info.plist` example

```xml
<key>UTImportedTypeDeclarations</key>
<array>
	<dict>
		<key>UTTypeIdentifier</key>
		<string>com.sindresorhus.simple-color-palette</string>
		<key>UTTypeDescription</key>
		<string>Simple Color Palette</string>
		<key>UTTypeConformsTo</key>
		<array>
			<string>public.json</string>
		</array>
		<key>UTTypeTagSpecification</key>
		<dict>
			<key>public.filename-extension</key>
			<array>
				<string>color-palette</string>
			</array>
			<key>public.mime-type</key>
			<string>application/x.sindresorhus.simple-color-palette+json</string>
		</dict>
	</dict>
</array>
<key>CFBundleDocumentTypes</key>
<array>
	<dict>
		<key>CFBundleTypeName</key>
		<string>Simple Color Palette</string>
		<key>LSItemContentTypes</key>
		<array>
			<string>com.sindresorhus.simple-color-palette</string>
		</array>
		<key>CFBundleTypeExtensions</key>
		<array>
			<string>color-palette</string>
		</array>
		<key>CFBundleTypeRole</key>
		<string>Viewer</string>
		<key>LSHandlerRank</key>
		<string>Alternate</string>
	</dict>
</array>
```

## FAQ

### Why JSON?

- Human-readable.
- Ubiquitous and supported everywhere.
- Easy to parse in any language.

### Why is the color space hard-coded?

- Simplicity and clarity - no ambiguity about color interpretation.
- Avoiding the complexity of color space conversion and management.
- Color space conversion is better handled at the app level when needed.

### What is “extended linear sRGB”?

The same color space as [sRGB](https://en.wikipedia.org/wiki/SRGB) but with two differences:
1. Extended: Values can go beyond 0-1 for HDR colors.
1. Linear: Raw RGB values without [gamma correction](https://en.wikipedia.org/wiki/Gamma_correction).

### What is gamma correction?

Standard sRGB uses a non-linear encoding to match human perception. This format uses raw linear values instead - better for color computations but requires conversion (gamma correction) for display.

### Why extended sRGB?

- Allows values outside 0-1 range for HDR and wide-gamut colors.
- Simple extension of familiar sRGB.
- Easy to clamp to standard sRGB when needed.

### Why linear color?

- Eliminates gamma correction ambiguity.
- More accurate color blending and interpolation.

### Why not Display P3 or other color spaces?

No benefit over extended sRGB since both can represent the same range of colors.

### Why no CMYK support?

CMYK is output-dependent and requires device-specific ICC profiles. This format is for authoring and sharing colors, not managing print workflows.

### Does the format support gradients?

No, only flat colors are supported.
