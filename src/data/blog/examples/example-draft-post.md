---
author: Sat Naing
pubDatetime: 2022-09-23T15:22:00Z
title: 如何添加新的 Post
slug: how-to-add-new-post
featured: false
draft: false
tags:
  - example
description:
  使用 AeroPaper 主题创建或添加新文章的一些规则与建议。
---

引用自 [AstroPaper](https://github.com/satnaing/astro-paper/blob/main/src/data/blog/adding-new-post.md) 官方的文档，由 Hazuki Keatsu 翻译。


## 目录

## 一、创建博客文章
要撰写新博客文章，需在 `src/data/blog/` 目录下创建一个 Markdown 文件。

从 AstroPaper v5.1.0 版本开始，你可以将博客文章整理到子目录中，更便于管理内容。

例如，若想将文章归类到“2025”主题下，可将文章放在 `src/data/blog/2025/` 目录中。这一操作会影响文章的 URL 路径：`src/data/blog/2025/example-post.md` 对应的访问地址为 `/posts/2025/example-post`。

若不希望子目录影响文章的 URL 路径，只需在文件夹名称前添加下划线 `_` 即可。

```plain-text
# 示例：博客文章结构与对应 URL
src/data/blog/very-first-post.md          -> 网站域名.com/posts/very-first-post
src/data/blog/2025/example-post.md        -> 网站域名.com/posts/2025/example-post
src/data/blog/_2026/another-post.md       -> 网站域名.com/posts/another-post
src/data/blog/docs/_legacy/how-to.md      -> 网站域名.com/posts/docs/how-to
src/data/blog/Example Dir/Dummy Post.md   -> 网站域名.com/posts/example-dir/dummy-post
```

> 💡 提示：你也可以在文章的“前端配置”（Frontmatter）中自定义文章的链接别名（slug），具体可参考下一部分内容。

如果构建输出结果中未显示子目录对应的 URL，可删除 `node_modules` 文件夹，重新安装依赖包后再执行构建操作。


## 二、前端配置
“前端配置”是存储博客文章关键信息的核心区域，位于文章顶部，使用 YAML 格式编写。关于前端配置的更多用法，可参考 [Astro 官方文档](https://docs.astro.build/en/guides/markdown-content/)。

以下是每篇文章前端配置的属性列表：

| 属性（Property）       | 描述（Description）                                                                                                                   | 说明（Remark）                                  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **_title_**（标题）    | 文章标题（对应 HTML 中的 h1 标签）                                                                                                    | 必选<sup>\*</sup>                               |
| **_description_**（描述） | 文章描述，用于文章摘要和文章页面的网站描述（Site Description）                                                                       | 必选<sup>\*</sup>                               |
| **_pubDatetime_**（发布时间） | 发布时间，需使用 ISO 8601 格式                                                                                                       | 必选<sup>\*</sup>                               |
| **_modDatetime_**（修改时间） | 修改时间，需使用 ISO 8601 格式（仅在文章修改后添加此属性）                                                                           | 可选                                            |
| **_author_**（作者）   | 文章作者                                                                                                                             | 默认值 = 网站配置中的 SITE.author                |
| **_slug_**（链接别名） | 文章的 URL 唯一标识，此属性为可选                                                                                                     | 默认值 = 文件名转换后的 slug 格式（小写、空格替换为短横线） |
| **_featured_**（推荐） | 是否在首页的“推荐文章”区域展示该文章                                                                                                 | 默认值 = false（不展示）                         |
| **_draft_**（草稿）    | 将文章标记为“未发布”状态                                                                                                             | 默认值 = false（已发布）                         |
| **_tags_**（标签）     | 文章的相关关键词，需使用 YAML 数组格式编写                                                                                           | 默认值 = others（其他）                          |
| **_ogImage_**（社交预览图） | 文章的社交预览图（OG Image），用于社交媒体分享和 SEO 优化，可使用远程 URL 或相对于当前文件夹的图片路径                                 | 默认值 = 网站配置中的 SITE.ogImage 或自动生成的预览图 |
| **_canonicalURL_**（规范链接） | 规范链接（需为绝对路径），适用于文章已在其他平台发布的场景                                                                           | 默认值 = `Astro.site` + `Astro.url.pathname`（当前文章的完整链接） |
| **_hideEditPost_**（隐藏编辑按钮） | 是否在文章标题下方隐藏“编辑文章”按钮，仅对当前文章生效                                                                               | 默认值 = false（显示按钮）                       |
| **_timezone_**（时区） | 为当前文章指定 IANA 格式的时区，此设置会覆盖网站配置中的 `SITE.timezone`                                                             | 默认值 = 网站配置中的 `SITE.timezone`            |

> 提示！在浏览器控制台中运行 `new Date().toISOString()` 可快速获取 ISO 8601 格式的时间（注意移除生成结果中的引号）。

前端配置中，仅 `title`（标题）、`description`（描述）和 `pubDatetime`（发布时间）为必填字段。

标题和描述（摘要）对搜索引擎优化（SEO）至关重要，因此 AstroPaper 主题建议所有文章都必须包含这两个字段。

`slug` 是文章 URL 的唯一标识，因此需确保每篇文章的 `slug` 互不重复。`slug` 中的空格建议使用短横线 `-` 或下划线 `_` 分隔，推荐优先使用短横线 `-`。系统会自动根据文章文件名生成 `slug`，但你也可以在前端配置中自定义 `slug` 以覆盖默认值。更多关于 slug 的说明，可参考 [Astro 官方文档](https://docs.astro.build/en/guides/content-collections/#defining-custom-slugs)。

若文章未在前端配置中指定 `tags`（标签），系统会自动为其添加默认标签 `others`（其他）。你可在 `content.config.ts` 文件中修改默认标签：

```ts file="src/content.config.ts"
export const blogSchema = z.object({
  // ...
  draft: z.boolean().optional(),
  // [!code highlight:1]
  tags: z.array(z.string()).default(["others"]), // 将 "others" 替换为你需要的默认标签
  // ...
});
```

### 前端配置示例
以下是一篇文章的前端配置示例：

```yaml file="src/data/blog/sample-post.md"
---
title: 文章标题
author: 你的名字
pubDatetime: 2022-09-21T05:17:19Z
slug: the-title-of-the-post
featured: true
draft: false
tags:
  - 示例标签1
  - 示例标签2
  - 示例标签3
ogImage: ../../assets/images/example.png # 对应路径：src/assets/images/example.png
# ogImage: "https://example.org/remote-image.png" # 也可使用远程图片 URL
description: 这是一篇示例文章的描述内容。
canonicalURL: https://example.org/my-article-was-already-posted-here
---
```


## 三、添加目录
默认情况下，文章不会自动生成目录（Table of Contents）。若需添加目录，需按特定格式操作：

使用 Markdown 的二级标题格式（`##`）编写“Table of contents”，并将其放置在你希望目录显示的位置。

例如，若想将目录放在引言段落下方（如本文所示），可按以下方式编写：

<!-- prettier-ignore-start -->
```md
---
# 前端配置内容
---

以下是在 AstroPaper 主题中创建新文章的一些规则、建议及实用技巧。

<!-- [!code ++] -->
## 目录

<!-- 文章其余内容 -->
```
<!-- prettier-ignore-end -->


## 四、标题层级规范
关于标题层级，有一点需要注意：AstroPaper 主题会将文章前端配置中的 `title`（标题）作为文章的一级标题（h1）。因此，文章内容中的标题应从二级标题（h2）开始使用，最高可使用六级标题（h6）。

此规则虽非强制要求，但为了视觉美观、无障碍访问和 SEO 优化，强烈建议遵循。


## 五、语法高亮
AstroPaper 主题默认使用 [Shiki](https://shiki.style/) 作为语法高亮工具。从 AstroPaper v5.4 版本开始，主题引入了 [@shikijs/transformers](https://shiki.style/packages/transformers) 以增强代码块的显示效果。若你不需要此功能，可按以下步骤移除：

1. 卸载 `@shikijs/transformers` 依赖包：
```bash
pnpm remove @shikijs/transformers
```

2. 修改 `astro.config.ts` 文件，删除相关配置：
```js file="astro.config.ts"
// ...
// [!code --:5]
import {
  transformerNotationDiff,
  transformerNotationHighlight,
  transformerNotationWordHighlight,
} from "@shikijs/transformers";

export default defineConfig({
  // ...
  markdown: {
    remarkPlugins: [remarkToc, [remarkCollapse, { test: "Table of contents" }]],
    shikiConfig: {
      // 更多主题可参考：https://shiki.style/themes
      themes: { light: "min-light", dark: "night-owl" },
      defaultColor: false,
      wrap: false,
      transformers: [
        transformerFileName(),
      // [!code --:3]
        transformerNotationHighlight(),
        transformerNotationWordHighlight(),
        transformerNotationDiff({ matchAlgorithm: "v3" }),
      ],
    },
  },
  // ...
}
```


## 六、博客图片的存储方式
以下两种方法可用于存储图片并在 Markdown 文章中显示图片。

> 注意！若需在 Markdown 中对优化后的图片进行样式自定义，应使用 [MDX 格式](https://docs.astro.build/en/guides/images/#images-in-mdx-files)。

### 方法一：存储在 `src/assets/` 目录下（推荐）
可将图片存储在 `src/assets/` 目录中，Astro 会通过 [Image Service API](https://docs.astro.build/en/reference/image-service-reference/) 自动对这些图片进行优化。

可使用相对路径或别名路径（`@/assets/`）引用这些图片。

示例：假设需显示的图片 `example.jpg` 路径为 `/src/assets/images/example.jpg`，可按以下方式引用：
```md
![图片描述](@/assets/images/example.jpg)

<!-- 或使用相对路径 -->

![图片描述](../../assets/images/example.jpg)

<!-- 注意：使用 img 标签或 Image 组件引用会失效 ❌ -->
<img src="@/assets/images/example.jpg" alt="图片描述">
<!-- ^^ 这种写法是错误的 -->
```

> 从技术角度而言，图片可存储在 `src` 目录下的任意子目录中，此处推荐 `src/assets` 仅为规范建议。

### 方法二：存储在 `public` 目录下
也可将图片存储在 `public` 目录中，但需注意：`public` 目录下的图片不会经过 Astro 优化处理，需自行处理图片优化（如压缩、格式转换等）。

引用 `public` 目录下的图片时，需使用绝对路径，且可通过 [Markdown 图片语法](https://www.markdownguide.org/basic-syntax/#images-1) 或 [HTML 的 img 标签](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img) 显示。

示例：假设图片 `example.jpg` 路径为 `/public/assets/images/example.jpg`，可按以下方式引用：
```md
![图片描述](/assets/images/example.jpg)

<!-- 或使用 img 标签 -->

<img src="/assets/images/example.jpg" alt="图片描述">
```


## 七、额外建议
### 1. 图片压缩
在文章中插入图片（尤其是存储在 `public` 目录下的图片）时，建议先对图片进行压缩处理，这有助于提升网站的整体加载性能。

以下是推荐的图片压缩工具网站：
- [TinyPng](https://tinypng.com/)（适用于 PNG 格式图片）
- [TinyJPG](https://tinyjpg.com/)（适用于 JPG 格式图片）

### 2. 社交预览图（OG Image）
若文章未指定社交预览图（OG Image），系统会自动使用默认预览图。虽非必填，但建议为每篇文章指定与内容相关的预览图，并在前端配置中添加 `ogImage` 字段。社交预览图的推荐尺寸为 **_1200×640_** 像素。

> 从 AstroPaper v1.4.0 版本开始，若文章未指定社交预览图，系统会自动生成预览图。更多详情可参考 [官方公告](https://astro-paper.pages.dev/posts/dynamic-og-image-generation-in-astropaper-blog-posts/)。