---
layout: post
title: 实验室私有内容 Agent MVP 部署总结：基于 Qwen3.5-35B 与 vLLM 的实战实践
date: 2026-03-30 07:13:30 +0000
author: Lab-Agent
categories:
- lab-notes
tags:
- LLM 部署
- vLLM
- Qwen3.5
- FastAPI
- 工程实践
- lab-agent
- qwen3.5
- vllm
- github-pages
---

> 本文档详细记录了在 lab11 服务器（6 张 RTX 3090）上成功部署私有内容 Agent MVP 的全过程。系统基于 Qwen3.5-35B-A3B 模型与 vLLM 推理框架，通过 FastAPI 实现了从文件解析到 GitHub Pages 发布的最小闭环。文章重点总结了多卡并行配置优化、服务启动预热策略及推理输出控制等关键技术问题的解决方案。

# 实验室私有内容 Agent MVP 部署总结

本文档总结了在实验室服务器 lab11 上成功部署私有内容 Agent MVP 的全过程。该系统基于 Qwen3.5-35B-A3B 模型和 vLLM 推理框架，通过 FastAPI 实现了从文件解析到 GitHub Pages 发布的最小闭环。部署过程中解决了多卡并行配置、服务启动预热及推理输出控制等关键技术问题，目前系统已具备处理 Markdown 和文本型 PDF 文件并生成博客文章的能力。

## 服务器与运行环境

本次部署的核心硬件资源为 lab11 服务器，配备 6 张 RTX 3090 GPU。项目代码与模型文件分别位于 `/data/local_userdata/cenmengyue/lab-agent` 和 `models` 目录下。为了确保依赖隔离与运行稳定，我们使用 conda 构建了独立运行环境。

核心组件配置如下：
- **模型**：Qwen3.5-35B-A3B
- **推理框架**：vLLM（提供 OpenAI 兼容接口）
- **服务层**：FastAPI 统一服务

## 多卡并行参数问题

在 6 张显卡的部署环境中，初始尝试使用标准的张量并行（Tensor Parallel）策略时，因上下文长度与卡数不整除导致启动失败。经过多次调试，我们最终确定了稳定的并行配置方案：

- **张量并行**：`TENSOR_PARALLEL_SIZE=1`
- **流水线并行**：`PIPELINE_PARALLEL_SIZE=6`
- **最大上下文长度**：`MODEL_MAX_LEN=8190`

该配置有效规避了显存分配冲突，确保了模型在多卡环境下的正常加载与推理。

## vLLM 启动与预热问题

vLLM 在启动阶段默认会进行多模态 profiling 和编译优化，这在特定硬件环境下可能导致服务启动超时或不稳定。为保障服务快速就绪，我们调整了以下环境变量：

- **强制 Eager 模式**：设置 `VLLM_ENFORCE_EAGER=1`，跳过部分编译步骤，降低启动延迟。
- **跳过多模态分析**：设置 `VLLM_SKIP_MM_PROFILING=1`，避免不必要的性能分析开销。

这些参数显著提升了服务启动的确定性和稳定性。

## Qwen3.5 推理输出控制问题

Qwen3.5 模型默认可能输出包含思考过程（reasoning）的内容，这会破坏我们期望的 JSON 结构化摘要结果。为确保应用层能稳定解析返回数据，我们在请求参数中显式禁用了思考过程：

- 设置 `LLM_ENABLE_THINKING=false`
- 设置 `chat_template_kwargs.enable_thinking=false`

该配置强制模型直接输出最终结论，保障了摘要生成和文章生成的结构化数据质量。

## 当前系统能力

经过上述优化，MVP 版本目前已打通以下完整功能闭环：

1. **文件输入**：支持接收 Markdown 或文本型 PDF 文件。
2. **内容处理**：能够解析文档内容，调用模型生成结构化摘要。
3. **文章生成**：将摘要转化为 Jekyll 兼容的 Markdown 格式博客文章。
4. **自动发布**：具备将生成的文章自动发布到 GitHub 博客仓库的能力。

## 总结与后续规划

当前系统已具备处理文本型 PDF 和 Markdown 文件并生成博客文章的能力，但在格式支持上仍有局限：暂不支持扫描版 OCR 识别、DOCX 格式解析及复杂的前端交互界面。后续工作将聚焦于扩展文件格式支持及优化前端展示体验.
