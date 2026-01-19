# E-Ink Display Simulator

A web-based simulator for e-ink displays with pixel-perfect Adafruit GFX font rendering. Designed for rapid UI prototyping without flashing hardware.

## Features

- **Pixel-perfect rendering** - Uses actual bitmap font data extracted from Adafruit GFX headers
- **Full drawing primitives** - Lines, rectangles, circles, triangles, rounded rectangles, XBitmap images
- **GFX-compatible API** - Mirrors Adafruit GFX library function signatures
- **TypeScript support** - Full type definitions for IDE autocomplete and type safety
- **Export options** - Download PNG or copy to clipboard
- **Scalable display** - 1x to 4x zoom with pixel-perfect scaling
- **Zero runtime dependencies** - Pure vanilla JavaScript

## Installation

```bash
npm install eink-simulator
```

## Quick Start

### Browser (Script Tag)

```html
<canvas id="display" width="296" height="128"></canvas>
<script src="node_modules/eink-simulator/dist/eink-simulator.bundle.js"></script>
<script>
  const { EinkSimulator } = window.EinkSimulator;

  const canvas = document.getElementById('display');
  const sim = new EinkSimulator(canvas, { width: 296, height: 128 });

  sim.clear();
  sim.fillRect(10, 10, 50, 30);
  sim.drawCircle(100, 50, 20);
  sim.drawText('Hello World', 150, 50);  // Uses built-in 5x7 font
</script>
```

### ES Modules

```typescript
import { EinkSimulator } from 'eink-simulator';

const canvas = document.getElementById('display') as HTMLCanvasElement;
const sim = new EinkSimulator(canvas, { width: 296, height: 128 });

sim.clear();
sim.fillRect(10, 10, 50, 30);
sim.drawText('Hello', 10, 50);
```

### Using Custom Fonts

The simulator includes a built-in 5x7 monospace font. For custom Adafruit GFX fonts, you need to extract font data from C header files.

```bash
# Extract fonts from Adafruit GFX headers
npx extract-fonts \
  --fonts-dir ./path/to/Adafruit_GFX_Library/Fonts \
  --fonts "FreeSans9pt7b,FreeMono12pt7b" \
  --output ./src/generated/font-data.ts
```

Then import and use the fonts:

```typescript
import { EinkSimulator } from 'eink-simulator';
import { FreeSans9pt7b } from './generated/font-data.js';

const sim = new EinkSimulator(canvas, { width: 296, height: 128 });
sim.drawText('Custom Font', 10, 50, FreeSans9pt7b);
```

## Font Extraction

### Using a Config File

Create `fonts.config.json`:

```json
{
  "fontsDir": "./path/to/Adafruit_GFX_Library/Fonts",
  "customFontsDir": "./src/fonts",
  "fonts": [
    "FreeSans9pt7b",
    "FreeSansBold12pt7b",
    "FreeMono9pt7b"
  ],
  "output": "./src/generated/font-data.ts"
}
```

Run the extractor:

```bash
npx extract-fonts --config fonts.config.json
```

### CLI Options

```
npx extract-fonts [options]

Options:
  --config       Path to JSON config file
  --fonts-dir    Directory containing Adafruit GFX font headers
  --custom-dir   Directory containing custom font headers (optional)
  --fonts        Comma-separated list of font names (without .h extension)
  --output       Output file path (default: stdout)
  --help, -h     Show help message
```

### Converting TTF/OTF to Adafruit GFX Format

Use the Adafruit GFX `fontconvert` tool to convert TrueType fonts:

```bash
# Build fontconvert (requires FreeType)
cd Adafruit_GFX_Library/fontconvert
make

# Convert a font
./fontconvert MyFont.ttf 12 > MyFont12pt7b.h
```

## API Reference

### EinkSimulator Class

```typescript
const sim = new EinkSimulator(canvas, {
  width: 296,      // Display width in pixels
  height: 128,     // Display height in pixels
  foreground: '#000000',  // Black (default)
  background: '#FFFFFF',  // White (default)
});
```

### Drawing Methods

| Method | Description |
|--------|-------------|
| `clear()` | Clear display with background color |
| `drawPixel(x, y, color?)` | Draw a single pixel |
| `drawLine(x0, y0, x1, y1, color?)` | Draw a line |
| `drawHLine(x, y, length, color?)` | Draw horizontal line |
| `drawVLine(x, y, length, color?)` | Draw vertical line |
| `drawRect(x, y, w, h, color?)` | Draw rectangle outline |
| `fillRect(x, y, w, h, color?)` | Fill rectangle |
| `drawCircle(x, y, r, color?)` | Draw circle outline |
| `fillCircle(x, y, r, color?)` | Fill circle |
| `drawTriangle(x0, y0, x1, y1, x2, y2, color?)` | Draw triangle outline |
| `fillTriangle(x0, y0, x1, y1, x2, y2, color?)` | Fill triangle |
| `drawRoundRect(x, y, w, h, r, color?)` | Draw rounded rectangle outline |
| `fillRoundRect(x, y, w, h, r, color?)` | Fill rounded rectangle |
| `drawXBitmap(x, y, bitmap, w, h, color?)` | Draw XBitmap image |
| `drawXBitmapScaled(x, y, bitmap, w, h, sw, sh, color?)` | Draw scaled XBitmap |

### Text Methods

| Method | Description |
|--------|-------------|
| `setFont(font)` | Set current font (null for built-in 5x7) |
| `setTextColor(color)` | Set text color |
| `drawText(text, x, y, font?, color?)` | Draw text at position |
| `drawTextCentered(text, x, y, width, font?, color?)` | Draw centered text |
| `drawTextRightAligned(text, rightX, y, font?, color?)` | Draw right-aligned text |
| `getTextWidth(text, font?)` | Get text width in pixels |
| `getTextBounds(text, font?)` | Get text bounds `{ w, h, yMin, yMax }` |

### Export Methods

| Method | Description |
|--------|-------------|
| `toDataURL()` | Get PNG as data URL |
| `toBlob()` | Get PNG as Blob (async) |
| `download(filename?)` | Download as PNG |
| `copyToClipboard()` | Copy to clipboard (async) |
| `setScale(scale)` | Set CSS scale (1-4x) |
| `getContext()` | Get the drawing context |

## Built-in 5x7 Font

The GFX library includes a built-in 5x7 monospace font that's always available:

```javascript
sim.setFont(null);  // Use built-in font
sim.drawText('Hello', 10, 10);

// Or pass null/undefined as the font parameter
sim.drawText('World', 10, 20, null);
```

**Important differences from GFXfont:**
- Uses **top-left positioning** (not baseline like custom fonts)
- Character width: 6px (5px glyph + 1px spacing)
- Character height: 7px

## Types

```typescript
interface GFXFont {
  name: string;
  bitmap: number[];
  glyphs: Record<number, GFXGlyph>;
  first: number;
  last: number;
  yAdvance: number;
}

interface GFXGlyph {
  offset: number;
  width: number;
  height: number;
  xAdvance: number;
  xOffset: number;
  yOffset: number;
}

interface DisplayConfig {
  width: number;
  height: number;
  foreground?: string;
  background?: string;
}

interface TextBounds {
  w: number;
  h: number;
  yMin: number;
  yMax: number;
}
```

## Project Structure

```
eink-simulator/
├── src/
│   ├── types.ts          # Type definitions
│   ├── primitives.ts     # Drawing primitives
│   ├── fonts.ts          # Font rendering
│   ├── simulator.ts      # Main simulator class
│   ├── index.ts          # Entry point
│   └── generated/
│       └── font-data.ts  # Your extracted fonts go here
├── scripts/
│   ├── extract-fonts.js  # Font extraction CLI tool
│   └── bundle.js         # Browser bundler
├── dist/                 # Compiled output
│   ├── index.js          # ES module
│   ├── index.d.ts        # Type definitions
│   └── eink-simulator.bundle.js  # Browser bundle
├── examples/
│   └── basic-shapes.html # Demo page
├── fonts.config.example.json
├── package.json
└── tsconfig.json
```

## Building from Source

```bash
# Clone the repository
git clone https://github.com/slavik0329/eink-simulator.git
cd eink-simulator

# Install dependencies
npm install

# Build TypeScript and create bundle
npm run build

# Watch for changes during development
npm run watch
```

## License

MIT
