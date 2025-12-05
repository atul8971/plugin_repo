# Test Case Management Command

This command helps you manage test cases by integrating Jira ticket analysis and Google Sheets operations.

## Input Analysis

First, analyze the user's input to understand what needs to be done:

1. **Extract Jira Ticket Information** (if provided):

   - Jira ticket ID (e.g., "PROJ-123")
   - Jira ticket URL (e.g., "https://jira.company.com/browse/PROJ-123")

2. **Extract Google Sheets Information** (if provided):

   - Google Sheets URL
   - Spreadsheet ID
   - Sheet name

3. **Extract Figma Design Information** (if provided):

   - Figma file URL (e.g., "https://figma.com/file/abc123/Design-Name")
   - Specific screens or flows to focus on
   - Design-related requirements

4. **Extract Task Details**:

   - What action should be performed (e.g., "create test cases", "update test status", "extract requirements")
   - Any specific parameters or filters

5. **Extract Number of Test Cases** (if provided):
   - Number of test cases to generate (e.g., "10 test cases", "generate 25 tests")
   - If not specified, default to 20 test cases

## Workflow Logic

Based on the input provided, follow this workflow:

### Step 1: Input Validation

If any required information is missing or unclear, use the AskUserQuestion tool to gather:

- Jira ticket ID/URL (if Jira analysis is needed)
- Google Sheets link (if sheet operations are needed)
- Figma design URL (if Figma analysis is needed)
- Task details describing what action to perform
- Number of test cases to generate (default: 20 if not specified)

### Step 2: Figma Design Analysis (if Figma link provided)

If a Figma design URL is provided:

1. Use the Task tool with `subagent_type='figma-agent'` to:

   - Generate detailed test steps referencing exact UI elements from Figma
   - Create comprehensive functional, visual, and usability test cases
   - Ensure test cases align with the actual design specifications
   - Include specific test data based on design inputs and forms
   - Cover all user flows and navigation patterns shown in the design

### Step 3: Jira Analysis (if Jira ticket provided)

If a Jira ticket ID or URL is provided:

1. Use the Task tool with `subagent_type='jira-ticket-analyzer'` to:

   - Fetch the complete ticket details (summary, description, acceptance criteria, comments)
   - Analyze any attachments (screenshots, documents, test data)
   - Extract relevant information such as:
     - Requirements and user stories
     - Test scenarios from acceptance criteria
     - Bug details and reproduction steps
     - Technical specifications
     - Related issues or dependencies

2. Process the returned Jira data based on the task_details:
   - If task is "extract test scenarios": Parse acceptance criteria and requirements
   - If task is "create bug report": Extract bug details, steps to reproduce, environment info
   - If task is "analyze requirements": Identify functional and non-functional requirements
   - Custom task handling based on user's specific task_details

### Step 4: Test Case Generation (if test generation is needed)

If the task requires generating comprehensive test cases:

**IMPORTANT: Test Case Format**

All generated test cases MUST follow this exact format with these columns:
- **Test Case ID**: Unique identifier (e.g., TC001, TC002)
- **Category**: Test category (e.g., Functional, Negative, Edge Case, Integration, E2E)
- **Priority**: Test priority (e.g., High, Medium, Low, Critical)
- **Test Objective**: Clear description of what is being tested
- **Preconditions**: Setup requirements or conditions before test execution. Each precondition MUST be on a new line within the cell (use line breaks between points)
- **Test Steps**: Numbered step-by-step instructions to execute the test. Each step MUST be on a new line within the cell (use line breaks between steps)
- **Expected Results**: Expected outcome for each step or overall test. Each expected result MUST be on a new line within the cell (use line breaks between points)

**CRITICAL: No Hallucination Policy**
- **DO NOT hallucinate or invent Test Steps, Expected Results, or Preconditions**
- Only include information that can be directly derived from:
  - Jira ticket description, acceptance criteria, and requirements
  - User-provided specifications
  - Existing documentation or test cases
  - Clear, verifiable feature behavior
- If certain details are unclear or missing from the source material:
  - Use generic, reasonable test steps that follow standard testing practices
  - Mark uncertain areas with "[Verify with team]" or "[Confirm expected behavior]"
  - Ask the user for clarification when critical information is missing
- **Be accurate and truthful** - better to have fewer, correct test cases than many test cases with made-up details
- Base all test case details on actual requirements, not assumptions

**Formatting Example:**
- Preconditions cell should contain:
  ```
  1. User is logged in
  2. User has admin privileges
  3. Test data is available
  ```
- Test Steps cell should contain:
  ```
  1. Navigate to dashboard
  2. Click on settings menu
  3. Select user management
  4. Click add new user button
  ```
- Expected Results cell should contain:
  ```
  1. Dashboard loads successfully
  2. Settings menu opens
  3. User management page displays
  4. New user form appears
  ```

**IMPORTANT: Check for Existing Test Cases First**

Before generating new test cases, ALWAYS check if test cases already exist:

1. **If Google Sheets link is provided:**

   - Use the Task tool with `subagent_type='google-sheets-manager'` to read existing test cases from the sheet
   - Review the existing test cases to understand what coverage already exists
   - Identify gaps in test coverage (missing scenarios, incomplete categories, etc.)
   - Note existing test case IDs, scenarios, and categories to avoid duplication

2. **If checking local test files:**

   - Use Glob to find test files (e.g., `**/*.test.js`, `**/*.spec.ts`, `**/test_*.py`)
   - Use Read to examine existing test files
   - Identify what test scenarios are already covered
   - Note any missing test categories or edge cases

3. **Generate only what's needed:**

   - Use the Task tool with `subagent_type='test-generator'` to:
     - Generate comprehensive functional test cases from requirements
     - Generate the specified number of test cases (default: 20 if not specified)
     - Create test cases for both AI and non-AI features
     - Cover all test categories: functional, negative, edge cases, end-to-end, and integration tests
     - Include AI evaluation tests only when the feature involves AI/ML components
     - **Ensure no duplicate test cases by excluding scenarios already covered**
     - Create detailed test steps with preconditions, expected results, and test data
     - Focus on filling gaps in existing test coverage
     - **CRITICAL: Do NOT hallucinate test details** - only create test cases based on actual requirements from Jira, Figma, or user input
     - If information is missing or unclear, use generic test steps or ask for clarification rather than inventing details
     - When Figma designs are available, include detailed UI/UX test cases with specific element references

4. When to use the test-generator:

   - **"create test cases from jira"**: After extracting requirements from Jira, check existing tests, then use test-generator to create comprehensive test cases
   - **"generate test suite"**: When user explicitly requests test case generation
   - **"create test coverage"**: When analyzing features that need testing
   - **"add missing tests"**: When augmenting existing test suites
   - After implementing new functionality that requires validation
   - Before releases to ensure proper test coverage

5. Test generation workflow:
   - **First: Check for existing test cases** (from Google Sheets or local test files)
   - Analyze existing coverage and identify gaps
   - Gather requirements (from Jira, Figma, or user input)
   - Extract the number of test cases to generate from user input (default: 20 if not specified)
   - Invoke the test-generator agent with the requirements, number of test cases, AND information about existing tests
   - If Figma designs are provided, ensure test cases include specific UI element references from the design
   - Receive comprehensive test suite covering all categories, excluding duplicates
   - Optionally write the generated test cases to Google Sheets (appending to existing cases, not replacing)

### Step 5: Google Sheets Operations (if sheet link provided)

If a Google Sheets link is provided:

**IMPORTANT: Ensure Test Case Format in Google Sheets**

When writing test cases to Google Sheets, ensure the sheet has these column headers in this exact order:
1. Test Case ID
2. Category
3. Priority
4. Test Objective
5. Preconditions (each precondition on a new line within the cell)
6. Test Steps (each step on a new line within the cell)
7. Expected Results (each result on a new line within the cell)

**CRITICAL: Multi-line Cell Formatting**
- For Preconditions, Test Steps, and Expected Results columns, use newline characters (`\n`) to separate each point within the cell
- In Google Sheets API, use `\n` to create line breaks within a single cell value
- Example: `"1. User is logged in\n2. User has admin privileges\n3. Test data is available"`

1. Use the Task tool with `subagent_type='google-sheets-manager'` to:

   - Read existing test case data (if updating)
   - Write new test cases extracted from Jira or generated by test-generator in the required format
   - Update test execution status
   - Create new sheets for test documentation with proper column headers
   - Append test results or metrics following the standard format

2. Operations based on task_details:
   - **"create test cases"**: Write test scenarios from Jira or test-generator to Google Sheets
   - **"update status"**: Update test execution status in existing sheet
   - **"export results"**: Read test data from Sheets and format it
   - **"sync from jira"**: Fetch Jira data and populate/update Google Sheets
   - Custom operations based on user's specific needs

### Step 6: Combined Workflow

**Figma + Test Generator Workflow:**
When Figma design link is provided and test generation is needed:

1. Invoke the figma-agent to fetch and analyze the Figma design file
2. Extract UI flows, user journeys, interactive elements, and design specifications
3. Check for existing test cases (local test files or any provided sheet)
4. Extract the number of test cases to generate from user input (default: 20)
5. Invoke the test-generator agent with:
   - Extracted UI flows and design specifications from Figma
   - Specified number of test cases to generate
   - Existing test case information
6. Receive comprehensive test suite with detailed UI/UX test cases that reference specific design elements
7. Report the generated test cases to the user

**Figma + Jira + Test Generator Workflow:**
When both Figma design and Jira ticket are provided:

1. Invoke the figma-agent to analyze the design and extract UI flows
2. Invoke the jira-ticket-analyzer agent to fetch requirements and acceptance criteria
3. Check for existing test cases (local test files or any provided sheet)
4. Extract the number of test cases to generate from user input (default: 20)
5. Invoke the test-generator agent with:
   - Combined requirements from both Jira (functional specs) and Figma (UI/UX specs)
   - Specified number of test cases to generate
   - Existing test case information
6. Receive comprehensive test suite covering both functional requirements and design specifications
7. Report the generated test cases to the user

**Jira + Test Generator Workflow:**
When Jira ticket is provided and test generation is needed:

1. Check for existing test cases (local test files or any provided sheet)
2. Extract the number of test cases to generate from user input (default: 20)
3. Invoke the jira-ticket-analyzer agent to fetch and analyze the Jira ticket
4. Extract requirements, acceptance criteria, and specifications
5. Invoke the test-generator agent with the extracted requirements, specified number of test cases, AND existing test case information
6. Receive comprehensive test suite covering all test categories (excluding duplicates)
7. Report the generated test cases to the user

**Jira + Google Sheets Workflow:**
When both Jira ticket and Google Sheets are provided:

1. Invoke the google-sheets-manager agent to read existing test cases (if any)
2. Invoke the jira-ticket-analyzer agent to fetch and analyze the Jira ticket
3. Process and structure the extracted data based on task_details
4. Invoke the google-sheets-manager agent to perform the sheet operations (append, not replace)
5. Ensure data flows correctly from Jira → Processing → Google Sheets

**Figma + Jira + Test Generator + Google Sheets Workflow:**
For complete test case management with design specifications (recommended):

1. **First: Read existing test cases** - Invoke the google-sheets-manager agent to read current test cases from Google Sheets
2. Analyze existing test coverage and identify gaps
3. Extract the number of test cases to generate from user input (default: 20)
4. Invoke the figma-agent to analyze the design and extract UI flows and specifications
5. Invoke the jira-ticket-analyzer agent to fetch Jira ticket details and extract requirements
6. Invoke the test-generator agent with:
   - Extracted UI flows and design specifications from Figma
   - Extracted requirements from Jira
   - Specified number of test cases to generate
   - Information about existing test cases
   - Specific gaps to fill in test coverage
7. Invoke the google-sheets-manager agent to **append** the generated test cases to Google Sheets (not replace existing ones)
8. Ensure data flows: Existing Tests (Read) → Figma + Jira → Test Generator → Google Sheets (Append)
9. Report completion with summary of:
   - Number of existing test cases found
   - Number of new test cases created
   - Total test coverage achieved
   - Sheet location

**Jira + Test Generator + Google Sheets Workflow:**
For complete test case management (recommended):

1. **First: Read existing test cases** - Invoke the google-sheets-manager agent to read current test cases from Google Sheets
2. Analyze existing test coverage and identify gaps
3. Extract the number of test cases to generate from user input (default: 20)
4. Invoke the jira-ticket-analyzer agent to fetch Jira ticket details and extract requirements
5. Invoke the test-generator agent with:
   - Extracted requirements from Jira
   - Specified number of test cases to generate
   - Information about existing test cases
   - Specific gaps to fill in test coverage
6. Invoke the google-sheets-manager agent to **append** the generated test cases to Google Sheets (not replace existing ones)
7. Ensure data flows: Existing Tests (Read) → Jira → Test Generator → Google Sheets (Append)
8. Report completion with summary of:
   - Number of existing test cases found
   - Number of new test cases created
   - Total test coverage achieved
   - Sheet location

Example combined workflow for "create test cases from Jira and Figma":

- **Step 1: Read existing test cases** from Google Sheets (if provided) or local test files
- Analyze existing coverage: identify what test scenarios, categories, and edge cases are already covered
- Extract the number of test cases to generate from user input (default: 20)
- Fetch Figma design specifications and extract UI flows, elements, and interactions (if Figma link provided)
- Fetch Jira ticket details and extract requirements, acceptance criteria, and specifications (if Jira ticket provided)
- Generate comprehensive test suite using test-generator (functional, negative, edge, e2e, integration tests, UI/UX tests)
  - Generate the specified number of test cases
  - Include detailed UI element references from Figma design (if available)
  - **Explicitly exclude already-covered test scenarios**
  - Focus on filling gaps in test coverage
- Structure new test cases into the required Google Sheets format (Test Case ID, Category, Priority, Test Objective, Preconditions, Test Steps, Expected Results)
- **Append** formatted test cases to the specified Google Sheet (preserving existing test cases)
- Report completion with summary:
  - Existing test cases: X
  - New test cases added: Y
  - Total coverage: X + Y test cases

## Task Details Examples

Here are common task_details and how they should be handled:

| Task Detail                                  | Action                                                                                                                                                                                                 |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| "extract test scenarios"                     | Get Jira ticket, parse acceptance criteria, return structured test scenarios                                                                                                                           |
| "create test cases" or "generate test cases" | **Check existing tests first**, extract number of test cases (default: 20), get Jira/Figma (if provided), use test-generator to create comprehensive test suite (excluding duplicates)                |
| "create test cases from figma"               | **Check existing tests first**, extract number of test cases (default: 20), use figma-agent to analyze design, use test-generator to create UI/UX test cases with specific element references         |
| "create test cases from figma and jira"      | **Check existing tests first**, extract number of test cases (default: 20), use figma-agent + jira-ticket-analyzer, use test-generator to create comprehensive test suite with design specifications |
| "create test cases in sheet"                 | **Read existing test cases from sheet**, extract number of test cases (default: 20), get Jira/Figma, use test-generator to create test cases, **append** to Google Sheets                             |
| "generate comprehensive test suite"          | **Check existing test coverage**, extract number of test cases (default: 20), use test-generator to create functional, negative, edge, e2e, and integration test cases (filling gaps only)             |
| "add missing tests"                          | **Read existing tests**, extract number of test cases (default: 20), identify gaps in coverage, use test-generator to create only missing test cases                                                   |
| "analyze figma design"                       | Use figma-agent to fetch UI flows, interactions, and design specifications from Figma file                                                                                                             |
| "update test execution status"               | Read Google Sheets, update status columns based on criteria                                                                                                                                            |
| "sync jira bugs to sheet"                    | **Check for existing bugs in sheet**, fetch Jira bug details, append new bugs to Google Sheets (avoid duplicates)                                                                                      |
| "generate test report"                       | Read Google Sheets test data, analyze, create summary report                                                                                                                                           |
| "export jira to excel format"                | Fetch Jira data, structure as tabular data, write to Google Sheets                                                                                                                                     |
| "create test coverage for feature"           | **Check existing test coverage**, extract number of test cases (default: 20), analyze feature requirements, use test-generator to create complete test coverage (filling gaps)                         |
| "audit test coverage"                        | Read existing test cases, analyze coverage gaps, report what's missing                                                                                                                                 |

## Instructions

When the user invokes this command:

1. **Parse the user input** to identify:

   - Jira ticket references
   - Figma design URLs (look for "figma.com/file/" links)
   - Google Sheets URLs
   - Task details (what action to perform)
   - Number of test cases to generate (look for phrases like "10 test cases", "generate 25 tests", "create 30", etc.)

2. **Validate inputs**:

   - If Jira ticket is mentioned but not clearly identified, ask for ticket ID/URL
   - If Figma design is mentioned but no link provided, ask for Figma URL
   - If Google Sheets is mentioned but no link provided, ask for sheet URL
   - If task_details is unclear, ask what specific action should be performed
   - If number of test cases is not specified, default to 20

3. **Execute in proper sequence**:

   - Figma operations first (if needed) to gather UI/UX specifications
   - Jira operations next (if needed) to gather functional requirements
   - Process and transform data based on task_details
   - Google Sheets operations last (if needed) to write/update data

4. **Provide clear feedback**:

   - Summarize what was fetched from Figma (UI flows, design specifications)
   - Summarize what was fetched from Jira (requirements, acceptance criteria)
   - Describe what operations were performed on Google Sheets
   - Report any errors or issues encountered
   - Confirm completion with relevant details

5. **Error handling**:
   - If Figma link is invalid or inaccessible, report clearly and ask for correct link
   - If Jira ticket not found, report clearly and offer alternatives
   - If Google Sheets access fails, explain the issue
   - If task_details cannot be understood, ask for clarification

## User's Input

{USER_INPUT}

---

**Now execute**:

1. Analyze the user input above
2. Determine what resources are provided (Figma link, Jira ticket, Google Sheets, task_details, number of test cases)
3. Extract the number of test cases to generate (default: 20 if not specified)
4. If information is missing, use AskUserQuestion to gather required details
5. **CRITICAL: If the task involves creating/generating test cases, FIRST check for existing test cases:**
   - If Google Sheets is provided: Read existing test cases from the sheet
   - If no sheet provided: Use Glob to find local test files and Read to examine them
   - Analyze existing test coverage to identify what already exists and what gaps need to be filled
6. Invoke appropriate agents in the correct sequence:
   - google-sheets-manager (FIRST - to read existing test cases if sheet is provided)
   - figma-agent (if Figma link provided and UI/UX specifications need to be extracted)
   - jira-ticket-analyzer (if Jira ticket provided and requirements need to be extracted)
   - test-generator (if test case generation is needed - provide existing test info, Figma specs, Jira requirements, and number of test cases to avoid duplicates)
   - google-sheets-manager (LAST - to append new test cases to sheet if provided)
7. Process results according to task_details
8. Provide clear summary of actions performed and results including:
   - Number of test cases requested (if specified) or using default (20)
   - Number of existing test cases found (if applicable)
   - Number of new test cases created
   - Total test coverage achieved
   - What gaps were filled
   - UI/UX coverage from Figma (if applicable)

**Recommended workflow for comprehensive test case creation:**

- **ALWAYS check for existing test cases BEFORE generating new ones** - this prevents duplication and ensures efficient test coverage
- **Extract the number of test cases to generate from user input** (default: 20 if not specified)
- **If Figma link is provided, FIRST use figma-agent to get UI flows information** - this enables generation of detailed UI/UX test cases with specific element references
- When the task involves creating or generating test cases, ALWAYS use the test-generator agent
- Provide the test-generator with:
  - Information about existing test cases so it can focus on filling gaps
  - UI flows and design specifications from Figma (if available)
  - Functional requirements from Jira (if available)
  - The specified number of test cases to generate
- The test-generator creates comprehensive test suites covering: functional, negative, edge cases, end-to-end, integration tests, UI/UX tests, and AI evaluation tests (when applicable)
- When writing to Google Sheets, APPEND new test cases rather than replacing existing ones
- This ensures complete test coverage without duplication and follows testing best practices
