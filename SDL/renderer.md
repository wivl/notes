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

## SDL_Texture


```cpp
SDL_Surface *surface = SDL_LoadBMP("demo.bmp");
int w = surface->w;
int h = surface->h;
SDL_Texture *texture = SDL_CreateTextureFromSurface(renderer, surface);
SDL_FreeSurface(surface);

// 显示在哪个位置
SDL_Rect dstrect;
dstrect.x = 100;
dstrect.y = 100;
dstrect.w = w;
dstrect.h = h;

// 显示素材的哪个部分
SDL_Rect *srcrect;
srcrect.x = 9;
srcrect.y = 0;
srcrect.w = 10;
srcrect.h = 10;

// 绘制
SDL_RenderCopy(renderer, texture, &srcrect, &dstrect);
// 旋转
SDL_RenderCopyEx(renderer, texture, &srcrect, &dstrect, float degree, SDL_Point *rotateCenter, SDL_FLIP_NONE);

// 销毁
SDL_DestoryTexture(texture);
```


<code>SDL_TEXTUREACCESS_TARGET</code>: 可以被渲染成 target，即允许以此纹理为绘图目标

```cpp
SDL_Texture *newTexture = SDL_CreateTexture(renderer, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_TARGET|SDL_TEXTUREACCESS_STATIC, w, h);
SDL_SetRenderTarget(renderer, newTexture);		// 设置此纹理为渲染器渲染目标

// 绘制一个矩形和一条直线
SDL_SetRenderDrawColor(renderer, 0, 0, 255, 255);
SDL_RenderClear(renderer);
SDL_SetRenderDrawColor(renderer, 0, 100, 100, 255);
SDL_RenderDrawLine(renderer, -100, -100, 300, 400);

// 设置渲染器渲染目标为默认
SDL_SetRenderTarget(renderer, NULL);
```

```cpp
// 设置 texture 透明度
SDL_SetTextureBlendMode(texture, SDL_BLENDMODE_BLEND);
SDL_SetTextureAlphaMod(texture, 100);
```


```cpp
// 颜色滤镜
SDL_SetTextureColorMod(texture, 255, 0, 0);
```
