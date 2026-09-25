<div align="right">

English | [简体中文](README.zh-CN.md)

</div>

# PalatMind — Windows Desktop AI Agent

PalatMind is an AI agent that runs on Windows and uses your computer the way a person does — it sees the screen, understands the page, and clicks with precision. Give it a single sentence and it becomes real, step-by-step operations on your desktop. Workflows that succeed once can be saved as one-click replayable automation scripts.

- Download: https://palatmind.com/download/
- Documentation: https://palatmind.com/docs/
- Questions & feedback: [Issues](https://github.com/tianyuleishen/palatmind-agent/issues)

---

# Making GUI Automation Precise: From Seeing the Screen to Replayable Scripts

For an AI operating a computer, the hard part is never "calling the mouse API" — it is **seeing clearly, locating precisely, and clicking reliably**. The full pipeline: OCR & icon recognition → page understanding → precise mouse control → step-by-step execution → distilling successful runs into scripts.

## 1. Seeing the Screen: OCR + a Small Icon-Element Model

An agent's first job is turning pixels into structured information. Relying on a large model to "look at the picture" is not enough: small text gets misread, icon semantics get guessed wrong, coordinates drift. Our approach is dual-channel perception: **precise OCR + a lightweight icon-element model**.

### 1.1 OCR: get the text together with its coordinates

Feeding a raw screenshot straight into OCR works poorly: system UI text is tiny, dark themes have low contrast, and high-DPI scaling complicates everything. Before recognition, we:

- **Upscale preprocessing**: 2x upscaling dramatically improves recognition of small UI text (taskbar, menus at ~12px);
- **Layered recognition**: a full-screen pass captures the global layout, then low-confidence regions are cropped, upscaled, and re-recognized;
- **Structured output**: not just text — every text block comes with its **bounding box + confidence**, which everything downstream depends on.

```text
# Pseudocode: structured OCR output
TextBlock {
    text: "Settings",
    box: (x, y, w, h),     # position on the screenshot
    confidence: 0.97
}
```

### 1.2 The small icon-element model: for what OCR can't read

Many clickable elements carry no text at all: the × close button, hamburger menus, toolbar icons, image buttons. For these we use a **lightweight object detection model** (YOLO-class) plus **icon semantic classification**:

- **Detect**: box every UI element on screen (buttons, inputs, icons, toggles…);
- **Classify**: judge each box's type and semantics (that's a "search icon", that's a "play button");
- **Keep it light**: the model must be small and fast — a single screen should be processed within a second, or the agent spends its life waiting.

### 1.3 Fusing the two channels

OCR text blocks + model icon boxes merge into a single **element list**, each entry carrying: type, text/semantics, coordinates, confidence. That list is the input to page understanding.

## 2. Understanding the Page: Knowing Where the Target Element Lives

An element list is only the beginning. The user says "click Settings" and the screen has 20 elements — which one? The core of this stage is **reasoning from the element list to a target coordinate**.

### 2.1 Layout zoning

First segment the screen into structured regions: title bar, sidebar, main content, status bar. "Settings" is most likely in the sidebar or title bar, not the middle of the body — regional priors massively shrink the search space.

### 2.2 Row grouping

File lists, chat threads, search results — in these UIs a row is one logical whole (icon + title + summary + actions). Elements on the same row must be **merged into a single logical unit**, otherwise you get "clicked the blank space inside the row" or "clicked the neighboring row" accidents.

```text
# Pseudocode: group elements into rows by y-axis proximity
if vertical_gap < threshold and heights are similar:
    merge into the same RowGroup
click_target = RowGroup's clickable area
```

### 2.3 Parent-child containment and state

- **Parent-child**: an icon belongs to a button, a label belongs to a card — clicking the child actually means clicking the parent's clickable area;
- **State**: is the button disabled? Is a dialog covering the target? Is the input focused? Operating on the wrong state is guaranteed failure.

### 2.4 Output: a target coordinate

Page understanding ultimately produces something very plain — **a coordinate + confidence**. All the reasoning above (semantic matching, regional priors, row grouping, state filtering) exists to compute that coordinate and attach "how sure I am". When confidence is low, prefer confirmation or retry over a blind click.

## 3. Clicking with Precision

The coordinate is computed — but the mouse operation itself is full of details. Get this wrong and everything above is wasted.

### 3.1 Match the operation to the scenario

| Scenario | Operation |
|----------|-----------|
| Open a file / confirm button | Single click |
| Launch a desktop icon / enter a folder | Double click |
| Open a context menu | Right click |
| Move a file / resize a window | Press → move → release (drag) |
| Scroll a long page | Wheel (mind direction and amount) |

### 3.2 A double click is not just "two fast clicks"

Windows has a system-level `DoubleClickTime` (~500ms by default) and a `DoubleClickSize` tolerance zone. Programmatic double clicks must land both clicks within those thresholds, or the system reads them as two single clicks. The opposite failure also exists: click too fast and the app ignores the second one.

### 3.3 Drags need interpolated segments

Teleporting the cursor from A to B and releasing will simply not register in many apps (custom-drawn UIs, game windows, web drag-and-drop). The right way:

```text
# Pseudocode: drag with interpolated segments
mouse_down(start)
loop: move 10~20px per step, sleep a few ms between steps  # mimic a human hand
mouse_up(end)
```

### 3.4 Verify immediately after every click

After each click, check right away that the expected effect appeared: did the window title change? Did the dialog open? Did the input get focus? **A click without verification is a landmine.** On failure, retry immediately instead of discovering at the end of the task that step one already went wrong.

## 4. Step-by-Step Execution

Once individual operations are precise, the task level needs a reliable execution framework.

### 4.1 Decompose the task into steps

A one-sentence task becomes an explicit step sequence, each step carrying: action, target (element description / coordinates), expected result. "Send the report on my desktop to Zhang San" becomes: open WeChat → search Zhang San → open the chat → click send-file → choose Desktop → double-click report.jpg → click send.

### 4.2 The perceive → locate → act → verify loop

Every step runs through the same closed loop:

```text
# Pseudocode: single-step execution loop
loop:
    screenshot → OCR + small model → element list     # perceive
    locate the target → coordinate + confidence       # locate
    perform the mouse operation                        # act
    verify the expected result appeared                # verify
    success → next step; failure → retry or adapt
```

### 4.3 Handling snags

Real desktops are full of surprises: popups appear out of nowhere, targets get covered, apps load slowly, networks stall. The framework must have:

- **Timeouts and retries**: per-step timeouts; on failure, retry with re-localization rather than re-clicking the same blind coordinate;
- **Popup handling**: recognize an unexpected dialog before deciding — dismiss it, route around it, or use it;
- **Fail fast**: after several consecutive verification failures, stop and report instead of "clicking into a bigger mess" with irreversible consequences.

## 5. Turning Successful Runs into Automation Scripts

The second time a task runs, it should not re-do all the perception and reasoning from scratch. We distill **verified execution traces into replayable scripts** — the single biggest speedup in the whole system.

### 5.1 What a script stores

Not a screen recording, but structured **steps + verification anchors + parameters**:

```text
# Pseudocode: structure of a distilled script
Step {
    action: "click",
    target: { semantic: "search button", anchor: "magnifier icon at the window's top-right" },
    verify: "search box has focus",     # anchor: proof this step succeeded
    params: { text: "{song_name}" }     # parameterized: replaceable next time
}
```

### 5.2 Second-level replay

A first run of "play a song in the music player" may take 1~2 minutes (perception, reasoning, trial and error). Script replay executes the steps directly and finishes **within 10 seconds** — all the "look at the screen and figure it out" time is gone, only necessary loading waits remain.

### 5.3 Anchor self-healing: scripts are not brittle recordings

The UI gets redesigned, buttons move — then what? The key is **verification anchors + re-localization**:

- Before each step, look up the element by anchor (semantic description + spatial relation);
- Found → operate using the recorded relative position (fast);
- Not found → fall back to real-time perception, then **update the script's anchor** (self-healing).

Scripts keep replay speed while real-time perception acts as the safety net — one UI redesign no longer invalidates everything.

### 5.4 Only save the successful

Only execution traces that **passed verification** may be distilled into scripts. Baking failed traces into a script means preserving the mistake and replaying it forever.

---

## Open-Source Technologies We Build On

The implementation of the pipeline above benefits greatly from these outstanding open-source projects and system interfaces (sources acknowledged):

| Technology | Source | Role in the pipeline |
|------------|--------|----------------------|
| **UI-TARS** | ByteDance · [github.com/bytedance/UI-TARS](https://github.com/bytedance/UI-TARS) | Reference for GUI screenshot understanding and visual element grounding |
| **PaddleOCR** | Baidu · [github.com/PaddlePaddle/PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) | On-screen text recognition (text + box + confidence) |
| **YOLO / Ultralytics** | Ultralytics · [github.com/ultralytics/ultralytics](https://github.com/ultralytics/ultralytics) | Lightweight UI element / icon detection |
| **Windows UI Automation** | Microsoft system API · [official docs](https://learn.microsoft.com/windows/win32/winauto/ui-automation-entry-page) | Control tree reading, element properties and Patterns — complements the visual channel |

---

## Try PalatMind

- Download: https://palatmind.com/download/
- Documentation: https://palatmind.com/docs/
- Questions or GUI automation discussions: [open an Issue](https://github.com/tianyuleishen/palatmind-agent/issues)
