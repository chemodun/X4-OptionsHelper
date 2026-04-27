# Options Helper

A library mod for **X4: Foundations** that provides reusable MD (Mission Director) script libraries for building [SirNukes Mod Support APIs](https://www.nexusmods.com/x4foundations/mods/503) options menus.

Instead of repeating the widget-construction boilerplate in every mod, declare your menu sections using the `Options_Helper` libraries and wire up a single handler cue per widget type.

## Requirements

- **X4: Foundations** version 8.00 or newer
- **SirNukes Mod Support APIs** version 1.95 or higher

## Installation

Place the `options_helper/` folder in your X4 extensions directory alongside your other mods.

---

## API Reference

All libraries live in the `md.Options_Helper` namespace. Widget-building libraries use `purpose="run_actions"` (called with `<run_actions ref="...">`) while layout and event-processing libraries use `purpose="include_actions"` (called with `<include_actions ref="...">`).

### Layout libraries

#### `Add_Empty_Row` — `include_actions`

Adds a thin visual separator row (border height) to the current options menu.

```xml
<include_actions ref="md.Options_Helper.Add_Empty_Row" />
```

---

#### `Add_Title_Row` — `run_actions`

Adds a centred section title row followed by an empty separator row.

- `title` *(required)* - Text string or text DB reference `{page, id}`
- `columns` *(default: `12`)* - Number of columns in the menu

```xml
<run_actions ref="md.Options_Helper.Add_Title_Row">
  <param name="title" value="{1972092401, 100}" />
  <param name="columns" value="12" />
</run_actions>
```

---

### Widget libraries

#### `Add_Checkbox` — `run_actions`

Adds a checkbox widget with a text label to its right. Automatically starts a new row when `col == 1`.

- `id` *(required)* - String identifier - used to build the widget id and the echo key `$<id>`
- `col` *(default: `1`)* - Column for the checkbox widget
- `textColSpan` *(default: `1`)* - Column span for the label text cell
- `text` *(default: `''`)* - Label text or text DB reference
- `data` *(required)* - Table holding the current value at key `$<id>`
- `active` *(default: `true`)* - Whether the widget is interactive
- `handle` *(default: `null`)* - Cue to signal on click

**Handler echo fields:**

- `event.param.$echo.$valueId` - table key string, e.g. `'$mySetting'`
- `event.param.$checked` - int `0` or `1` after the click

```xml
<run_actions ref="md.Options_Helper.Add_Checkbox">
  <param name="id" value="'mySetting'" />
  <param name="col" value="1" />
  <param name="textColSpan" value="11" />
  <param name="text" value="{1972092401, 110}" />
  <param name="data" value="@$mySettings" />
  <param name="handle" value="Handle_Checkbox" />
</run_actions>
```

---

#### `Add_Dropdown` — `run_actions`

Adds a label text cell followed by a dropdown widget on the same row. Automatically starts a new row when `col == 1`.

- `id` *(required)* - String identifier - used to build the widget id and the echo key `$<id>`
- `options` *(required)* - List of `[$text = ..., $value = ...]` option entries
- `currentIndex` *(default: `1`)* - 1-based index of the initially selected option
- `col` *(default: `1`)* - Column for the label text cell
- `textColSpan` *(default: `1`)* - Column span for the label
- `text` *(default: `''`)* - Label text or text DB reference
- `dropdownColSpan` *(default: `1`)* - Column span for the dropdown widget
- `active` *(default: `true`)* - Whether the widget is interactive
- `handle` *(default: `null`)* - Cue to signal on option confirm

**Handler echo fields:**

- `event.param.$echo.$valueId` - table key string, e.g. `'$myOption'`
- `event.param.$option.$value` - value of the selected option

```xml
<run_actions ref="md.Options_Helper.Add_Dropdown">
  <param name="id" value="'myOption'" />
  <param name="options" value="$myOptions" />
  <param name="currentIndex" value="$currentIndex" />
  <param name="col" value="1" />
  <param name="textColSpan" value="7" />
  <param name="text" value="{1972092401, 120}" />
  <param name="dropdownColSpan" value="5" />
  <param name="handle" value="Handle_Dropdown" />
</run_actions>
```

---

#### `Add_Slider` — `run_actions`

Adds a non-selectable label row followed by a slider widget row. Always creates a two-row pair.

- `id` *(required)* - String identifier - used to build the widget id and the echo key `$<id>`
- `text` *(default: `''`)* - Label text or text DB reference shown above the slider
- `min` *(default: `0`)* - Minimum slider value
- `max` *(default: `100`)* - Maximum slider value
- `step` *(default: `1`)* - Step increment between slider positions
- `suffix` *(default: `''`)* - String appended to the displayed current value
- `colSpan` *(default: `1`)* - Column span for both the label and slider cells
- `data` *(required)* - Table holding the current value at key `$<id>`
- `readOnly` *(default: `false`)* - Whether the slider is display-only
- `handle` *(default: `null`)* - Cue to signal on slider confirm

**Handler echo fields:**

- `event.param.$echo.$valueId` - table key string, e.g. `'$mySlider'`
- `event.param.$value` - confirmed longfloat value

```xml
<run_actions ref="md.Options_Helper.Add_Slider">
  <param name="id" value="'myThreshold'" />
  <param name="min" value="0" />
  <param name="max" value="1000000" />
  <param name="step" value="1000" />
  <param name="text" value="{1972092401, 130}" />
  <param name="suffix" value="' Cr'" />
  <param name="colSpan" value="12" />
  <param name="data" value="@$mySettings" />
  <param name="handle" value="Handle_Slider" />
</run_actions>
```

---

### Event-processing libraries

These three libraries are `include_actions` — they run inline inside your handler cue, where `event.param` is in scope. Before calling, assign `$resultTable` to the table you want written.

#### `Process_Dropdown_Changed` — `include_actions`

Reads `event.param.$echo.$valueId` and `event.param.$option.$value`, writes the selected value into `$resultTable`.

```xml
<cue name="Handle_Dropdown" instantiate="true">
  <conditions><event_cue_signalled /></conditions>
  <actions>
    <set_value name="$resultTable" exact="@$mySettings" />
    <include_actions ref="md.Options_Helper.Process_Dropdown_Changed" />
    <remove_value name="$resultTable" />
  </actions>
</cue>
```

---

#### `Process_Slider_Changed` — `include_actions`

Reads `event.param.$echo.$valueId` and `event.param.$value`, writes the confirmed slider value into `$resultTable`.

```xml
<cue name="Handle_Slider" instantiate="true">
  <conditions><event_cue_signalled /></conditions>
  <actions>
    <set_value name="$resultTable" exact="@$mySettings" />
    <include_actions ref="md.Options_Helper.Process_Slider_Changed" />
    <remove_value name="$resultTable" />
  </actions>
</cue>
```

---

#### `Process_Checkbox_Changed` — `include_actions`

Reads `event.param.$echo.$valueId` and `event.param.$checked` (int `0`/`1`), writes the checked state into `$resultTable`.

```xml
<cue name="Handle_Checkbox" instantiate="true">
  <conditions><event_cue_signalled /></conditions>
  <actions>
    <set_value name="$resultTable" exact="@$mySettings" />
    <include_actions ref="md.Options_Helper.Process_Checkbox_Changed" />
    <remove_value name="$resultTable" />
  </actions>
</cue>
```

---

## Complete Usage Example

```xml
<!-- Register your menu on API reload -->
<cue name="Register_Menu" instantiate="true">
  <conditions>
    <event_cue_signalled cue="md.Simple_Menu_API.Reloaded" />
  </conditions>
  <actions>
    <signal_cue_instantly cue="md.Simple_Menu_API.Register_Options_Menu"
      param="table[
        $id      = 'my_mod_options',
        $columns = 12,
        $title   = 'My Mod Options',
        $onOpen  = My_Menu_Open,
      ]" />
  </actions>
</cue>

<!-- Build menu content on open -->
<cue name="My_Menu_Open" instantiate="true">
  <conditions><event_cue_signalled /></conditions>
  <actions>
    <!-- load your settings table into $mySettings here -->

    <include_actions ref="md.Options_Helper.Add_Empty_Row" />
    <run_actions ref="md.Options_Helper.Add_Title_Row">
      <param name="title" value="'General Settings'" />
    </run_actions>

    <run_actions ref="md.Options_Helper.Add_Checkbox">
      <param name="id" value="'enabled'" />
      <param name="col" value="1" />
      <param name="textColSpan" value="11" />
      <param name="text" value="'Enable feature'" />
      <param name="data" value="@$mySettings" />
      <param name="handle" value="Handle_Checkbox" />
    </run_actions>

    <run_actions ref="md.Options_Helper.Add_Slider">
      <param name="id" value="'threshold'" />
      <param name="min" value="0" />
      <param name="max" value="100000" />
      <param name="step" value="1000" />
      <param name="text" value="'Minimum threshold'" />
      <param name="suffix" value="' Cr'" />
      <param name="colSpan" value="12" />
      <param name="data" value="@$mySettings" />
      <param name="handle" value="Handle_Slider" />
    </run_actions>
  </actions>
</cue>

<!-- Handlers -->
<cue name="Handle_Checkbox" instantiate="true">
  <conditions><event_cue_signalled /></conditions>
  <actions>
    <set_value name="$resultTable" exact="@$mySettings" />
    <include_actions ref="md.Options_Helper.Process_Checkbox_Changed" />
    <remove_value name="$resultTable" />
    <!-- persist $mySettings here -->
  </actions>
</cue>

<cue name="Handle_Slider" instantiate="true">
  <conditions><event_cue_signalled /></conditions>
  <actions>
    <set_value name="$resultTable" exact="@$mySettings" />
    <include_actions ref="md.Options_Helper.Process_Slider_Changed" />
    <remove_value name="$resultTable" />
    <!-- persist $mySettings here -->
  </actions>
</cue>
```

---

## Credits

- **Author**: Chem O`Dun, on [Nexus Mods](https://next.nexusmods.com/profile/ChemODun/mods?gameId=2659) and [Steam Workshop](https://steamcommunity.com/id/chemodun/myworkshopfiles/?appid=392160)
- *"X4: Foundations"* is a trademark of [Egosoft](https://www.egosoft.com).

## Acknowledgements

- [EGOSOFT](https://www.egosoft.com) - for the X series.
- [SirNukes](https://next.nexusmods.com/profile/sirnukes?gameId=2659) - for the `Mod Support APIs` that power the UI hooks.
