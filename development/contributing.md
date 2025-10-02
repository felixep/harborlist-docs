# Contributing to HarborList

## Welcome Contributors

Thank you for your interest in contributing to HarborList! This guide will help you get started with contributing to our boat listing platform.

## Getting Started

### Prerequisites
- Node.js 18+ and npm
- Git
- AWS CLI (for infrastructure work)
- Basic knowledge of React and TypeScript

### Development Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd harborlist
   ```

2. **Install dependencies**
   ```bash
   # Backend dependencies
   cd backend && npm install
   
   # Frontend dependencies
   cd ../frontend && npm install
   
   # Infrastructure dependencies
   cd ../infrastructure && npm install
   ```

3. **Set up environment**
   ```bash
   # Copy environment templates
   cp frontend/.env.example frontend/.env.local
   cp config/secrets.template.json config/secrets.json
   ```

4. **Start development servers**
   ```bash
   # Start backend (in one terminal)
   cd backend && npm run dev
   
   # Start frontend (in another terminal)
   cd frontend && npm run dev
   ```

## How to Contribute

### Types of Contributions

We welcome various types of contributions:
- **Bug fixes** - Help us fix issues
- **Feature development** - Add new functionality
- **Documentation** - Improve our docs
- **Testing** - Add or improve tests
- **Performance** - Optimize existing code

### Contribution Workflow

1. **Find or create an issue**
   - Check existing issues first
   - Create a new issue for bugs or features
   - Discuss major changes before starting

2. **Fork and branch**
   ```bash
   # Fork the repository on GitHub
   # Clone your fork
   git clone <your-fork-url>
   
   # Create a feature branch
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow our coding standards
   - Write tests for new functionality
   - Update documentation as needed
   - Test your changes thoroughly

4. **Commit your changes**
   ```bash
   # Stage your changes
   git add .
   
   # Commit with descriptive message
   git commit -m "feat: add boat search filtering"
   ```

5. **Push and create PR**
   ```bash
   # Push to your fork
   git push origin feature/your-feature-name
   
   # Create pull request on GitHub
   ```

### Pull Request Guidelines

#### Before Submitting
- [ ] Code follows our style guidelines
- [ ] Tests pass locally
- [ ] Documentation is updated
- [ ] Commit messages are clear
- [ ] Branch is up to date with main

#### PR Description Template
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Performance improvement

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No breaking changes
```

## Development Guidelines

### Code Quality
- Write clean, readable code
- Follow TypeScript best practices
- Use meaningful variable names
- Add comments for complex logic

### Testing Requirements
- Unit tests for new functions
- Integration tests for API changes
- E2E tests for user workflows
- Maintain test coverage above 80%

### Documentation
- Update README files
- Document API changes
- Add inline code comments
- Update architecture docs

## Review Process

### What to Expect
1. **Automated checks** - CI/CD runs tests and linting
2. **Code review** - Team members review your code
3. **Feedback** - Address any requested changes
4. **Approval** - PR approved and merged

### Review Criteria
- Code quality and style
- Test coverage
- Documentation completeness
- Security considerations
- Performance impact

## Community Guidelines

### Code of Conduct
- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn and grow
- Maintain professional communication

### Getting Help
- **Issues** - Ask questions in GitHub issues
- **Discussions** - Use GitHub discussions for general topics
- **Documentation** - Check existing docs first
- **Team** - Reach out to maintainers for guidance

## Recognition

Contributors are recognized through:
- GitHub contributor list
- Release notes mentions
- Community acknowledgments
- Maintainer opportunities

## Development Resources

### Useful Commands
```bash
# Run all tests
npm run test

# Lint code
npm run lint

# Build for production
npm run build

# Type checking
npm run type-check
```

### Project Structure
```
harborlist/
├── backend/           # Node.js API server
├── frontend/          # React application
├── infrastructure/    # AWS CDK infrastructure
├── docs/             # Documentation
└── scripts/          # Utility scripts
```

Thank you for contributing to HarborList! Your efforts help make boat listing and discovery better for everyone.
