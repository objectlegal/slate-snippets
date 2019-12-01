# slate-snippets
Slate snippets

# General

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
ReactEditor.findPath(editor, node)
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
const nextpath = Editor.next(editor, editor.selection.anchor.path, 'text')[1]
Editor.setSelection(editor, {anchor:{path:nextpath, offset:0}, focus:{path:nextpath, offset:0}})
```
Test insert after above commands
```
Editor.insertText(editor, 'text in the following text node')
```


# Some helper functions

## Create a withPlugins hook
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


Non sophisticated helper fn used by addData above
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


