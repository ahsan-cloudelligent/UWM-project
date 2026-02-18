# Planning Mode Operations

## When to Use Planning vs Build Mode

- **Use Planning Mode** for: New features, complex refactors, architecture decisions, or when requirements are unclear
- **Use Build Mode** for: Bug fixes, minor updates, well-defined tasks, or when implementation path is clear

## Planning Workflow

1. **Requirements Gathering** - Define user needs, acceptance criteria, and constraints
2. **Research** - Investigate existing patterns, libraries, and technical approaches
3. **Design** - Create component structure, data flow, and interaction patterns
4. **Task Breakdown** - Split work into implementable chunks with clear deliverables

## Frontend Planning Questions

**Component Architecture:**
- What components are needed and how do they compose?
- Which components should be reusable vs feature-specific?
- How will props and data flow between components?

**UX Flows:**
- What are the key user journeys and interaction patterns?
- How should loading, error, and empty states be handled?
- What feedback should users receive for their actions?

**State Management:**
- Where should state live (local, lifted, global)?
- What data needs to persist vs can be ephemeral?
- How will state updates trigger UI changes?

**Accessibility:**
- What ARIA labels and roles are needed?
- How will keyboard navigation work?
- Are there screen reader considerations?

**Styling Approach:**
- Which styling method fits the component needs?
- What design tokens or theme values apply?
- How will responsive behavior work?

**Performance Considerations:**
- What can be lazy loaded or code split?
- Are there expensive operations to optimize?
- How will this impact bundle size?

## Complete Planning

When planning is complete, call `switch_to_execution` to begin implementation.

---

# Frontend Development Standards

These standards define how we design, build, and maintain frontend applications to ensure consistency, quality, performance, and long‑term maintainability.

---

## 1. General Principles

- **Clarity over cleverness**: Write code that is easy to read and understand.
- **Consistency matters**: Follow existing patterns before introducing new ones.
- **User-first mindset**: Prioritize usability, accessibility, and performance.
- **Scalable by default**: Assume the codebase will grow.
- **Fail loudly**: Errors should be visible and actionable during development.

---

## 2. Technology Standards

- Use **modern, actively maintained frameworks and libraries**.
- Prefer **TypeScript** over JavaScript for type safety.
- Avoid unnecessary dependencies; evaluate bundle size and maintenance cost.
- Lock dependency versions and update them regularly.

---

## 3. Project Structure

- Organize code by **feature/domain**, not by file type.
- Keep related files close together (component, styles, tests).
- Avoid deep nesting (max 3–4 levels).
- Use clear, predictable folder names.

Example:

```text
features/
  auth/
    Login.tsx
    Login.test.tsx
    auth.api.ts
```

---

## 4. Naming Conventions

- Use **descriptive, intention‑revealing names**.
- Components: `PascalCase`
- Hooks & utilities: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Files should mirror exported names.

Avoid abbreviations unless universally understood.

---

## 5. Component Design

- Components should follow **single responsibility**.
- Prefer **composition over inheritance**.
- Keep components small and focused.
- Separate presentational and business logic when complexity grows.
- Avoid large, multi-purpose components.

---

## 6. State Management

- Prefer **local state** when possible.
- Lift state only when necessary.
- Avoid global state for ephemeral UI state.
- Keep state minimal and normalized.
- Clearly document side effects.

---

## 7. Styling Standards

- Use a single styling approach per project.
- Styles must be **scoped** to avoid global leakage.
- Avoid inline styles except for dynamic values.
- Use design tokens for colors, spacing, typography.
- Never hardcode magic values.

---

## 8. Code Formatting & Linting (Mandatory)

All frontend projects must use standardized code formatting and linting with strict enforcement.

### Required Tools

- **Prettier**: Code formatter (mandatory, standard config)
- **ESLint**: Code linter (mandatory, standard config)  
- **Husky + lint-staged**: Pre-commit hooks (strict enforcement - blocks commits)

### Required Packages

```bash
npm install --save-dev prettier eslint husky lint-staged
```

### Configuration Files

**.prettierrc**
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

**.eslintrc.json**
```json
{
  "extends": ["eslint:recommended", "@typescript-eslint/recommended"],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "rules": {
    "no-unused-vars": "error",
    "no-console": "warn"
  }
}
```

**package.json (lint-staged config)**
```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "eslint . --fix",
    "lint:check": "eslint .",
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["prettier --write", "eslint --fix"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

**.husky/pre-commit**
```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

### CI/CD Requirements

- Pipeline must include `npm run format:check` and `npm run lint:check`
- Pipeline must fail if formatting or linting checks fail
- No exceptions - all code must pass checks before merge

---

## 9. Accessibility (Required)

- Follow **WCAG 2.1 AA** standards.
- All interactive elements must be keyboard accessible.
- Use semantic HTML elements.
- Provide accessible labels for inputs and controls.
- Ensure sufficient color contrast.

Accessibility issues are considered **bugs**, not enhancements.

---

## 10. Performance

- Optimize for perceived performance.
- Avoid unnecessary re-renders.
- Lazy load routes and heavy components.
- Optimize images and fonts.
- Measure performance before and after changes.

---

## 11. Error Handling

- Handle errors explicitly.
- Provide user-friendly error messages.
- Never silently fail.
- Log errors in a consistent, centralized way.

---

## 12. Testing & Quality Assurance

### 12.1 QA Principles

- Quality is everyone's responsibility, not just QA engineers
- Automation-first mindset for all repeatable tests
- Test early and continuously - bugs are cheaper to fix early
- Tests must be reliable, readable, and maintainable

### 12.2 Required Test Layers

**Unit Testing:**
- Test component logic, hooks, and utilities in isolation
- Tests must be fast, deterministic, and isolated
- Focus on business logic and edge cases
- No network or API calls in unit tests

**Component Testing:**
- Test component rendering and user interactions
- Validate props, state changes, and event handlers
- Test accessibility features (ARIA labels, keyboard navigation)
- Mock external dependencies

**Integration Testing:**
- Test component integration with APIs and services
- Validate data flow between components
- Test real implementations where possible
- Mock only external or third-party dependencies

**End-to-End (E2E) Testing (Mandatory):**
- Validate complete user flows from UI to backend
- Must run against realistic environments
- Cover critical business paths and regressions
- Use Playwright for E2E automation

### 12.3 Playwright E2E Testing (Required)

Playwright is the **mandatory standard** for frontend E2E testing.

**Playwright Requirements:**
- Use Playwright for:
  - UI regression tests
  - Cross-browser testing (Chrome, Firefox, Safari)
  - Accessibility validation
  - Critical user flow automation
- Tests must:
  - Be stable and non-flaky
  - Avoid hard-coded waits (use proper wait strategies)
  - Use `data-testid` attributes instead of CSS selectors
  - Run in headless mode in CI
  - Be independent and parallelizable
- Mock unstable external services
- Keep test data reproducible and resettable

**Example Playwright Best Practices:**
```typescript
// Use data-testid for stable selectors
await page.getByTestId('login-button').click();

// Proper waiting strategies
await page.waitForSelector('[data-testid="dashboard"]');

// Accessibility checks
await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
```

### 12.4 Test Automation Standards

**Automation Requirements:**
- All critical user flows must be automated
- Regression testing must be automated
- Manual testing allowed only for exploratory testing
- Tests must be part of CI/CD pipeline
- Failed tests must block deployment

**Test Coverage Goals:**
- Unit tests: >80% coverage for business logic
- Component tests: All reusable components
- E2E tests: All critical user journeys
- Accessibility tests: All interactive components

### 12.5 Frontend-Specific QA Standards

**UI Behavior:**
- UI must match design specifications
- Loading states must be tested
- Error states must be handled gracefully
- Empty states must be user-friendly

**Accessibility (WCAG 2.1 AA):**
- All interactive elements keyboard accessible
- Proper ARIA labels and roles
- Sufficient color contrast
- Screen reader compatibility
- Focus management tested

**Cross-Browser Compatibility:**
- Test on Chrome, Firefox, Safari
- Verify responsive behavior on different screen sizes
- Test on mobile devices (iOS, Android)

**Visual Regression:**
- Automate visual regression testing where possible
- Use snapshot testing for critical UI components
- Review visual changes in PR process

**Performance Testing:**
- Measure and optimize:
  - Initial load time
  - Time to interactive
  - First contentful paint
  - Largest contentful paint
- Lazy load non-critical resources
- Optimize bundle size
- Monitor performance metrics in CI

### 12.6 Security Testing

**Frontend Security Validation:**
- Validate input sanitization
- Test XSS prevention
- Verify secure data handling
- Check for exposed sensitive data
- Validate authentication flows
- Test authorization boundaries

**Dependency Security:**
- Scan dependencies for vulnerabilities
- Keep dependencies up to date
- Use automated security scanning in CI

### 12.7 Test Data Management

- Use realistic but non-sensitive test data
- Never use production data in tests
- Test data must be reproducible and resettable
- Automate test data cleanup

### 12.8 CI/CD Integration

**Pipeline Requirements:**
- All tests run automatically on every commit
- Builds must fail on:
  - Test failures
  - Coverage violations
  - Security vulnerabilities
  - Accessibility violations
- Test results must be visible and actionable
- Flaky tests must be fixed or removed

### 12.9 Bug Management

**Bug Reporting Standards:**
- Clear reproduction steps
- Expected vs actual behavior
- Browser and environment details
- Screenshots or videos when applicable
- Console errors and network logs

**Bug Severity:**
- Critical: Blocks core functionality, security issues
- High: Major feature broken, poor UX
- Medium: Minor feature issues, edge cases
- Low: Cosmetic issues, minor improvements

**Bug Resolution:**
- Critical bugs block releases
- Root cause analysis for major issues
- Regression tests added for fixed bugs

### 12.10 Definition of Done (Frontend QA)

A frontend feature is **Done** only when:

- [ ] Unit tests written and passing
- [ ] Component tests cover all interactions
- [ ] Playwright E2E tests cover critical flows
- [ ] Accessibility validated (WCAG 2.1 AA)
- [ ] Cross-browser compatibility verified
- [ ] Performance impact measured and acceptable
- [ ] Security checks passed
- [ ] No critical or high-severity bugs
- [ ] Visual regression reviewed
- [ ] Documentation updated
- [ ] Code review approved
- [ ] All CI checks passing

---

## 13. Code Quality & Reviews

- All code must be reviewed before merging.
- Keep pull requests small and focused.
- Explain *why* changes are made, not just *what*.
- Address review feedback promptly.
- Code formatting and linting are automated - focus reviews on logic and design.

---

## 14. Documentation

- Public components and utilities must be documented.
- Document non-obvious decisions.
- Keep documentation close to the code.

---

## 15. Security

- Never trust user input.
- Avoid exposing sensitive data in the frontend.
- Sanitize and validate data where appropriate.
- Keep dependencies up to date to patch vulnerabilities.

---

## 16. Definition of Done

A frontend task is considered complete when:

- Code meets these standards
- Code is formatted with Prettier and passes ESLint
- Unit tests written and passing (>80% coverage for business logic)
- Component tests cover all interactions
- Playwright E2E tests cover critical flows
- Accessibility validated (WCAG 2.1 AA)
- Cross-browser compatibility verified
- Performance impact measured and acceptable
- Security checks passed
- No critical or high-severity bugs
- Visual regression reviewed
- Documentation is updated
- Code review approved
- All CI checks passing

---

**These standards are living guidelines and should evolve with the product and team.**
