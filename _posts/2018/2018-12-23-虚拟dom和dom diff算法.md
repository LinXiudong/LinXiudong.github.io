---
layout: mypost
title: 虚拟dom和dom diff算法
categories: [算法]
---

# dom diff算法的概念

![p07][07]

DIFF算法在执行时有三个维度，分别是Tree DIFF、Component DIFF和Element DIFF，执行时按顺序依次执行，它们的差异仅仅因为DIFF粒度不同、执行先后顺序不同。 

![p08][08]

Tree DIFF是对树的每一层进行遍历，如果某组件不存在了，则会直接销毁。如图所示，左边是旧属，右边是新属，第一层是R组件，一模一样，不会发生变化；第二层进入Component DIFF，同一类型组件继续比较下去，发现A组件没有，所以直接删掉A、B、C组件；继续第三层，重新创建A、B、C组件。 

![p09][09]

如图所示，第一层遍历完，进行第二层遍历时，D和G组件是不同类型的组件，不同类型组件直接进行替换，将D删掉，再将G重建。 

![p10][10]

Element DIFF紧接着以上统一类型组件继续比较下去，常见类型就是列表。同一个列表由旧变新有三种行为，插入、移动和删除，它的比较策略是对于每一个列表指定key，先将所有列表遍历一遍，确定要新增和删除的，再确定需要移动的。如图所示，第一步将D删掉，第二步增加E，再次执行时A和B只需要移动位置即可。


# 虚拟dom和dom diff算法

所谓的virtual dom，也就是虚拟节点。它通过JS的Object对象模拟DOM中的节点，然后再通过特定的render方法将其渲染成真实的DOM节点。
  
对于页面的重新渲染一般的做法是通过操作dom，重置innerHTML去完成这样一件事情。而virtual dom则是通过JS层面的计算，返回一个patch对象，即补丁对象，再通过特定的操作解析patch对象，完成页面的重新渲染。具体virtual dom 渲染的流程如图所示
 
![p01][01]
 

# 实现一个utils方法库 
 
 ```
const _ = exports

_.setAttr = function setAttr (node, key, value) {
  switch (key) {
    case 'style':
      node.style.cssText = value
      break;
    case 'value':
      let tagName = node.tagName || ''
      tagName = tagName.toLowerCase()
      if (
        tagName === 'input' || tagName === 'textarea'
      ) {
        node.value = value
      } else {
        // 如果节点不是 input 或者 textarea, 则使用 `setAttribute` 去设置属性
        node.setAttribute(key, value)
      }
      break;
    default:
      node.setAttribute(key, value)
      break;
  }
}

_.slice = function slice (arrayLike, index) {
  return Array.prototype.slice.call(arrayLike, index)
}


_.type = function type (obj) {
  return Object.prototype.toString.call(obj).replace(/\[object\s|\]/g, '')
}

_.isArray = function isArray (list) {
  return _.type(list) === 'Array'
}

_.toArray = function toArray (listLike) {
  if (!listLike) return []

  let list = []
  for (let i = 0, l = listLike.length; i < l; i++) {
    list.push(listLike[i])
  }
  return list
}

_.isString = function isString (list) {
  return _.type(list) === 'String'
}

_.isElementNode = function (node) {
  return node.nodeType === 1
}
```

# 实现一个Element 

这里我们需要做的事很easy，那就是实现一个Object去模拟DOM节点的展示形式。真实节点如下

```html
<ul id="list">
  <li class="item">item1</li>
  <li class="item">item2</li>
  <li class="item">item3</li>
</ul>
``` 

我们需要完成一个Element模拟上面的真实节点，形式如下
```javascript
let ul = {
  tagName: 'ul',
  attrs: {
    id: 'list'
  },
  children: [
    { tagName: 'li', attrs: { class: 'item' }, children: ['item1'] },
    { tagName: 'li', attrs: { class: 'item' }, children: ['item1'] },
    { tagName: 'li', attrs: { class: 'item' }, children: ['item1'] },
  ]
}
```

看到这里，我们可以看到的是el对象中的tagName，attrs，children都可以提取出来到Element中去，即

```javascript
class Element {
  constructor(tagName, attrs, children) {
    this.tagName  = tagName
    this.attrs    = attrs
    this.children = children
  }
}
function el (tagName, attrs, children) {
  return new Element(tagName, attrs, children)
}
module.exports = el;
```

那么上面的ul就可以用更简化的方式进行书写了，即

```javascript
let ul = el(
  'ul', { id: 'list' }, 
  [
    el('li', { class: 'item' }, ['Item 1']),
    el('li', { class: 'item' }, ['Item 2']),
    el('li', { class: 'item' }, ['Item 3'])
])
```

到这我们Element算是实现一半，剩下的则是提供一个render函数，将Element对象渲染成真实的DOM节点。完整的Element的代码如下
 
 ```javascript
import _ from './utils'

/**
 * @class Element Virtrual Dom
 * @param { String } tagName
 * @param { Object } attrs   Element's attrs, 如: { id: 'list' }
 * @param { Array <Element|String> } 可以是Element对象，也可以只是字符串，即textNode
 */
class Element {
  constructor(tagName, attrs, children) {
    // 如果只有两个参数
    if (_.isArray(attrs)) {
      children = attrs
      attrs = {}
    }

    this.tagName  = tagName
    this.attrs    = attrs || {}
    this.children = children
    // 设置this.key属性，为了后面list diff做准备
    this.key = attrs
      ? attrs.key
      : void 0
  }

  render () {
    let el    = document.createElement(this.tagName)
    let attrs = this.attrs

    for (let attrName in attrs) { // 设置节点的DOM属性
      let attrValue = attrs[attrName]
      _.setAttr(el, attrName, attrValue)
    }

    let children = this.children || []
    children.forEach(child => {
      let childEl = child instanceof Element
        ? child.render() // 若子节点也是虚拟节点，递归进行构建
        : document.createTextNode(child)  // 若是字符串，直接构建文本节点
      el.appendChild(childEl)
    })

    return el
  }
}
function el (tagName, attrs, children) {
  return new Element(tagName, attrs, children)
}
module.exports = el;


// 其他js中执行
let ulRoot = ul.render()
document.body.appendChild(ulRoot);
```
 
至此，我们的Element便得以实现了。
 
 
# 实现diff算法 

这里我们做的就是实现一个diff算法进行虚拟节点Element的对比，并返回一个patch对象，用来存储两个节点不同的地方。这也是整个virtual dom实现最核心的一步。而diff算法又包含了两个不一样的算法，一个是O(n)，一个则是O(max(m, n))。

## 同层级元素比较（O(n)） 

首先，我们的知道的是，如果元素之间进行完全的一个比较，即新旧Element对象的父元素，本身，子元素之间进行一个混杂的比较，其实现的时间复杂度为O(n^3)。但是在我们前端开发中，很少会出现跨层级处理节点，所以这里我们会做一个同级元素之间的一个比较，则其时间复杂度则为O(n)。算法流程如图所示

![p02][02]

在这里，我们做同级元素比较时，可能会出现四种情况

- 整个元素都不一样，即元素被replace掉 
- 元素的attrs不一样 
- 元素的text文本不一样 
- 元素顺序被替换，即元素需要reorder 

上面的第四种情况属于diff的第二种算法，这里我们先不讨论，我们在后面再进行详细的讨论 

针对以上四种情况，我们先设置四个常量进行表示。diff入口方法及四种状态如下

```javascript
const REPLACE = 0  // replace => 0
const ATTRS   = 1  // attrs   => 1
const TEXT    = 2  // text    => 2
const REORDER = 3  // reorder => 3

// diff 入口，比较新旧两棵树的差异
function diff (oldTree, newTree) {
  let index   = 0
  let patches = {} // 用来记录每个节点差异的补丁对象
  walk(oldTree, newTree, index, patches)
  return patches
}
```

状态定义好了，接下来开搞。我们一个一个实现，获取到每个状态的不同。这里需要注意的一点就是，我们这里的diff比较只会和上面的流程图显示的一样，只会两两之间进行比较，如果有节点remove掉，这里会pass掉，直接走list diff。

首先我们先从最顶层的元素依次往下进行比较，直到最后一层元素结束，并把每个层级的差异存到patch对象中去，即实现walk方法

```javascript
/**
 * walk 遍历查找节点差异
 * @param  { Object } oldNode
 * @param  { Object } newNode
 * @param  { Number } index   - currentNodeIndex
 * @param  { Object } patches - 记录节点差异的对象
 */
function walk (oldNode, newNode, index, patches) {
  let currentPatch = []

  // 如果oldNode被remove掉了
  if (newNode === null || newNode === undefined) {
    // 先不做操作, 具体交给 list diff 处理
  }
  // 比较文本之间的不同
  else if (_.isString(oldNode) && _.isString(newNode)) {
    if (newNode !== oldNode) currentPatch.push({ type: TEXT, content: newNode })
  }
  // 比较attrs的不同
  else if (
    oldNode.tagName === newNode.tagName &&
    oldNode.key     === newNode.key
  ) {
    let attrsPatches = diffAttrs(oldNode, newNode)
    if (attrsPatches) {
      currentPatch.push({ type: ATTRS, attrs: attrsPatches })
    }
    // 递归进行子节点的diff比较
    diffChildren(oldNode.children, newNode.children, index, patches)
  }
  else {
    currentPatch.push({ type: REPLACE, node: newNode})
  }

  if (currentPatch.length) {
    patches[index] = currentPatch
  }
}

function diffAttrs (oldNode, newNode) {
  let count    = 0
  let oldAttrs = oldNode.attrs
  let newAttrs = newNode.attrs

  let key, value
  let attrsPatches = {}

  // 如果存在不同的 attrs
  for (key in oldAttrs) {
    value = oldAttrs[key]
    // 如果 oldAttrs 移除掉一些 attrs, newAttrs[key] === undefined
    if (newAttrs[key] !== value) {
      count++
      attrsPatches[key] = newAttrs[key]
    }
  }
  // 如果存在新的 attr
  for (key in newAttrs) {
    value = newAttrs[key]
    if (!oldAttrs.hasOwnProperty(key)) {
      count++
      attrsPatches[key] = value
    }
  }

  if (count === 0) {
    return null
  }

  return attrsPatches
}
```

实际上我们需要对新旧元素进行一个深度的遍历，为每个节点加上一个唯一的标记，具体流程如图所示

![p03][03]

我们接下来要做的一件事情就很明确了，那就是在做深度遍历比较差异的时候，将每个元素节点，标记上一个唯一的标识。具体做法如下

```javascript
// 设置节点唯一标识
let key_id = 0
// diff with children
function diffChildren (oldChildren, newChildren, index, patches) {
  // 存放当前node的标识，初始化值为 0
  let currentNodeIndex = index

  oldChildren.forEach((child, i) => {
    key_id++
    let newChild = newChildren[i]
    currentNodeIndex = key_id

    // 递归继续比较
    walk(child, newChild, currentNodeIndex, patches)
  })
}
```

咱调用一下看下效果，看看两个不同的Element对象比较会返回一个哪种形式的patch对象

```javascript
let ul = el('ul', { id: 'list' }, [
  el('li', { class: 'item' }, ['Item 1']),
  el('li', { class: 'item' }, ['Item 2'])
])
let ul1 = el('ul', { id: 'list1' }, [
  el('li', { class: 'item1' }, ['Item 4']),
  el('li', { class: 'item2' }, ['Item 5'])
])
let patches = diff(ul, ul1);
console.log(patches);
```

结果如图

![p04][04]

完整的diff代码如下（包含了调用list diff的方法，如果你在跟着文章踩坑的话，把里面一些代码注释掉即可）

```javascript
import _ from './utils'
import listDiff from './list-diff'

const REPLACE = 0
const ATTRS   = 1
const TEXT    = 2
const REORDER = 3

// diff 入口，比较新旧两棵树的差异
function diff (oldTree, newTree) {
  let index   = 0
  let patches = {} // 用来记录每个节点差异的补丁对象
  walk(oldTree, newTree, index, patches)
  return patches
}

/**
 * walk 遍历查找节点差异
 * @param  { Object } oldNode
 * @param  { Object } newNode
 * @param  { Number } index   - currentNodeIndex
 * @param  { Object } patches - 记录节点差异的对象
 */
function walk (oldNode, newNode, index, patches) {

  let currentPatch = []

  // 如果oldNode被remove掉了，即 newNode === null的时候
  if (newNode === null || newNode === undefined) {
    // 先不做操作, 具体交给 list diff 处理
  }
  // 比较文本之间的不同
  else if (_.isString(oldNode) && _.isString(newNode)) {
    if (newNode !== oldNode) currentPatch.push({ type: TEXT, content: newNode })
  }
  // 比较attrs的不同
  else if (
    oldNode.tagName === newNode.tagName &&
    oldNode.key     === newNode.key
  ) {
    let attrsPatches = diffAttrs(oldNode, newNode)
    if (attrsPatches) {
      currentPatch.push({ type: ATTRS, attrs: attrsPatches })
    }
    // 递归进行子节点的diff比较
    diffChildren(oldNode.children, newNode.children, index, patches, currentPatch)
  }
  else {
    currentPatch.push({ type: REPLACE, node: newNode})
  }

  if (currentPatch.length) {
    patches[index] = currentPatch
  }
}

function diffAttrs (oldNode, newNode) {
  let count    = 0
  let oldAttrs = oldNode.attrs
  let newAttrs = newNode.attrs

  let key, value
  let attrsPatches = {}

  // 如果存在不同的 attrs
  for (key in oldAttrs) {
    value = oldAttrs[key]
    // 如果 oldAttrs 移除掉一些 attrs, newAttrs[key] === undefined
    if (newAttrs[key] !== value) {
      count++
      attrsPatches[key] = newAttrs[key]
    }
  }
  // 如果存在新的 attr
  for (key in newAttrs) {
    value = newAttrs[key]
    if (!oldAttrs.hasOwnProperty(key)) {
      attrsPatches[key] = value
    }
  }

  if (count === 0) {
    return null
  }

  return attrsPatches
}

// 设置节点唯一标识
let key_id = 0
// diff with children
function diffChildren (oldChildren, newChildren, index, patches, currentPatch) {
  let diffs = listDiff(oldChildren, newChildren, 'key')
  newChildren = diffs.children

  if (diffs.moves.length) {
    let reorderPatch = { type: REORDER, moves: diffs.moves }
    currentPatch.push(reorderPatch)
  }

  // 存放当前node的标识，初始化值为 0
  let currentNodeIndex = index

  oldChildren.forEach((child, i) => {
    key_id++
    let newChild = newChildren[i]
    currentNodeIndex = key_id

    // 递归继续比较
    walk(child, newChild, currentNodeIndex, patches)
  })
}

module.exports = diff
```

## listDiff实现 O(m*n) => O(max(m, n)) 

首先我们得明确一下为什么需要list diff这种算法的存在，list diff做的一件事情是怎样的，然后它又是如何做到这么一件事情的。
   
举个栗子，我有新旧两个Element对象，分别为

```javascript
let oldTree = el('ul', { id: 'list' }, [
  el('li', { class: 'item1' }, ['Item 1']),
  el('li', { class: 'item2' }, ['Item 2']),
  el('li', { class: 'item3' }, ['Item 3'])
])
let newTree = el('ul', { id: 'list' }, [
  el('li', { class: 'item3' }, ['Item 3']),
  el('li', { class: 'item1' }, ['Item 1']),
  el('li', { class: 'item2' }, ['Item 2'])
])
```

如果要进行diff比较的话，我们直接用上面的方法就能比较出来，但我们可以看出来这里只做了一次节点的move。如果直接按照上面的diff进行比较，并且通过后面的patch方法进行patch对象的解析渲染，那么将需要操作三次DOM节点才能完成视图最后的update。

当然，如果只有三个节点的话那还好，我们的浏览器还能吃的消，看不出啥性能上的区别。那么问题来了，如果有N多节点，并且这些节点只是做了一小部分remove，insert，move的操作，那么如果我们还是按照一一对应的DOM操作进行DOM的重新渲染，那岂不是操作太昂贵？

所以，才会衍生出list diff这种算法，专门进行负责收集remove，insert，move操作，当然对于这个操作我们需要提前在节点的attrs里面申明一个DOM属性，表示该节点的唯一性。另外上张图说明一下list diff的时间复杂度，小伙伴们可以看图了解一下

![p05][05]

接下来我们举个具体的例子说明一下list diff具体如何进行操作的，代码如下

```javascript
let oldTree = el('ul', { id: 'list' }, [
  el('li', { key: 1 }, ['Item 1']),
  el('li', {}, ['Item']),
  el('li', { key: 2 }, ['Item 2']),
  el('li', { key: 3 }, ['Item 3'])
])
let newTree = el('ul', { id: 'list' }, [
  el('li', { key: 3 }, ['Item 3']),
  el('li', { key: 1 }, ['Item 1']),
  el('li', {}, ['Item']),
  el('li', { key: 4 }, ['Item 4'])
])
```

对于上面例子中的新旧节点的差异对比，如果我说直接让小伙伴们看代码捋清楚节点操作的流程，估计大家都会说我耍流氓。所以我整理了一幅流程图，解释了list diff具体如何进行计算节点差异的，如下

![p06][06]

我们看图说话，list diff做的事情就很简单明了啦。

第一步，newChildren向oldChildren的形式靠近进行操作（移动操作，代码中做法是直接遍历oldChildren进行操作），得到simulateChildren = [key1, 无key, null, key3] 

- step1.oldChildren第一个元素key1对应newChildren中的第二个元素 
- step2.oldChildren第二个元素无key对应newChildren中的第三个元素 
- step3.oldChildren第三个元素key2在newChildren中找不到，直接设为null 
- step4.oldChildren第四个元素key3对应newChildren中的第一个元素 

第二步，稍微处理一下得出的simulateChildren，将null元素以及newChildren中的新元素加入，得到simulateChildren = [key1, 无key, key3, key4] 

第三步，将得出的simulateChildren向newChildren的形式靠近，并将这里的移动操作全部记录下来（注：元素的move操作这里会当成remove和insert操作的结合）。所以最后我们得出上图中的一个moves数组，存储了所有节点移动类的操作。 

整体流程我们捋清楚了，接下来要做的事情就会简单很多了。我们只需要用代码把上面列出来要做的事情得以实现即可。（注：这里本来我是想分步骤一步一步实现，但是每一步牵扯到的东西有点多，怕到时贴出来的代码太多，我还是直接把list diff所有代码写上注释贴上吧）

```javascript
/**
 * Diff two list in O(N).
 * @param {Array} oldList - 原始列表
 * @param {Array} newList - 经过一些操作的得出的新列表
 * @return {Object} - {moves: <Array>}
 *                  - moves list操作记录的集合
 */
function diff (oldList, newList, key) {
  let oldMap = getKeyIndexAndFree(oldList, key)
  let newMap = getKeyIndexAndFree(newList, key)

  let newFree = newMap.free

  let oldKeyIndex = oldMap.keyIndex
  let newKeyIndex = newMap.keyIndex
  // 记录所有move操作
  let moves = []

  // a simulate list
  let children = []
  let i = 0
  let item
  let itemKey
  let freeIndex = 0

  // newList 向 oldList 的形式靠近进行操作
  while (i < oldList.length) {
    item = oldList[i]
    itemKey = getItemKey(item, key)
    if (itemKey) {
      if (!newKeyIndex.hasOwnProperty(itemKey)) {
        children.push(null)
      } else {
        let newItemIndex = newKeyIndex[itemKey]
        children.push(newList[newItemIndex])
      }
    } else {
      let freeItem = newFree[freeIndex++]
      children.push(freeItem || null)
    }
    i++
  }
  let simulateList = children.slice(0)

  // 移除列表中一些不存在的元素
  i = 0
  while (i < simulateList.length) {
    if (simulateList[i] === null) {
      remove(i)
      removeSimulate(i)
    } else {
      i++
    }
  }
  // i  => new list
  // j  => simulateList
  let j = i = 0
  while (i < newList.length) {
    item = newList[i]
    itemKey = getItemKey(item, key)

    let simulateItem = simulateList[j]
    let simulateItemKey = getItemKey(simulateItem, key)

    if (simulateItem) {
      if (itemKey === simulateItemKey) {
        j++
      }
      else {
        // 如果移除掉当前的 simulateItem 可以让 item在一个正确的位置，那么直接移除
        let nextItemKey = getItemKey(simulateList[j + 1], key)
        if (nextItemKey === itemKey) {
          remove(i)
          removeSimulate(j)
          j++ // 移除后，当前j的值是正确的，直接自加进入下一循环
        } else {
          // 否则直接将item 执行 insert
          insert(i, item)
        }
      }
    // 如果是新的 item, 直接执行 inesrt
    } else {
      insert(i, item)
    }
    i++
  }
  // if j is not remove to the end, remove all the rest item
  // let k = 0;
  // while (j++ < simulateList.length) {
  //   remove(k + i);
  //   k++;
  // }

  // 记录remove操作
  function remove (index) {
    let move = {index: index, type: 0}
    moves.push(move)
  }
  // 记录insert操作
  function insert (index, item) {
    let move = {index: index, item: item, type: 1}
    moves.push(move)
  }
  // 移除simulateList中对应实际list中remove掉节点的元素
  function removeSimulate (index) {
    simulateList.splice(index, 1)
  }
  // 返回所有操作记录
  return {
    moves: moves,
    children: children
  }
}
/**
 * 将 list转变成  key-item keyIndex 对象的形式进行展示.
 * @param {Array} list
 * @param {String|Function} key
 */
function getKeyIndexAndFree (list, key) {
  let keyIndex = {}
  let free = []
  for (let i = 0, len = list.length; i < len; i++) {
    let item = list[i]
    let itemKey = getItemKey(item, key)
    if (itemKey) {
      keyIndex[itemKey] = i
    } else {
      free.push(item)
    }
  }

  // 返回 key-item keyIndex
  return {
    keyIndex: keyIndex,
    free: free
  }
}

function getItemKey (item, key) {
  if (!item || !key) return void 0
  return typeof key === 'string'
    ? item[key]
    : key(item)
}

module.exports = diff
```

## 实现patch，解析patch对象 

这里的操作还是蛮简单的。因为他和前面的操作思路基本一致，前面是遍历Element，给其唯一的标识，那么这里则是顺着patch对象提供的唯一的键值进行解析的。直接给大家上一些深度遍历的代码

```javascript
function patch (rootNode, patches) {
  let walker = { index: 0 }
  walk(rootNode, walker, patches)
}

function walk (node, walker, patches) {
  let currentPatches = patches[walker.index] // 从patches取出当前节点的差异

  let len = node.childNodes
    ? node.childNodes.length
    : 0
  for (let i = 0; i < len; i++) { // 深度遍历子节点
    let child = node.childNodes[i]
    walker.index++
    walk(child, walker, patches)
  }

  if (currentPatches) {
    dealPatches(node, currentPatches)  // 对当前节点进行DOM操作
  }
}
```

历史总是惊人的相似，现在小伙伴应该知道之前深度遍历给Element每个节点加上唯一标识的好处了吧。OK，接下来我们根据不同类型的差异对当前节点进行操作

```
function dealPatches (node, currentPatches) {
  currentPatches.forEach(currentPatch => {
    switch (currentPatch.type) {
      case REPLACE:
        let newNode = (typeof currentPatch.node === 'string')
          ? document.createTextNode(currentPatch.node)
          : currentPatch.node.render()
        node.parentNode.replaceChild(newNode, node)
        break
      case REORDER:
        reorderChildren(node, currentPatch.moves)
        break
      case ATTRS:
        setProps(node, currentPatch.props)
        break
      case TEXT:
        if (node.textContent) {
          node.textContent = currentPatch.content
        } else {
          // for ie
          node.nodeValue = currentPatch.content
        }
        break
      default:
        throw new Error('Unknown patch type ' + currentPatch.type)
    }
  })
}
```

具体的setAttrs和reorder的实现如下

```javascript
function setAttrs (node, props) {
  for (let key in props) {
    if (props[key] === void 0) {
      node.removeAttribute(key)
    } else {
      let value = props[key]
      _.setAttr(node, key, value)
    }
  }
}
function reorderChildren (node, moves) {
  let staticNodeList = _.toArray(node.childNodes)
  let maps = {} // 存储含有key特殊字段的节点

  staticNodeList.forEach(node => {
    // 如果当前节点是ElementNode，通过maps将含有key字段的节点进行存储
    if (_.isElementNode(node)) {
      let key = node.getAttribute('key')
      if (key) {
        maps[key] = node
      }
    }
  })

  moves.forEach(move => {
    let index = move.index
    if (move.type === 0) { // remove item
      if (staticNodeList[index] === node.childNodes[index]) { // maybe have been removed for inserting
        node.removeChild(node.childNodes[index])
      }
      staticNodeList.splice(index, 1)
    } else if (move.type === 1) { // insert item
      let insertNode = maps[move.item.key]
        ? maps[move.item.key] // reuse old item
        : (typeof move.item === 'object')
            ? move.item.render()
            : document.createTextNode(move.item)
      staticNodeList.splice(index, 0, insertNode)
      node.insertBefore(insertNode, node.childNodes[index] || null)
    }
  })
}
```


[原文1 优秀前端系列（五）-Virtual Dom && Diff](https://blog.csdn.net/mm782642353/article/details/82356025)

[原文2 非常简单易懂的DOM Diff算法简析](https://blog.csdn.net/one_girl/article/details/81086289)



[01]: 01.png
[02]: 02.png
[03]: 03.png
[04]: 04.png
[05]: 05.png
[06]: 06.png
[07]: 07.png
[08]: 08.png
[09]: 09.png
[10]: 10.png
