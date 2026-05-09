---
title: "Introducing OS Level Actions in Amazon Bedrock AgentCore Browser"
url: "https://aws.amazon.com/blogs/machine-learning/introducing-os-level-actions-in-amazon-bedrock-agentcore-browser/"
date: "2026-05-05"
author: "Evandro Franco"
feed_url: "https://aws.amazon.com/blogs/machine-learning/feed/"
---
AI agents that automate web workflows operate within the browser’s web layer, the DOM that Playwright and the Chrome DevTools Protocol (CDP) expose. AgentCore Browser provides a secure, isolated browser environment for this, and it works well for the vast majority of automation: navigating pages, filling forms, clicking elements, extracting content. But the web layer has a hard boundary. Anything that the operating system renders (native dialogs, security prompts, certificate choosers, context menus, even Chrome settings) sits outside the DOM entirely.
