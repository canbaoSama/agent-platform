<script setup lang="ts">
const modules = [
  ["Agent Studio", "设计、调试和发布 Agent 工作流"],
  ["Knowledge", "管理文档、索引、检索与引用"],
  ["Tools & MCP", "注册工具并控制权限与审批"],
  ["Runs", "查看运行状态、步骤、成本与 Trace"],
  ["Evaluation", "管理数据集、指标与回归测试"],
  ["Governance", "租户、成员、角色、审计与密钥"],
];
</script>

<template>
  <main>
    <section class="hero">
      <p class="eyebrow">AGENT PLATFORM / FOUNDATION</p>
      <h1>把大模型能力变成<br />稳定的业务系统</h1>
      <p class="intro">
        统一连接模型、知识、工具、记忆和工作流，并为每次 Agent 运行提供权限控制、人工审批、评测与可观测性。
      </p>
      <div class="status"><span />基础工程已就绪</div>
    </section>

    <section class="grid" aria-label="平台模块">
      <article v-for="[name, description] in modules" :key="name">
        <p>{{ name }}</p>
        <span>{{ description }}</span>
      </article>
    </section>
  </main>
</template>

