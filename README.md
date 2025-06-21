# Universal-Automation-Protocol
Universal Automation Protocol(UAP) is an open standard created by HuBrowser to define and run automation tasks on any device or platform

**UAP makes automation accessible, portable, and future-proof across platforms.**

Universal Automation Protocol (UAP) is an open standard created by HuBrowser to define and run automation tasks on any device or platform. Whether you’re a beginner or an expert, UAP lets you record, import, and export automation tasks in a simple, readable format. Run your automations with any UAP-compatible app—no vendor lock-in.

If you do not have technical knowledge, you can still read this page and easily understand and write your own automation steps.
If you have technical knowledge, you will see how this fits into enterprise UI testing and cross-platform automation.


-   **Human-friendly:** Uses `JSON` for easy reading and writing.
-   **Type-safe:** Protocol types are defined in `TypeScript` for reliability.
-   **AI-first:** Designed for the new era of mobile AI agents, solving challenges that legacy standards like Playwright or Selenium can’t.
-   **Cross-platform:** Enables sharing automation scripts between platforms—even if HuBrowser doesn’t support iPhone, your scripts can still work.

## Terms

-   **Mode**: 3 [modes](/guide/features/parrot-assistant/agent/highlights#%EF%B8%8F-adaptability-meets-predictability) we support:
    -   `instruction`: Natural Language instruction for the action
    -   `recording`: Recorded automation steps
    -   `script`: Scripted automation steps
-   **Harden**: Switches from a more flexible mode to a more deterministic mode (e.g. from `instruction` to `script`), making the automation faster and more precise.
-   **Soften**: Switches from a more deterministic mode to a more flexible mode (e.g. from `script` to `instruction`), making the automation more adaptable but potentially slower.

## Example: UAP Action (JSON)

```json
[
	{
		"command": "goal",
		"goal": "Submit the form"
	},
	{
		"command": "tap",
		"selector": "#submit-button",
		"displayMsg": "Tap the submit button"
	},
	{
		"command": "wait",
		"duration": 2000,
		"displayMsg": "Wait for 2 seconds"
	},
	{
		"command": "assert",
		"assertion": {
			"type": "visible",
			"selector": "#success-message"
		},
		"displayMsg": "Check if success message is visible"
	}
]
```

This example shows a simple form fill workflow. UAP actions are easy to write and understand.

## Automation Config (TypeScript)

```js
/**
 * Types for AI actions (specs only)
 * Adapt best practices from Playwright and Puppeteer for AI era
 */

// Common base for all AI actions
export interface CommonAIAction {
	/**
	 * The command to execute (e.g. 'tap', 'wait', 'open_link', etc)
	 */
	command: string
	/**
	 * Optional message to display to the user
	 */
	displayMsg?: string
	/**
	 * If true, do not wait for this action to finish before proceeding to the next task (AI era best practice)
	 * Defaults to false
	 */
	noWait?: boolean
	/**
	 * Wait strategy
	 */
	waitUntil?:
		| 'domcontentloaded'
		| 'load'
		| 'networkidle'
		| 'visible'
		| 'hidden'
}

export interface AIGoalAction extends CommonAIAction {
	command: 'goal'
	goal: string // natural language goal
}

export interface AITapAction extends CommonAIAction {
	command: 'tap'
	selector: string
}

export interface AIHoverAction extends CommonAIAction {
	command: 'hover'
	selector: string
}

export interface AIDragAction extends CommonAIAction {
	command: 'drag'
	sourceSelector: string
	targetSelector: string
}

export interface AITypeAction extends CommonAIAction {
	command: 'type'
	selector: string
	text: string
}

export interface AIKeyboardPressAction extends CommonAIAction {
	command: 'keyboard_press'
	keys: string[]
}

export interface AIScrollAction extends CommonAIAction {
	command: 'scroll'
	selector?: string
	direction: 'up' | 'down' | 'left' | 'right'
	amount?: number // pixels or percent
}

export interface AIScrollToAction extends CommonAIAction {
	command: 'scroll_to'
	selector?: string
	edge: 'top' | 'bottom' | 'left' | 'right'
}

export interface AIWaitAction extends CommonAIAction {
	command: 'wait'
	duration: number // milliseconds
}

export interface AIOpenLinkAction extends CommonAIAction {
	command: 'open_link'
	url: string
}

export interface AIZoomAction extends CommonAIAction {
	command: 'zoom'
	zoomLevel: number // e.g. 1.0 = 100%, 2.0 = 200%
	selector?: string // optional, for zooming a specific element
}

export interface AIMultitouchAction extends CommonAIAction {
	command: 'multitouch'
	events: Array<{
		type: 'pinch' | 'spread' | 'rotate' | 'swipe'
		start: { x: number; y: number }
		end: { x: number; y: number }
		fingers?: number // default 2
		angle?: number // for rotate
		amount?: number // for pinch/spread (pixels or percent)
	}>
	selector?: string // optional, for targeting a specific element
}

export interface AIGestureAction extends CommonAIAction {
	command: 'gesture'
	gestureType:
		| 'swipe'
		| 'flick'
		| 'longpress'
		| 'doubletap'
		| 'press'
		| 'pan'
		| 'rotate' // exclude 'pinch' and 'zoom' to avoid overlap
	start: { x: number; y: number }
	end?: { x: number; y: number }
	duration?: number // ms, for longpress or press
	fingers?: number // default 1
	angle?: number // for rotate
	selector?: string // optional, for targeting a specific element
}

export interface AIGoToTabAction extends CommonAIAction {
	command: 'go_to_tab'
	description: string
}

/**
 * AISetValueAction: Set the value of an input or element.
 */
export interface AISetValueAction extends CommonAIAction {
  command: 'set_value'
  selector: string
  value: string | number | boolean
}

/**
 * AIAssertAction: Assert a condition on the UI or data.
 */
export interface AIAssertAction extends CommonAIAction {
  command: 'assert'
  assertion:
	| { type: 'visible'; selector: string }
	| { type: 'not_visible'; selector: string }
	| { type: 'text_equals'; selector: string; value: string }
	| { type: 'value_equals'; selector: string; value: string | number | boolean }
}

/**
 * AIActionType: Accepts any of the known action types, or a custom command with arbitrary params.
 * This enables extension with custom commands and parameters.
 */
export type AIActionType =
	// stochastic (random) actions
	| AIGoalAction
	// deterministic actions
	| AITapAction
	| AIWaitAction
	| AIHoverAction
	| AIDragAction
	| AITypeAction
	| AIKeyboardPressAction
	| AIScrollAction
	| AIScrollToAction
	| AIOpenLinkAction
	| AIZoomAction
	| AIMultitouchAction
	| AIGestureAction
	| AIGoToTabAction
	| CustomAIAction
	| AISetValueAction
	| AIAssertAction

/**
 * CustomAIAction: Allows any string as command and arbitrary params.
 */
export interface CustomAIAction extends CommonAIAction {
	command: string // any string
	[name: string]: any // allow arbitrary params
}
```

## General Structure Control

UAP uses familiar schema concepts to keep definitions clear and extensible:

-   **properties**: Defines the attributes of an object.
-   **anyOf**: Supports multiple possible types or structures.
-   **required**: Specifies fields that must be provided.
-   **type**: Restricts the data type of a value.
-   **default**: Provides a default value.
-   **item_count_range**: Controls the minimum and maximum length of an array.

### Tool Use

UAP also supports describing tool and function calls in a structured way:

```json
{
	"tool_choice": { "function": { "name": "" } },
	"tools": [
		{
			"function": {
				"description": "",
				"name": "",
				"parameters": {}
			},
			"type": "function"
		}
	]
}
```

This structure defines how tools and functions are described in UAP.

## Maintenance and Versioning

-   **version**: v25.1
-   Open Source License: [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)
-   AI evolves quickly, but we try to keep changes predictable:
    -   At most 1 breaking change in 2 months, when you need to redownload the automation protocol file.
-   **Graceful fallback:** If an environment doesn’t support a certain action, UAP-compatible apps should ensure your automation won’t break unexpectedly.
