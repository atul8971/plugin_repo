# Playwright POM Framework Skill

You are an expert in the Playwright Python UI Automation Framework following the Page Object Model (POM) design pattern.

## Framework Architecture

This project follows a three-layer abstraction pattern:
```
Tests (What to test) → Steps (Business Logic) → Pages (UI Interactions)
```

### Directory Structure
```
playwright_test_fw/
├── pages/              # Page Object Models
│   ├── base_page.py   # Base page with common actions
│   ├── login_page.py  # Page-specific objects
│   └── products_page.py
├── steps/              # Business logic layer
│   ├── login_steps.py # Workflow steps
│   └── search_steps.py
├── tests/              # Test cases
│   ├── test_login.py
│   └── test_product_search.py
├── utils/              # Utility modules
│   └── logger.py      # Custom logging
├── conftest.py         # Pytest fixtures
└── pytest.ini          # Pytest configuration
```

## Coding Standards

### 1. Page Objects (pages/)
- **Inheritance**: All page classes MUST inherit from `BasePage`
- **Locator Strategy**: Prefer role-based locators > test-id > CSS
  - Format: `"role=button[name='Login']"`
  - Example: `self.login_button = "role=button[name='Login']"`
- **Locators as Attributes**: Define all locators as class attributes in `__init__`
- **Methods**: Page-specific actions (enter_email, click_login_button, etc.)
- **No Business Logic**: Pages should only handle UI interactions

**Example Pattern:**
```python
from playwright.sync_api import Page
from pages.base_page import BasePage

class MyPage(BasePage):
    def __init__(self, page: Page):
        super().__init__(page)

        # Define locators
        self.email_input = "role=textbox[name='email']"
        self.submit_button = "role=button[name='Submit']"

    def enter_email(self, email: str) -> None:
        self.logger.info(f"Entering email: {email}")
        self.fill(self.email_input, email)

    def click_submit(self) -> None:
        self.logger.info("Clicking submit button")
        self.click(self.submit_button)
```

### 2. Step Files (steps/)
- **Purpose**: Contains business logic and workflows
- **Dependencies**: Import page objects and use them
- **Logging**: Use `Logger.log_step()` for major workflow steps
- **Assertions**: Business-level verifications belong here
- **Return Values**: Can return boolean for verification steps

**Example Pattern:**
```python
from playwright.sync_api import Page
from pages.my_page import MyPage
from utils.logger import Logger

class MySteps:
    def __init__(self, page: Page):
        self.page = page
        self.my_page = MyPage(page)
        self.logger = Logger.get_logger(__name__)

    def complete_workflow(self, email: str) -> None:
        Logger.log_step(self.logger, f"Completing workflow for {email}")
        self.my_page.enter_email(email)
        self.my_page.click_submit()
        self.logger.info("Workflow completed")

    def verify_success(self) -> bool:
        Logger.log_step(self.logger, "Verifying success")
        is_successful = "/dashboard" in self.page.url
        Logger.log_assertion(self.logger, "Navigation successful", is_successful)
        return is_successful
```

### 3. Test Files (tests/)
- **Naming**: `test_<feature_name>.py`
- **Class-based**: Group related tests in classes
- **Markers**: Use pytest markers (@pytest.mark.smoke, @pytest.mark.login)
- **Fixtures**: Use `setup_page` fixture which auto-navigates to URL
- **Steps Pattern**: Instantiate step classes, not page objects directly
- **Assertions**: Keep tests focused on high-level assertions

**Example Pattern:**
```python
import pytest
from playwright.sync_api import Page
from steps.my_steps import MySteps

class TestMyFeature:
    @pytest.mark.smoke
    def test_my_scenario(self, setup_page: Page):
        page = setup_page
        my_steps = MySteps(page)

        # Perform actions
        my_steps.complete_workflow("test@example.com")

        # Assert
        assert my_steps.verify_success(), "Workflow should succeed"
```

### 4. BasePage Inheritance
All pages inherit these common methods:
- **Navigation**: `navigate(url)`, `reload()`, `go_back()`
- **Actions**: `click()`, `fill()`, `hover()`, `select_option()`, `check()`
- **Waits**: `wait_for_selector()`, `wait_for_url()`, `wait_for_load_state()`
- **Getters**: `get_text()`, `get_attribute()`, `get_current_url()`
- **Assertions**: `assert_element_visible()`, `assert_text()`, `assert_url()`
- **Checks**: `is_visible()`, `is_enabled()`, `is_checked()`

### 5. Logging Standards
- Use `self.logger.info()` for action logs
- Use `Logger.log_step()` for major workflow steps
- Use `Logger.log_assertion()` for verification logging
- All logging is automatic in BasePage methods

### 6. Test Configuration
- **URL Parameter**: Tests use `--url` CLI argument
- **Setup Fixture**: `setup_page` fixture auto-navigates to URL
- **Example**: `pytest --url=https://example.com tests/test_login.py`

## When Creating New Code

### For New Page Objects:
1. Inherit from BasePage
2. Define all locators in `__init__` as attributes
3. Use role-based locators when possible
4. Create methods for page-specific actions
5. Keep it UI-focused, no business logic

### For New Step Files:
1. Import necessary page objects
2. Instantiate page objects in `__init__`
3. Use `Logger.log_step()` for workflows
4. Use `Logger.log_assertion()` for verifications
5. Combine page actions into business workflows

### For New Tests:
1. Use class-based structure
2. Add pytest markers for categorization
3. Use `setup_page` fixture
4. Instantiate step classes (not page objects)
5. Keep assertions high-level and meaningful

## Important Rules
- NEVER instantiate page objects directly in tests - use step files
- ALWAYS define locators as class attributes, not method variables
- PREFER role-based locators over CSS or XPath
- USE Logger.log_step() for major actions in step files
- KEEP page objects focused on UI, steps focused on business logic
- INHERIT all page classes from BasePage
- USE the BasePage methods instead of direct Playwright calls

## File Naming Conventions
- Pages: `<page_name>_page.py` (e.g., `login_page.py`)
- Steps: `<feature_name>_steps.py` (e.g., `login_steps.py`)
- Tests: `test_<feature_name>.py` (e.g., `test_login.py`)
- Classes: `<Name>Page`, `<Name>Steps`, `Test<Name>`
