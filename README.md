# Hytale-UI-Markdown
Claude compilation of the UI Markdown System in Hytale(errors may exist)


# Hytale UI Markup Language Documentation

## Overview

Hytale uses a custom UI markup language for defining user interfaces. This document covers the syntax, components, properties, and patterns discovered through reverse engineering and testing.

---

## Table of Contents

1. [File Structure](#file-structure)
2. [Imports and References](#imports-and-references)
3. [Variables and Templates](#variables-and-templates)
4. [Components](#components)
5. [Properties](#properties)
6. [Styling](#styling)
7. [Layout System](#layout-system)
8. [Common Patterns](#common-patterns)
9. [Known Limitations](#known-limitations)
10. [Examples](#examples)

---

## File Structure

UI files use the `.ui` extension and follow this general structure:

```
// Comments use double slashes
$ImportAlias = "path/to/file.ui";

@VariableName = Value;

@TemplateName = ComponentType {
  Property: Value;
};

ComponentType #RootElementId {
  Property: Value;
  
  ChildComponent {
    Property: Value;
  }
}
```

---

## Imports and References

### Importing Files

Use `$` prefix to import other UI files:

```
$C = "Common.ui";
$Sounds = "Sounds.ui";
$CustomFile = "path/to/CustomFile.ui";
```

### Referencing Imported Content

Access imported variables/templates using dot notation:

```
$C.@DecoratedContainer    // Template from Common.ui
$C.@TitleStyle            // Style from Common.ui
$Sounds.@ButtonsLight     // Sound definition from Sounds.ui
```

---

## Variables and Templates

### Simple Variables

Use `@` prefix for variables:

```
@TitleHeight = 38;
@SlotSize = 48;
@BackgroundColor = #1a2030;
@Padding = 8;
```

### Template Definitions

Templates are reusable component definitions:

```
@InventorySlot = Group {
  Anchor: (Width: 48, Height: 48);
  Background: (Color: #1a2030(0.9));
};

@StatLabel = Label {
  Anchor: (Height: 18, Width: 87);
  Style: (FontSize: 13, RenderBold: true, TextColor: #878e9c);
  Text: "0";
};
```

### Parameterized Templates

Templates can have parameters using `@ParamName` inside:

```
@Title = Group {
  @Text = "DEFAULT";  // Parameter with default value
  
  Label {
    Text: @Text;
    Style: (FontSize: 18, Alignment: Center);
  }
};

// Usage:
@Title { @Text = "INVENTORY"; }
```

### Template Inheritance (Spread Operator)

Use `...` to inherit and extend styles:

```
@ExtendedStyle = (
  ...@BaseStyle,
  FontSize: 20
);
```

---

## Components

### Basic Components

| Component | Description |
|-----------|-------------|
| `Group` | Container for other elements, like a div |
| `Label` | Text display |
| `Button` | Clickable button (no text) |
| `TextButton` | Button with text |
| `ItemGrid` | Inventory grid for items |
| `ItemIcon` | Displays an item icon |

### Advanced Components

| Component | Description |
|-----------|-------------|
| `CharacterPreviewComponent` | 3D character model preview |
| `TabNavigation` | Tab-based navigation |
| `Slider` | Slider input |
| `TextField` | Text input field |
| `CompactTextField` | Collapsible text field |
| `ScrollView` | Scrollable container |
| `DropdownBox` | Dropdown selector |

---

## Properties

### Universal Properties (All Components)

| Property | Type | Description |
|----------|------|-------------|
| `Anchor` | Anchor | Position and size |
| `Background` | Background | Background color/texture |
| `Padding` | Padding | Inner spacing |
| `Margin` | Margin | Outer spacing |
| `Visible` | bool | Show/hide component |
| `LayoutMode` | LayoutMode | Child layout direction |
| `FlexWeight` | number | Flex grow factor |

### Label Properties

| Property | Type | Description |
|----------|------|-------------|
| `Text` | string | Display text |
| `Style` | LabelStyle | Text styling |

**IMPORTANT:** Labels do NOT support `HorizontalAlignment` or `VerticalAlignment` as direct properties. These must be inside the `Style` block.

```
// ❌ WRONG - Will crash!
Label {
  Text: "Hello";
  HorizontalAlignment: Center;
}

// ✅ CORRECT
Label {
  Text: "Hello";
  Style: (Alignment: Center, VerticalAlignment: Center);
}
```

### Button Properties

| Property | Type | Description |
|----------|------|-------------|
| `Style` | ButtonStyle | Button appearance states |

### TextButton Properties

| Property | Type | Description |
|----------|------|-------------|
| `Text` | string | Button label |
| `Style` | TextButtonStyle | Button + text styling |

### ItemGrid Properties

| Property | Type | Description |
|----------|------|-------------|
| `SlotsPerRow` | int | Items per row |
| `Style` | ItemGridStyle | Grid styling |

---

## Styling

### LabelStyle Properties

All properties go inside `Style: (...)` for Labels:

```
Style: (
  FontSize: 14,
  FontName: "Default",        // or "Secondary"
  TextColor: #ffffff,
  RenderBold: true,
  RenderUppercase: true,
  Alignment: Center,          // Horizontal: Start, Center, End
  VerticalAlignment: Center,  // Start, Center, End
  HorizontalAlignment: Center, // Alternative to Alignment
  Wrap: true,
  LetterSpacing: 1
)
```

### ButtonStyle Definition

```
@MyButtonStyle = ButtonStyle(
  Default: (Background: "button.png"),
  Hovered: (Background: "button_hover.png"),
  Pressed: (Background: "button_press.png"),
  Disabled: (Background: "button_disabled.png"),
  Sounds: @ButtonSounds
);
```

### TextButtonStyle Definition

```
@MyTextButtonStyle = TextButtonStyle(
  Default: (Background: @BgDefault, LabelStyle: @LabelDefault),
  Hovered: (Background: @BgHover, LabelStyle: @LabelHover),
  Pressed: (Background: @BgPress, LabelStyle: @LabelPress),
  Disabled: (Background: @BgDisabled, LabelStyle: @LabelDisabled),
  Sounds: @ButtonSounds
);
```

### Background Options

```
// Solid color
Background: (Color: #1a2030);

// Solid color with alpha (0.0 - 1.0)
Background: (Color: #1a2030(0.8));

// Texture
Background: "path/to/image.png";

// Texture with properties
Background: (TexturePath: "image.png", Border: 4);

// 9-slice/patch texture
Background: (TexturePath: "patch.png", Border: 20);
Background: (TexturePath: "patch.png", HorizontalBorder: 35, VerticalBorder: 0);

// PatchStyle for buttons
Background: PatchStyle(TexturePath: "button.png", Border: 12);
```

### Color Format

```
#RRGGBB          // Hex color
#RRGGBB(alpha)   // Hex with alpha (0.0 - 1.0)
#ffffff          // White
#000000(0.5)     // 50% transparent black
#00000000        // Fully transparent (hex alpha)
```

---

## Layout System

### Anchor System

Position and size elements:

```
Anchor: (
  Width: 100,      // Fixed width
  Height: 50,      // Fixed height
  Top: 10,         // Distance from parent top
  Bottom: 10,      // Distance from parent bottom
  Left: 10,        // Distance from parent left
  Right: 10,       // Distance from parent right
  Horizontal: 10,  // Left + Right shorthand
  Vertical: 10,    // Top + Bottom shorthand
  Full: 10         // All sides shorthand
);
```

**Note:** You can combine directional anchors:
- `Top` + `Bottom` = stretch vertically
- `Left` + `Right` = stretch horizontally
- `Width` + `Left` = fixed width, positioned from left

### LayoutMode

Controls how children are arranged:

```
LayoutMode: Top;     // Stack vertically from top
LayoutMode: Bottom;  // Stack vertically from bottom
LayoutMode: Left;    // Stack horizontally from left
LayoutMode: Right;   // Stack horizontally from right
LayoutMode: Middle;  // Center children
```

### FlexWeight

For flexible sizing in layouts:

```
Group {
  LayoutMode: Left;
  
  Group { FlexWeight: 1; }  // Takes 1/3 of space
  Group { FlexWeight: 2; }  // Takes 2/3 of space
}
```

### Padding

Inner spacing:

```
Padding: (Full: 8);              // All sides
Padding: (Horizontal: 10);       // Left + Right
Padding: (Vertical: 10);         // Top + Bottom
Padding: (Top: 5, Left: 10);     // Specific sides
Padding: (Top: 5, Left: 10, Right: 10, Bottom: 5);
```

---

## Common Patterns

### Container with Title

```
$C.@DecoratedContainer #MyPanel {
  Anchor: (Width: 400, Height: 500, Right: 20, Top: 100);
  
  #Title {
    Label #TitleLabel {
      Style: (FontSize: 18, RenderBold: true, Alignment: Center);
      Text: "PANEL TITLE";
    }
  }
  
  #Content {
    LayoutMode: Top;
    Padding: (Full: 8);
    
    // Content goes here
  }
}
```

### Horizontal Row of Items

```
Group {
  LayoutMode: Left;
  Anchor: (Height: 50);
  
  Group { FlexWeight: 1; /* Item 1 */ }
  Group { Anchor: (Width: 8); }  // Spacer
  Group { FlexWeight: 1; /* Item 2 */ }
  Group { Anchor: (Width: 8); }  // Spacer
  Group { FlexWeight: 1; /* Item 3 */ }
}
```

### Vertical Stack

```
Group {
  LayoutMode: Top;
  
  Label { Text: "Row 1"; Anchor: (Height: 30); }
  Label { Text: "Row 2"; Anchor: (Height: 30); }
  Label { Text: "Row 3"; Anchor: (Height: 30); }
}
```

### Grid of Buttons (Manual)

```
Group #SkillGrid {
  LayoutMode: Top;
  
  // Row 1
  Group {
    LayoutMode: Left;
    Anchor: (Height: 52);
    
    Button { Anchor: (Width: 84, Height: 48); }
    Button { Anchor: (Width: 84, Height: 48); }
    Button { Anchor: (Width: 84, Height: 48); }
  }
  
  // Row 2
  Group {
    LayoutMode: Left;
    Anchor: (Height: 52);
    
    Button { Anchor: (Width: 84, Height: 48); }
    Button { Anchor: (Width: 84, Height: 48); }
    Button { Anchor: (Width: 84, Height: 48); }
  }
}
```

### Inventory Slot with Border Effect

```
@InventorySlot = Group {
  Anchor: (Width: 48, Height: 48);
  
  Group {
    Anchor: (Full: 1);
    Background: (Color: #3a4a60);  // Border color
    Padding: (Full: 1);            // Border width
    
    Group {
      Anchor: (Full: 1);
      Background: (Color: #1a2030);  // Inner color
    }
  }
};
```

---

## Known Limitations

### Invalid Properties on Label

These properties do NOT work as direct Label properties:
- `HorizontalAlignment` - Use `Style: (Alignment: ...)` instead
- `VerticalAlignment` - Use `Style: (VerticalAlignment: ...)` instead

### Reserved/Invalid Features

- `ZIndex` - May cause parser crashes
- `IsOpaque` - Use color alpha instead: `#000000(0.5)`
- `Margin` on some components - May not work everywhere

### File Dependencies

- Files must be in the correct path relative to the server's UI folder
- Missing imports will cause load failures
- Circular imports may cause issues

---

## Complete Example

```
// OSRS-style Skills Panel
$C = "Common.ui";

@SkillSlot = Group {
  Anchor: (Width: 84, Height: 48);
  Background: (Color: #131a26(0.8));
  LayoutMode: Top;
};

$C.@DecoratedContainer #SkillsPanel {
  Anchor: (Right: 10, Width: 280, Top: 100, Height: 520);
  LayoutMode: Top;
  
  #Title {
    $C.@Title { @Text = "SKILLS"; }
  }
  
  #Content {
    LayoutMode: Top;
    Padding: (Full: 8);
    
    Group #SkillGrid {
      LayoutMode: Top;
      
      // Row 1: Attack, Strength, Defence
      Group {
        LayoutMode: Left;
        Anchor: (Height: 52);
        
        @SkillSlot {
          Label #AttackName { 
            Style: (FontSize: 9, TextColor: #b8c5d6, Alignment: Center); 
            Text: "Attack";
          }
          Label #AttackLevel { 
            Style: (FontSize: 11, TextColor: #ffffff, RenderBold: true, Alignment: Center); 
            Text: "1";
          }
        }
        
        @SkillSlot {
          Label #StrengthName { 
            Style: (FontSize: 9, TextColor: #b8c5d6, Alignment: Center); 
            Text: "Strength";
          }
          Label #StrengthLevel { 
            Style: (FontSize: 11, TextColor: #ffffff, RenderBold: true, Alignment: Center); 
            Text: "1";
          }
        }
        
        @SkillSlot {
          Label #DefenceName { 
            Style: (FontSize: 9, TextColor: #b8c5d6, Alignment: Center); 
            Text: "Defence";
          }
          Label #DefenceLevel { 
            Style: (FontSize: 11, TextColor: #ffffff, RenderBold: true, Alignment: Center); 
            Text: "1";
          }
        }
      }
      
      // More rows...
    }
    
    // Total Level
    Group {
      Anchor: (Height: 40);
      Background: (Color: #1a2530);
      
      Label #TotalLevel {
        Style: (FontSize: 14, RenderBold: true, TextColor: #ffd700, Alignment: Center, VerticalAlignment: Center);
        Text: "Total: 23";
      }
    }
  }
}
```

---

## Sounds Definition Example

```
// Sounds.ui
@ButtonsLightActivate = "Sounds/ButtonsLightActivate.ogg";
@ButtonsLightHover = "Sounds/ButtonsLightHover.ogg";

@ButtonsLight = (
  Activate: (
    SoundPath: @ButtonsLightActivate,
    MinPitch: -0.4,
    MaxPitch: 0.4,
    Volume: 4
  ),
  MouseHover: (
    SoundPath: @ButtonsLightHover,
    Volume: 6
  )
);
```

---

## Quick Reference Card

### Must-Know Rules

1. **Label alignment** → Always inside `Style: (...)`
2. **Colors** → `#RRGGBB` or `#RRGGBB(alpha)`
3. **Imports** → `$Alias = "file.ui";`
4. **Variables** → `@Name = value;`
5. **Templates** → `@Name = Component { ... };`
6. **IDs** → `#ElementId` after component type
7. **Comments** → `// single line only`

### Layout Quick Reference

```
LayoutMode: Top;    // Vertical stack ↓
LayoutMode: Left;   // Horizontal stack →
FlexWeight: 1;      // Flexible sizing
Anchor: (Width: X, Height: Y);  // Fixed size
Anchor: (Full: X);  // Fill with margin
```

---

*Document Version: 1.0*
*Last Updated: January 2026*
*Based on Hytale Creator Tools observation and testing*
