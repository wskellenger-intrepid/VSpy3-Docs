# Graphical Panel Controls: SVG Graphic

An SVG Graphic control shows a drawing you made in a vector graphics program such as Inkscape. The drawing can react to signal values and can set signals. You can build gauges, bars, indicator lights, buttons or a picture of a vehicle that changes as the vehicle runs.

Vehicle Spy does not draw the graphic itself. The drawing, and any animation it has, comes from the SVG file. Vehicle Spy only connects the names in the SVG file to signals.

## How an SVG Is Connected to Signals

The SVG file declares the **names** it reads and the names it writes. You choose the **signal** for each name in the property grid.

* A name the SVG reads is an **input**. Vehicle Spy sends the value of the signal to the SVG.
* A name the SVG writes is an **output**. When the SVG sets it, Vehicle Spy sets the signal.

The names have no meaning to Vehicle Spy. `rpm` is only a label the author of the SVG chose. You can connect `rpm` to any signal, for example a temperature. The graphic does not know what the number means, so a gauge drawn with a scale of 0 to 8000 still shows 0 to 8000.

When you load an SVG file, Vehicle Spy finds the names in it and adds one row to the property grid for each name:

* `In: rpm` is an input called `rpm`.
* `Out: horn` is an output called `horn`.

Double-click a row to choose its signal or expression, the same way you do for other graphical panel controls.

## Add an SVG Graphic to a Panel

1. Add an **SVG Graphic** control to the panel the same way you add any other tool. See [Add or Delete a Tool](../graphical-panels-add-or-delete-a-tool.md).
2. Select the control. Double-click the **SVG File** row, and choose an `.svg` file.
3. Double-click each **In:** row and choose the signal or expression to show.
4. Double-click each **Out:** row and choose the signal to set.
5. Start the measurement. The graphic appears.

**Table 1: SVG Graphic Control Properties**

| Property | Function and Options |
| -------- | -------------------- |
| SVG File | Double-click to choose the SVG file. Shows the file name once a file is loaded. The file is stored inside the setup, so the setup does not need the file afterward. |
| In: _name_ | One row for each input in the SVG. Double-click to choose the signal or expression whose value is sent to the SVG. If the signal is a text signal, or **Evaluate as text** is checked for it, the SVG receives text. Otherwise it receives a number, or `null` when the value is not a valid number. |
| Out: _name_ | One row for each output in the SVG. Double-click to choose the signal that is set when the SVG sets this output. Until you choose a signal, the output does nothing. |

A list of common properties for all controls can be found under [Common Control Properties](graphical-panel-controls-common-control-properties.md). The **Transparent** and **BackColor** properties set the color behind the drawing. A transparent control uses the color of the panel, so controls behind the graphic do not show through.

The SVG Graphic control also works on a [floating panel](../floating-panels.md).

## When the Graphic Is Shown

Vehicle Spy runs the graphic, with its script and signal values, only while the measurement is running. When it is not running, the control shows a picture of the SVG as it looks before any value arrives, for example a gauge with its needle at its starting position. You can select, move and resize the control as usual.

Vehicle Spy takes the picture again when you load another SVG, change the color behind the drawing, or finish resizing the control. While you resize, the old picture keeps its shape and the new one appears a moment after you let go.

The graphic is drawn with the Microsoft Edge WebView2 runtime. Until the first picture is ready, and if a picture cannot be taken, the control shows a box with the words **SVG Graphic** and the name of the file instead. A picture cannot be taken when the runtime is not available on the computer, or when the SVG is larger than about 2 MB.

## Prepare the SVG

You can draw in any vector graphics program. Then you add attributes that name the inputs and outputs. In Inkscape, use **Edit > XML Editor** (Ctrl+Shift+X): select an object, and add an attribute with the **+** button.

**Table 2: Attributes Vehicle Spy Looks For**

| Attribute | Put it on | What it does |
| --------- | --------- | ------------ |
| `data-vspy-in="name"` | Any element | Declares the input `name`. When the value changes, Vehicle Spy writes it into the element if the element is a `text` or `tspan`. Every element with the attribute also gets the current value in a `data-vspy-value` attribute. A script or a style sheet can use it. |
| `data-vspy-out="name"` | Any element | Declares the output `name`. Clicking the element sets the output to 1. |
| `data-vspy-click="value"` | An element with `data-vspy-out` | The value to set when the element is clicked instead of 1. |

A name can contain letters, digits, underscores, hyphens and periods. Vehicle Spy ignores a name with any other character, such as a space. Names are case-sensitive. A name that is declared several times appears once in the property grid. Inputs and outputs are listed separately, so the same name can be an input and an output.

In Inkscape, a text object contains a `tspan` element. Add `data-vspy-in` to the `tspan` so the text keeps its formatting.

Vehicle Spy finds the names by reading the text of the file. Every name has to be written in the file, and a name inside an XML comment counts too. A script cannot invent a name while it runs.

Vehicle Spy scales the drawing evenly to fit the control and centers it, so a control with another shape than the drawing shows empty bands at its sides or its top and bottom. The SVG needs a `viewBox` attribute for this. Inkscape always writes one. Without it, the drawing keeps its own size and is not scaled.

## Animate the Drawing With a Script

An attribute can show a number in a text object, but cannot move or color a shape. To do that, put a script in the SVG. Vehicle Spy runs it in the browser that draws the graphic.

The script receives values and sends values like this:

* **When values arrive**, the page gets an event named `vspy` on `document`. `event.detail` holds only the inputs that changed, for example `{rpm: 5200}`. When the graphic is first shown, it holds all inputs that have a value.
* `vspy.values` holds the latest value of every input received so far.
* `vspy.set("name", value)` sets the output `name`. This needs the attribute `data-vspy-out="name"` on some element, so that the output exists. Any element works, including a `desc` element that is only there to declare the name.

`vspy` exists only after your script has first run, so call `vspy.set` and read `vspy.values` inside an event or click handler, never at the top level of the script.

A click sets the output once and never sets it back, so the `horn` output in the example stays at 1. For a momentary button, call `vspy.set("horn", 1)` when the pointer goes down and `vspy.set("horn", 0)` when it goes up, from `pointerdown` and `pointerup` handlers.

Put the script in a CDATA section, so the file stays valid XML and programs such as Inkscape can open it.

This example shows the input `rpm` as a number, grows a bar with it, and sets the output `horn` when the button is clicked:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 120">
  <text data-vspy-in="rpm" x="100" y="40" text-anchor="middle" font-size="24">0</text>
  <rect id="bar" x="10" y="60" width="0" height="20" fill="steelblue"/>
  <g data-vspy-out="horn" data-vspy-click="1" style="cursor:pointer">
    <rect x="70" y="90" width="60" height="24" rx="4" fill="#446"/>
    <text x="100" y="107" fill="#fff" text-anchor="middle" font-size="14"
          pointer-events="none">HORN</text>
  </g>
  <script><![CDATA[
    document.addEventListener('vspy', function (e) {
      if ('rpm' in e.detail)
        document.getElementById('bar').setAttribute('width',
          180 * Math.min(e.detail.rpm, 8000) / 8000);
    });
  ]]></script>
</svg>
```

Loading it adds the rows **In: rpm** and **Out: horn**.

To build a different gauge, change the scale in the script, not in Vehicle Spy. To add a second input, declare a second name in the file and read it in the same event.

## Things to Know

* **The setup keeps its own copy.** If you change the SVG file afterward, double-click **SVG File** and choose it again. Inputs and outputs whose names are still in the file keep their signals. A name that you rename or remove loses its signal, and a new name starts with none.
* **Outputs set signals only while the measurement is running.**
* **The page has no network access.** Links, external images, external fonts and external scripts do not load. Put images inside the SVG as `data:` addresses. Fonts cannot be embedded: the SVG can only use fonts that are installed on the computer. A link in the SVG goes nowhere, and a script cannot open a new window.
* **Size.** An SVG file larger than about 2 MB cannot be shown. Vehicle Spy adds a message to the **Problem Log** that the SVG could not be loaded.
* **A hidden control** does not show the graphic and does not update. A control is hidden when it is on a tab that is not selected, or when the Text API or C code hides it.
