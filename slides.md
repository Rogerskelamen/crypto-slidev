---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: 现代密码学项目报告
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# 现代密码学项目报告

MP-SPDZ项目部分代码分析

<div class="pt-12 text-base">
  <div class="m-2">汇报人：钟毅文</div>
  <div>2024年6月12日</div>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/data61/MP-SPDZ" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->



---
transition: slide-left
---

# 主要内容

本次讲解的内容

<br>

<div v-click class="ml-10">
  <h3>编译执行</h3>

  这是理解项目代码行为的基本工作

  **意义**：通过分析编译行为和执行流，可以对深入研究项目提供帮助
</div>

<br>

<div v-click class="ml-10">
  <h3>指令系统</h3>

  该项目有一整套完整的指令系统（类似于CISC指令集）

  更准确来说应该是建立在虚拟机上的字节码指令

  **意义**：理解这套指令系统，可以帮助我们更加轻松地添加自定义协议和指令
</div>

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<!--
Here is another comment.
-->



---
src: ./pages/compile.md
---


---
src: ./pages/instructions.md
---


---
layout: end
---

# 项目汇报结束

感谢聆听