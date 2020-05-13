```javascript
export type TypeOfMode = number;

export const NoMode = 0b00000;
export const StrictMode = 0b00001;

export const BlockingMode = 0b00010;
export const ConcurrentMode = 0b00100;
export const ProfileMode = 0b01000;
export const DebugTracingMode = 0b10000;
```

定义应用所处的模式，平常开发中用到的比较多的就是NoMode以及StrictMode了。