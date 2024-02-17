Introduction
------------

This project is an emscripten port of glfw written in C++ for the web/wasm platform.

Goal
----

The main goal of this project is to provide as many features from glfw as possible (in a browser context).

Since this project is targeting the web/webassembly platform, which runs in more recent web browsers, it is also trying
to focus on using the most recent features and not use deprecated features (for example, uses `keyboardEvent.key` 
vs `keyboardEvent.charcode`). As a result, this implementation will most likely not work in older browsers.

Since the code is written in C++, it is trying to minimize the amount of javascript code to remain clean and lean.

A wishful goal would be to potentially replace the built-in emscripten/glfw code in the future if this effort 
is successful, or most likely, offer it as an option.

Status
------

![Compiles](https://github.com/pongasoft/emscripten-glfw/actions/workflows/main.yml/badge.svg)

The project is now mature enough and has reached 1.0.0 status.

Main supported features:
* can create as many windows as you want, each one associated to a different canvas (use 
  `emscripten_glfw_set_next_window_canvas_selector("#canvas2")` to specify which canvas to use)
* resizable window/canvas (use `emscripten_glfw_make_canvas_resizable(...)` to make the canvas resizable by user.
  Use `"window"` as the resize selector for full frame canvas (ex: ImGui))
* mouse (includes sticky button behavior)
* keyboard (includes sticky key behavior)
* joystick/gamepad
* fullscreen
* Hi DPI 
* all glfw cursors
* window opacity
* size constraints (size limits and aspect ratio)
* visibility
* focus
* timer

Demo
----

![emscripten_glfw](https://github.com/pongasoft/emscripten-glfw/releases/download/v1.0.0/emscripten_glfw.png)

Checkout the [live demo](https://pongasoft.github.io/emscripten-glfw/test/demo/main.html) of the example code. Note that you
need to use a "modern" browser to see it in action. Currently tested on Google Chrome 120+ and Firefox 121+. 

The [code](test/demo/src) for the demo is included in this project.

The demo shows 2 canvases each created via a `glfwCreateWindow` and shows how they respond to keyboard and mouse events
(using direct apis, like `glfwGetMouseButton` or callback apis like `glfwSetMouseButtonCallback`)

- canvas1 is hi dpi aware (`glfwWindowHint(GLFW_SCALE_TO_MONITOR, GLFW_TRUE)`)
- canvas2 is **not** hi dpi aware (but can be made so with the "Enable" Hi DPI Aware button)
- canvas2 is fully resizable (use the square handle to resize) (`emscripten_glfw_make_canvas_resizable(window2, "#canvas2-container", "#canvas2-handle")`)

You can enable/disable each window/canvas independently:

- When 2 (or more) canvases are present, the canvas that has focus can receive keyboard events. If no other element on 
  the page has focus, then the last canvas that had the focus will receive these events. Clicking with the left mouse 
  button on a canvas gives it focus.
- When there is only 1 canvas, the implementation try to be smart about it and will route keyboard (and other relevant) 
  events to the single canvas provided that nothing else has focus (the 'Change focus/Text' field is used to test 
  this feature since clicking in the text field grabs the focus).

The demo uses webgl to render a triangle (the hellow world of gpu rendering...).

Examples
--------

<table>
<thead>
<tr><th>Example</th><th>Note</th></tr>
</thead>
  <tbody>
  <tr><td><a href="https://pongasoft.github.io/emscripten-glfw/test/demo/main.html">Demo</a> (<a href="test/demo">src</a>)</td><td>Main test/demo which demonstrates most features of the implementation</td></tr>
  <tr>
    <td><a href="https://pongasoft.github.io/emscripten-glfw/examples/example_asyncify/main.html">example_asyncify</a> (<a href="examples/example_asyncify">src</a>)</td>
    <td>The purpose of this example is to demonstrate how to use asyncify which allows the code to be written like you
      would for a normal desktop application</td>
  </tr>
  <tr>
    <td><a href="https://pongasoft.github.io/emscripten-glfw/examples/example_hi_dpi/main.html">example_hi_dpi</a> (<a href="examples/example_hi_dpi">src</a>)</td>
    <td>The purpose of this example is to demonstrate how to make the window Hi DPI aware</td>
  </tr>
  <tr>
    <td><a href="https://pongasoft.github.io/emscripten-glfw/examples/example_minimal/main.html">example_minimal</a> (<a href="examples/example_minimal">src</a>)</td>
    <td>The purpose of this example is to be as minimal as possible: initializes glfw, create window, then destroy it and terminate glfw.
      Uses the default shell that comes with emscripten</td>
  </tr>
  <tr>
    <td><a href="https://pongasoft.github.io/emscripten-glfw/examples/example_resizable_container/main.html">example_resizable_container</a> (<a href="examples/example_resizable_container">src</a>)</td>
    <td>The purpose of this example is to demonstrate how to make the canvas resizable with another container (a
      surrounding div) driving its size. The container width is proportional to the size of the window and so as the
      window gets resized so does the div and so does the canvas</td>
  </tr>
  <tr>
    <td><a href="https://pongasoft.github.io/emscripten-glfw/examples/example_resizable_container_with_handle/main.html">example_resizable_container_with_handle</a> (<a href="examples/example_resizable_container_with_handle">src</a>)</td>
    <td>The purpose of this example is to demonstrate how to make the canvas resizable with a container that has a handle.
      The handle can be dragged around (left mouse drag) and the div is resized accordingly which in turn resizes the
      canvas, making the canvas truly resizable like a window</td>
  </tr>
  <tr>
    <td><a href="https://pongasoft.github.io/emscripten-glfw/examples/example_resizable_full_window/main.html">example_resizable_full_window</a> (<a href="examples/example_resizable_full_window">src</a>)</td>
    <td>The purpose of this example is to demonstrate how to make the canvas resizable and occupy the full window</td>
  </tr>
  </tbody>
</table>


Usage
-----

Check the [Usage](docs/Usage.md) documentation for details on how to use this implementation. Note that care has been
taken to backward compatible with the pure javascript implementation built-in in emscripten.

Building
--------

### Using emscripten port

Since emscripten 3.1.54, using this port is really easy via the `--use-port=contrib.glfw3` option.

Example:

```sh
emcc --use-port=contrib.glfw3 main.cpp -o build/index.html
```

The port can be configured with the following options:

| Option               | Description                                                                                               |
|----------------------|-----------------------------------------------------------------------------------------------------------|
| `disableJoystick`    | Boolean to disable support for joystick entirely, which can be useful if you don't need it due to polling |
| `disableWarning`     | Boolean to disable warnings emitted by the library (for example when using non supported features)        |
| `disableMultiWindow` | Boolean to disable multi window support which makes the code smaller and faster if you don't need it      |

Example using `disableWarning` and `disableMultiWindow`:

```sh
emcc --use-port=contrib.glfw3:disableWarning=true:disableMultiWindow=true main.cpp -o build
```

Release Notes
-------------

#### 1.0.4 - 2024/01/24

- Fixed version string

#### 1.0.3 - 2024/01/23

- Fixed `emscripten_glfw3.h` to work as a C file
- Added `create-archive` target

#### 1.0.2 - 2024/01/22

- Added `EMSCRIPTEN_GLFW3_DISABLE_JOYSTICK` as a CMake option
- Made joystick code truly conditional on `EMSCRIPTEN_GLFW3_DISABLE_JOYSTICK` define
- Added `EMSCRIPTEN_GLFW3_DISABLE_MULTI_WINDOW_SUPPORT` as a CMake option and made the multi window code conditional
  on `EMSCRIPTEN_GLFW3_DISABLE_MULTI_WINDOW_SUPPORT` define
- Misc: added github workflow to compile the code/display badge

#### 1.0.1 - 2024/01/21
 
- Fixed import

#### 1.0.0 - 2024/01/21

- First 1.0.0 release
- Added examples
- Added documentation
- Fixed some issues
- Removed `GLFW_EMSCRIPTEN_CANVAS_SELECTOR` window hint in favor of a new api `emscripten_glfw_set_next_window_canvas_selector`
- Removed `GLFW_EMSCRIPTEN_CANVAS_RESIZE_SELECTOR` and `Module.glfwSetCanvasResizableSelector` in favor of a new 
  api `emscripten_glfw_make_canvas_resizable`
- This new api also offer the ability to deal with the handle automatically
- Implemented `getWindowPosition` (canvas position in the browser window)
- Implemented all timer apis (`glfwSetTime`, `glfwGetTimerValue` and `glfwGetTimerFrequency`)
- Implemented `glfwExtensionSupported`
- Implemented `glfwSetWindowTitle` (changes the browser window title)

#### wip-0.5.0 - 2024/01/12

- Added support for resizable canvas (`glfwWindowHintString(GLFW_EMSCRIPTEN_CANVAS_RESIZE_SELECTOR, "#canvas2-container")` 
  from c/c++ code or `Module.glfwSetCanvasResizableSelector('#canvas2', '#canvas2-container')` from javascript) 
- Added support fo visibility (`glfwShowWindow` and `glfwHideWindow`)
- Added support for `GLFW_FOCUS_ON_SHOW` window hint/attribute
- Added support for dynamic Hi DPI Awareness (`GLFW_SCALE_TO_MONITOR` can be used in `glfwSetWindowAttrib`)
- Added support for "sticky" mouse button and keyboard
- Added support for window size constraints (`glfwSetWindowSizeLimits` and `glfwSetWindowAspectRatio`)
- Added support for providing a callback function in javascript to be notified when a window is created (`Module.glfwOnWindowCreated`)

#### wip-0.4.0 - 2024/01/03

- Added support for joystick/gamepad
  - Joystick support can be disabled via `EMSCRIPTEN_GLFW3_DISABLE_JOYSTICK` compilation flag

#### wip-0.3.0 - 2023/12/31

- Added support for input mode `GLFW_CURSOR` (handle all use cases: Normal / Hidden / Locked)
- Added support for glfw defined cursors (implemented `glfwCreateStandardCursor` and `glfwSetCursor`)
- Added support for window opacity (implemented `glfwGetWindowOpacity` and `glfwSetWindowOpacity`)

#### wip-0.2.0 - 2023/12/28

- remembers the last window that had focus so that some events can be sent to it even if no window has 
  focus (ex: requesting fullscreen)
- added support for mouse wheel (`glfwSetScrollCallback`)
- added support for mouse enter/leave (`glfwSetCursorEnterCallback`)

#### wip-0.1.0 - 2023/12/26

- first public version


Misc
----

This project includes the `glfw3.h` header (`external/GLFW/glfw3.h`) which uses a [ZLib license](https://www.glfw.org/license.html)

Licensing
---------

- Apache 2.0 License. This project can be used according to the terms of the Apache 2.0 license.
