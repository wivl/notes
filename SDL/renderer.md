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

## 实例

```c
#include <stdio.h>
#include <stdbool.h>
#include <stdlib.h>
#include "SDL.h"

void scc(int code) {
	if (code < 0) {
		fprintf(stderr, "SDL ERROR: %s\n", SDL_GetError());
		exit(1);
	}
}

void *scp(void *ptr) {
	if (ptr == NULL) {
		fprintf(stderr, "SDL ERROR: %s\n", SDL_GetError());
		exit(1);
	}
	return ptr;
}

bool quit = false;

void event_handler(SDL_Event *event) {
    while (SDL_PollEvent(event)) {
			switch (event->type) {
			case SDL_QUIT:
				quit = true;
				break;
		}
	}
}

int main(void) {
	scc(SDL_Init(SDL_INIT_VIDEO));

	SDL_Window *window = scp(SDL_CreateWindow("engine3d", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, 800, 600, SDL_WINDOW_RESIZABLE));
	SDL_Renderer *renderer = scp(SDL_CreateRenderer(window, -1, SDL_RENDERER_SOFTWARE)); // SDL_WINDOW_ACCELERATED will cause splash 

	SDL_SetRenderDrawBlendMode(renderer, SDL_BLENDMODE_BLEND);	// 允许透明

	// SDL_FlashWindow(window, SDL_FLASH_BRIEFLY);	// 设置任务栏标志闪烁
	SDL_Event event = {0};
	while (!quit) {	// 一次循环绘制一帧
		event_handler(&event);

        SDL_SetRenderDrawColor(renderer, 0, 0, 0, 255);
        SDL_RenderClear(renderer);

		SDL_RenderPresent(renderer);

	}

	scc(SDL_RenderClear(renderer));
	SDL_DestroyRenderer(renderer);
	SDL_Quit();
	return 0;
}
```