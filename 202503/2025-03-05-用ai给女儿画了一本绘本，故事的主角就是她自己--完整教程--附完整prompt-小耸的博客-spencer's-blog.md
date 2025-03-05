# 用AI给女儿画了一本绘本，故事的主角就是她自己--完整教程--附完整prompt | 小耸的博客 Spencer's Blog
- URL: https://xiaosong.fun/2024/04/15/anan/
- Added At: 2025-03-05 04:18:14
- [Link To Text](2025-03-05-用ai给女儿画了一本绘本，故事的主角就是她自己--完整教程--附完整prompt-小耸的博客-spencer's-blog_raw.md)

## TL;DR
作者利用Midjourney和ChatGPT为女儿创作了一本个性化绘本，通过Midjourney v6的新功能解决了角色一致性问题，并使用ChatGPT生成故事和提示词。整个过程展示了AI工具在创意领域的潜力，尤其是角色一致性和复杂场景生成方面的进步。作者分享了详细的提示词和绘图过程，为其他用户提供了参考。

## Summary
1. **项目背景**：
   - 作者使用AI工具Midjourney和ChatGPT为女儿创作了一本绘本，故事的主角是女儿自己。
   - 在Midjourney v6版本之前，角色一致性难以保持，v6版本引入了`–cref`和`–sref`参数，解决了这一问题。

2. **生成主角参考图**：
   - 使用Midjourney基于一张照片生成主角参考图，确保后续绘图中主角的一致性。
   - 提示词要求：背景干净（纯色），普通站姿，无夸张动作。
   - 示例提示词：`[原始照片url] a 3-year-old cute Chinese girl in a red one-piece dress, full length portrait, pure background, children’s story book illustration, picture book style, --chaos 20`。

3. **生成绘本故事**：
   - 使用ChatGPT生成适合3岁女孩的儿童绘本故事，分画面撰写，包含“插画描述”和“旁白文本”。
   - 提示词示例：`请你扮演一个儿童绘本作家，并且精通用midjourney生成图画的方法。请你写一个适合于3岁女孩的儿童绘本故事。请分画面撰写，每幅画面中，都包含“插画描述”和“旁白文本”两部分。`

4. **将故事转为Midjourney提示词**：
   - 使用ChatGPT将每页的插画描述转为Midjourney提示词，但生成的提示词需要大量修正。
   - 问题：
     - ChatGPT将无关信息纳入提示词（如“creatively”）。
     - 缺少画面风格相关的提示词和参数。

5. **使用提示词绘图**：
   - 根据修订后的提示词，使用Midjourney生成绘本插图。
   - 保持风格一致的技巧：
     - 所有画面使用统一的风格提示词（如`colored ink drawing, giving the scene a magical, dreamlike, and engaging atmosphere, A heartwarming children's book illustration.`）。
     - 使用`–cref`参数保持角色一致性，引用主角参考图。
     - 使用`–sref`参数保持画面质感连续性，将上一幅画面的输出作为下一幅画面的输入。
     - 多次尝试调整提示词，直到生成满意的画面。

6. **将旁白加入图画**：
   - 使用绘图工具（如PS、美图秀秀或Win10自带的“画图”程序）将旁白贴入图片中。

7. **成果展示**：
   - 展示了绘本的封面和16幅插图，每幅插图均附有详细的Midjourney提示词。
   - 示例提示词：
     - 图1：`A joyful three-year-old girl surrounded by colorful balloons and toys in her room, with sparkling eyes full of imagination. ink drawing, giving the scene a magical, dreamlike, and engaging atmosphere, A heartwarming children's book illustration. --cref [主角参考图url] --cw 50 --chaos 30`
     - 图6：`In heavy rain, some kittens, puppies and a littyle girl, are running at full speed, looking for shelter in the garden, dark clouds gather and the wind starts to pick up. colored ink drawing, giving the scene a magical, dreamlike, and engaging atmosphere, A children's book illustration. --cref [主角参考图url] --cw 20 --sref [图5 url] --chaos 40 --v 6.0 --no house`

8. **总结**：
   - 通过Midjourney和ChatGPT的协作，作者成功创作了一本以女儿为主角的绘本。
   - 项目展示了AI工具在创意领域的潜力，尤其是在保持角色一致性和生成复杂场景方面的进步。
   - 作者分享了完整的提示词和绘图过程，为其他用户提供了参考。
