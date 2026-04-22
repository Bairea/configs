# Claude Skills Batch Installation

## Batch Install Command

逐条执行:

```bash
npx skills add obra/superpowers -g -y
npx skills add vercel-labs/agent-browser@agent-browser -g -y
npx skills add panniantong/agent-reach@agent-reach -g -y
npx skills add charon-fan/agent-playbook@self-improving-agent -g -y
npx skills add useai-pro/openclaw-skills-security@skill-vetter -g -y
npx skills add anthropics/skills@skill-creator -g -y
npx skills add vercel-labs/skills@find-skills -g -y
npx skills add anthropics/skills@frontend-design -g -y
npx skills add tanweai/pua@pua -g -y
npx skills add https://github.com/anthropics/skills --skill pptx -g -y
npx skills add anthropics/skills@docx -g -y
npx skills add anthropics/skills@xlsx -g -y
npx skills add anthropics/skills@pdf -g -y
npx skills add https://github.com/github/awesome-copilot --skill git-commit -g -y
```

## 验证安装

```bash
npx skills list
```

## 卸载技能

```bash
npx skills remove <skill-name>
```
