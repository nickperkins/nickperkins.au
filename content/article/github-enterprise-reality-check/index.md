---
title: "GitHub Enterprise Reality Check: A DevOps Engineer's Perspective"
date: 2025-07-25T14:30:00+10:00
draft: false

categories: [Platform Engineering]
tags: [github enterprise, devops, ci/cd, platform engineering, github actions]
toc: false
author: "Nick Perkins"
---

Over the past couple of years, I've been working extensively with GitHub Enterprise in large organisational settings. While GitHub has built an exceptional developer experience and continues to ship features at an impressive pace, the reality is that GitHub Enterprise still has significant gaps when it comes to enterprise-scale operations. Let me share what I've discovered about the current state of GitHub Enterprise from a platform engineering perspective.

<!-- LTeX: enabled=false -->
{{< imgproc img="james-harrison-vpOeXr5wmR4-unsplash.jpg" command="Resize" options="x400" alt="A laptop computer displaying a code editor with syntax-highlighted programming code on a dark background, showing colorful text in green, yellow, and purple. The laptop sits on a clean desk surface next to a small potted succulent plant in a decorative white pot, creating a modern, minimalist workspace aesthetic." >}}Photo by <a href="https://unsplash.com/@jstrippa?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">James Harrison</a> on <a href="https://unsplash.com/photos/black-laptop-computer-turned-on-on-table-vpOeXr5wmR4?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
      {{< /imgproc >}}
<!-- LTeX: enabled=true -->

## The Repository-Centric Challenge

GitHub's architecture is fundamentally repository-focused, which works brilliantly for individual projects and small teams. However, this design becomes a significant limitation when you're managing hundreds or thousands of repositories across an enterprise. Features that should operate at an organisational level often require manual configuration per repository, or worse, custom tooling to manage at scale.

Take OIDC authentication for deployment workflows. In theory, it's a fantastic security improvement over long-lived secrets. In practice, configuring OIDC across hundreds of repositories becomes a management nightmare. Each repository needs its own trust relationships configured, and there's no centralised way to manage these relationships or enforce consistent policies. You end up building custom automation just to maintain basic security hygiene.

The same pattern emerges with deployment environments. GitHub's environment protection rules are powerful when configured, but try scaling that across an enterprise and you'll quickly find yourself writing infrastructure as code just to manage your GitHub configuration. It's infrastructure to manage your infrastructure—not exactly the simplification we were hoping for.

## Self-Hosted Runners: Where's the Enterprise Story?

Perhaps the most glaring example of GitHub's enterprise limitations is the self-hosted Actions runners story. For any serious enterprise workload, you need self-hosted runners for security, compliance, or performance reasons. GitHub's solution? You get to choose between hosting Actions Runner Controller in Kubernetes or building your own autoscaling and monitoring infrastructure from scratch.

Actions Runner Controller (ARC) is a reasonable solution if you're already heavily invested in Kubernetes, but it's essentially GitHub outsourcing the operational complexity to you. You're responsible for runner lifecycle management, scaling, security patching, monitoring, and troubleshooting. Meanwhile, GitHub-hosted runners get all of this for free, but they're not suitable for most enterprise workloads due to security and compliance constraints.

Compare this to Azure DevOps or GitLab, where enterprise-grade runner management is built into the platform. You shouldn't need a degree in Kubernetes operations just to run your CI/CD pipelines reliably.

## Continuous Integration vs Continuous Deployment

Here's where things get interesting: GitHub Actions excels at continuous integration but struggles with continuous deployment at enterprise scale. For CI workloads—running tests, building artifacts, code quality checks—GitHub Actions is genuinely excellent. The developer experience is outstanding, the ecosystem is rich, and the integration with pull requests is seamless.

However, when it comes to continuous deployment and delivery, GitHub Actions starts to show its limitations. Deployment orchestration across multiple environments, approval workflows, rollback mechanisms, and deployment tracking all feel like afterthoughts rather than core platform capabilities. You end up building complex workflows that would be standard features in dedicated CD platforms.

This creates an interesting architectural decision: should you split your CI and CD concerns? I'm increasingly seeing organisations adopt this hybrid approach—using GitHub Actions for CI while handling CD through more mature platforms like Azure DevOps, GitLab, or dedicated deployment tools. It's not the unified workflow GitHub promotes, but it plays to each platform's strengths.

<!-- LTeX: enabled=false -->
{{< imgproc img="matthew-henry-Rd9uwddKoRA-unsplash(1).jpg" command="Resize" options="x400" alt="A large, deteriorating water slide complex photographed in winter, with multiple intertwining blue slides winding around a tall central structure. The slides appear abandoned and unused, set against a snowy landscape with bare trees in the background. The infrastructure looks weathered and in disrepair, creating a stark, desolate atmosphere." >}}Photo by <a href="https://unsplash.com/@matthewhenry?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Matthew Henry</a> on <a href="https://unsplash.com/photos/blue-plastic-spiral-slides-Rd9uwddKoRA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>{{< /imgproc >}}
<!-- LTeX: enabled=true -->

## The Cost Equation

The feature gaps become even more concerning when you factor in cost. GitHub Actions pricing for enterprise workloads can quickly become eye-watering, especially when you're running complex workflows or need significant compute resources. The per-minute pricing model incentivises short, focused builds, but enterprise applications often require longer-running processes for comprehensive testing or deployment.

When you add up the costs of GitHub Enterprise licensing, Actions minutes, storage for artifacts and packages, plus the engineering time needed to build tooling to fill the enterprise gaps, the total cost of ownership starts looking quite different from the initial sticker price. Azure DevOps or GitLab can deliver similar capabilities with more predictable pricing and less custom engineering required.

## What GitHub Gets Right

Don't get me wrong—GitHub has many strengths that keep organisations coming back. The developer experience is genuinely best-in-class. Features like Dependabot, CodeQL, and the GitHub ecosystem provide tremendous value. The pace of feature development is impressive, and GitHub clearly understands developer needs.

The integration between code, issues, pull requests, and Actions creates a cohesive workflow that developers love. For organisations prioritising developer productivity and code quality, these benefits are significant and shouldn't be dismissed.

## Making GitHub Enterprise Work

If you're committed to GitHub Enterprise, here are some strategies that can help:

**Embrace the hybrid approach**: Use GitHub Actions for CI and a dedicated CD platform for deployment orchestration. This gives you the best developer experience while avoiding GitHub's CD limitations.

**Invest in automation**: You'll need custom tooling to manage repository configuration at scale. Treat this as infrastructure as code and build it properly—you'll be maintaining it for years.

**Budget for the total cost**: Include engineering time for custom tooling, higher Actions consumption than expected, and potential integration costs with other platforms.

**Plan your runner strategy carefully**: If you need self-hosted runners, invest time in understanding ARC or plan for significant infrastructure engineering. Don't underestimate the operational overhead.

## The Bottom Line

GitHub Enterprise isn't a complete enterprise platform yet—it's an excellent developer platform with enterprise aspirations. The repository-centric architecture and gaps in enterprise-scale operations mean you'll likely need additional tooling and platforms to build a complete DevOps pipeline.

This doesn't make GitHub Enterprise a poor choice, but it does mean you need to go in with realistic expectations. If developer experience is your primary concern and you're willing to invest in platform engineering to fill the gaps, GitHub Enterprise can work well. However, if you need a complete, enterprise-ready DevOps platform out of the box, you might find better value elsewhere.

The future may well belong to GitHub as they continue to address these enterprise gaps, but for now, a pragmatic approach acknowledges both the platform's strengths and its limitations. Choose GitHub Enterprise for what it does best, and complement it with other tools where needed.
