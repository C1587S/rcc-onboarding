# RCC Onboarding Guide

A practical onboarding guide for new CIL predocs working with RCC systems and multidimensional datasets.

## Quick Start

### Local Development

```bash
# Clone the repository
git clone https://github.com/ClimateImpactLab/rcc-onboarding.git
cd rcc-onboarding

# Install dependencies
pip install -r requirements.txt

# Serve locally
mkdocs serve
```

Visit `http://127.0.0.1:8000` to preview the site.

### Deployment

The site automatically deploys to GitHub Pages when changes are pushed to the `main` branch.

## Contributing

### Adding new content

1. Create or edit markdown files in the `docs/` directory
2. Update navigation in `mkdocs.yml` if adding new pages
3. Test locally with `mkdocs serve`
4. Submit a pull request

### Content guidelines

- Use clear, practical examples
- Include code snippets for common tasks
- Link to related sections when helpful
- Keep troubleshooting sections up to date

## Structure

- `getting-started/` - SSH connection, interactive sessions, first Slurm jobs
- `environment-setup/` - Conda environments and package management
- `hands-on-tutorial/` - Complete workflow using mortality datasets
- `reference/` - Detailed documentation for ongoing use
- `troubleshooting/` - Common issues and solutions

## Getting Help

- Check existing documentation and troubleshooting sections
- Search [GitHub Discussions](https://github.com/your-org/rcc-onboarding/discussions)
- Open an issue for documentation bugs or improvement suggestions

## License

This project is licensed under the MIT License - see the LICENSE file for details.