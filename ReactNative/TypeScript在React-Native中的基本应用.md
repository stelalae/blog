# TypeScript在React-Native中的基本应用

[toc]

```JavaScript
// 基础样式
export interface TransformsStyle {
    transform?: (
        | PerpectiveTransform
        | RotateTransform
        | RotateXTransform
        | RotateYTransform
        | RotateZTransform
        | ScaleTransform
        | ScaleXTransform
        | ScaleYTransform
        | TranslateXTransform
        | TranslateYTransform
        | SkewXTransform
        | SkewYTransform
    )[];
    transformMatrix?: Array<number>;
    rotation?: number;
    scaleX?: number;
    scaleY?: number;
    translateX?: number;
    translateY?: number;
}
export interface ShadowStyleIOS {
    shadowColor?: string;
    shadowOffset?: { width: number; height: number };
    shadowOpacity?: number;
    shadowRadius?: number;
}
/**
 * Flex Prop Types
 * @see https://facebook.github.io/react-native/docs/flexbox.html#proptypes
 * @see https://facebook.github.io/react-native/docs/layout-props.html
 * @see https://github.com/facebook/react-native/blob/master/Libraries/StyleSheet/LayoutPropTypes.js
 */
export interface FlexStyle {
    alignContent?: 'flex-start' | 'flex-end' | 'center' | 'stretch' | 'space-between' | 'space-around';
    alignItems?: FlexAlignType;
    alignSelf?: 'auto' | FlexAlignType;
    aspectRatio?: number;
    borderBottomWidth?: number;
    borderEndWidth?: number | string;
    borderLeftWidth?: number;
    borderRightWidth?: number;
    borderStartWidth?: number | string;
    borderTopWidth?: number;
    borderWidth?: number;
    bottom?: number | string;
    display?: 'none' | 'flex';
    end?: number | string;
    flex?: number;
    flexBasis?: number | string;
    flexDirection?: 'row' | 'column' | 'row-reverse' | 'column-reverse';
    flexGrow?: number;
    flexShrink?: number;
    flexWrap?: 'wrap' | 'nowrap' | 'wrap-reverse';
    height?: number | string;
    justifyContent?: 'flex-start' | 'flex-end' | 'center' | 'space-between' | 'space-around' | 'space-evenly';
    left?: number | string;
    margin?: number | string;
    marginBottom?: number | string;
    marginEnd?: number | string;
    marginHorizontal?: number | string;
    marginLeft?: number | string;
    marginRight?: number | string;
    marginStart?: number | string;
    marginTop?: number | string;
    marginVertical?: number | string;
    maxHeight?: number | string;
    maxWidth?: number | string;
    minHeight?: number | string;
    minWidth?: number | string;
    overflow?: 'visible' | 'hidden' | 'scroll';
    padding?: number | string;
    paddingBottom?: number | string;
    paddingEnd?: number | string;
    paddingHorizontal?: number | string;
    paddingLeft?: number | string;
    paddingRight?: number | string;
    paddingStart?: number | string;
    paddingTop?: number | string;
    paddingVertical?: number | string;
    position?: 'absolute' | 'relative';
    right?: number | string;
    start?: number | string;
    top?: number | string;
    width?: number | string;
    zIndex?: number;
    /**
     * @platform ios
     */
    direction?: 'inherit' | 'ltr' | 'rtl';
}
/**
 * @see https://facebook.github.io/react-native/docs/view.html#style
 * @see https://github.com/facebook/react-native/blob/master/Libraries/Components/View/ViewStylePropTypes.js
 */
export interface ViewStyle extends FlexStyle, ShadowStyleIOS, TransformsStyle {
    backfaceVisibility?: 'visible' | 'hidden';
    backgroundColor?: string;
    borderBottomColor?: string;
    borderBottomEndRadius?: number;
    borderBottomLeftRadius?: number;
    borderBottomRightRadius?: number;
    borderBottomStartRadius?: number;
    borderBottomWidth?: number;
    borderColor?: string;
    borderEndColor?: string;
    borderLeftColor?: string;
    borderLeftWidth?: number;
    borderRadius?: number;
    borderRightColor?: string;
    borderRightWidth?: number;
    borderStartColor?: string;
    borderStyle?: 'solid' | 'dotted' | 'dashed';
    borderTopColor?: string;
    borderTopEndRadius?: number;
    borderTopLeftRadius?: number;
    borderTopRightRadius?: number;
    borderTopStartRadius?: number;
    borderTopWidth?: number;
    borderWidth?: number;
    opacity?: number;
    testID?: string;
    /**
     * Sets the elevation of a view, using Android's underlying
     * [elevation API](https://developer.android.com/training/material/shadows-clipping.html#Elevation).
     * This adds a drop shadow to the item and affects z-order for overlapping views.
     * Only supported on Android 5.0+, has no effect on earlier versions.
     *
     * @platform android
     */
    elevation?: number;
}
// Text组件的样式和属性
export type FontVariant = 'small-caps' | 'oldstyle-nums' | 'lining-nums' | 'tabular-nums' | 'proportional-nums';
export interface TextStyleIOS extends ViewStyle {
    fontVariant?: FontVariant[];
    letterSpacing?: number;
    textDecorationColor?: string;
    textDecorationStyle?: 'solid' | 'double' | 'dotted' | 'dashed';
    writingDirection?: 'auto' | 'ltr' | 'rtl';
}
export interface TextStyleAndroid extends ViewStyle {
    textAlignVertical?: 'auto' | 'top' | 'bottom' | 'center';
    includeFontPadding?: boolean;
}
// @see https://facebook.github.io/react-native/docs/text.html#style
export interface TextStyle extends TextStyleIOS, TextStyleAndroid, ViewStyle {
    color?: string;
    fontFamily?: string;
    fontSize?: number;
    fontStyle?: 'normal' | 'italic';
    /**
     * Specifies font weight. The values 'normal' and 'bold' are supported
     * for most fonts. Not all fonts have a variant for each of the numeric
     * values, in that case the closest one is chosen.
     */
    fontWeight?: 'normal' | 'bold' | '100' | '200' | '300' | '400' | '500' | '600' | '700' | '800' | '900';
    letterSpacing?: number;
    lineHeight?: number;
    textAlign?: 'auto' | 'left' | 'right' | 'center' | 'justify';
    textDecorationLine?: 'none' | 'underline' | 'line-through' | 'underline line-through';
    textDecorationStyle?: 'solid' | 'double' | 'dotted' | 'dashed';
    textDecorationColor?: string;
    textShadowColor?: string;
    textShadowOffset?: { width: number; height: number };
    textShadowRadius?: number;
    textTransform?: 'none' | 'capitalize' | 'uppercase' | 'lowercase';
    testID?: string;
}
export interface TextPropsIOS { 
    adjustsFontSizeToFit?: boolean; 
    minimumFontScale?: number; 
    suppressHighlighting?: boolean;
}
export interface TextPropsAndroid { 
    selectable?: boolean; 
    selectionColor?: string; 
    textBreakStrategy?: 'simple' | 'highQuality' | 'balanced';
}
// https://facebook.github.io/react-native/docs/text.html#props
export interface TextProps extends TextPropsIOS, TextPropsAndroid, AccessibilityProps { 
    allowFontScaling?: boolean; 
    ellipsizeMode?: 'head' | 'middle' | 'tail' | 'clip'; 
    lineBreakMode?: 'head' | 'middle' | 'tail' | 'clip'; 
    numberOfLines?: number; 
    onLayout?: (event: LayoutChangeEvent) => void; 
    onPress?: (event: GestureResponderEvent) => void; 
    onLongPress?: (event: GestureResponderEvent) => void; 
    style?: StyleProp<TextStyle>; 
    testID?: string; 
    nativeID?: string;
    maxFontSizeMultiplier?: number | null;
}
/**
 * A React component for displaying text which supports nesting, styling, and touch handling.
 */
declare class TextComponent extends React.Component<TextProps> { }
declare const TextBase: Constructor<NativeMethodsMixin> & typeof TextComponent;
export class Text extends TextBase { }
```

从上面能看出，组件大都有各自的 Props，然后里面有 Style 和支持的属性。Style 往往由 iOS特有、Android特有、两个平台共有等三部分组成，这样子做到了差异性、复用性、维护性。

命名方面，样式以 Style 结尾，属性已 Props 结尾。如果是平台特有，则在名字后追加平台名字 iOS 或 Android。

### 样式Style篇

react-native内置组件中要重点关注几个常用组件样式： FlexStyle、ViewStyle、TextStyle、ImageStyle。

先看看这篇文章 [聊聊怎么写react-native上的样式吧](https://segmentfault.com/a/1190000013332489)，文中总结出下面几点：

#### 分离样式和页面

除非组件特别小，否则都不应该用”css-in-js”模式。通过StyleSheet.create来集中定义样式，或提取一些能复用的样式使用对象合并或扩展运算符合并后再集中定义，这样组件的 style 属性接收单个对象或数组方式，来实现基本的复用和继承，[官网示例](https://reactnative.cn/docs/style/)。
```
import React, { Component } from 'react';
import { AppRegistry, StyleSheet, Text, View } from 'react-native';

export default class LotsOfStyles extends Component {
  render() {
    return (
      <View>
        <Text style={styles.red}>just red</Text>
        <Text style={{
            color: "blue",
            fontWeight: "bold",
            fontSize: 30,
        }}>just bigblue</Text>
        <Text style={[styles.bigblue, styles.red]}>bigblue, then red</Text>
        <Text style={[styles.red, styles.bigblue]}>red, then bigblue</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  bigblue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});

AppRegistry.registerComponent('LotsOfStyles', () => LotsOfStyles);
```


#### 提取项目级的公共属性

从项目在页面上的表现可以看出，使用复用组件可以提高项目质量，样式方面则是将颜色、尺寸等基础属性单独提取到一个 module 中后 export，通过此方式提高项目的可配置性等。
```
import { StyleSheet } from 'react-native';
import {
  px,
  COLOR_BG_RED,
  COLOR_BG_GREEN,
  STYLE_FR_VC_HSB,
  STYLE_FR_VC_HC,
  STYLE_FR_VC_HFS,
} from 'MyStyle';


export default StyleSheet.create({
    // TODO
});
* 归类提取的公共样式：既然定义了项目的公共属性，为了方便管理，那对提取出的样式进行分类。
// color.tsx
export default {
  COLOR_BG_RED,
  COLOR_BG_GREEN,
  // ...
}

// size.tsx
export default {
  // ...
}

// layout.tsx
export default {
  // ...
}

// index.tsx 
export { default * color } from 'color';
export { default * size } from ’size';
export { default * layout } from ‘layout';

import { color, size, layout } from 'MyStyle';
```

#### 通过混合器去创造模板样式

通过函数来实现样式的计算，实现样式的动态能力，如按钮禁用和启用各自不同颜色、不同数量的参数来模拟传统css开发的简写属性。一般情况下，我们习惯在View等组件上面去做样式的运算，这样子没有做到样式和页面的分离。
```
// 模拟传统css开发，这里的形参顺序遵循css中的 “上、右、下、左”
const layout = {
  margin(...arg) {
    let margin = {};
    switch (arg.length) {
      case 1:
        margin = {
          marginTop: arg[0],
          marginRight: arg[0],
          marginBottom: arg[0],
          marginLeft: arg[0],
        };
        break;
      case 2:
        margin = {
          marginVertical: arg[0],
          marginHorizontal: arg[1],
        };
        break;
      case 3:
        margin = {
          marginTop: arg[0],
          marginHorizontal: arg[1],
          marginBottom: arg[2],
        };
        break;
      case 4:
        margin = {
          marginTop: arg[0],
          marginRight: arg[1],
          marginBottom: arg[2],
          marginLeft: arg[3],
        };
        break;
      default:
        break;
    }
    return margin;
  },
};

const styles = StyleSheet.create({
    lines: {
      height: px(88),
      backgroundColor: color.background,
      ...layout.border(10px, 5px, 10px, 5px),
    },
});


// 背景色条件控制
const styleCondition = {
    backgroundColor: (enable: boolean): ViewStyle => {
        return { backgroundColor: enable ? 'red' : 'green' };
    },
};

<View style={[styles.sectionContainer, styleCondition.backgroundColor(enable)]} />
// 等效于
<View style={[styles.sectionContainer, { backgroundColor: enable ? 'red' : 'green' }]} />
```

### 页面Containers篇

[React SFC 无状态组件及多种组件写法](https://blog.lbinin.com/frontEnd/React/React-SFC.html)

React 中创建组件的三种方法：。

* ES5：使用 React.createClass 方法，不推荐！！！（跳过）
* ES6：
    * 有状态类，通过继承 React.Component 或 React.PureComponent。
    * 无状态类（组件）。
* SFC：指TpeScript 中的无状态组件，在大型项目或者组件中经常被使用，未来 React 也会对 SFC 做一些专门的优化，优点：
    * 适当减少代码量，可读性增强；
    * 无状态，统一移交给高阶组件（HOC）或者 Redux 进行管理；
    * 能获取到 ref；

```JavaScript
import * as React from 'react';

interface AppProps {
  title?: React.ReactNode;
}

const App: React.SFC<AppProps> = props => {
  const {title} = props;
  let ref;

  return (
    <div className="App">
        <h1 className="App-title">{title}</h1>
        <div className="App" ref={ref => node = ref}>{title}</div>
    </div>
  )
}
```

关于TypeScript与 React 的更多总结，推荐几篇文章：
* [TypeScript 2.8下的终极React组件模式](https://juejin.im/post/5b07caf16fb9a07aa83f2977)
* [TypeScript 在 React 中使用总结](https://juejin.im/post/5bab4d59f265da0aec22629b)
* [三千字讲清TypeScript与React的实战技巧](https://juejin.im/post/5d3aad8b6fb9a07ecb0bef5e)
* [TypeScript中高级应用与最佳实践](https://juejin.im/post/5d4285ddf265da03dd3d514b)

### 组件Components篇

[Stylesheets in React Native with TypeScript Revisited](https://medium.com/@zvona/stylesheets-in-react-native-with-typescript-revisited-6b4ba0a899d2) 中有个比较完美的例子：
```JavaScript
import * as React from 'react';
import {
  Image,
  ImageStyle,
  StyleProp,
  StyleSheet,
  Text,
  TextStyle,
  TouchableHighlight,
  View,
  ViewStyle,
} from 'react-native';

interface IProps {
  label: string;
  buttonStyle?: StyleProp<ViewStyle>;
  labelStyle?: StyleProp<TextStyle>;
}

interface Styles {
  button: ViewStyle;
  icon: ImageStyle;
  label: TextStyle;
}

const styles = StyleSheet.create<Styles>({
  button: {
    flexDirection: 'row',
    backgroundColor: '#336699',
  },

  icon: {
    width: 16,
    height: 16,
  },

  label: {
    color: '#F8F8F8',
    textAlign: 'center',
  },
});

const Button: React.SFC<IProps> = (props): JSX.Element => (
  <TouchableHighlight>
    <View style={[styles.button, props.buttonStyle]}>
      <Image style={styles.icon} source={require('./assets/someCoolIcon.png')} />
      <Text style={[styles.label, props.labelStyle]}>{props.label}</Text>
    </View>
  </TouchableHighlight>
);
```

参考资料：
* [聊聊怎么写react-native上的样式吧](https://segmentfault.com/a/1190000013332489)
* [React SFC 无状态组件及多种组件写法](https://blog.lbinin.com/frontEnd/React/React-SFC.html)