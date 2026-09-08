# 001Lib — Roblox Lua UI Library

A clean, dark-themed executor UI library for Roblox. Features a draggable window with a tab sidebar, smooth tween animations, and a full set of UI components: buttons, toggles, sliders, dropdowns, color pickers, textboxes, stats, labels, warnings, and notifications.

---

## Quick Start

Load the library from the raw GitHub source:

```lua
local Library = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/mjzzy/001Lib/refs/heads/main/src.lua"
))()
```

Then create a window and at least one tab before adding any elements:

```lua
local Window = Library:CreateWindow({ Title = "My Script" })
local MainTab = Window:CreateTab({ Name = "Main", Icon = "house" })
```

---

## Library API

### `Library:CreateWindow(cfg)`

Creates and displays the main window. Returns a `Window` object.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Title` | `string` | `"Lib Name"` | Text shown in the top bar |
| `Accent` | `Color3` | `Color3.fromRGB(100, 160, 255)` | Accent colour used across all elements |
| `ToggleKey` | `Enum.KeyCode` | `Enum.KeyCode.RightShift` | Key that shows/hides the window |

**Example:**

```lua
local Window = Library:CreateWindow({
    Title       = "My Cheat",
    Accent      = Color3.fromRGB(120, 200, 100),
    ToggleKey   = Enum.KeyCode.Insert,
})
```

**Window controls (built-in):**

| Button | Action |
|--------|--------|
| `×` (red on hover) | Destroys the GUI entirely |
| `−` (minus) | Minimises/restores the window body |
| YouTube icon | Copies the YouTube link to clipboard and notifies |
| Discord icon | Opens the Discord invite via RPC and/or copies link |

---

### `Library:Notify(cfg)`

Shows a slide-in notification card in the bottom-right corner.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Title` | `string` | `"Notification"` | Bold heading text |
| `Content` | `string` | *(none)* | Optional body text beneath the title |
| `Duration` | `number` | `4` | Seconds before the card fades out |

**Example:**

```lua
Library:Notify({
    Title    = "Script Loaded",
    Content  = "Welcome back! Press RightShift to toggle.",
    Duration = 5,
})
```

---

## Tab API

### `Window:CreateTab(cfg)`

Adds a tab button to the left sidebar. Returns a `Tab` object. The first tab created is automatically selected.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Tab"` | Label shown next to the icon |
| `Icon` | `string` | `"circle"` | Builder Icon name (see note below) |

> **Icon names** use Roblox's `BuilderIcons` font. Common ones: `"house"`, `"gear"`, `"person"`, `"sword"`, `"flag"`, `"star"`, `"shield"`, `"eye"`, `"lock"`, `"bolt"`.

**Example:**

```lua
local CombatTab  = Window:CreateTab({ Name = "Combat",   Icon = "sword"  })
local VisualTab  = Window:CreateTab({ Name = "Visuals",  Icon = "eye"    })
local MiscTab    = Window:CreateTab({ Name = "Misc",     Icon = "gear"   })
```

---

## Element Reference

All elements below are methods on a `Tab` object and return an API table with at minimum a `.Instance` field (the underlying `Frame`).

---

### 1. `Tab:CreateLabel(text)`

A simple text label. No background — renders inline between other elements.

```lua
local lbl = MainTab:CreateLabel("-- Player Options --")

-- Update text later:
lbl:Set("-- Updated Header --")
```

---

### 2. `Tab:CreateWarning(text)`

A highlighted warning row with an amber triangle icon and amber border.

```lua
local warn = MainTab:CreateWarning("This feature may cause lag.")

-- Update text later:
warn:Set("Disabled in this game version.")
```

---

### 3. `Tab:CreateButton(cfg)`

A clickable button. Flashes accent colour on press.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Button"` | Button label text |
| `Callback` | `function` | *(none)* | Called when the button is clicked |

```lua
MainTab:CreateButton({
    Name     = "Teleport to Spawn",
    Callback = function()
        game.Players.LocalPlayer.Character:PivotTo(
            workspace.SpawnLocation.CFrame
        )
    end,
})
```

---

### 4. `Tab:CreateToggle(cfg)`

A toggle switch with an animated knob. Fires the callback with the new boolean state on every change.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Toggle"` | Label text |
| `Default` | `boolean` | `false` | Starting state |
| `Callback` | `function` | *(none)* | Called with `(state: boolean)` |

```lua
local speedToggle = MainTab:CreateToggle({
    Name     = "Speed Hack",
    Default  = false,
    Callback = function(state)
        local char = game.Players.LocalPlayer.Character
        if char then
            char.Humanoid.WalkSpeed = state and 60 or 16
        end
    end,
})

-- Read or set the state programmatically:
print(speedToggle:Get())   -- false
speedToggle:Set(true)      -- activates and fires callback
```

---

### 5. `Tab:CreateStat(cfg)`

A read-only key/value row. Useful for displaying live stats (ping, speed, position, etc.).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Stat"` | Left-side label |
| `Value` | `any` | `"-"` | Initial right-side value (converted to string) |

```lua
local pingStat = MainTab:CreateStat({
    Name  = "Ping",
    Value = "...",
})

-- Update every second:
task.spawn(function()
    while true do
        pingStat:Set(math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()))
        task.wait(1)
    end
end)
```

---

### 6. `Tab:CreateSlider(cfg)`

A draggable slider that snaps to an increment.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Slider"` | Label text |
| `Min` | `number` | `0` | Minimum value |
| `Max` | `number` | `100` | Maximum value |
| `Default` | `number` | `Min` | Starting value |
| `Increment` | `number` | `1` | Snap step size |
| `Callback` | `function` | *(none)* | Called with `(value: number)` while dragging |

```lua
local fovSlider = VisualTab:CreateSlider({
    Name      = "FOV",
    Min       = 70,
    Max       = 120,
    Default   = 90,
    Increment = 5,
    Callback  = function(val)
        workspace.CurrentCamera.FieldOfView = val
    end,
})

-- Read or set:
print(fovSlider:Get())
fovSlider:Set(100)
```

---

### 7. `Tab:CreateTextbox(cfg)`

A text input field. Fires the callback on focus-lost (Enter or click-away).

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Textbox"` | Label shown on the left |
| `Default` | `string` | `""` | Pre-filled text |
| `Placeholder` | `string` | `"..."` | Greyed placeholder text |
| `MinWidth` | `number` | `56` | Min pixel width of the input box |
| `MaxWidth` | `number` | `180` | Max pixel width of the input box |
| `Callback` | `function` | *(none)* | Called with `(text: string)` on focus-lost |

```lua
local nameBox = MiscTab:CreateTextbox({
    Name        = "Target Player",
    Placeholder = "Username...",
    Callback    = function(text)
        print("Targeting:", text)
    end,
})

-- Read or set:
print(nameBox:Get())
nameBox:Set("Roblox")
```

---

### 8. `Tab:CreateColorPicker(cfg)`

An expandable inline HSV colour picker (SV square + hue bar). Click the swatch row to open/close it.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Color"` | Label text |
| `Default` | `Color3` | Red (`255,0,0`) | Starting colour |
| `Callback` | `function` | *(none)* | Called with `(color: Color3)` as you drag |

```lua
local espColor = VisualTab:CreateColorPicker({
    Name     = "ESP Colour",
    Default  = Color3.fromRGB(255, 50, 50),
    Callback = function(color)
        -- apply to your ESP boxes here
        print("New colour:", color)
    end,
})

-- Read or set:
print(espColor:Get())
espColor:Set(Color3.fromRGB(0, 255, 128))
```

---

### 9. `Tab:CreateDropdown(cfg)`

An expandable dropdown list. Supports both single-select and multi-select modes.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `Name` | `string` | `"Dropdown"` | Label text |
| `Options` | `{string}` | `{}` | List of option strings |
| `Default` | `string \| {string}` | *(none)* | Pre-selected option(s) |
| `Multi` | `boolean` | `false` | Allow multiple selections |
| `Callback` | `function` | *(none)* | Single: `(selected: string)`, Multi: `(selected: {string})` |

**Single-select example:**

```lua
local teamDrop = CombatTab:CreateDropdown({
    Name    = "Team",
    Options = { "Red", "Blue", "Green" },
    Default = "Red",
    Callback = function(choice)
        print("Team selected:", choice)
    end,
})

print(teamDrop:Get())   -- "Red"
teamDrop:Set("Blue")
```

**Multi-select example:**

```lua
local itemDrop = MiscTab:CreateDropdown({
    Name    = "Items to Collect",
    Options = { "Coins", "Gems", "Keys", "Hearts" },
    Default = { "Coins", "Keys" },
    Multi   = true,
    Callback = function(choices)
        -- choices is a table of all currently selected strings
        print(table.concat(choices, ", "))
    end,
})

-- Refresh the option list at runtime:
itemDrop:Refresh({ "Coins", "Gems", "Keys", "Hearts", "Stars" })
```

---

## Complete Example Script

```lua
-- Load library
local Library = loadstring(game:HttpGet(
    "https://raw.githubusercontent.com/mjzzy/001Lib/refs/heads/main/src.lua"
))()

-- Create window
local Window = Library:CreateWindow({
    Title    = "My Hub",
    Accent   = Color3.fromRGB(100, 160, 255),
    ToggleKey = Enum.KeyCode.RightShift,
})

-- ── Combat Tab ──────────────────────────────────────────────────────
local Combat = Window:CreateTab({ Name = "Combat", Icon = "sword" })

Combat:CreateLabel("── Movement ──")

local speedToggle = Combat:CreateToggle({
    Name     = "Speed Hack",
    Default  = false,
    Callback = function(state)
        local hum = game.Players.LocalPlayer.Character and
                    game.Players.LocalPlayer.Character:FindFirstChild("Humanoid")
        if hum then hum.WalkSpeed = state and 60 or 16 end
    end,
})

Combat:CreateSlider({
    Name      = "Walk Speed",
    Min       = 16,
    Max       = 200,
    Default   = 16,
    Increment = 2,
    Callback  = function(val)
        local hum = game.Players.LocalPlayer.Character and
                    game.Players.LocalPlayer.Character:FindFirstChild("Humanoid")
        if hum then hum.WalkSpeed = val end
    end,
})

Combat:CreateButton({
    Name     = "Reset Character",
    Callback = function()
        game.Players.LocalPlayer.Character:FindFirstChild("Humanoid").Health = 0
    end,
})

-- ── Visuals Tab ──────────────────────────────────────────────────────
local Visuals = Window:CreateTab({ Name = "Visuals", Icon = "eye" })

Visuals:CreateWarning("ESP may be detected in some anti-cheats.")

Visuals:CreateToggle({
    Name     = "Player ESP",
    Default  = false,
    Callback = function(state)
        print("ESP:", state)
    end,
})

Visuals:CreateColorPicker({
    Name     = "ESP Colour",
    Default  = Color3.fromRGB(255, 80, 80),
    Callback = function(color)
        print("ESP colour changed:", color)
    end,
})

Visuals:CreateSlider({
    Name      = "FOV",
    Min       = 70,
    Max       = 120,
    Default   = 90,
    Increment = 1,
    Callback  = function(val)
        workspace.CurrentCamera.FieldOfView = val
    end,
})

-- ── Misc Tab ─────────────────────────────────────────────────────────
local Misc = Window:CreateTab({ Name = "Misc", Icon = "gear" })

local pingStat = Misc:CreateStat({ Name = "Ping", Value = "..." })

Misc:CreateDropdown({
    Name     = "Game Mode",
    Options  = { "Normal", "Turbo", "Ghost" },
    Default  = "Normal",
    Callback = function(choice)
        Library:Notify({ Title = "Mode Changed", Content = "Now in " .. choice .. " mode." })
    end,
})

Misc:CreateTextbox({
    Name        = "Target Player",
    Placeholder = "Username...",
    Callback    = function(text)
        Library:Notify({ Title = "Target Set", Content = text, Duration = 3 })
    end,
})

-- Live ping updater
task.spawn(function()
    while true do
        pingStat:Set(
            math.floor(
                game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()
            )
        )
        task.wait(1)
    end
end)

-- Startup notification
Library:Notify({
    Title   = "Loaded",
    Content = "Press RightShift to toggle the window.",
    Duration = 5,
})
```

---

## Element API Summary

| Element | Create Method | Returned Methods |
|---------|--------------|-----------------|
| Label | `CreateLabel(text)` | `:Set(text)` |
| Warning | `CreateWarning(text)` | `:Set(text)` |
| Button | `CreateButton(cfg)` | *(none)* |
| Toggle | `CreateToggle(cfg)` | `:Set(bool)`, `:Get()` |
| Stat | `CreateStat(cfg)` | `:Set(value)` |
| Slider | `CreateSlider(cfg)` | `:Set(number)`, `:Get()` |
| Textbox | `CreateTextbox(cfg)` | `:Set(text)`, `:Get()` |
| Color Picker | `CreateColorPicker(cfg)` | `:Set(Color3)`, `:Get()` |
| Dropdown | `CreateDropdown(cfg)` | `:Set(val)`, `:Get()`, `:Refresh(opts)` |

---

## Notes

- **Icons** — the library uses Roblox's `BuilderIcons` font. Pass any valid icon name string to `Icon` in `CreateTab`, or look up names in the Roblox `BuilderIcons` documentation.
- **Theme** — the colour theme is defined internally. You can change the `Accent` colour per-window via `CreateWindow({ Accent = Color3... })`.
- **ToggleKey** — defaults to `RightShift`. Pass any `Enum.KeyCode` to override.
- **Executor compatibility** — clipboard (`setclipboard`/`toclipboard`) and HTTP request functions are resolved automatically across common executors (Synapse, Fluxus, etc.).
- **Mobile & touch** — dragging, sliders, and color pickers all support touch input natively.
