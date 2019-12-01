# slate-snippets
Slate snippets

# General

##Queries
Current anchor path
```
editor.selection.anchor.path
```


Trim a path to get the parent (e.g. transform [4,2,3] to [4,2])
```
const parentPath = Path.parent(path);
```


Get node by path
```
const node = Node.get(editor, path)
```

Get path by node
```
const path = ReactEditor.findPath(editor, node)
```

Find a node's DOM node
```
const domNode = ReactEditor.toDOMNode(editor, node)
```

Get closest block
```
const [blockNode, path] = Editor.match(editor, path, 'block');
```

Insert text at selection
```
Editor.insertText(editor, 'some text');
```
## Commands
Insert nodes at selection
```
Editor.insertNodes(editor, [
    {type:'inline_type', children:[{text: 'some text', marks:[]}]},
    {text: ' and some text after the inline', marks: []}
  ]
);
```

Insert node at beginning of document
```
Editor.insertNodes(editor, [
    {type:'paragraph', children:[{text: 'some text', marks:[]}]},
  ],
  {at:[0]}
);
```

Set node
```
Editor.setNodes(editor, {type: 'paragraph'}, {at: path})
```

Insert inline + text & navigate to text
```
Editor.insertNodes(editor, [
    { type: 'link', url:'x', children: [{ text:'mja', marks:[] }] },
    { text: '', marks:[] },
]);
const nextPoint = Editor.after(editor, editor.selection.anchor);
Editor.setSelection(editor, {anchor:nextPoint, focus:nextPoint})
```
Test insert after above commands
```
Editor.insertText(editor, 'text in the following text node')
```


# Setup & Helpers

## Create a withPlugins hook / composer
```
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

Find the node and path by custom id
```
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

Add or replace data for a node. Doesn't overwrite.
Call with editor.addData(data, node) or editor.addData(data, path)
```
editor.addData = (data, nodeOrPath) => {
    const isNode = Node.isNode(nodeOrPath);
    if(!isNode && !Path.isPath(nodeOrPath)) return;
    const node = isNode ? nodeOrPath : Node.get(editor, nodeOrPath);
    const path = !isNode ? nodeOrPath : ReactEditor.findPath(editor, node);
    Editor.setNodes(editor, {data: dataAdder(data, node.data)}, {at: path})
}
```

You could use 'deepmerge' or other library as a 'dataAdder'
Or use a non sophisticated function as follows
```
const dataAdder = (inputData, existingData) => {
    if(typeof existingData !== 'object' || Array.isArray(existingData) || existingData === null) existingData = {};
    for(var prop in inputData) {
      if(typeof inputData[prop] === 'object' && !Array.isArray(inputData[prop]) && inputData[prop] !== null)
        existingData[prop] = dataAdder(inputData[prop], existingData[prop]);
      else
        existingData[prop] = inputData[prop];
    }
    return existingData;
}
```


