# Generate Playwright Automation Code

skill: playwright-pom

You are tasked with generating complete Playwright automation code following this project's Page Object Model (POM) architecture.

## Input Analysis

First, analyze the user's input to determine the appropriate workflow:

1. **If the user provides a JIRA ticket ID or URL** (e.g., "PROJ-123", "https://jira.company.com/browse/PROJ-123"):
   - Use the Task tool with subagent_type='general-purpose' to invoke the jira-issue-analyzer agent
   - The agent will extract test scenarios and automation steps from the JIRA ticket
   - Then proceed to generate automation code based on the extracted information

2. **If the user provides specific automation steps** (e.g., "Navigate to login page, enter credentials, click submit"):
   - Proceed directly to generating automation code using the steps provided
   - Skip the JIRA analysis step

3. **If the user input is empty or unclear**:
   - Use the AskUserQuestion tool to prompt the user:
     - "Please provide either a JIRA ticket ID/URL for analysis, or describe the specific UI automation steps you want to automate"
   - Wait for user response before proceeding

## Instructions

Once you have clear automation steps (either from JIRA or user input):

1. **Analyze the UI automation steps**
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

## User's Input

{USER_INPUT}

---

**Now execute**:
- First determine the input type (JIRA ticket, automation steps, or unclear)
- If JIRA ticket: invoke jira-issue-analyzer agent first
- If automation steps: proceed directly to playwright-code-generator agent
- If unclear: ask user for clarification
