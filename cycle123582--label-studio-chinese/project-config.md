---
trigger: always_on
description: Writing and updating cypress integration tests
---

## Cypress Test Generation Rules for Label Studio

### Project Structure and Organization

**Test File Structure:**
- Tests should be placed in `web/libs/editor/tests/integration/e2e/` with semantic folder organization
- Follow the existing folder structure: `core/`, `image_segmentation/`, `control_tags/`, `audio/`, `video/`, `timeseries/`, `relations/`, `outliner/`, `bulk_mode/`, `config/`, `drafts/`, `linking_modes/`, `ner/`, `sync/`, `view_all/`
- Test files should end with `.cy.ts` extension
- Test data should be placed in `web/libs/editor/tests/integration/data/` following the same folder structure

**File Naming Convention:**
- Use descriptive names that reflect the feature being tested
- Use kebab-case for file names (e.g., `audio-regions.cy.ts`, `image-segmentation.cy.ts`)
- Group related tests in logical folders

### Import Standards

**Required Imports:**
Always import helpers from the centralized helper library:
```typescript
import { LabelStudio, ImageView, Sidebar, Labels, Hotkeys } from "@humansignal/frontend-test/helpers/LSF";
```

**Test Data Imports:**
Import test data from the data folder with relative paths:
```typescript
import { configName, dataName, resultName } from "../../data/folder_name/file_name";
```

**Available Helpers:**
- `LabelStudio` - Core initialization and control
- `ImageView` - Image interaction and drawing
- `VideoView` - Video playback and interaction  
- `AudioView` - Audio playback and regions
- `Sidebar` - Outliner and region management
- `Labels` - Label selection and management
- `Hotkeys` - Cross-platform keyboard shortcuts (Mac/PC compatibility)
- `Taxonomy`, `Choices`, `DateTime`, `Number`, `Rating`, `Textarea` - Control tag helpers
- `Relations` - Relationship management
- `ToolBar` - Toolbar interactions
- `Modals` - Modal dialog handling
- `Tooltip` - Tooltip verification

### Test Structure Standards

**Basic Test Structure:**
```typescript
describe("Feature Name - Specific Area", () => {
  it("should perform specific action", () => {
    // Test implementation
  });
});
```

**Nested Describes:**
Use nested describe blocks for logical grouping:
```typescript
describe("Image Segmentation", () => {
  describe("Rectangle Tool", () => {
    it("should draw rectangle", () => {
      // Test implementation
    });
  });
});
```

### LabelStudio Initialization Patterns

**Simple Initialization:**
```typescript
LabelStudio.init({
  config: configString,
  task: {
    id: 1,
    annotations: [{ id: 1001, result: [] }],
    predictions: [],
    data: { image: "url" },
  },
});
```

**Fluent API Initialization (Preferred):**
```typescript
LabelStudio.params()
  .config(configString)
  .data(dataObject)
  .withResult(expectedResult)
  .init();
```

**With Additional Parameters:**
```typescript
LabelStudio.params()
  .config(config)
  .data(data)
  .withResult([])
  .withInterface("panel")
  .withEventListener("eventName", handlerFunction)
  .withParam("customParam", value)
  .init();
```

### Required Test Preparation Steps

**Always Include:**
1. LabelStudio initialization
2. Wait for objects ready: `LabelStudio.waitForObjectsReady();`
3. (optional, usually waitForObjectsReady is enough) Wait for media loading (for image/video/audio): `ImageView.waitForImage();`
4. Initial state verification: `Sidebar.hasNoRegions();`
5. (optional, if possible) Some state verification after actions, for example: `Sidebar.hasRegions(count);`

### Interaction Patterns

**Image Interactions:**
```typescript
// Wait for image to load
ImageView.waitForImage();

// Select tools
ImageView.selectRectangleToolByButton();
ImageView.selectPolygonToolByButton();

// Drawing operations
ImageView.drawRect(x, y, width, height);
ImageView.drawRectRelative(0.1, 0.1, 0.4, 0.8); // Preferred

// Click interactions
ImageView.clickAt(x, y);
ImageView.clickAtRelative(0.5, 0.5); // Preferred

// Screenshot comparisons
ImageView.capture("screenshot_name");
ImageView.canvasShouldChange("screenshot_name", threshold);
```

**Label Management:**
```typescript
// Select labels before drawing
Labels.select("Label Name");

// Verify label selection
Labels.isSelected("Label Name");
```

**Sidebar Operations:**
```typescript
// Region verification
Sidebar.hasRegions(count);
Sidebar.hasNoRegions();
Sidebar.hasSelectedRegions(count);

// Region manipulation
Sidebar.toggleRegionVisibility(index);
Sidebar.toggleRegionSelection(index);
```

### Assertion Patterns

**Standard Cypress Assertions:**
```typescript
cy.get(selector).should("be.visible");
cy.get(selector).should("have.text", "expected text");
cy.get(selector).should("have.class", "class-name");
```

**Custom Helper Assertions:**
```typescript
Sidebar.hasRegions(expectedCount);
Sidebar.hasSelectedRegions(expectedCount);
ImageView.canvasShouldChange("screenshot", threshold);
```

**Window Object Access:**
```typescript
cy.window().then((win) => {
  expect(win.Htx.annotationStore.selected.names.get("image")).to.exist;
});
```

### Test Data Structure

**Configuration Format:**
```typescript
export const configName = `
  <View>
    <Image name="img" value="$image"/>
    <RectangleLabels name="tag" toName="img">
      <Label value="Planet"/>
      <Label value="Moonwalker" background="blue"/>
    </RectangleLabels>
  </View>
`;
```

**Data Format:**
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cycle123582/label-studio-Chinese](https://github.com/cycle123582/label-studio-Chinese) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
