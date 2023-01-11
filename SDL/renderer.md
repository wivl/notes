# SDL Renderer

## 部分常用函数

```c
SDL_Renderer *renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);
```

|flag|含义|
|---------------------|-------|
|SDL_RENDERER_SOFTWARE|CPU 渲染|
|SDL_RENDERER_ACCELERATED|GPU 加速|
|SDL_RENDERER_PRESENTVSYNC|垂直同步|
|SDL_RENDERER_TARGETTEXTURE|支持纹理|
|0|SDL 自行选择|

```c
// 使用 RGBA
SDL_SetRenderDrawColor(renderer, 0, 255, 0, 0);
```

```c
// 使用设定好的颜色在 x, y 处绘制一个点
SDL_RenderDrawPoint(renderer, x, y);
```

```c
// 绘制一条线段
SDL_RenderDrawLine(renderer, x1, y1, x2, y2);
```

```c
// 绘制矩形框
SDL_Rect rect = {.x = 50, .y = 50, .w = 100, .h = 100};
SDL_RenderDrawRect(renderer, &rect);
```

```c
SDL_RenderFillRect(renderer, &rect);
```

