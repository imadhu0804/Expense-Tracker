# ExpenseTracker Pro - Services Documentation

## Overview

ExpenseTracker Pro uses a service-oriented architecture to handle all business logic and data operations. Each service is responsible for a specific domain of functionality, providing a clean separation of concerns and making the codebase maintainable and testable.

---

## Services

### 1. **expenseService**

Manages all expense-related operations including CRUD operations and data persistence.

#### Key Responsibilities:
- Create, read, update, and delete expenses
- Retrieve all expenses or filter by specific criteria
- Persist expense data to local storage
- Validate expense data before operations

#### Common Methods:
```typescript
getAllExpenses(): Promise<Expense[]>
createExpense(data: ExpenseFormData): Promise<Expense>
updateExpense(id: number, data: ExpenseFormData): Promise<Expense>
deleteExpense(id: number): Promise<void>
```

#### Use Cases:
- Adding a new expense from the Add Expense page
- Fetching expenses for the dashboard and expense list
- Updating expense details from the edit modal
- Deleting expenses from the expense table

---

### 2. **budgetService**

Handles budget goal creation, tracking, and monitoring against actual spending.

#### Key Responsibilities:
- Set and manage monthly budget limits per category
- Track current spending against budget goals
- Calculate budget utilization percentages
- Generate budget alerts and notifications
- Update spent amounts based on expense data

#### Common Methods:
```typescript
getAllBudgets(): Promise<BudgetGoal[]>
setBudget(category: string, amount: number, month: number, year: number): Promise<void>
updateSpentAmount(category: string, month: number, year: number, spent: number): Promise<void>
getBudgetForCategory(category: string, month: number, year: number): Promise<BudgetGoal | null>
```

#### Use Cases:
- Creating budget goals from the Budget page
- Displaying budget cards with progress indicators
- Triggering budget alerts on the dashboard
- Calculating budget achievement for spending streaks

---

### 3. **currencyService**

Manages multi-currency support, exchange rates, and currency conversions.

#### Key Responsibilities:
- Fetch real-time exchange rates from external APIs
- Store and cache currency rates
- Convert amounts between different currencies
- Manage user's preferred currency settings
- Handle currency formatting based on locale

#### Common Methods:
```typescript
fetchRates(): Promise<void>
convertAmount(amount: number, fromCurrency: string, toCurrency: string): number
getCurrentCurrency(): string
setCurrentCurrency(currency: string): void
formatCurrency(amount: number, currency?: string): string
```

#### Use Cases:
- Converting expenses to different currencies on the dashboard
- Displaying amounts in user's preferred currency
- Multi-currency expense tracking for international users
- Currency selector in settings

---

### 4. **themeService**

Handles application theming, dark mode, and visual customization.

#### Key Responsibilities:
- Toggle between light and dark themes
- Persist theme preferences to local storage
- Apply theme changes across the application
- Provide theme-aware color schemes for charts and UI elements
- Support custom color palettes

#### Common Methods:
```typescript
getTheme(): 'light' | 'dark'
setTheme(theme: 'light' | 'dark'): void
toggleTheme(): void
getChartColors(): string[]
```

#### Use Cases:
- Theme toggle in the navigation bar or settings
- Applying consistent colors across dashboard charts
- User preference persistence across sessions
- Dynamic styling based on theme state

---

### 5. **smartCategoryService**

Provides AI-powered category suggestions and intelligent expense categorization.

#### Key Responsibilities:
- Analyze expense titles and descriptions
- Suggest appropriate categories based on keywords
- Learn from user categorization patterns
- Provide smart auto-complete for expense titles
- Improve suggestions over time

#### Common Methods:
```typescript
suggestCategory(title: string, description?: string): Promise<string>
getSimilarExpenses(title: string): Expense[]
learnFromUserChoice(title: string, category: string): void
getPopularCategories(): string[]
```

#### Use Cases:
- Auto-suggesting categories when adding expenses
- Providing smart defaults in the expense form
- Learning from user behavior to improve accuracy
- Category recommendations in the expense form

---

### 6. **recurringExpenseService**

Manages recurring expenses, subscriptions, and automated expense creation.

#### Key Responsibilities:
- Create and manage recurring expense templates
- Automatically generate expenses based on schedules
- Track subscription renewals and due dates
- Send notifications for upcoming recurring expenses
- Handle different recurrence patterns (daily, weekly, monthly, yearly)

#### Common Methods:
```typescript
getAllRecurringExpenses(): Promise<RecurringExpense[]>
createRecurringExpense(data: RecurringExpenseData): Promise<RecurringExpense>
updateRecurringExpense(id: number, data: RecurringExpenseData): Promise<RecurringExpense>
deleteRecurringExpense(id: number): Promise<void>
generateDueExpenses(): Promise<Expense[]>
```

#### Use Cases:
- Setting up monthly subscription tracking (Netflix, Spotify, etc.)
- Automating rent or mortgage payments
- Managing recurring bills and utilities
- Notification alerts for upcoming recurring expenses

---

### 7. **exportService**

Handles data export functionality in various formats for reporting and backup.

#### Key Responsibilities:
- Export expense data to CSV format
- Generate PDF reports with charts and summaries
- Create JSON backups of all data
- Format data for different export types
- Handle date range filtering for exports

#### Common Methods:
```typescript
exportToCSV(expenses: Expense[], filename?: string): void
exportToPDF(expenses: Expense[], options?: PDFOptions): Promise<void>
exportToJSON(data: object, filename?: string): void
generateReport(expenses: Expense[], format: 'csv' | 'pdf' | 'json'): Promise<void>
```

#### Use Cases:
- Exporting expenses for tax preparation
- Creating monthly/yearly financial reports
- Backing up expense data
- Sharing expense reports via the Export Modal
- Integration with accounting software

---

## Service Architecture

### Design Principles

1. **Single Responsibility**: Each service handles one domain of functionality
2. **Dependency Injection**: Services can be easily mocked for testing
3. **Async Operations**: All data operations return Promises for better UX
4. **Error Handling**: Services throw meaningful errors that can be caught by UI components
5. **Data Persistence**: Services handle their own data storage logic

### Data Flow

```
User Interaction → Component → Service → Data Layer (localStorage/API)
                                  ↓
                          State Update
                                  ↓
                          UI Re-render
```

### Service Integration

Services work together to provide comprehensive functionality:

- **expenseService** + **budgetService**: Track spending against budget goals
- **expenseService** + **smartCategoryService**: Provide intelligent category suggestions
- **expenseService** + **exportService**: Generate expense reports
- **recurringExpenseService** + **expenseService**: Auto-create recurring expenses
- **currencyService** + **expenseService**: Multi-currency expense tracking

---

## Usage Examples

### Adding an Expense with Smart Categories

```typescript
// In AddExpense component
const handleSubmit = async (formData: ExpenseFormData) => {
  // Smart category suggestion
  const suggestedCategory = await smartCategoryService.suggestCategory(
    formData.title, 
    formData.notes
  );
  
  // Create expense
  await expenseService.createExpense({
    ...formData,
    category: formData.category || suggestedCategory
  });
  
  // Update budget tracking
  const currentDate = new Date();
  await budgetService.updateSpentAmount(
    formData.category,
    currentDate.getMonth() + 1,
    currentDate.getFullYear(),
    formData.amount
  );
};
```

### Checking Budget Status

```typescript
// In Dashboard component
const checkBudgetAlerts = async () => {
  const budgets = await budgetService.getAllBudgets();
  const expenses = await expenseService.getAllExpenses();
  
  budgets.forEach(budget => {
    const percentage = (budget.currentSpent / budget.monthlyLimit) * 100;
    if (percentage > 80) {
      // Show alert
      showWarningToast(`${budget.category} budget is ${percentage}% used`);
    }
  });
};
```

### Exporting Data

```typescript
// In Export Modal
const handleExport = async (format: 'csv' | 'pdf' | 'json') => {
  const expenses = await expenseService.getAllExpenses();
  
  switch(format) {
    case 'csv':
      exportService.exportToCSV(expenses);
      break;
    case 'pdf':
      await exportService.exportToPDF(expenses, { includeCharts: true });
      break;
    case 'json':
      exportService.exportToJSON({ expenses, budgets, settings });
      break;
  }
};
```

---

## Best Practices

1. **Always handle errors**: Wrap service calls in try-catch blocks
2. **Show loading states**: Use loading flags while async operations are in progress
3. **Provide user feedback**: Show success/error toasts after operations
4. **Validate data**: Services should validate input before processing
5. **Keep services pure**: Avoid direct DOM manipulation in services
6. **Use TypeScript**: Leverage type safety for service methods and data structures

---

## Testing Services

Each service should have comprehensive unit tests covering:

- Happy path scenarios
- Error handling
- Edge cases
- Data validation
- State management

```typescript
describe('expenseService', () => {
  it('should create an expense successfully', async () => {
    const formData: ExpenseFormData = {
      title: 'Groceries',
      amount: 50,
      date: '2025-10-15',
      category: 'Food',
      notes: 'Weekly shopping'
    };
    
    const expense = await expenseService.createExpense(formData);
    expect(expense).toBeDefined();
    expect(expense.title).toBe('Groceries');
  });
});
```

---

## Future Enhancements

- **analyticsService**: Advanced spending analytics and predictions
- **syncService**: Cloud synchronization across devices
- **notificationService**: Push notifications for budgets and recurring expenses
- **importService**: Import data from other expense tracking apps
- **aiInsightsService**: Machine learning-based spending insights and recommendations

---

## Contributing

When adding new services:

1. Create a new service file in `/services`
2. Define clear interfaces for methods
3. Add comprehensive error handling
4. Write unit tests
5. Update this documentation
6. Ensure service follows existing patterns

---

## Support

For issues or questions about services:
- Check the source code documentation
- Review the component usage examples
- Refer to the type definitions in `/types`
- Contact the development team

---

**Version**: 1.0.0  
**Last Updated**: October 2025  
**Maintained By**: ExpenseTracker Pro Team
