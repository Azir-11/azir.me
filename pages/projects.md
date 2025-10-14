---
title: Projects - Anthony Fu
display: Projects
description: List of projects that I am proud of
wrapperClass: 'text-center'
art: dots
projects:
  Current Focus:
    - name: 'SoybeanAdmin'
      link: 'https://github.com/soybeanjs/soybean-admin'
      desc: '一个清新优雅、高颜值且功能强大的后台管理模板'
      icon: 'Soybean'
    - name: 'GinVueAdmin'
      link: 'https://github.com/flipped-aurora/gin-vue-admin'
      desc: 'Vite+Vue3+Gin拥有AI辅助的基础开发平台，企业级业务AI+开发解决方案'
      icon: 'GinVueAdmin'

  SoybeanJS:
    - name: 'SoybeanAdmin'
      link: 'https://github.com/soybeanjs/soybean-admin'
      desc: '一个清新优雅、高颜值且功能强大的后台管理模板'
      icon: 'Soybean'
    - name: 'SoybeanAdminElementPlus'
      link: 'https://github.com/soybeanjs/soybean-admin'
      desc: 'soybean-admin 的 Element Plus 版本'
      icon: 'Soybean'
    - name: 'SoybeanAdminAntDesignVue'
      link: 'https://github.com/soybeanjs/soybean-admin'
      desc: 'soybean-admin 的 Ant Design Vue 版本'
      icon: 'Soybean'
    - name: 'SoybeanAdminReact'
      link: 'https://github.com/soybeanjs/soybean-admin'
      desc: 'soybean-admin 的 React 版本'
      icon: 'Soybean'
    - name: 'SoybeanUi'
      link: 'https://github.com/soybeanjs/soybean-ui'
      desc: 'SoybeanUI is an elegant and accessible UI library like shadcn for Vue3, it is based on @soybean-ui/primitives and UnoCSS'
      icon: 'Soybean'

  FlippedAurora:
    - name: 'GinVueAdmin'
      link: 'https://github.com/flipped-aurora/gin-vue-admin'
      desc: 'Vite+Vue3+Gin拥有AI辅助的基础开发平台，企业级业务AI+开发解决方案'
      icon: 'GinVueAdmin'

  Vectutil:
    - name: 'func2md'
      link: 'https://github.com/Vectutil/func2md'
      desc: '一个用于从 JSDoc 和 TypeScript 类型生成 VitePress 文档的 Vite 插件'
      icon: 'i-tabler-transform'
    - name: 'sendx'
      link: 'https://github.com/Vectutil/sendx'
      desc: '企业微信/钉钉/等平台的消息推送SDK，提供多种消息类型支持'
      icon: 'i-token-push'
    - name: 'batchx'
      link: 'https://github.com/Vectutil/batchx'
      desc: '基于内存的轻量批量处理队列'
      icon: 'i-mingcute-task-2-line'

  Utils & Libraries:
    - name: 'Arcdash'
      link: 'https://github.com/Azir-11/arcdash'
      desc: '现代、简洁、类型化、强大的业务函数库'
      icon: 'i-ic-baseline-speed'

  VS Code:
    - name: 'Azir theme'
      link: 'https://github.com/Azir-11/azir-theme'
      desc: '基于个人偏好制作的 VS Code 主题'
      icon: 'i-material-icon-theme-folder-theme saturate-0'

  Browser Extension:
    - name: 'Quick Prompt'
      link: 'https://github.com/wenyuanw/quick-prompt'
      desc: '提示词管理与快捷输入浏览器插件'
      icon: 'i-tabler-input-ai'

  Applications:
    - name: 'ChatGPT-Desktop'
      link: 'https://github.com/Synaptrix/ChatGPT-Desktop'
      desc: 'Fuel your productivity with ChatGPT-Desktop - Blazingly fast and supercharged!'
      icon: 'ChatGPTDesktop'
---

<!-- @layout-full-width -->
<ListProjects :projects="frontmatter.projects" />
