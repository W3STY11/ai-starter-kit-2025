# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an educational resource repository focused on AI tools and learning, not a traditional software project. It serves as a curated collection of AI learning resources for beginners, organized into structured paths and categories.

## Repository Structure

```
ai-starter-kit-2025/
├── learning-paths/          # Structured learning journeys by audience
│   ├── executives/          # For business leaders
│   ├── professionals/       # For working professionals
│   ├── analysts-writers/    # For content creators and analysts
│   └── technical-beginners/ # For those wanting to build with AI
├── resources/               # Main resource collection
│   ├── by-category/         # Resources organized by type
│   │   ├── ai-tools/        # ChatGPT, Claude, Gemini, etc.
│   │   ├── no-code/         # No-code AI platforms
│   │   ├── learning-materials/  # Courses, books, videos
│   │   └── communities/     # Discord, Reddit, newsletters
│   └── curated lists (.md, .csv)  # Top picks and recommendations
├── examples/                # Practical implementations
│   ├── prompts/             # Ready-to-use AI prompts
│   ├── workflows/           # Complete automation examples
│   └── use-cases/           # Industry-specific applications
├── guides/                  # How-to documentation
│   ├── setup-guides/        # Tool setup instructions
│   ├── best-practices/      # Usage guidelines
│   └── troubleshooting/     # Common issues and solutions
├── tools/                   # Supporting materials
│   ├── templates/           # Reusable templates
│   ├── checklists/          # Task checklists
│   └── comparison-charts/   # Tool comparisons
└── docs/                    # Additional documentation
```

## Key Principles

1. **Beginner-First**: All content must be accessible to complete AI beginners
2. **Quality Over Quantity**: Only include verified, high-quality resources
3. **Practical Focus**: Prioritize actionable, immediately useful content
4. **Recent Content**: Resources should be updated within the last 12 months
5. **Unbiased**: Educational focus, not promotional

## Resource Quality Criteria

When adding or evaluating resources, ensure they meet these standards:

### Required:
- Beginner-friendly language and approach
- Recently updated (within 6-12 months)
- From credible, authoritative sources
- Practical and actionable content
- Educational, not sales-focused

### Preferred:
- Free or low-cost access
- Multiple learning formats available
- Active community support
- Regular updates and maintenance

## Common Tasks

### Adding New Resources
1. Verify the resource meets quality criteria (see CONTRIBUTING.md)
2. Place in appropriate category under `/resources/by-category/`
3. Update relevant README files in that category
4. Add to curated lists if exceptional quality
5. Test all links before committing

### Creating Learning Paths
1. Each path should have clear goals and time estimates
2. Include resources for different learning styles
3. Progress from simple to complex topics
4. Provide practical exercises at each stage

### Maintaining Links
1. Regularly check for broken links
2. Update to HTTPS when available
3. Use descriptive link text (avoid "click here")
4. Include archive links for critical resources

### Content Formatting
- Use consistent Markdown formatting
- Include emojis for visual navigation (sparingly)
- Ensure mobile-friendly formatting
- Clear headings and subheadings
- Tables for comparisons and quick reference

## File Naming Conventions

- Use lowercase with hyphens: `example-guide.md`
- Be descriptive: `chatgpt-business-prompts.md` not `prompts.md`
- README.md for category overviews
- Numbered files for sequential content: `01-getting-started.md`

## Important Files

- `README.md` - Main repository overview and quick start
- `CONTRIBUTING.md` - Guidelines for contributors
- `resources/top-10-starter-pack.md` - Essential resources for beginners
- `resources/curated-list.csv` - Master list of all resources
- Learning path README files - Define goals and structure for each path

## Notes

- This is NOT a code repository - no build/test commands needed
- Zone.Identifier files are Windows metadata (safe to ignore)
- Focus on content quality and organization, not code functionality
- Community-driven with regular updates