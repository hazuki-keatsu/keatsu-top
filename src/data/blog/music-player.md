---
author: Hazuki Keatsu
pubDatetime: 2025-08-27T09:12:00Z
title: 基于 Astro 和 Typescript 的浮动音乐播放器
featured: false
draft: false
tags:
  - astro
  - typescript
description:
  介绍了 AeroPaper 主题中的音乐播放器的实现方法。
---

## 目录

## 效果展示

![对音乐播放器的三种状态的预览](/assets/blog/music-player/preview.webp)

## 使用的技术栈

[Astro](https://astro.build/) 和 [TypeScript](https://www.typescriptlang.org/)

因此，你能在任何的使用Astro和TypeScript的项目中轻松植入这个音乐播放器组件。

## 项目结构

```plain-text
├─ components
│  └─ MusicPlayer.astro # 样式文件
├─ utils
│  └─ musicPlayer.ts    # 脚本文件
└─ constants.ts         # 配置文件
```

这几个文件都可以在开源项目[AeroPaper](https://github.com/hazuki-keatsu/aero-paper)中获取到。

## 使用方法

在任何你需要加入的页面直接插入即可，例如在[AeroPaper](https://github.com/hazuki-keatsu/aero-paper)中的使用：

```astro
<body>
	<slot />
	{(() => {
		// 读取用户配置
		if (!SITE.musicPlayer.enabled) return false;
		if (SITE.musicPlayer.autoShow) return true;
		
		// 获取当前页面路径，移除尾部斜杠并标准化
		const currentPath = Astro.url.pathname.replace(/\/$/, '') || "/";
		const currentPage = currentPath === "/" ? "home" : currentPath.split("/")[1];
		
		// 检查是否在指定页面显示
		return SITE.musicPlayer.showInPages.includes("all") ||
						SITE.musicPlayer.showInPages.includes(currentPage) ||
						SITE.musicPlayer.showInPages.includes(currentPath);
	<!-- [!code ++] -->
	})() && <MusicPlayer />}
	
	<!-- 音乐播放器独立脚本 -->
	<script is:inline>
		// 确保音乐播放器在页面切换时保持状态
		document.addEventListener('astro:after-swap', () => {
			const player = document.getElementById('music-player');
			if (player && !player.hasAttribute('data-astro-persist')) {
				// 如果播放器存在但没有persist属性，添加它
				player.setAttribute('data-astro-persist', '');
			}
		});
	</script>
</body>
```

## 代码实现

### MusicPlayer.astro

```astro
---
// 导入播放列表数据
import { SONG_LIST } from '../constants';
---

<div id="music-player" transition:persist="music-player" class="fixed bottom-5 left-5 z-50 backdrop-blur-lg rounded-xl text-foreground font-mono shadow-2xl transition-all duration-300 ease-in-out overflow-hidden minimized">
  <!-- 最小化状态的按钮 -->
  <div 
    id="player-button" 
    class="content-card flex items-center justify-center cursor-pointer rounded-xl transition-all duration-200 hover:bg-accent/20 group player-button-size"
  >
    <svg class="text-accent group-hover:scale-110 transition-transform duration-200" style="width: 1.5rem; height: 1.5rem;" viewBox="0 0 24 24" fill="currentColor">
      <path d="M8 5v14l11-7z"/>
    </svg>
  </div>

  <!-- 展开状态的播放器 -->
  <div id="player-expanded" class="p-4 hidden player-expanded-width content-card">
    <!-- 头部控制区域 -->
    <div class="flex items-center justify-between mb-4">
      <div class="flex items-center flex-1 min-w-0">
        <img 
          id="cover-image" 
          src={SONG_LIST[0].cover} 
          alt="封面" 
          class="rounded-lg object-cover mr-3 shadow-md"
          style="width: 3rem; height: 3rem;"
        >
        <div class="flex-1 min-w-0">
          <div id="song-title" class="text-sm font-semibold text-foreground truncate mb-1">
            {SONG_LIST[0].title}
          </div>
          <div id="song-artist" class="text-xs text-accent truncate">
            {SONG_LIST[0].artist}
          </div>
        </div>
      </div>
      <button 
        id="minimize-btn" 
        class="flex items-center justify-center rounded-full bg-transparent hover:bg-accent/20 transition-all duration-200 opacity-70 hover:opacity-100 ml-2"
        style="width: 2rem; height: 2rem;"
      >
        <svg style="width: 1rem; height: 1rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M19 13H5v-2h14v2z"/>
        </svg>
      </button>
    </div>

    <!-- 进度条 -->
    <div class="flex items-center gap-2 mb-4">
      <span id="current-time" class="text-xs text-accent min-w-[35px] text-center">0:00</span>
      <div class="flex-1 relative">
        <div class="bg-muted/30 rounded-full overflow-hidden" style="height: 0.25rem;">
          <div id="progress-fill" class="h-full bg-gradient-to-r from-accent to-secondary-highlight rounded-full" style="width: 0%"></div>
        </div>
        <input 
          type="range" 
          id="progress-slider" 
          min="0" 
          max="100" 
          value="0" 
          class="absolute inset-0 w-full h-full opacity-0 cursor-pointer"
        >
      </div>
      <span id="duration" class="text-xs text-accent min-w-[35px] text-center">0:00</span>
    </div>

    <!-- 控制按钮 -->
    <div class="flex items-center justify-center gap-4 mb-4">
      <button 
        id="shuffle-btn" 
        class="mode-btn flex items-center justify-center rounded-full transition-all duration-300 opacity-60 hover:opacity-100 text-foreground"
        style="width: 2.5rem; height: 2.5rem;"
      >
        <svg style="width: 1.1rem; height: 1.1rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M10.59 9.17L5.41 4 4 5.41l5.17 5.17 1.42-1.41zM14.5 4l2.04 2.04L4 18.59 5.41 20 17.96 7.46 20 9.5V4h-5.5zm.33 9.41l-1.41 1.41 3.13 3.13L14.5 20H20v-5.5l-2.04 2.04-3.13-3.13z"/>
        </svg>
        <span class="mode-status">随机</span>
      </button>
      <button 
        id="prev-btn" 
        class="flex items-center justify-center rounded-full bg-transparent hover:bg-accent/20 transition-all duration-200 opacity-80 hover:opacity-100 hover:scale-105 text-foreground"
        style="width: 2.5rem; height: 2.5rem; position: relative;"
      >
        <svg style="width: 1.25rem; height: 1.25rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M6 6h2v12H6zm3.5 6l8.5 6V6z"/>
        </svg>
				<span class="mode-status">上一曲</span>
      </button>
      <button 
        id="play-pause-btn" 
        class="flex items-center justify-center rounded-full bg-accent hover:bg-accent/80 transition-all duration-200 hover:scale-110 shadow-lg text-background"
        style="width: 3rem; height: 3rem; position: relative;"
      >
        <svg id="play-icon" style="width: 1.5rem; height: 1.5rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M8 5v14l11-7z"/>
        </svg>
        <svg id="pause-icon" class="hidden" style="width: 1.5rem; height: 1.5rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z"/>
        </svg>
				<span class="mode-status play-pause-status">播放/暂停</span>
      </button>
      <button 
        id="next-btn" 
        class="flex items-center justify-center rounded-full bg-transparent hover:bg-accent/20 transition-all duration-200 opacity-80 hover:opacity-100 hover:scale-105 text-foreground"
        style="width: 2.5rem; height: 2.5rem; position: relative;"
      >
        <svg style="width: 1.25rem; height: 1.25rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M6 18l8.5-6L6 6v12zM16 6v12h2V6h-2z"/>
        </svg>
				<span class="mode-status">下一曲</span>
      </button>
      <button 
        id="repeat-btn" 
        class="mode-btn flex items-center justify-center rounded-full transition-all duration-300 opacity-60 hover:opacity-100 text-foreground"
        style="width: 2.5rem; height: 2.5rem;"
      >
        <svg id="repeat-off-icon" style="width: 1.1rem; height: 1.1rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M7 7h10v3l4-4-4-4v3H5v6h2V7zm10 10H7v-3l-4 4 4 4v-3h12v-6h-2v4z"/>
        </svg>
        <svg id="repeat-one-icon" class="hidden" style="width: 1.1rem; height: 1.1rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M7 7h10v3l4-4-4-4v3H5v6h2V7zm10 10H7v-3l-4 4 4 4v-3h12v-6h-2v4zm-4-2V9h-1l-2 1v1h1.5v4H13z"/>
        </svg>
				<span class="mode-status">循环</span>
      </button>
    </div>

    <!-- 音量控制 -->
    <div class="flex items-center gap-2 mb-3">
      <button 
        id="volume-btn" 
        class="flex items-center justify-center rounded-full bg-transparent hover:bg-accent/20 transition-all duration-200 opacity-80 hover:opacity-100 text-foreground"
        style="width: 2rem; height: 2rem;"
        title="静音/取消静音"
      >
        <!-- 正常音量图标 -->
        <svg id="volume-on-icon" class="text-accent" style="width: 1rem; height: 1rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z"/>
        </svg>
        <!-- 静音图标 -->
        <svg id="volume-off-icon" class="text-accent hidden" style="width: 1rem; height: 1rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M16.5 12c0-1.77-1.02-3.29-2.5-4.03v2.21l2.45 2.45c.03-.2.05-.41.05-.63zm2.5 0c0 .94-.2 1.82-.54 2.64l1.51 1.51C20.63 14.91 21 13.5 21 12c0-4.28-2.99-7.86-7-8.77v2.06c2.89.86 5 3.54 5 6.71zM4.27 3L3 4.27 7.73 9H3v6h4l5 5v-6.73l4.25 4.25c-.67.52-1.42.93-2.25 1.18v2.06c1.38-.31 2.63-.95 3.69-1.81L19.73 21 21 19.73l-9-9L4.27 3zM12 4L9.91 6.09 12 8.18V4z"/>
        </svg>
      </button>
      <div class="flex-1 relative">
        <div class="bg-muted/30 rounded-full" style="height: 0.25rem;">
          <div id="volume-fill" class="h-full bg-gradient-to-r from-secondary-highlight to-accent rounded-full w-1/2"></div>
        </div>
        <input 
          type="range" 
          id="volume-slider" 
          min="0" 
          max="100" 
          value="50" 
          class="absolute inset-0 w-full h-full opacity-0 cursor-pointer"
        >
      </div>
      <button 
        id="playlist-btn" 
        class="flex text-accent items-center justify-center rounded-full bg-transparent hover:bg-accent/20 transition-all duration-200 opacity-80 hover:opacity-100"
        style="width: 2rem; height: 2rem;"
        title="播放列表"
      >
        <svg style="width: 1.25rem; height: 1.25rem;" viewBox="0 0 24 24" fill="currentColor">
          <path d="M15 6H3v2h12V6zm0 4H3v2h12v-2zM3 16h8v-2H3v2zM17 6v8.18c-.31-.11-.65-.18-1-.18-1.66 0-3 1.34-3 3s1.34 3 3 3 3-1.34 3-3V8h3V6h-5z"/>
        </svg>
      </button>
    </div>

    <!-- 播放列表 -->
    <div id="playlist" class="playlist-container max-h-48 pt-2 custom-scrollbar" style="overflow-y: hidden">
      {SONG_LIST.map((song, index) => (
        <div 
          class="playlist-item flex items-center p-2 cursor-pointer rounded-md transition-all duration-200 hover:bg-accent/20 group" 
          data-index={index}
        >
          <img 
            src={song.cover} 
            alt={song.title} 
            class="rounded object-cover mr-2 group-hover:scale-105 transition-transform duration-200"
            style="width: 2rem; height: 2rem;"
          >
          <div class="flex-1 min-w-0">
            <div class="text-xs font-medium text-foreground truncate group-hover:text-accent transition-colors">
              {song.title}
            </div>
            <div class="text-xs text-accent truncate">
              {song.artist}
            </div>
          </div>
          <div class="rounded-full bg-accent opacity-0 group-hover:opacity-100 transition-opacity duration-200" style="width: 0.5rem; height: 0.5rem;"></div>
        </div>
      ))}
    </div>
  </div>

  <!-- 隐藏的音频元素 -->
  <audio id="audio-player" preload="metadata">
    <source src={SONG_LIST[0].src} type="audio/mpeg">
    您的浏览器不支持音频播放。
  </audio>
</div>

<style>
  /* 音乐播放器样式 - 使用项目配色方案 */
  .music-player-bg {
    background: rgba(var(--color-background) / 0.85);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(var(--color-border) / 0.3);
  }
  
  @media (prefers-color-scheme: dark) {
    html[data-theme="dark"] .music-player-bg {
      background: rgba(52, 63, 96, 0.85);
      border: 1px solid rgba(255, 255, 255, 0.15);
    }
  }
  
  /* 自定义样式 */
  .minimized {
    width: 4rem; /* 64px, equivalent to w-16 */
    height: 4rem; /* 64px, equivalent to h-16 */
  }
  
  .expanded {
    width: 20rem; /* 320px, equivalent to w-80 */
    height: auto;
    max-height: 31.25rem; /* 500px */
  }
  
  .player-button-size {
    width: 4rem; /* 64px, equivalent to w-16 */
    height: 4rem; /* 64px, equivalent to h-16 */
  }
  
  .player-expanded-width {
    width: 20rem; /* 320px, equivalent to w-80 */
  }
  
  .minimized #player-expanded {
    display: none;
  }
  
  .expanded #player-button {
    display: none;
  }
  
  .expanded #player-expanded {
    display: block;
  }

  /* 自定义滚动条 */
  .custom-scrollbar::-webkit-scrollbar {
    width: 0.25rem; /* 4px, equivalent to w-1 */
  }
  
  .custom-scrollbar::-webkit-scrollbar-track {
    background: rgba(var(--color-muted) / 0.3);
    border-radius: 0.25rem;
  }
  
  .custom-scrollbar::-webkit-scrollbar-thumb {
    background: rgba(var(--color-accent) / 0.5);
    border-radius: 0.25rem;
  }
  
  .custom-scrollbar::-webkit-scrollbar-thumb:hover {
    background: rgba(var(--color-accent) / 0.7);
  }

  /* 播放列表活动状态 */
  .playlist-item.active {
    background: rgba(var(--color-accent) / 0.2);
    border-left: 2px solid var(--color-accent);
  }

  .playlist-item.active .rounded-full {
    opacity: 1 !important;
  }

  /* 播放列表过渡动画 */
  .playlist-container {
    max-height: 0;
    opacity: 0;
    padding-top: 0;
    overflow: hidden;
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    transform-origin: top;
  }
  
  .playlist-container.show {
    max-height: 12rem; /* 192px, equivalent to max-h-48 */
    opacity: 1;
    padding-top: 0.5rem; /* 8px, equivalent to pt-2 */
    overflow-y: auto;
  }
  
  .playlist-container .playlist-item {
    transform: translateY(-15px);
    opacity: 0;
    transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  }
  
  .playlist-container.show .playlist-item {
    transform: translateY(0);
    opacity: 1;
  }
  
  /* 播放列表项延迟动画 */
  .playlist-container.show .playlist-item:nth-child(1) { transition-delay: 0.1s; }
  .playlist-container.show .playlist-item:nth-child(2) { transition-delay: 0.15s; }
  .playlist-container.show .playlist-item:nth-child(3) { transition-delay: 0.2s; }
  .playlist-container.show .playlist-item:nth-child(4) { transition-delay: 0.25s; }
  .playlist-container.show .playlist-item:nth-child(5) { transition-delay: 0.3s; }
  
  /* 播放列表按钮增强样式 */
  #playlist-btn {
    transition: all 0.2s ease-out;
  }
  
  #playlist-btn:hover {
    transform: scale(1.1);
  }
  
  #playlist-btn:active {
    transform: scale(0.95);
  }

  /* 播放模式按钮样式 */
  .mode-btn, #shuffle-btn, #repeat-btn {
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    background: rgba(var(--color-muted) / 0.1);
    border: 1px solid rgba(var(--color-border) / 0.3);
  }
  
  .mode-btn:hover, #shuffle-btn:hover, #repeat-btn:hover {
    transform: scale(1.1);
    background: rgba(var(--color-accent) / 0.1);
    border-color: rgba(var(--color-accent) / 0.3);
  }
  
  .mode-btn:active, #shuffle-btn:active, #repeat-btn:active {
    transform: scale(0.95);
  }
  
  /* 播放模式激活状态 - 更明显的视觉效果 */
  .mode-btn.active, #shuffle-btn.active, #repeat-btn.active {
    opacity: 1 !important;
    background: linear-gradient(135deg, rgba(var(--color-accent) / 0.8), rgba(var(--color-secondary-highlight) / 0.6));
    // color: var(--color-accent);
    border: 1px solid var(--color-accent);
    box-shadow: 
      0 0 0 2px rgba(var(--color-accent) / 0.2),
      0 4px 12px rgba(var(--color-accent) / 0.3),
      inset 0 1px 0 rgba(255, 255, 255, 0.2);
    transform: scale(1.05);
  }
  
  .mode-btn.active:hover, #shuffle-btn.active:hover, #repeat-btn.active:hover {
    background: linear-gradient(135deg, rgba(var(--color-accent) / 0.9), rgba(var(--color-secondary-highlight) / 0.7));
    box-shadow: 
      0 0 0 3px rgba(var(--color-accent) / 0.3),
      0 6px 16px rgba(var(--color-accent) / 0.4),
      inset 0 1px 0 rgba(255, 255, 255, 0.3);
    transform: scale(1.1);
  }
  
  /* 激活状态脉冲动画 */
  @keyframes pulse-glow {
    0% { 
      box-shadow: 
        0 0 0 2px rgba(var(--color-accent) / 0.2),
        0 4px 12px rgba(var(--color-accent) / 0.3),
        inset 0 1px 0 rgba(255, 255, 255, 0.2);
    }
    50% { 
      box-shadow: 
        0 0 0 3px rgba(var(--color-accent) / 0.4),
        0 6px 16px rgba(var(--color-accent) / 0.5),
        inset 0 1px 0 rgba(255, 255, 255, 0.3);
    }
    100% { 
      box-shadow: 
        0 0 0 2px rgba(var(--color-accent) / 0.2),
        0 4px 12px rgba(var(--color-accent) / 0.3),
        inset 0 1px 0 rgba(255, 255, 255, 0.2);
    }
  }
  
  .mode-btn.active, #shuffle-btn.active, #repeat-btn.active {
    animation: pulse-glow 2s ease-in-out infinite;
  }
  
  /* 图标增强效果 */
  .mode-btn svg, #shuffle-btn svg, #repeat-btn svg {
    transition: all 0.2s ease-out;
    filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.1));
  }
  
  .mode-btn.active svg, #shuffle-btn.active svg, #repeat-btn.active svg {
    filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
    transform: scale(1.1);
  }
  
  /* 激活指示器 */
  .mode-btn.active::after, #shuffle-btn.active::after, #repeat-btn.active::after {
    content: '';
    position: absolute;
    top: -2px;
    right: -2px;
    width: 6px;
    height: 6px;
    background: var(--color-secondary-highlight);
    border-radius: 50%;
    box-shadow: 0 0 0 1px var(--color-background), 0 0 8px rgba(var(--color-secondary-highlight) / 0.6);
    animation: indicator-pulse 1.5s ease-in-out infinite;
  }
  
  /* 单曲循环特殊样式 */
  #repeat-btn.repeat-one {
    background: linear-gradient(135deg, rgba(var(--color-secondary-highlight) / 0.8), rgba(var(--color-accent) / 0.6)) !important;
  }
  
  #repeat-btn.repeat-one::after {
    background: var(--color-accent) !important;
    box-shadow: 0 0 0 1px var(--color-background), 0 0 12px rgba(var(--color-accent) / 0.8) !important;
  }
  
  #repeat-btn.repeat-one:hover {
    background: linear-gradient(135deg, rgba(var(--color-secondary-highlight) / 0.9), rgba(var(--color-accent) / 0.7)) !important;
  }
  
  /* 随机播放特殊样式 */
  #shuffle-btn.active {
    background: linear-gradient(135deg, rgba(var(--color-accent) / 0.8), rgba(var(--color-secondary-highlight) / 0.6)) !important;
  }
  
  #shuffle-btn.active svg {
    animation: shuffle-shake 0.8s ease-in-out infinite;
  }
  
  @keyframes shuffle-shake {
    0%, 100% { transform: scale(1.1) rotate(0deg); }
    25% { transform: scale(1.1) rotate(-2deg); }
    50% { transform: scale(1.1) rotate(2deg); }
    75% { transform: scale(1.1) rotate(-1deg); }
  }
  
  /* 状态指示文字 */
  .mode-status {
    position: absolute;
    bottom: -18px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 0.625rem; /* 10px */
    color: var(--color-accent);
    background: rgba(var(--color-background) / 0.9);
    padding: 1px 4px;
    border-radius: 2px;
    border: 1px solid rgba(var(--color-accent) / 0.3);
    white-space: nowrap;
    opacity: 0;
    transition: opacity 0.3s ease-out;
    pointer-events: none;
    font-weight: 500;
    backdrop-filter: blur(4px);
  }
  
  .mode-btn.active .mode-status,
  #shuffle-btn.active .mode-status,
  #repeat-btn.active .mode-status {
    opacity: 1;
  }
  
  .mode-btn:hover .mode-status,
  #shuffle-btn:hover .mode-status,
  #repeat-btn:hover .mode-status,
  #prev-btn:hover .mode-status,
  #next-btn:hover .mode-status,
  #play-pause-btn:hover .mode-status {
    opacity: 0.8;
  }
  
  /* 播放/暂停按钮状态文字特殊样式 */
  .play-pause-status {
    color: var(--color-background) !important;
    background: rgba(var(--color-background) / 0.2) !important;
    border-color: rgba(var(--color-background) / 0.3) !important;
  }
  
  /* 播放/暂停按钮在播放状态下的细微变化 */
  #play-pause-btn.playing {
    box-shadow: 0 0 12px rgba(var(--color-accent) / 0.4);
  }
  
  @keyframes indicator-pulse {
    0%, 100% { 
      transform: scale(1);
      opacity: 1;
    }
    50% { 
      transform: scale(1.2);
      opacity: 0.8;
    }
  }

  /* 进度条动画效果 */
  #progress-fill {
    transition: width 0.1s ease-out, transform 0.15s ease-out;
    transform-origin: left center;
  }
  
  #progress-slider {
    transition: opacity 0.2s ease-out;
  }
  
  #progress-slider:hover {
    opacity: 0.3 !important;
  }
  
  /* 进度条发光效果 */
  @keyframes progress-glow {
    0% { box-shadow: 0 0 5px rgba(var(--color-accent) / 0.5); }
    50% { box-shadow: 0 0 15px rgba(var(--color-accent) / 0.8); }
    100% { box-shadow: 0 0 5px rgba(var(--color-accent) / 0.5); }
  }
  
  #progress-fill:active {
    animation: progress-glow 0.6s ease-out;
  }

  /* 音量控制动画效果 */
  #volume-fill {
    transition: width 0.1s ease-out, transform 0.15s ease-out;
    transform-origin: left center;
  }
  
  #volume-btn {
    transition: all 0.2s ease-out;
  }
  
  #volume-btn:hover {
    transform: scale(1.1);
  }
  
  #volume-btn:active {
    transform: scale(0.95);
  }
  
  /* 音量图标动画 */
  #volume-on-icon, #volume-off-icon {
    transition: opacity 0.2s ease-out, transform 0.2s ease-out;
  }
  
  /* 音量滑块增强样式 */
  #volume-slider {
    transition: opacity 0.2s ease-out;
  }
  
  #volume-slider:hover {
    opacity: 0.3 !important;
  }
  
  /* 音量条发光效果 */
  @keyframes volume-glow {
    0% { box-shadow: 0 0 5px rgba(var(--color-secondary-highlight) / 0.5); }
    50% { box-shadow: 0 0 15px rgba(var(--color-secondary-highlight) / 0.8); }
    100% { box-shadow: 0 0 5px rgba(var(--color-secondary-highlight) / 0.5); }
  }
  
  #volume-fill:active {
    animation: volume-glow 0.6s ease-out;
  }

  /* 响应式设计 */
  @media (max-width: 480px) {
    .expanded {
      width: calc(100vw - 2.5rem);
      max-width: 20rem; /* 320px, equivalent to max-w-80 */
    }
  }
</style>

<script>
  import { MusicPlayer } from '../utils/musicPlayer.ts';
	import { SONG_LIST } from '../constants'

  // 全局变量声明
  declare global {
    interface Window {
      musicPlayerInstance?: MusicPlayer;
    }
  }

  // 当DOM加载完成后初始化播放器
  function initPlayer() {
    // 检查是否已经初始化过了
    if (window.musicPlayerInstance) {
      return; // 已经初始化，不需要重复
    }
    
    try {
      const player = new MusicPlayer(SONG_LIST);
      window.musicPlayerInstance = player;
      console.log('音乐播放器初始化成功');
    } catch (error) {
      console.error('音乐播放器初始化失败:', error);
    }
  }

  // 页面加载时初始化
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initPlayer);
  } else {
    initPlayer();
  }

  // 监听页面切换事件，确保播放器在新页面中仍然工作
  document.addEventListener('astro:after-swap', () => {
    // transition:persist会保持DOM元素，但我们需要确保事件监听器仍然有效
    // 如果实例不存在，重新初始化
    if (!window.musicPlayerInstance) {
      setTimeout(initPlayer, 10);
    }
  });
</script>
```

### musicPlayer.ts

```typescript
// 音乐播放器数据类型定义
export interface Song {
  id: number;
  title: string;
  artist: string;
  cover: string;
  src: string;
}

// 播放模式枚举
export enum PlayMode {
  Normal = 'normal',    // 正常播放
  Shuffle = 'shuffle',  // 随机播放
  Repeat = 'repeat',    // 列表循环
  RepeatOne = 'repeat-one' // 单曲循环
}

// 音乐播放器类
export class MusicPlayer {
  private playlist: Song[];
  private originalPlaylist: Song[]; // 保存原始播放列表
  private currentTrackIndex: number = 0;
  private isPlaying: boolean = false;
  private isPlaylistVisible: boolean = false;
  private isMuted: boolean = false;
  private previousVolume: number = 50; // 保存静音前的音量
  private playMode: PlayMode = PlayMode.Normal; // 当前播放模式
  
  // DOM元素
  private player!: HTMLDivElement;
  private playerButton!: HTMLDivElement;
  private audio!: HTMLAudioElement;
  private playPauseBtn!: HTMLButtonElement;
  private playIcon!: SVGElement;
  private pauseIcon!: SVGElement;
  private prevBtn!: HTMLButtonElement;
  private nextBtn!: HTMLButtonElement;
  private minimizeBtn!: HTMLButtonElement;
  private progressSlider!: HTMLInputElement;
  private progressFill!: HTMLDivElement;
  private volumeFill!: HTMLDivElement;
  private currentTimeEl!: HTMLSpanElement;
  private durationEl!: HTMLSpanElement;
  private volumeSlider!: HTMLInputElement;
  private coverImage!: HTMLImageElement;
  private songTitle!: HTMLDivElement;
  private songArtist!: HTMLDivElement;
  private playlistBtn!: HTMLButtonElement;
  private volumeBtn!: HTMLButtonElement;
  private volumeOnIcon!: SVGElement;
  private volumeOffIcon!: SVGElement;
  private playlistEl!: HTMLDivElement;
  private playlistItems!: NodeListOf<HTMLDivElement>;
  private volumeTooltip?: HTMLDivElement; // 音量提示框
  private progressTooltip?: HTMLDivElement; // 进度提示框
  private shuffleBtn!: HTMLButtonElement; // 随机播放按钮
  private repeatBtn!: HTMLButtonElement; // 循环播放按钮
  private repeatOffIcon!: SVGElement; // 循环关闭图标
  private repeatOneIcon!: SVGElement; // 单曲循环图标
  private repeatStatus!: HTMLSpanElement; // 循环状态文字

  constructor(playlist: Song[]) {
    this.playlist = [...playlist]; // 创建副本
    this.originalPlaylist = [...playlist]; // 保存原始列表
    this.initElements();
    this.bindEvents();
    this.loadTrack(this.currentTrackIndex);
    this.setVolume(); // 初始化音量显示
    this.updateVolumeIcon(); // 初始化音量图标
    this.createVolumeTooltip(); // 创建音量提示框
    this.createProgressTooltip(); // 创建进度提示框
    this.updateButtonTitle(); // 初始化按钮提示文字
  }

  private initElements(): void {
    this.player = document.getElementById('music-player') as HTMLDivElement;
    this.playerButton = document.getElementById('player-button') as HTMLDivElement;
    this.audio = document.getElementById('audio-player') as HTMLAudioElement;
    this.playPauseBtn = document.getElementById('play-pause-btn') as HTMLButtonElement;
    this.playIcon = document.getElementById('play-icon') as unknown as SVGElement;
    this.pauseIcon = document.getElementById('pause-icon') as unknown as SVGElement;
    this.prevBtn = document.getElementById('prev-btn') as HTMLButtonElement;
    this.nextBtn = document.getElementById('next-btn') as HTMLButtonElement;
    this.minimizeBtn = document.getElementById('minimize-btn') as HTMLButtonElement;
    this.progressSlider = document.getElementById('progress-slider') as HTMLInputElement;
    this.progressFill = document.getElementById('progress-fill') as HTMLDivElement;
    this.volumeFill = document.getElementById('volume-fill') as HTMLDivElement;
    this.currentTimeEl = document.getElementById('current-time') as HTMLSpanElement;
    this.durationEl = document.getElementById('duration') as HTMLSpanElement;
    this.volumeSlider = document.getElementById('volume-slider') as HTMLInputElement;
    this.coverImage = document.getElementById('cover-image') as HTMLImageElement;
    this.songTitle = document.getElementById('song-title') as HTMLDivElement;
    this.songArtist = document.getElementById('song-artist') as HTMLDivElement;
    this.playlistBtn = document.getElementById('playlist-btn') as HTMLButtonElement;
    this.volumeBtn = document.getElementById('volume-btn') as HTMLButtonElement;
    this.volumeOnIcon = document.getElementById('volume-on-icon') as unknown as SVGElement;
    this.volumeOffIcon = document.getElementById('volume-off-icon') as unknown as SVGElement;
    this.playlistEl = document.getElementById('playlist') as HTMLDivElement;
    this.playlistItems = document.querySelectorAll('.playlist-item') as NodeListOf<HTMLDivElement>;
    this.shuffleBtn = document.getElementById('shuffle-btn') as HTMLButtonElement;
    this.repeatBtn = document.getElementById('repeat-btn') as HTMLButtonElement;
    this.repeatOffIcon = document.getElementById('repeat-off-icon') as unknown as SVGElement;
    this.repeatOneIcon = document.getElementById('repeat-one-icon') as unknown as SVGElement;
    this.repeatStatus = document.getElementById('repeat-status') as HTMLSpanElement;
  }

  private bindEvents(): void {
    this.playerButton?.addEventListener('click', () => this.expand());
    this.minimizeBtn?.addEventListener('click', () => this.minimize());
    this.playPauseBtn?.addEventListener('click', () => this.togglePlay());
    this.prevBtn?.addEventListener('click', () => this.previousTrack());
    this.nextBtn?.addEventListener('click', () => this.nextTrack());
    this.progressSlider?.addEventListener('input', () => this.seek());
    this.progressSlider?.addEventListener('mousedown', () => this.showProgressTooltip());
    this.progressSlider?.addEventListener('mouseup', () => this.hideProgressTooltip());
    this.progressSlider?.addEventListener('mouseleave', () => this.hideProgressTooltip());
    this.audio?.addEventListener('timeupdate', () => this.updateProgress());
    this.audio?.addEventListener('loadedmetadata', () => this.updateDuration());
    this.audio?.addEventListener('ended', () => this.onTrackEnded());
    this.volumeSlider?.addEventListener('input', () => this.setVolume());
    this.volumeSlider?.addEventListener('mousedown', () => this.showVolumeTooltip());
    this.volumeSlider?.addEventListener('mouseup', () => this.hideVolumeTooltip());
    this.volumeSlider?.addEventListener('mouseleave', () => this.hideVolumeTooltip());
    this.playlistBtn?.addEventListener('click', () => this.togglePlaylist());
    this.volumeBtn?.addEventListener('click', () => this.toggleMute());
    this.shuffleBtn?.addEventListener('click', () => this.toggleShuffle());
    this.repeatBtn?.addEventListener('click', () => this.toggleRepeat());
    
    this.playlistItems?.forEach((item, index) => {
      item.addEventListener('click', () => this.selectTrack(index));
    });
  }

  private formatTime(seconds: number): string {
    if (isNaN(seconds)) return '0:00';
    const mins = Math.floor(seconds / 60);
    const secs = Math.floor(seconds % 60);
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  }

  private expand(): void {
    this.player?.classList.remove('minimized');
    this.player?.classList.add('expanded');
  }

  private minimize(): void {
    this.player?.classList.remove('expanded');
    this.player?.classList.add('minimized');
    // 最小化时隐藏播放列表
    this.hidePlaylist();
  }

  private togglePlay(): void {
    if (this.isPlaying) {
      this.pause();
    } else {
      this.play();
    }
  }

  private play(): void {
    this.audio?.play().then(() => {
      this.isPlaying = true;
      this.playIcon?.classList.add('hidden');
      this.pauseIcon?.classList.remove('hidden');
      this.playPauseBtn?.classList.add('playing');
    }).catch((error: Error) => {
      console.error('播放失败:', error);
    });
  }

  private pause(): void {
    this.audio?.pause();
    this.isPlaying = false;
    this.playIcon?.classList.remove('hidden');
    this.pauseIcon?.classList.add('hidden');
    this.playPauseBtn?.classList.remove('playing');
  }

  private previousTrack(): void {
    const wasPlaying = this.isPlaying; // 记录当前播放状态
    
    if (this.playMode === PlayMode.Shuffle) {
      // 随机播放模式：随机选择一首歌
      this.playRandomTrack();
    } else {
      // 正常模式或循环模式：播放上一首
      this.currentTrackIndex = this.currentTrackIndex > 0 
        ? this.currentTrackIndex - 1 
        : this.playlist.length - 1;
      this.loadTrack(this.currentTrackIndex);
    }
    
    // 如果之前在播放，继续播放新的音轨
    if (wasPlaying) {
      this.play();
    }
  }

  private nextTrack(): void {
    const wasPlaying = this.isPlaying; // 记录当前播放状态
    
    switch (this.playMode) {
      case PlayMode.RepeatOne:
        // 单曲循环：重新播放当前歌曲
        this.loadTrack(this.currentTrackIndex);
        break;
      case PlayMode.Shuffle:
        // 随机播放：随机选择下一首
        this.playRandomTrack();
        break;
      case PlayMode.Repeat:
      case PlayMode.Normal:
      default:
        // 正常播放或列表循环：播放下一首
        this.currentTrackIndex = this.currentTrackIndex < this.playlist.length - 1 
          ? this.currentTrackIndex + 1 
          : 0;
        this.loadTrack(this.currentTrackIndex);
        break;
    }
    
    // 如果之前在播放，继续播放新的音轨
    if (wasPlaying) {
      this.play();
    }
  }

  private loadTrack(index: number): void {
    const track = this.playlist[index];
    if (this.audio) this.audio.src = track.src;
    if (this.coverImage) this.coverImage.src = track.cover;
    if (this.songTitle) this.songTitle.textContent = track.title;
    if (this.songArtist) this.songArtist.textContent = track.artist;
    
    // 重置播放状态和按钮状态
    this.isPlaying = false;
    this.playIcon?.classList.remove('hidden');
    this.pauseIcon?.classList.add('hidden');
    this.playPauseBtn?.classList.remove('playing');
    
    // 更新播放列表活动状态
    this.updatePlaylistActiveState(index);
    
    // 重置进度
    if (this.progressSlider) this.progressSlider.value = '0';
    if (this.progressFill) this.animateProgressBar('0%');
    if (this.currentTimeEl) this.currentTimeEl.textContent = '0:00';
  }

  private updatePlaylistActiveState(activeIndex: number): void {
    this.playlistItems?.forEach((item, index) => {
      if (index === activeIndex) {
        item.classList.add('active');
      } else {
        item.classList.remove('active');
      }
    });
  }

  private selectTrack(index: number): void {
    this.currentTrackIndex = index;
    this.loadTrack(index);
    if (this.isPlaying) {
      this.play();
    }
  }

  private seek(): void {
    if (this.progressSlider && this.audio) {
      const seekTime = (parseFloat(this.progressSlider.value) / 100) * this.audio.duration;
      this.audio.currentTime = seekTime;
      // 添加进度条动画
      this.animateProgressBar(this.progressSlider.value + '%');
      // 显示进度提示
      this.showProgressTooltip();
    }
  }

  private updateProgress(): void {
    if (this.audio?.duration) {
      const progress = (this.audio.currentTime / this.audio.duration) * 100;
      if (this.progressSlider) this.progressSlider.value = progress.toString();
      if (this.progressFill) {
        // 使用动画更新进度条，但在自动播放时使用更短的动画时间
        this.animateProgressBar(progress + '%', true);
      }
      if (this.currentTimeEl) this.currentTimeEl.textContent = this.formatTime(this.audio.currentTime);
    }
  }

  private updateDuration(): void {
    if (this.durationEl && this.audio) {
      this.durationEl.textContent = this.formatTime(this.audio.duration);
    }
  }

  private setVolume(): void {
    if (this.volumeSlider && this.audio && this.volumeFill) {
      const volume = parseFloat(this.volumeSlider.value) / 100;
      this.audio.volume = volume;
      
      // 添加缓动动画到音量条
      this.animateVolumeBar(this.volumeSlider.value + '%');
      
      // 显示实时音量值
      this.showVolumeTooltip();
      
      // 如果不是手动设置为0，则取消静音状态
      if (volume > 0 && this.isMuted) {
        this.isMuted = false;
        this.updateVolumeIcon();
      } else if (volume === 0 && !this.isMuted) {
        // 如果手动调节到0，设置为静音状态
        this.isMuted = true;
        this.updateVolumeIcon();
      }
    }
  }

  private toggleMute(): void {
    if (!this.audio || !this.volumeSlider || !this.volumeFill || !this.volumeBtn) return;
    
    // 添加按钮点击动画
    this.addButtonClickAnimation(this.volumeBtn);
    
    if (this.isMuted) {
      // 取消静音：恢复之前的音量
      this.audio.volume = this.previousVolume / 100;
      this.volumeSlider.value = this.previousVolume.toString();
      this.animateVolumeBar(this.previousVolume + '%');
      this.isMuted = false;
    } else {
      // 静音：保存当前音量并设置为0
      this.previousVolume = parseFloat(this.volumeSlider.value);
      this.audio.volume = 0;
      this.volumeSlider.value = '0';
      this.animateVolumeBar('0%');
      this.isMuted = true;
    }
    
    this.updateVolumeIcon();
  }

  private updateVolumeIcon(): void {
    if (!this.volumeOnIcon || !this.volumeOffIcon) return;
    
    if (this.isMuted || (this.audio && this.audio.volume === 0)) {
      this.volumeOnIcon.classList.add('hidden');
      this.volumeOffIcon.classList.remove('hidden');
      // 添加图标切换动画
      this.addIconSwitchAnimation(this.volumeOffIcon);
    } else {
      this.volumeOnIcon.classList.remove('hidden');
      this.volumeOffIcon.classList.add('hidden');
      // 添加图标切换动画
      this.addIconSwitchAnimation(this.volumeOnIcon);
    }
  }

  // 音量条缓动动画
  private animateVolumeBar(targetWidth: string): void {
    if (!this.volumeFill) return;
    
    // 添加CSS过渡效果
    this.volumeFill.style.transition = 'width 0.3s cubic-bezier(0.4, 0, 0.2, 1)';
    this.volumeFill.style.width = targetWidth;
    
    // 添加脉冲效果
    this.volumeFill.style.transform = 'scaleY(1.2)';
    
    setTimeout(() => {
      if (this.volumeFill) {
        this.volumeFill.style.transform = 'scaleY(1)';
      }
    }, 150);
    
    // 清除过渡效果，避免影响后续操作
    setTimeout(() => {
      if (this.volumeFill) {
        this.volumeFill.style.transition = '';
      }
    }, 300);
  }

  // 按钮点击动画
  private addButtonClickAnimation(button: HTMLElement): void {
    // 添加点击波纹效果
    button.style.transform = 'scale(0.95)';
    button.style.transition = 'transform 0.1s ease-out';
    
    setTimeout(() => {
      button.style.transform = 'scale(1.05)';
    }, 50);
    
    setTimeout(() => {
      button.style.transform = 'scale(1)';
    }, 150);
    
    setTimeout(() => {
      button.style.transition = '';
    }, 200);
  }

  // 图标切换动画
  private addIconSwitchAnimation(icon: SVGElement): void {
    // 添加弹性缩放动画
    icon.style.transform = 'scale(0.8)';
    icon.style.transition = 'transform 0.2s cubic-bezier(0.68, -0.55, 0.265, 1.55)';
    
    setTimeout(() => {
      icon.style.transform = 'scale(1)';
    }, 50);
    
    setTimeout(() => {
      icon.style.transition = '';
    }, 250);
  }

  // 创建音量提示框
  private createVolumeTooltip(): void {
    this.volumeTooltip = document.createElement('div');
    this.volumeTooltip.className = 'volume-tooltip';
    this.volumeTooltip.style.cssText = `
      position: absolute;
      bottom: 100%;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 4px 8px;
      border-radius: 4px;
      font-size: 12px;
      white-space: nowrap;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.2s ease-out;
      z-index: 1000;
    `;
    
    // 添加到音量滑块的父容器
    if (this.volumeSlider && this.volumeSlider.parentElement) {
      this.volumeSlider.parentElement.style.position = 'relative';
      this.volumeSlider.parentElement.appendChild(this.volumeTooltip);
    }
  }

  // 显示音量提示框
  private showVolumeTooltip(): void {
    if (this.volumeTooltip && this.volumeSlider) {
      const volume = Math.round(parseFloat(this.volumeSlider.value));
      this.volumeTooltip.textContent = `${volume}%`;
      this.volumeTooltip.style.opacity = '1';
    }
  }

  // 隐藏音量提示框
  private hideVolumeTooltip(): void {
    if (this.volumeTooltip) {
      this.volumeTooltip.style.opacity = '0';
    }
  }

  // 进度条缓动动画
  private animateProgressBar(targetWidth: string, isAutoUpdate: boolean = false): void {
    if (!this.progressFill) return;
    
    // 自动更新时使用更短的动画时间，手动拖拽时使用较长的动画时间
    const duration = isAutoUpdate ? '0.1s' : '0.3s';
    const curve = isAutoUpdate ? 'ease-out' : 'cubic-bezier(0.4, 0, 0.2, 1)';
    
    // 添加CSS过渡效果
    this.progressFill.style.transition = `width ${duration} ${curve}`;
    this.progressFill.style.width = targetWidth;
    
    if (!isAutoUpdate) {
      // 手动拖拽时添加脉冲效果
      this.progressFill.style.transform = 'scaleY(1.5)';
      
      setTimeout(() => {
        if (this.progressFill) {
          this.progressFill.style.transform = 'scaleY(1)';
        }
      }, 150);
    }
    
    // 清除过渡效果，避免影响后续操作
    setTimeout(() => {
      if (this.progressFill && !isAutoUpdate) {
        this.progressFill.style.transition = '';
      }
    }, isAutoUpdate ? 100 : 300);
  }

  // 创建进度提示框
  private createProgressTooltip(): void {
    this.progressTooltip = document.createElement('div');
    this.progressTooltip.className = 'progress-tooltip';
    this.progressTooltip.style.cssText = `
      position: absolute;
      bottom: 100%;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 4px 8px;
      border-radius: 4px;
      font-size: 12px;
      white-space: nowrap;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.2s ease-out;
      z-index: 1000;
      margin-bottom: 8px;
    `;
    
    // 添加到进度滑块的父容器
    if (this.progressSlider && this.progressSlider.parentElement) {
      this.progressSlider.parentElement.style.position = 'relative';
      this.progressSlider.parentElement.appendChild(this.progressTooltip);
    }
  }

  // 显示进度提示框
  private showProgressTooltip(): void {
    if (this.progressTooltip && this.progressSlider && this.audio) {
      const progress = parseFloat(this.progressSlider.value);
      const currentTime = (progress / 100) * this.audio.duration;
      this.progressTooltip.textContent = this.formatTime(currentTime);
      this.progressTooltip.style.opacity = '1';
    }
  }

  // 隐藏进度提示框
  private hideProgressTooltip(): void {
    if (this.progressTooltip) {
      this.progressTooltip.style.opacity = '0';
    }
  }

  private togglePlaylist(): void {
    // 添加按钮点击动画
    if (this.playlistBtn) {
      this.addButtonClickAnimation(this.playlistBtn);
    }
    
    this.isPlaylistVisible = !this.isPlaylistVisible;
    if (this.playlistEl) {
      if (this.isPlaylistVisible) {
        // 展开播放列表动画
        this.showPlaylist();
      } else {
        // 收起播放列表动画
        this.hidePlaylist();
      }
    }
  }

  private showPlaylist(): void {
    if (this.playlistEl) {
      // 添加显示类来触发动画
      this.playlistEl.classList.add('show');
      this.isPlaylistVisible = true;
    }
  }

  private hidePlaylist(): void {
    if (this.playlistEl) {
      // 移除显示类来触发收起动画
      this.playlistEl.classList.remove('show');
      this.isPlaylistVisible = false;
    }
  }

  private onTrackEnded(): void {
    // 根据播放模式处理歌曲结束
    switch (this.playMode) {
      case PlayMode.RepeatOne:
        // 单曲循环：重新播放当前歌曲
        if (this.isPlaying) {
          this.play();
        }
        break;
      case PlayMode.Repeat:
      case PlayMode.Shuffle:
        // 列表循环或随机播放：播放下一首
        this.nextTrack();
        if (this.isPlaying) {
          setTimeout(() => this.play(), 100); // 稍微延迟以确保加载完成
        }
        break;
      case PlayMode.Normal:
      default:
        // 正常模式：检查是否是最后一首
        if (this.currentTrackIndex < this.playlist.length - 1) {
          this.nextTrack();
          if (this.isPlaying) {
            setTimeout(() => this.play(), 100);
          }
        } else {
          // 最后一首歌结束，停止播放
          this.pause();
        }
        break;
    }
  }

  // 播放模式相关方法
  private toggleShuffle(): void {
    if (this.shuffleBtn) {
      this.addButtonClickAnimation(this.shuffleBtn);
    }

    if (this.playMode === PlayMode.Shuffle) {
      // 关闭随机播放
      this.playMode = PlayMode.Normal;
      this.shuffleBtn?.classList.remove('active');
      // 恢复原始播放列表顺序
      this.restoreOriginalPlaylist();
    } else {
      // 开启随机播放
      this.playMode = PlayMode.Shuffle;
      this.shuffleBtn?.classList.add('active');
      // 如果当前是循环模式，则关闭循环
      if (this.repeatBtn?.classList.contains('active')) {
        this.playMode = PlayMode.Shuffle; // 随机播放优先
        this.repeatBtn.classList.remove('active');
      }
      // 打乱播放列表
      this.shufflePlaylist();
    }
    
    this.updateButtonTitle();
  }

  private toggleRepeat(): void {
    if (this.repeatBtn) {
      this.addButtonClickAnimation(this.repeatBtn);
    }

    switch (this.playMode) {
      case PlayMode.Normal:
      case PlayMode.Shuffle:
        // 切换到列表循环
        this.playMode = PlayMode.Repeat;
        this.repeatBtn?.classList.add('active');
        this.repeatBtn?.classList.remove('repeat-one'); // 移除单曲循环样式
        this.repeatOffIcon?.classList.remove('hidden');
        this.repeatOneIcon?.classList.add('hidden');
        if (this.repeatStatus) this.repeatStatus.textContent = '列表';
        // 如果当前是随机模式，则关闭随机
        if (this.shuffleBtn?.classList.contains('active')) {
          this.shuffleBtn.classList.remove('active');
          this.restoreOriginalPlaylist();
        }
        break;
      case PlayMode.Repeat:
        // 切换到单曲循环
        this.playMode = PlayMode.RepeatOne;
        this.repeatBtn?.classList.add('repeat-one'); // 添加单曲循环样式
        this.repeatOffIcon?.classList.add('hidden');
        this.repeatOneIcon?.classList.remove('hidden');
        if (this.repeatStatus) this.repeatStatus.textContent = '单曲';
        break;
      case PlayMode.RepeatOne:
        // 关闭循环
        this.playMode = PlayMode.Normal;
        this.repeatBtn?.classList.remove('active');
        this.repeatBtn?.classList.remove('repeat-one'); // 移除单曲循环样式
        this.repeatOffIcon?.classList.remove('hidden');
        this.repeatOneIcon?.classList.add('hidden');
        if (this.repeatStatus) this.repeatStatus.textContent = '循环';
        break;
    }
    
    this.updateButtonTitle();
  }

  private shufflePlaylist(): void {
    // 保存当前播放的歌曲
    const currentSong = this.playlist[this.currentTrackIndex];
    
    // 创建不包含当前歌曲的列表
    const otherSongs = this.playlist.filter((_, index) => index !== this.currentTrackIndex);
    
    // 打乱其他歌曲
    for (let i = otherSongs.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [otherSongs[i], otherSongs[j]] = [otherSongs[j], otherSongs[i]];
    }
    
    // 重建播放列表，当前歌曲保持在第一位
    this.playlist = [currentSong, ...otherSongs];
    this.currentTrackIndex = 0;
  }

  private restoreOriginalPlaylist(): void {
    // 保存当前播放的歌曲
    const currentSong = this.playlist[this.currentTrackIndex];
    
    // 恢复原始播放列表
    this.playlist = [...this.originalPlaylist];
    
    // 找到当前歌曲在原始列表中的位置
    this.currentTrackIndex = this.playlist.findIndex(song => song.id === currentSong.id);
    if (this.currentTrackIndex === -1) {
      this.currentTrackIndex = 0;
    }
  }

  private playRandomTrack(): void {
    // 生成随机索引，确保不重复当前歌曲
    let randomIndex;
    do {
      randomIndex = Math.floor(Math.random() * this.playlist.length);
    } while (randomIndex === this.currentTrackIndex && this.playlist.length > 1);
    
    this.currentTrackIndex = randomIndex;
    this.loadTrack(this.currentTrackIndex);
  }

  private updateButtonTitle(): void {
    // 更新按钮提示文字
    if (this.shuffleBtn) {
      this.shuffleBtn.title = this.playMode === PlayMode.Shuffle ? '关闭随机播放' : '随机播放';
    }
    
    if (this.repeatBtn) {
      switch (this.playMode) {
        case PlayMode.Repeat:
          this.repeatBtn.title = '单曲循环';
          break;
        case PlayMode.RepeatOne:
          this.repeatBtn.title = '关闭循环';
          break;
        default:
          this.repeatBtn.title = '列表循环';
          break;
      }
    }
  }

  // 获取当前播放器状态
  public getState() {
    return {
      isExpanded: this.player?.classList.contains('expanded') || false,
      currentTrack: this.currentTrackIndex,
      isPlaying: this.isPlaying,
      currentTime: this.audio?.currentTime || 0,
      volume: this.audio?.volume ? this.audio.volume * 100 : 50,
      playMode: this.playMode,
      isMuted: this.isMuted,
      isPlaylistVisible: this.isPlaylistVisible,
      previousVolume: this.previousVolume
    };
  }

  // 恢复播放器状态
  public restoreState(state: any) {
    if (!state) return;

    // 恢复播放状态
    if (state.currentTrack !== undefined && state.currentTrack !== this.currentTrackIndex) {
      this.currentTrackIndex = state.currentTrack;
      this.loadTrack(this.currentTrackIndex);
    }

    // 恢复音量
    if (state.volume !== undefined) {
      this.audio.volume = state.volume / 100;
      this.volumeSlider.value = state.volume.toString();
      this.setVolume();
    }

    // 恢复播放模式
    if (state.playMode !== undefined) {
      this.playMode = state.playMode;
      this.updateModeButtons();
    }

    // 恢复静音状态
    if (state.isMuted !== undefined) {
      this.isMuted = state.isMuted;
      this.updateVolumeIcon();
    }

    // 恢复其他状态
    if (state.previousVolume !== undefined) {
      this.previousVolume = state.previousVolume;
    }

    // 恢复展开状态
    if (state.isExpanded) {
      this.expand();
    } else {
      this.minimize();
    }

    // 恢复播放列表可见性
    if (state.isPlaylistVisible) {
      this.showPlaylist();
    }

    // 恢复播放位置（如果音频已加载）
    if (state.currentTime && this.audio.readyState >= 2) {
      this.audio.currentTime = state.currentTime;
    }

    // 如果之前在播放，恢复播放状态
    if (state.isPlaying) {
      // 延迟一下确保音频加载完成
      setTimeout(() => {
        this.play();
      }, 100);
    }
  }

  // 清理播放器资源
  public destroy() {
    // 暂停播放
    if (this.audio) {
      this.audio.pause();
    }

    // 移除提示框
    if (this.volumeTooltip && this.volumeTooltip.parentNode) {
      this.volumeTooltip.parentNode.removeChild(this.volumeTooltip);
    }
    if (this.progressTooltip && this.progressTooltip.parentNode) {
      this.progressTooltip.parentNode.removeChild(this.progressTooltip);
    }
  }

  // 更新模式按钮状态的辅助方法
  private updateModeButtons() {
    // 更新随机播放按钮
    if (this.shuffleBtn) {
      if (this.playMode === PlayMode.Shuffle) {
        this.shuffleBtn.classList.add('active');
      } else {
        this.shuffleBtn.classList.remove('active');
      }
    }

    // 更新循环播放按钮
    if (this.repeatBtn && this.repeatOffIcon && this.repeatOneIcon) {
      this.repeatBtn.classList.remove('active', 'repeat-one');
      
      switch (this.playMode) {
        case PlayMode.Repeat:
          this.repeatBtn.classList.add('active');
          this.repeatOffIcon.classList.remove('hidden');
          this.repeatOneIcon.classList.add('hidden');
          break;
        case PlayMode.RepeatOne:
          this.repeatBtn.classList.add('active', 'repeat-one');
          this.repeatOffIcon.classList.add('hidden');
          this.repeatOneIcon.classList.remove('hidden');
          break;
        default:
          this.repeatOffIcon.classList.remove('hidden');
          this.repeatOneIcon.classList.add('hidden');
          break;
      }
    }

    this.updateButtonTitle();
  }
}

```

### constants.ts

```typescript
interface Song {
  id: number;
  title: string;
  artist: string;
  cover: string;
  src: string;
}

export const SONG_LIST: Song[] = [
  {
    id: 1,
    title: "spiral",
    artist: "LONGMAN",
    cover: "/LONGMAN%20-%20spiral.jpg",
    src: "/LONGMAN%20-%20spiral.mp3"
  },
  {
    id: 2,
    title: "Dye the sky. -25 colors-",
    artist: "シャイニーカラーズ",
    cover: "http://p2.music.126.net/tM1S4-q5Tt2cYUs0LTrcrw==/109951170278147013.jpg",
    src: "https://er-sycdn.kuwo.cn/f2eda961da3391e9449d1f47431a4a83/68ae5fdf/resource/30106/trackmedia/M800003xGgeK3y8O46.mp3"
  }
] as const;

```

## 总结

整体的代码量虽然有点大，主要是在样式和动画的设计上。整体的实现并不是很复杂。

做这么一个玩意主要还是对于博客关于页面的一个拓展，用户可以将自己喜欢的歌曲展示在这里。

> 代码基于MIT协议开源，详情请见[AeroPaper](https://github.com/hazuki-keatsu/aero-paper)