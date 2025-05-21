---
author: Axton
pubDatetime: 2025-05-21T15:22:00Z
modDatetime: 2025-05-21T15:22:00Z
title: React Native 语义化主题系统模板
featured: false
draft: false
tags:
  - React Native
description:
   这里是一个完整的 React Native 语义化主题系统模板，适用于支持多主题（如 light、dark、blue）的大型项目结构。
---

这里是一个完整的 React Native 语义化主题系统模板，适用于支持多主题（如 light、dark、blue）的大型项目结构。

---

## 📁 目录结构

```
src/
├── themes/
│   ├── types.ts
│   ├── light.ts
│   ├── dark.ts
│   ├── blue.ts
│   ├── index.ts
├── hooks/
│   └── useThemeColors.ts
├── components/
│   ├── ThemedText.tsx
│   ├── ThemedView.tsx
│   └── ThemedButton.tsx
├── context/
│   └── ThemeProvider.tsx
```

---

## 1️⃣ `themes/types.ts`

```ts
export type ThemeType = 'light' | 'dark' | 'blue';

export interface ThemeColors {
  text: {
    primary: string;
    secondary: string;
    error: string;
    link: string;
  };
  background: {
    page: string;
    card: string;
    modal: string;
  };
  border: {
    default: string;
    focused: string;
  };
  button: {
    primary: string;
    danger: string;
    disabled: string;
    text: string;
  };
}
```

---

## 2️⃣ 主题文件（light.ts、dark.ts、blue.ts）

例如 `light.ts`：

```ts
import { ThemeColors } from './types';

export const light: ThemeColors = {
  text: {
    primary: '#000',
    secondary: '#666',
    error: '#f00',
    link: '#1e90ff',
  },
  background: {
    page: '#fff',
    card: '#f8f8f8',
    modal: '#eee',
  },
  border: {
    default: '#ccc',
    focused: '#007bff',
  },
  button: {
    primary: '#007bff',
    danger: '#ff4444',
    disabled: '#aaa',
    text: '#fff',
  },
};
```

同理 `dark.ts`、`blue.ts` 只要颜色改动即可。

---

## 3️⃣ `themes/index.ts`

```ts
import { ThemeColors, ThemeType } from './types';
import { light } from './light';
import { dark } from './dark';
import { blue } from './blue';

export const themes: Record<ThemeType, ThemeColors> = {
  light,
  dark,
  blue,
};
```

---

## 4️⃣ `context/ThemeProvider.tsx`

```tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';
import { ThemeType } from '../themes/types';
import { themes } from '../themes';

interface ThemeContextValue {
  themeType: ThemeType;
  colors: typeof themes.light;
  setTheme: (theme: ThemeType) => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

export const ThemeProvider = ({ children }: { children: ReactNode }) => {
  const [themeType, setThemeType] = useState<ThemeType>('light');

  const value: ThemeContextValue = {
    themeType,
    colors: themes[themeType],
    setTheme: setThemeType,
  };

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
};

export const useThemeContext = () => {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useThemeContext must be used within ThemeProvider');
  return ctx;
};
```

---

## 5️⃣ `hooks/useThemeColors.ts`

```ts
import { useThemeContext } from '../context/ThemeProvider';

export const useColors = () => useThemeContext().colors;
```

---

## 6️⃣ `components/ThemedText.tsx`

```tsx
import React from 'react';
import { Text, TextProps, StyleProp, TextStyle } from 'react-native';
import { useColors } from '../hooks/useThemeColors';

interface Props extends TextProps {
  color?: keyof ReturnType<typeof useColors>['text'];
  style?: StyleProp<TextStyle>;
}

export const ThemedText: React.FC<Props> = ({ color = 'primary', style, ...rest }) => {
  const colors = useColors();
  return <Text style={[{ color: colors.text[color] }, style]} {...rest} />;
};
```

---

## 7️⃣ `components/ThemedView.tsx`

```tsx
import React from 'react';
import { View, ViewProps, StyleProp, ViewStyle } from 'react-native';
import { useColors } from '../hooks/useThemeColors';

interface Props extends ViewProps {
  variant?: keyof ReturnType<typeof useColors>['background'];
  style?: StyleProp<ViewStyle>;
}

export const ThemedView: React.FC<Props> = ({ variant = 'page', style, ...rest }) => {
  const colors = useColors();
  return <View style={[{ backgroundColor: colors.background[variant] }, style]} {...rest} />;
};
```

---

## 8️⃣ `components/ThemedButton.tsx`

```tsx
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';
import { useColors } from '../hooks/useThemeColors';

interface Props {
  title: string;
  onPress: () => void;
  variant?: keyof ReturnType<typeof useColors>['button'];
  disabled?: boolean;
}

export const ThemedButton: React.FC<Props> = ({
  title,
  onPress,
  variant = 'primary',
  disabled = false,
}) => {
  const colors = useColors();
  const bg = disabled ? colors.button.disabled : colors.button[variant];
  const textColor = colors.button.text;

  return (
    <TouchableOpacity
      style={[styles.button, { backgroundColor: bg, opacity: disabled ? 0.6 : 1 }]}
      onPress={onPress}
      disabled={disabled}
    >
      <Text style={[styles.text, { color: textColor }]}>{title}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    padding: 12,
    borderRadius: 6,
    alignItems: 'center',
  },
  text: {
    fontWeight: '600',
    fontSize: 16,
  },
});
```

---

## 9️⃣ 应用顶层包裹 `ThemeProvider`

### `App.tsx`

```tsx
import React from 'react';
import { ThemeProvider } from './src/context/ThemeProvider';
import HomeScreen from './src/screens/HomeScreen';

export default function App() {
  return (
    <ThemeProvider>
      <HomeScreen />
    </ThemeProvider>
  );
}
```

---

## 🔟 示例页面 `HomeScreen.tsx`

```tsx
import React from 'react';
import { ThemedView } from '../components/ThemedView';
import { ThemedText } from '../components/ThemedText';
import { ThemedButton } from '../components/ThemedButton';
import { useThemeContext } from '../context/ThemeProvider';

export default function HomeScreen() {
  const { themeType, setTheme } = useThemeContext();

  return (
    <ThemedView variant="page" style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <ThemedText color="primary" style={{ marginBottom: 20 }}>
        当前主题：{themeType}
      </ThemedText>

      <ThemedButton title="切换到 dark" onPress={() => setTheme('dark')} />
    </ThemedView>
  );
}
```

---

## ✅ 总结

| 特性                   | 支持 |
| -------------------- | -- |
| 多主题（light/dark/blue） | ✅  |
| 语义化颜色结构              | ✅  |
| 可拓展性强（新增主题、颜色结构）     | ✅  |
| 中大型项目可维护性高           | ✅  |
| 支持封装组件复用             | ✅  |

---

