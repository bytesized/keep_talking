# "Keep Talking and Nobody Explodes" Expert's Tool

This tool is a simple one-page web app that is meant to replace the Bomb Defusal Manual from the game "Keep Talking and Nobody Explodes". I would not recommend using it without first being familiar with the original manual. Although the tool does give instructions, it may still be unclear how to use some parts of the tool if you are not already familiar with the game.

I often play the game remotely, and I wanted to easily be able to share the tool without having to  distribute a whole bunch of files or give any kind of installation instructions, so I made the (perhaps questionable) decision to put the entire tool in a single HTML file.

## Requirements

I would recommend using this with Firefox. It's entirely possible that it works with other web browsers, but I couldn't say for sure at the moment.

The tool only uses standard front-end web technologies (HTML, CSS, JS), so no web server is needed.

## Running the Tool

Simply download the HTML file and open it in the browser. I may eventually set up Github to host this file so that you can visit it without downloading it to a local file first. But I have not yet made it available that way.

## Usage

I think that pretty much all of the module tools should be sufficiently explained by their included instructions, but I will make a few notes here:

### Navigation

Although the tool initially loads a list of all the module tools, there isn't a good way to get back to this page. It's not really the quickest way to find them anyways. The search bar at the top of the page is pretty simplistic, but it seems to work pretty well. It should be excellent if you input the correct name of the module. It also recognizes a few other related words, as well as any text that is on that module, in an effort to make them a bit easier to find. It does not, however, have the ability to correct spelling. But it usually turns up the desired page within a couple of key presses anyways, so that doesn't seem like a big deal.

### Common Wires

The tool will always ask you for all the wires, in order. Strictly speaking, this isn't necessary; some wire values will not end up being used. But I think that the bomb expert having to communicate which wire colors are needed is more time consuming than the bomb technician just listing them all.

### Morse Code

This has always been the most frustrating module for me. It just seems like it's surprisingly hard for the bomb technician to read the Morse out correctly. This tool attempts to make things a bit easier by attempting some error correction. More required corrections will be reflected by a lower "Accuracy" value in the word table. In general, the tool is most effective if you avoid entering partial letters. That is to say, you shouldn't start or end in the middle of a letter.

### Complicated Wires

This module tool works a bit differently to other, similar ones. It starts with all the buttons pre-selected. It seems inefficient to communicate the things that the wire doesn't have, so I didn't want to need to press buttons for those things. By starting out will all the qualities set to not be present, you can just click the buttons for the qualities that the wires do have. Remember to press the "Reset" button between wires.

### Passwords

This module tool will work perfectly fine if you read out all the wheels in order, but it tries to suggest which wheels may offer more information by moving the cursor to that input. For example, if all the remaining possible words have the same second character, knowing the possibilities for that character slot won't offer much information. So the module tool will suggest asking for the possible characters in a different slot first.

### Knobs

This module tool will always open in a popup. This allows you to open it without losing your place in the module you are currently working on. This can be very useful if the knob needs to be dealt with when you are in the middle of solving another module. Just click the X in the top right corner to go back to the module you were in the middle of solving

### New Bomb

The tool remembers things between modules that do not change over the course of the bomb (ex: the number of batteries). The "New Bomb" page can be used to force the tool to forget those things.

## Implementation

Some of the module tools are one-offs, but most of them (pretty much all the ones with the sequences of questions answered with big, blue radio buttons) use a little framework that I wrote called an `InstructionTree`. It got a little convoluted and hacky, but it does have some nice features.

It's simplest possible usage is to have an instruction (a `prompt`) with a bunch of choices (`suboptions`). Each suboption is its own `InstructionTree` description, potentially with its own `prompt` and `suboptions`. This could look something like this:

```
"Do you like chocolate ice cream"?
Yes ->
        "Chocolate ice cream for you!"
No  ->
        "Do you like strawberry?"
        Yes ->
                "Strawberry ice cream for you!"
        No  ->
                "That's all we've got, sorry."
```

In practice, none of them really end up being this static. Often we want to do things like having all paths lead to the same next prompt, but then be able to figure out which button was pressed later. The `InstructionTree` description options that allow this are all explained in `InstructionTree`'s header comment.

If the answer to an already-answered prompt is changed (say a mistake was made on Simon, so the strike count from the first prompt is changed), the `InstructionTree` framework will backtrack, undoing the prompts that were shown since then and the buttons that were selected. Those elements will be removed from the DOM, as well as the corresponding `InstructionTree` state. This is necessary, because changing the answer to a question _may_ change the path we take through the `InstructionTree`. However, it will sometimes be appropriate for the values picked to be saved in some way. The `InstructionTree` can then include a `prepicked_suboption` function, which allows a suboption to be picked without requiring user interaction. This often allows it to _look_ like you are just changing the answer to a previous prompt without changing any of the subsequent ones. But, under the hood, those decisions are being undone and re-chosen. This ensures that we are always on the correct path of the `InstructionTree`.

In order for many of these features to work, functions included in the `InstructionTree` need to be able to store some state. The `InstructionTree` provides access to places to store state that will be automatically managed. The functions can access the state through the `InstructionTree` itself, which will be provided as the first argument to all of these functions, and is typically called `it`. This first is `it.state`. This will initially be an empty object, and will be automatically reset when the module tool page's reset button is pressed (or after navigation away from and back to the page). The second is `it.cds_state`, which stands for "Current Depth-Specific State". This state will be forgotten if the `InstructionTree` has to backtrack past the stage when it was set. Previous `cds_state` objects can be accessed by iterating through `it.depth_specific_state`. This makes it possible, for example, for us to use the same `InstructionTree` description many times in a row, and then search back through `it.depth_specific_state` to see how many times we've done it so far.
