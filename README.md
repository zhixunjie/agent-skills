# agent-skills


~~~ shell
# https://github.com/zhixunjie/agent-skills
npx skills add zhixunjie/agent-skills

# local skills
SKILL_NAME=new-llm
mkdir -p ${PATH_PROJECT_LEARN_PYTHON}/ai-llm/ai-agent/agent-skills/.claude/skills
ln -sf ${PATH_PROJECT_LEARN_PYTHON}/ai-llm/ai-agent/agent-skills/my-agent-skills/skills/${SKILL_NAME} \
${PATH_PROJECT_LEARN_PYTHON}/ai-llm/ai-agent/agent-skills/.claude/skills/${SKILL_NAME}
~~~