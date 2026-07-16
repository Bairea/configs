# Claude Skills Batch Installation

## Batch Install Command

逐条执行:

```bash
npx skills add obra/superpowers -g -y
npx skills add mattpocock/skills -g -y
npx skills add useai-pro/openclaw-skills-security@skill-vetter -g -y
npx skills add anthropics/skills@skill-creator -g -y
```

可选skills，一般推荐放在项目中安装，如果项目没有.claude目录则自动创建
```Bash
# 前端与视觉
npx skills add vercel-labs/agent-skills --skill vercel-react-best-practices --skill web-design-guidelines --skill vercel-react-native-skills -y
npx skills add anthropics/skills --skill frontend-design --skill webapp-testing --skill canvas-design -y

# flutter
npx skills add flutter/skills -y
# go
npx skills add samber/cc-skills-golang -y
# rust
npx skills add actionbook/rust-skills -y

# 联网获取信息
npx skills add https://github.com/browser-act/skills --skill browser-act-skill-forge
npx skills add https://github.com/browser-act/skills --skill browser-act
npx skills add https://github.com/tavily-ai/skills --skill tavily-search -y

# 文档
npx skills add anthropics/skills --skill pptx --skill docx --skill xlsx --skill pdf -y

# 其他
npx skills add charon-fan/agent-playbook@self-improving-agent -y
npx skills add vercel-labs/skills@find-skills -y
npx skills add tanweai/pua@pua -y
npx skills add github/awesome-copilot --skill git-commit -y
npx skills add othmanadi/planning-with-files --skill planning-with-files-zh -y
```

## 验证安装

```bash
npx skills list
```

## 卸载技能

```bash
npx skills remove <skill-name>
```
