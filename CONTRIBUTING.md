# Contributing to Music Bot

Thank you for considering contributing to this project! 🎉

## 📋 How to Contribute

### Reporting Bugs

If you found a bug, please open an [issue](https://github.com/carloswimmer/chatbot-youtube-music/issues) including:

- Clear description of the problem
- Steps to reproduce
- Expected vs. actual behavior
- Screenshots (if applicable)
- Node.js version and operating system

### Suggesting Enhancements

Feature suggestions are welcome! Please:

1. Check if a similar issue doesn't already exist
2. Clearly describe the proposed feature
3. Explain why it would be useful
4. Provide usage examples (if possible)

### Pull Requests

1. **Fork** the repository
2. **Clone** your fork locally
   ```bash
   git clone https://github.com/your-username/chatbot-youtube-music.git
   cd chatbot-youtube-music
   ```

3. **Create a branch** for your feature
   ```bash
   git checkout -b feature/my-feature
   ```

4. **Make your changes** following the project conventions

5. **Test** your changes
   ```bash
   pnpm dev
   # Test the functionality manually
   ```

6. **Commit** your changes with clear messages
   ```bash
   git commit -m "feat: add feature X"
   ```

7. **Push** to your fork
   ```bash
   git push origin feature/my-feature
   ```

8. Open a **Pull Request** to the `main` branch

## 📝 Code Conventions

### Commits

We follow the [Conventional Commits](https://www.conventionalcommits.org/) standard:

- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `style:` - Formatting, semicolons, etc
- `refactor:` - Code refactoring
- `test:` - Adding or fixing tests
- `chore:` - Build tasks, configs, etc

Examples:
```
feat: add artist search
fix: correct streaming response error
docs: update README with new examples
```

### Code

- Use **TypeScript** for all new code
- Follow the configured **ESLint** rules
- Keep functions small and focused
- Add **comments** when necessary
- Use **descriptive names** for variables and functions

### Structure

```typescript
// ✅ Good
async function generateEmbedding(text: string): Promise<number[]> {
  // Implementation
}

// ❌ Bad
async function ge(t: string): Promise<any> {
  // Implementation
}
```

## 🧪 Testing

Before submitting a PR, make sure:

- [ ] Code compiles without errors (`pnpm build`)
- [ ] Linter passes (`pnpm lint`)
- [ ] Manually tested the functionality
- [ ] No new warnings introduced

## 📚 Resources

- [Next.js Docs](https://nextjs.org/docs)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)
- [Drizzle ORM](https://orm.drizzle.team/docs)
- [Google Gemini API](https://ai.google.dev/docs)

## 💬 Questions?

If you have questions about contributing, feel free to:

- Open an issue with the `question` tag
- Contact via email
- Comment on an existing issue

---

Thank you for contributing! 🙏

