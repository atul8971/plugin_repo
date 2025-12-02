# Generate Playwright Automation Code

You are tasked with generating complete Playwright automation code following this project's Page Object Model (POM) architecture.

## Instructions

1. **Analyze the user's UI automation steps** provided after this prompt
2. **Use the playwright-code-generator agent** to create:
   - Page object classes (in `pages/`)
   - Step files with business logic (in `steps/`)
   - Test files with test cases (in `tests/`)

3. **Follow the project conventions**:
   - All page objects must inherit from `BasePage` (pages/base_page.py)
   - Use BasePage methods (click, fill, wait_for_selector, etc.) instead of raw Playwright
   - Maintain three-layer separation: Tests → Steps → Pages
   - Use Logger utility for logging (utils/logger.py)
   - Apply appropriate pytest markers (@pytest.mark.smoke, @pytest.mark.regression, etc.)
   - Follow the locator strategy priority: role > test-id > CSS > XPath

4. **Generate complete, executable code** that:
   - Can be run immediately with `pytest`
   - Follows all architectural patterns in CLAUDE.md
   - Includes proper assertions and logging
   - Uses the existing conftest.py fixtures (setup_page)

## User's Automation Steps

{USER_INPUT}

---

**Now execute**: Use the Task tool with subagent_type='playwright-code-generator' to generate the complete automation code for the steps described above.
