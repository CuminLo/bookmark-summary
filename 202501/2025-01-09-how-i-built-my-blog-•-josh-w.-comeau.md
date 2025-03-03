# How I Built My Blog • Josh W. Comeau
- URL: https://www.joshwcomeau.com/blog/how-i-built-my-blog-v2/
- Added At: 2025-01-09 08:44:11
- [Link To Text](2025-01-09-how-i-built-my-blog-•-josh-w.-comeau_raw.md)

## TL;DR
Josh W. Comeau 最近完成了个人博客的全面改版，采用了Next.js、MDX、Linaria等现代前端技术，优化了内容管理、样式、代码展示、互动功能等细节。通过引入Server Components、自定义主题、实时同步效果等功能，提升了用户体验和可访问性。改版后的博客不仅外观精致，还具备强大的交互性和搜索功能，展示了现代前端开发的趋势与挑战。

## Summary
1. **博客改版概述**：
   - Josh W. Comeau 最近完成了个人博客的全面改版，新版本在设计和功能上进行了优化。
   - 改版后的博客在外观上更加精致，但主要的变化在于技术栈的更新和细节的改进。

2. **技术栈选择**：
   - **核心技术**：使用了Next.js、MDX、Linaria等现代前端技术。
   - **选择原因**：
     - 博客文章使用MDX编写，需要一流的MDX支持。
     - 为了减少与其他项目（如课程平台）的上下文切换摩擦，选择了Next.js。
     - 希望深入体验React的最新特性，如Server Components和Actions。

3. **内容管理**：
   - **MDX的使用**：MDX结合了Markdown和JSX，允许在文章中嵌入自定义的React组件，增强了内容的交互性。
   - **工作流程**：直接在VS Code中编辑MDX文件，并通过Git提交。文章元数据通过文件顶部的frontmatter设置。

4. **样式与CSS**：
   - **样式库切换**：从styled-components切换到Linaria，以兼容React Server Components。
   - **挑战与解决方案**：Linaria与Next.js的集成遇到了一些问题，但通过逐步调试和优化，最终实现了平滑运行。

5. **代码片段展示**：
   - **语法高亮**：使用Shiki进行代码语法高亮，Shiki在编译时工作，减少了客户端负担。
   - **自定义主题**：设计了自定义的代码片段主题，并提供了下载链接供他人使用。

6. **代码游乐场**：
   - **React游乐场**：使用Sandpack创建交互式代码编辑器。
   - **静态HTML/CSS游乐场**：使用自定义的Playground工具，避免依赖Service Workers。

7. **数据库与互动功能**：
   - **点赞功能**：实现了基于MongoDB的点赞系统，用户可以通过IP地址匿名点赞。
   - **技术实现**：使用Next.js的Route Handler处理点赞逻辑，无需单独的后端服务。

8. **细节优化**：
   - **元素一致性**：花费大量时间确保组件在不同上下文中的样式一致性。
   - **彩虹效果**：在主页添加了响应鼠标的彩虹效果，并通过PartyKit实现实时同步。
   - **视图过渡**：使用View Transitions API实现页面切换时的动画效果。

9. **搜索功能**：
   - **搜索实现**：使用Algolia提供强大的搜索功能，支持模糊匹配。
   - **用户体验**：设计了清除搜索项的动画效果，增强了交互体验。

10. **图标与微交互**：
    - **图标优化**：基于Feather Icons重构了图标，并添加了微交互效果。
    - **动画实现**：使用React Spring实现图标的动态效果。

11. **可访问性改进**：
    - **媒体查询**：改用rem单位进行媒体查询，提升可访问性。
    - **持续学习**：不断学习并应用新的可访问性知识，确保博客对所有用户友好。

12. **App Router与Pages Router对比**：
    - **优点**：App Router的Server Components模型更自然，功能更强大。
    - **挑战**：性能优化方面存在挑战，特别是CSS模块的打包问题。

13. **总结**：
    - 博客改版涉及多个方面的优化，从技术栈的选择到细节的打磨，体现了现代前端开发的趋势和挑战。
    - 通过不断学习和实践，Josh W. Comeau 成功打造了一个功能丰富、用户体验优秀的个人博客。
