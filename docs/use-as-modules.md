# Use as node modules

## Overview

![overview](./resources/architecture.png)


`textlint` module expose these header at [index.js](../src/index.js)

```js
// Level of abstraction(descending order)
// cli > TextLintEngine > TextLintCore(textlint)
// See: https://github.com/textlint/textlint/blob/master/docs/use-as-modules.md
module.exports = {
    // Command line interface
    cli: require("./cli"),
    // TextLintEngine is a wrapper around `textlint` for linting **multiple** files
    // include formatter, detecting utils
    // <Recommend>: It is easy to use
    // You can see engine/textlint-engine-core.js for more detail
    TextLintEngine: require("./textlint-engine"),
    // TextFixEngine is a wrapper around `textlint` for linting **multiple** files
    // include formatter, detecting utils
    // <Recommend>: It is easy to use
    // You can see engine/textlint-engine-core.js for more detail
    TextFixEngine: require("./textfix-engine"),
    // It is a singleton object of TextLintCore
    // Recommend: use TextLintCore
    textlint: require("./textlint"),
    // Core API for linting a **single** text or file.
    TextLintCore: require("./textlint-core")
};

```

Recommend to use `TextLintEngine`.

## Architecture

See [src/README.md](../src/README.md) for details.

### CLI(Command Line Interface)

CLI parse command arguments, and run Engine with the options.

### Engine

textlint has two engines `TextLintEngine` and `TextFixEngine`.

Both engine

- handle **multiple** files or text string.
- return a array of `TextLintResult` or `TextLintFixResult`
    - actually, return a Promise like `Promise<TextLintResult[]>`

### Core

textlint's core 

- handle a **single** file or text string.
- return `TextLintResult` or `TextLintFixResult`
    - actually, return a Promise like `Promise<TextLintResult>`

## Example

Lint a file:

See [example/node-module/lint-file.js](example/node-module/lint-file.js)

```js
var TextLintEngine = require("textlint").TextLintEngine;
var path = require("path");
function lintFile(filePath) {
    /**
     * TextLintConfig
     * See https://github.com/textlint/textlint/blob/master/typings/textlint.d.ts
     */
    var options = {
        // load rules from [../rules]
        rulePaths: [path.join(__dirname, "..", "rules/")],
        formatterName: "pretty-error"
    };
    var engine = new TextLintEngine(options);
    var filePathList = [path.resolve(process.cwd(), filePath)];
    engine.executeOnFiles(filePathList).then(function(results){
        /* 
        See https://github.com/textlint/textlint/blob/master/typings/textlint.d.ts
        messages are TextLintMessage` array.
        [
            "filePath": "path/to/file",
            "messages" :[
                {
                    id: "rule-name",
                    message:"lint message",
                    line: 1, // 1-based columns(TextLintMessage)
                    column:1 // 1-based columns(TextLintMessage)
                }
            ]
        ]
         */
        if (engine.isErrorResults(results)) {
            var output = engine.formatResults(results);
            console.log(output);
        }
    }).catch(function(error){
        console.error(error);
    });
}
```

## Testing

You can use [textlint-tester](https://github.com/textlint/textlint-tester "textlint-tester") for testing your custom rule.

- [textlint-tester](https://github.com/textlint/textlint-tester "textlint-tester")
- [rule.md](./rule.md)

Consult link: [spellcheck-tech-word-textlint-rule/test.js at master · azu/spellcheck-tech-word-textlint-rule](https://github.com/azu/spellcheck-tech-word-textlint-rule/blob/master/test/test.js "spellcheck-tech-word-textlint-rule/test.js at master · azu/spellcheck-tech-word-textlint-rule")
