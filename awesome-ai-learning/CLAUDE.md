# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository is a comprehensive template and guide for building professional AI/ML projects. It provides best practices, templates, and examples for structuring, developing, and deploying machine learning projects.

## Current Structure

```
awesome-ai-learning/
├── README.md              # Main overview of the template repository
├── CONTRIBUTING.md        # Guidelines for contributors
├── LICENSE               # MIT License
├── CLAUDE.md            # This file
├── .gitignore           # Git ignore patterns
│
├── templates/            # Ready-to-use project templates
│   ├── project/         # Full project templates (PyTorch, TensorFlow, etc.)
│   ├── model/           # Model documentation templates
│   ├── data/            # Data pipeline templates
│   └── deployment/      # Deployment configuration templates
│
├── guides/              # Comprehensive guides
│   ├── setup/          # Environment setup guides
│   ├── development/    # Development best practices
│   ├── deployment/     # Deployment strategies
│   └── best-practices/ # General ML best practices
│
├── examples/           # Working example implementations
│   ├── pytorch/        # PyTorch examples
│   ├── tensorflow/     # TensorFlow examples
│   └── deployment/     # Deployment examples
│
├── resources/          # Additional resources and links
│
└── docs/              # Extended documentation

```

## Key Principles

1. **Template-First**: This is a template repository for AI/ML projects, not a project itself
2. **Best Practices**: Every recommendation should follow industry best practices
3. **Production-Ready**: All templates should be suitable for production use
4. **Clear Documentation**: Every file should be well-documented
5. **Practical Examples**: Include working examples, not just theory

## Content Guidelines

### Templates
- Should be complete and ready to use
- Include all necessary configuration files
- Follow consistent naming conventions
- Include inline documentation

### Guides
- Step-by-step instructions
- Include code examples
- Explain the "why" not just the "how"
- Link to relevant resources

### Examples
- Must be fully functional
- Include requirements.txt
- Have clear README files
- Demonstrate best practices

## Common Tasks

### Adding New Templates
1. Create template in appropriate directory
2. Include comprehensive documentation
3. Add example usage
4. Update main README navigation

### Creating Guides
1. Use clear, structured markdown
2. Include practical examples
3. Test all commands/code
4. Add troubleshooting section

### Updating Examples
1. Ensure code runs without errors
2. Use latest stable versions
3. Include performance benchmarks
4. Document any assumptions

## URL Standards
- All internal links should be relative
- External links should be to stable, reputable sources
- Prefer official documentation links
- Test all links before committing

## Quality Checklist
- [ ] All code examples tested
- [ ] Links verified
- [ ] Consistent formatting
- [ ] Clear navigation
- [ ] No placeholder content
- [ ] Production-ready recommendations

## Repository Maintenance

This repository should be:
- Regularly updated with new best practices
- Kept simple and focused
- Free of outdated information
- Easy to navigate and understand

## Notes
- This replaced a previous repository that aggregated AI/ML repositories
- Focus is now on providing templates and guides for building AI/ML projects
- Target audience is developers building production AI/ML systems