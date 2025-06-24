---
title: "Using local GitLab CI infrastructure for your GitHub project"
search_exclude: true
description: "Leverage local GitLab CI runners and specialized hardware for GitHub-hosted projects when standard CI resources are insufficient"
contributors: []
page_id: github_gitlab_ci_integration
related_pages:
  your_tasks: [ci_cd, task_automation_github_actions, task_automation_gitlab_ci_cd]
training:
  - name: "GitLab CI/CD Documentation"
    registry: "GitLab"
    url: "https://docs.gitlab.com/ee/ci/"
  - name: "GitHub Actions Documentation"
    registry: "GitHub"
    url: "https://docs.github.com/en/actions"
  - name: "Alpaka Job Matrix Library"
    registry: "GitHub"
    url: "https://github.com/alpaka-group/alpaka-job-matrix-library"
---

## How can I use my organization's local GitLab CI infrastructure for a GitHub-hosted project?

### Description

Many research software projects are hosted on {% tool "github" %} to benefit from its large open-source community and collaboration features. However, GitHub's free CI resources may be insufficient for complex research software that requires specialized hardware (GPUs, specific CPU architectures), extensive testing matrices, or simply more computational resources than the free tier provides. Organizations often have local {% tool "gitlab" %} instances with powerful runners and specialized hardware that could address these limitations.

Research projects like PIConGPU and Alpaka demonstrate this challenge perfectly. These projects require testing across multiple hardware configurations and extensive parameter combinations that exceed GitHub's free tier capabilities, yet benefit from GitHub's collaborative ecosystem for open-source development.

### Considerations

- Resource limitations on GitHub: Free {% tool "github-actions" %} have monthly limits and lack access to specialized hardware like GPUs or alternative CPU architectures required for performance-portable research software
- Available local infrastructure: Your organization may have {% tool "gitlab-ci-cd" %} runners with specialized hardware including multiple CPU architectures (ARM, Power, AMD, Intel), GPUs (NVIDIA, AMD), HPC systems, or different operating systems (macOS, Windows)
- Fork contribution workflow: Contributors often work from forks, which complicates direct integration with external CI systems, especially important for open-source research projects with distributed contributors
- GitLab Premium limitations: GitLab's native GitHub integration requires Premium edition and has limitations with fork pull requests, making it unsuitable for many research organizations
- Security considerations: External contributors need access to CI results without compromising internal infrastructure security
- Webhook reliability: Ensuring consistent communication between GitHub events and GitLab CI execution
- Status reporting complexity: Maintaining clear CI status visibility on GitHub while running on external infrastructure

### Solutions

#### Repository Mirroring Infrastructure

- Implement GitLab native project mirroring: Configure {% tool "gitlab" %} to automatically synchronize your GitHub repository to a local GitLab instance. Initially mirror only local branches to maintain security and avoid unnecessary synchronization of all external forks.

- Configure selective branch mirroring: Set up mirroring rules that focus on main development branches and exclude temporary or experimental branches to reduce synchronization overhead.

- Set up mirror monitoring: Implement monitoring to ensure mirroring remains active and responsive, with automated alerts for synchronization failures.

#### Fork Integration System

- Deploy automated fork mirroring bot: Create a webhook-driven bot that monitors GitHub pull request events from forks and automatically creates corresponding branches in the GitLab mirror. Use systematic naming conventions such as `username-repo-feature-branch` to maintain organization and traceability.

- Implement webhook security: Configure webhook signatures and validation to ensure only legitimate GitHub events trigger mirroring operations, preventing unauthorized access to your internal GitLab infrastructure.

- Handle fork branch lifecycle: Implement automated cleanup of mirrored fork branches when corresponding pull requests are closed or merged, maintaining repository hygiene.

#### CI Status Integration

- Configure bidirectional status reporting: Implement a system that sends GitLab pipeline status back to GitHub using commit hashes for identification. This ensures pull request status checks are properly updated regardless of the execution platform.

- Set up detailed status descriptions: Provide clear, descriptive status messages that indicate the testing platform, job types, and specific failure reasons to help developers understand CI results.

- Implement status aggregation: For complex pipelines with multiple stages, aggregate status information to provide clear pass/fail indicators while maintaining access to detailed logs.

#### Access Management and Security

- Establish guest access procedures: Create documented procedures for external contributors to access GitLab CI logs and results, including temporary guest access workflows that maintain security boundaries.

- Configure permission mapping: Establish clear mapping between GitHub repository permissions and GitLab project access levels to ensure appropriate access control.

- Implement audit logging: Maintain comprehensive logs of all cross-platform CI activities for security monitoring and troubleshooting.

#### Infrastructure Configuration

- Deploy specialized GitLab runners: Configure runners with the specific hardware configurations your project requires:

| Runner Type | Hardware Configuration | Use Case |
|-------------|----------------------|----------|
| Standard | x86_64, 8-16 GB RAM | Build testing, unit tests |
| GPU-NVIDIA | x86_64 + NVIDIA Tesla/RTX | CUDA development |
| GPU-AMD | x86_64 + AMD Radeon | ROCm/HIP testing |
| ARM | ARM64, 16 GB RAM | Cross-platform validation |
| PowerPC | ppc64le, 64 GB RAM | HPC compatibility |

- Configure runner tagging: Implement comprehensive tagging systems that allow jobs to target specific hardware configurations while maintaining flexibility for resource allocation.

- Set up runner pools: Organize runners into pools based on hardware capabilities and project requirements to ensure fair resource distribution across multiple projects.

#### Monitoring and Maintenance

- Implement comprehensive monitoring: Monitor webhook processing, mirroring bot health, runner availability, and pipeline execution metrics to ensure reliable service.

- Configure automated alerts: Set up alerting for critical failures in the integration chain, including mirroring delays, webhook processing errors, and runner unavailability.

- Establish maintenance procedures: Create documented procedures for routine maintenance, including runner updates, bot deployments, and mirror configuration changes.

## How do I ensure reliable integration between GitHub and GitLab CI systems?

### Description

Maintaining a stable, reliable integration between GitHub repositories and GitLab CI infrastructure requires careful attention to the communication pathways, error handling, and monitoring systems that connect these platforms. The integration must handle various failure modes gracefully while providing clear feedback to developers.

### Considerations

- Network reliability: Communication between GitHub and GitLab may experience intermittent failures, requiring robust retry mechanisms
- Authentication management: Maintaining valid authentication tokens across both platforms while ensuring security
- Rate limiting: Both GitHub and GitLab APIs have rate limits that must be respected to avoid service interruptions
- Error propagation: Failures in any part of the integration chain should be clearly communicated to developers
- Synchronization delays: Mirroring introduces latency that must be managed to provide timely feedback
- Configuration drift: Keeping webhook configurations and mirroring settings synchronized as projects evolve

### Solutions

- Implement robust webhook processing: Design webhook handlers with comprehensive error handling, retry logic, and dead letter queues to ensure no GitHub events are lost due to temporary failures.

- Configure webhook redundancy: Set up multiple webhook endpoints with failover mechanisms to ensure continuous operation even during maintenance or unexpected outages.

- Establish authentication token rotation: Implement automated token rotation for both GitHub and GitLab APIs to maintain long-term reliability without manual intervention.

- Create comprehensive error logging: Implement detailed logging throughout the integration pipeline to facilitate troubleshooting and identify patterns in failures.

- Set up health monitoring dashboards: Create monitoring dashboards that provide real-time visibility into the integration health, including webhook processing rates, mirroring delays, and CI execution metrics.

## How do I handle the complexity of managing both GitHub and GitLab workflows?

### Description

Running a hybrid GitHub-GitLab CI setup introduces workflow complexity that needs to be managed carefully to maintain developer productivity and project maintainability. This complexity is amplified when dealing with research software that requires specialized testing infrastructure and has contributors with varying levels of CI/CD expertise.

### Considerations

- Developer experience: Contributors should have a seamless experience regardless of which CI system is running their code
- Debugging complexity: Failed CI runs may require access to GitLab logs while the discussion happens on GitHub
- Permission management: Different access controls between GitHub and GitLab may create permission mismatches
- Documentation maintenance: Hybrid workflows require comprehensive documentation that stays current with both platforms
- Onboarding complexity: New contributors need to understand both the GitHub collaboration model and GitLab CI execution environment

### Solutions

- Provide comprehensive documentation: Create detailed documentation explaining the hybrid CI setup, including troubleshooting guides and examples specific to your research domain. Include step-by-step guides for common developer workflows.

- Implement automated documentation updates: Set up systems to automatically update documentation when CI configurations change, ensuring documentation remains current.

- Create developer onboarding guides: Develop specific onboarding materials that explain how to work with the hybrid system, including how to access logs, interpret status messages, and request runner access.

- Establish clear escalation procedures: Document clear procedures for developers to follow when they encounter CI issues that require GitLab access or infrastructure support.

- Implement automated issue routing: Create systems that automatically route CI-related issues to appropriate support channels based on whether they originate from GitHub or GitLab components.

## References

This approach was successfully implemented by the Helmholtz-Zentrum Dresden-Rossendorf for the Alpaka project, demonstrating how research organizations can effectively combine GitHub's collaborative features with local specialized CI infrastructure. The implementation showcases the infrastructure integration techniques that make complex research software CI pipelines practical and maintainable.

Further resources:

- [Continuous Integration in Complex Research Software - Handling Complexity](https://zenodo.org/records/14643958)
- [PIConGPU](https://github.com/ComputationalRadiationPhysics/picongpu)
- [Alpaka](https://github.com/alpaka-group/alpaka)
- [Alpaka Job Matrix Library](https://github.com/alpaka-group/alpaka-job-matrix-library)
- [Container Registry for CI Images](https://codebase.helmholtz.cloud/crp/alpaka-group-container)
- [Dynamic CI Pipelines in GitLab](https://docs.gitlab.com/ee/ci/pipelines/downstream_pipelines.html#dynamic-child-pipelines)