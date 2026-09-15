# AGENTS.md

## Purpose

This repository is a knowledge base for question and answer (Q&A). The agent's answers are distilled into the corresponding knowledge documents.

## Repository Structure

The knowledge documents must form a two-level tree. Directory and file names are in Chinese:

- Level 1: domains, e.g. 计算机网络 (Computer Networks), 软件工程 (Software Engineering), 人工智能 (Artificial Intelligence), Git, Linux.
- Level 2: subdomains of each domain, each stored as one Markdown file. For example, under 计算机网络, `协议.md` and `传输.md` are separate Markdown files; under 软件工程, `软件过程.md` and `软件设计.md` are separate Markdown files.

```
Here is the template of repo tree.
.
├── 计算机网络/
│   ├── 协议.md
│   └── 传输.md
├── 软件工程/
│   ├── 软件过程.md
│   └── 软件设计.md
├── 人工智能/
├── Git/
└── Linux/
```

## Answer Rules

1. When working in this repository, answer in Chinese.
2. For every term from the fields of computer science, software engineering, or artificial intelligence, mark the corresponding English term at its first occurrence, e.g. 协议（Protocol）.
3. Be concise and precise: answer only the question, and do not include information that the question does not ask about.

## Writing Knowledge into Documents

- Determine the level-1 domain and level-2 subdomain of the question.
- Read the corresponding knowledge document first.
- Do not copy the answer verbatim. Distill the new knowledge from the answer and supplement it into the corresponding document.
- If the corresponding document does not exist, obtain consent first, then create it (and its domain directory, if needed) and write the knowledge into it.

## Rule Changes

Supplements or changes to these rules will be communicated to the developer and submitted only after approval.
