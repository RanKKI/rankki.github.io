---
title: "React Native SVG File to Component"
date: 2023-09-17T16:32:38+10:00
draft: false
tags:
 - SVG
 - React
categories:
lang: en
---

When creating a React Native app, you may want to use SVG files as icons. This is a quick guide on how to convert an SVG file to a React Native component.

# Use SVG as component

It's easy to create SVG elements in your RN project by using [react-native-svg](https://github.com/react-native-community/react-native-svg`).

```
yarn add react-native-svg
```

```typescript
import * as React from 'react';
import Svg, { Circle, Rect } from 'react-native-svg';

export default function SvgComponent(props) {
  return (
    <Svg height="50%" width="50%" viewBox="0 0 100 100" {...props}>
      <Circle cx="50" cy="50" r="45" stroke="blue" strokeWidth="2.5" fill="green" />
      <Rect x="15" y="15" width="70" height="70" stroke="red" strokeWidth="2" fill="yellow" />
    </Svg>
  );
}
```

# Use SVG file

But, it doesn't support SVG files directly, in order to import a SVG file as a React Component, you will need to use [react-native-svg-transformer](https://github.com/kristerkari/react-native-svg-transformer).

```
yarn add react-native-svg-transformer
```

```typescript
import Logo from "./logo.svg";

export default function SvgComponent(props) {
  return (
    <Logo width={120} height={40} />
  );
}
```

## Attribute replacement

Sometime, you may wish change the color (or other attributes) of the SVG element in your code.

By adding this `.svgrrc.[js|json]` file to your root directory, the `react-native-svg-transformer` will replace the attribute value with the given props dynamically.

```javascript
module.exports = {
  replaceAttrValues: {
    '#000': '{props.fill}',
  },
}
```

---

```typescript
import Logo from "./logo.svg";

export default function SvgComponent(props) {
  return (
    <Logo width={120} height={40} fill={"red"}/>
  );
}
```

> Note: You have to set the `fill` to `#000` in all SVG files, otherwise the `react-native-svg-transformer` will not be able to replace the value.

## Change fill color based on the theme

Because I'm using `styled-components` to style my React app, and I want to the icon's color follow the theme color, so I implemented the [custom template](https://react-svgr.com/docs/custom-templates/) of `svgr.

Modify `.svgrrc.[js|json]` to use the custom template

```javascript
module.exports = {
  replaceAttrValues: {
    '#000': '{props.fill || theme.primaryIcon}',
  },
  template: require('./.svg.template.js'),
}
```

By using `props.fill || theme.primaryIcon`, the fill color will be changed to `theme.primaryIcon` if the `fill` prop in the code is not provided, which means I still can override the fill color in the code.

### Template

```javascript
const template = (
  { jsx, imports, interfaces, componentName, props, exports },
  { tpl }
) => {
  return tpl`
${imports};
import { useTheme } from "@/theme"; // Manually import the theme

${interfaces};

const ${componentName} = (${props}) => {
  const theme = useTheme(); // Use the hook here
  return (${jsx})
}

${exports};
`
}

module.exports = template
```