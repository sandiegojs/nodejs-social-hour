# AST Parsing Challenge

This challenge sounds more complex than it really is, we're going to use babylon and jscodeshift to read our bespoke javascript and convert it into an AST that operates like a weird hybrid between the dom, beautiful soup output and jquery dom manipulation.

Tools like [ASTExplorer.net](http://astexplorer.net) make it easy to test out an ast transform without having to install lots of tools.

Here is a relatively low-key transform, it HULKifies all your variables:

```javascript
// Press ctrl+space for code completion
export default function transformer(file, api) {
  const j = api.jscodeshift;

  return j(file.source).find(j.Identifier).forEach(path => {
    const HULKNAME = path.node.name.toUpperCase() + '_SMASH';

    const pathName = path.name
    const pathType = path.parent.node.type;

    const notCall = pathType !== 'CallExpression';
    const notType = pathType !== 'GenericTypeAnnotation';
    const notProperty = pathName !== 'property';

    if (notCall && notType && notProperty) {
    j(path).replaceWith(j.identifier(HULKNAME))
    }

  })
  .toSource();
}
```

Notes:

- most operations will return a `path` which always has a `node` property. The `node` is the thing you see in AST Tree view in the top right pane in ASTExplorer.net
- ctrl+space is essential for figuring out what methods are availabe to the property you are inspecting.
- accessing a `.parent` will return a `path`, so you will always have to manually check the node to reason about it
- like jquery, you can always call `j()` on any path you have a handle on to perform action
- `j` has two sorts of properties: `j.identifier` is an Identifier factory and `j.Identifier` is a _type definition_. it's easy to switch the two and have a failing query.
- `path.name` refers to the name of the _property name_ on the parent node.
  ![](http://dl.dropboxusercontent.com/u/406291/Screenshots/w3bc.png)
  in this example there are two `Identifier`s, one is the parent's `object` and the selected one is the parent's `property`. (for example, `input.map`, `map` is the selected `Identifier`. Its `.name` is `"property"` and its `.node.name` is `"map"`)
- `path.node.name` will usually be the variable name, function name, etc (and will only be present on nodes that have names)

Challenges:

## better variables

Oftentimes when writing map functions, people write code like

```javascript
// @flow

const baz = (input: str[]) => input.map((el, idx) => `${el}:${idx}`);

const qux = (input: str[]) => input.reduce(function(acc, val, idx) { return `${el}:${idx}`});

const idx = (foo: str) => idx(`${foo}${foo}`);
```

but what _is_ `idx`? i think the developer meant `index`.

**challenge 1** Write a codemod that replaces all `"idx"` `Identifier`s with the easier to read `index`.

> you can build off the earlier hulk sample code to do this except you will want to look at `path.node.name`

**challenge 1a** Update your codemod so that it doesn't rename the third function named `idx` (who knows why it was named that, anyway).

**challenge 2** Write a codemod to troll your friends that replaces all "tion" identifiers like "caption", "mention", "conclusion" to their funnier "conclusino", "mentino", "captino" etc. but leaves _"option"_ unaltered...

**challenge 3** write a codemod that converts `_.map(arr, fn)` calls to `arr.map(fn)`

**challenge 4** write a codemod that converts import from `index.js` or `index` to python-style imports from the package: i.e. `import Foo from './src/foo/index.js'` to `import Foo from './src/foo';`

**challenge 5** write a codemod that inserts module imports for all lodash methods used in a file. I.e. if you have code like `_.mapValues(obj, someFn)` the codemod should add `import mapValues from 'lodash/mapValues'`;

**challenge 5a** update the last codemod to also rename lodash methods from `_.mapValues(obj, someFn)` -> `mapValues(obj, someFn)`

**challenge 6** Code analysis: write a codemod that adds a comment to the top of each file that gives stats about how long (in lines) each function in the file is. (bonus: include average + std dev function length)

**challenge 7** Alphabetize the imports in a file.

**challenge 8** sort the imports in a file by line-length (i.e.)
  ```javascript
  import Foo from 'bar';
  import BazQux from 'baz-qux';
  ```
