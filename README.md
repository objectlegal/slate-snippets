# slate-snippets
Slate snippets

# General

## Queries

### Paths
#### Current anchor path
```js
editor.selection.anchor.path
```


#### Trim a path to get the parent (e.g. transform [4,2,3] to [4,2])
```js
const parentPath = Path.parent(path);
```

### Nodes, domNode, path
#### Get node by path
```js
const node = Node.get(editor, path)
```

#### Get path by node
```js
const path = ReactEditor.findPath(editor, node)
```

#### Find a node's DOM node
```js
const domNode = ReactEditor.toDOMNode(editor, node)
```

#### Get closest block
```js
const [blockNode, path] = Editor.above(editor, {match: n => editor.isBlock(n)})
```

#### Get parent and path of node
```js
const [node, path] = Editor.parent(editor, nodePath)  // <- using specific node path, see above how to get paths
```

#### move the cursor to the end of the document
```js
ReactEditor.focus(editor);
Transforms.select(editor, Editor.end(editor, []));
```

## Commands

#### Insert text at selection
```js
Transforms.insertText(editor, 'some text');
```

#### Insert nodes at selection
```js
Transforms.insertNodes(editor, [
    {type:'inline_type', children:[{text: 'some text', marks:[]}]},
    {text: ' and some text after the inline', marks: []}
  ]
);
```

#### Insert node at beginning of document
```js
Transforms.insertNodes(editor, [
    {type:'paragraph', children:[{text: 'some text', marks:[]}]},
  ],
  {at:[0]}
);
```

#### Set node
```js
Transforms.setNodes(editor, {type: 'paragraph'}, {at: path})
```

#### Set node text
```js
Transforms.insertText(editor, 'new text', {at: path})
```

#### Insert inline + text & navigate to text
```js
Transforms.insertNodes(editor, [
    { type: 'link', url:'x', children: [{ text:'mja', marks:[] }] },
    { text: '', marks:[] },
]);
const nextPoint = Editor.after(editor, editor.selection.anchor);
Editor.setSelection(editor, {anchor:nextPoint, focus:nextPoint})
```
#### Insert text after above commands
```js
Transforms.insertText(editor, 'text in the following text node')
```


# Setup & Helpers

## Create a withPlugins hook / composer
```js
import * as plugins from './plugins/';

export const withPlugins = (editor) => {
    for(let plugin in plugins) {
        if(typeof plugins[plugin] !== 'function') continue;
        const pluginEditor = plugins[plugin](editor);
        if(pluginEditor !== editor) continue; // Invalid plugin
        editor = pluginEditor;
    }
    return editor;
}
```
the file plugins/index.js exports all the plugins, e.g. export * from "./plugin1", export * from "./plugin2" etc


## Other helpers

#### Find the node and path by custom id
```js
const findById = (root, id, path=[]) => {
  if(!root || !root.children || !id) return;
	const childLen = root.children.length;
	for(let i=0;i<childLen;i++) {
		const child = root.children[i];
        if(child.data && child.id === id) return [child, [...path, i]];
        const potential = child.children && nodeById(child, id, [...path, i]);
        if(potential) return potential;
	}
}
```

## Useful editor extending methods

#### Add or replace data for a node. Doesn't overwrite.
Call with editor.addData(data, node) or editor.addData(data, path)
```js
editor.addData = (data, nodeOrPath) => {
    const isNode = Node.isNode(nodeOrPath);
    if(!isNode && !Path.isPath(nodeOrPath)) return;
    const node = isNode ? nodeOrPath : Node.get(editor, nodeOrPath);
    const path = !isNode ? nodeOrPath : ReactEditor.findPath(editor, node);
    Editor.setNodes(editor, {data: deepmerge(data, node.data)}, {at: path})
}
```

You could use 'deepmerge' or other library for merging data



## Convert data from v0.47 to v.0.50+
```js
const convertNode = (node) => {
  const { object, type, data, nodes, ...rest } = node
  // We drop `object`, pull up data, convert `nodes` to children and copy the rest across
  const element = {
    type,
    ...rest,
    ...(nodes ? { children: nodes.map(convertNode) } : {}),
  }
  if ((!element.type || element.type === 'text') && typeof element.text !== 'undefined') {
    delete element.type;
    if(Array.isArray(element.marks)) {
      for(const mark of element.marks) {
        if(typeof mark === 'string')
          element[mark] = true
        else if(typeof mark === 'object' && mark.type) {
          element[mark.type] = mark.hasOwnProperty('value') ? mark.value : true
        }
      }
    }
  }
  if(data) element.data = data;
  // if(element.type) element.type = element.type.replace("-", "_")
  
  // Atomic blocks must now have children
  if (element.type && !element.children) {
    element.children = [
      {
        text: '',
      },
    ]
  }
  return element
};
const convertSlate047to050 = (object) => {
  const { nodes } = object.document
  return nodes.map(convertNode)
}
```

