---
title: github copilot 使用分享
layout: cover
class: text-center
---
<style>
  .two-cols-header { 
    grid-gap: 16px; 
  }
</style>

<img src="https://github.gallerycdn.vsassets.io/extensions/github/copilot-chat/0.8.2023091402/1694713114386/Microsoft.VisualStudio.Services.Icons.Default" style="width: 100px;margin:0 auto">

# github copilot 使用分享


---

<Toc maxDepth="2"></Toc>

---
layout: section
---

# 如何讓 copilot 不只是高級 auto complete

---

## 了解 copilot 的咒語是什麼（prompt）

1. **Open files: 開啟相關的檔案**
    - 在使用 Copilot 時，將相關的檔案開啟在 VS Code 中，有助於設定這個語境，並讓 Copilot 看到您的專案的整體架構。
2. **Top level comment: 最上層的註解**
    - 在您正在使用的檔案中，一個最上層的註解，可以幫助 Copilot 理解您將要創建的程式碼片段的整體語境。
3. **Appropriate includes and references: 適當的引用和參考**
    - 這也可以幫助 Copilot 知道您希望它在製作建議時使用哪些框架、庫及其版本。
4. **Meaningful function names: 有意義的函式名稱**
5. **Specific and well-scoped function comments: 特定且範圍良好的函式註解**
6. **Prime Copilot with sample code: 使用範例程式碼**

ref: [vscode/docs/AI-Tools](https://code.visualstudio.com/docs/editor/artificial-intelligence#_provide-ai-context)

---

## 案例：單行咒語

### string
```js
const gePageLayoutKeyWithoutDevice = pageLayoutKey => {
  // PC__JOB_LIST -> JOB_LIST
  return pageLayoutKey.split("__")[1];
};
```

### sort
```js
const result = styleArray.sort((a,b) => {
  // pc--* > mobile--* > app--*
  const deviceOrder = ["pc", "mobile", "app"]
  const aDevice = a.split("--")[0]
  const bDevice = b.split("--")[0]
  const aDeviceIndex = deviceOrder.indexOf(aDevice)
  const bDeviceIndex = deviceOrder.indexOf(bDevice)
  if(aDeviceIndex !== bDeviceIndex) {
    return aDeviceIndex - bDeviceIndex
  } else {
    // string order
    return a.localeCompare(b)
  }
})
```

---
layout: two-cols-header
---
## 案例：範例咒語

::left::

```js
/*  
input:{
        PC:{
          [hash-p1]:{},
          [hash-p2]:{},
          [hash-p3]:{},
          [hash-p4]:{},
        },
        MOBILE:{
          [hash-m1]:{},
          [hash-m2]:{},
        },
        APP:{
          [hash-a1]:{},
          [hash-a2]:{},
          [hash-a3]:{},
        }
      }
return: [
          [Obj:hash-p1, Obj:hash-m1, Obj:hash-a1 ],
          [Obj:hash-p2, Obj:hash-m2, Obj:hash-a2 ],
          [Obj:hash-p3, {}, Obj:hash-a3 ],
          [Obj:hash-p4, {}, {} ],
        ]
*/
```

::right::

```js
function adapter(deviceData){
  const deviceKeys = Object.keys(deviceData)
  const deviceValues = Object.values(deviceData)
  const deviceEntries = Object.entries(deviceData)
  const deviceEntriesLength = deviceEntries.length

  const deviceEntriesLengths = deviceEntries.map(([,value]) => {
    return Object.keys(value).length
  })
  const maxDeviceEntriesLength = Math.max(...deviceEntriesLengths)

  const result = []
  for(let i = 0; i < maxDeviceEntriesLength; i++){
    const row = []
    for(let j = 0; j < deviceEntriesLength; j++){
      const [key,value] = deviceEntries[j]
      const item = value[Object.keys(value)[i]] || {}
      row.push(item)
    }
    result.push(row)
  }
  return result
}
```

---
layout: section
---

# github copilot chat

---

## Chat view

![](https://code.visualstudio.com/assets/docs/editor/artificial-intelligence/copilot-view.png)

---

## Slash commands

![](https://code.visualstudio.com/assets/docs/editor/artificial-intelligence/slash-commands.png)

---

## Inline chat

![](https://code.visualstudio.com/assets/docs/editor/artificial-intelligence/inline-chat.png)

---

## Quick Chat

![](https://code.visualstudio.com/assets/docs/editor/artificial-intelligence/quick-chat-dropdown.png)

---
layout: two-cols-header
---
## 案例：打包套件引入方法轉換
<br>

> can you convert to vite import?

::left::
### webpack
```js
const requireResult = require.context(
  "../components/adsTypes",
  true,
  /index\.vue$|AdsType\d*\.vue$/,
  "lazy"
);

const requireAll = requireResult =>
  requireResult.keys().reduce((acc, key) => {
    const name = key.match(/AdsType\d*/)[0];
    acc[name] = () => requireResult(key);
    return acc;
  }, {});
```
::right::
### vite
```js
const requireResult = import.meta.glob(
  "../components/adsTypes/**/{index,AdsType*}.vue"
);

const requireAll = async (requireResult) => {
  const acc = {};
  for (const path in requireResult) {
    const name = path.match(/AdsType\d*/)[0];
    acc[name] = () => requireResult[path]();
  }
  return acc;
};

const adsTypes = await requireAll(requireResult);
```

---
layout: two-cols-header
---
## 案例：css 框架轉換
<br>

::left::
### style
```css
  demo {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    display: flex;
    justify-content: center;
    align-items: center;
  }
```

::right::
### bootstrap 
> can you convert to bootstrap?
```html
<template>
  <div class="position-relative d-flex justify-content-center align-items-center">
    <!-- Your content here -->
  </div>
</template>
```

### tailwind css 
> can you convert to tailwind css?
```html
<template>
  <div class="absolute inset-0 flex justify-center items-center">
    <!-- Your content here -->
  </div>
</template>
```
---
layout: two-cols-header
---
## 案例：前端框架轉換
<br>

> convert to vue template

::left::
### react jsx
```jsx
export const Table = props => {
  return  <table name={'adcode'} >
    <thead>
    <tr>
      <th class="device" >type id</th>
      <th class="device" >媒體</th>
      <th class="site" >site name</th>
      <th class="site" >site id</th>
      <th class="channel" >channel name</th>
      <th class="channel" >channel id</th>
      <th >board name</th>
      <th >board id</th>
      <th >style</th>
    </tr>
    </thead>
    <Tbody source={props.source}/>
  </table>
}
```
::right::
### vue sfc
```html
<template>
  <table name="adcode">
    <thead>
      <tr>
        <th class="device">type id</th>
        <th class="device">媒體</th>
        <th class="site">site name</th>
        <th class="site">site id</th>
        <th class="channel">channel name</th>
        <th class="channel">channel id</th>
        <th>board name</th>
        <th>board id</th>
        <th>style</th>
      </tr>
    </thead>
    <Tbody :source="source" />
  </table>
</template>
<script> /* {...} */ </script>
```

---

# reference

- [vscode/docs/AI-Tools](https://code.visualstudio.com/docs/editor/artificial-intelligence#_chat-features)

