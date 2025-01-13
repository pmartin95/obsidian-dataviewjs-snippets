The following snippet parses the vault looking for files that have the following:
1. The tag #log/expense. You can customize that part of this script to change it to some other tag.
2. A property called date that is a valid date.
3. A numeric property called price. 

```dataviewjs
let today = dv.date("today"); // Current date
let thirtyDaysAgo = dv.date("today").minus({ days: 30 }); // Date 30 days ago

// Filter pages by the #log/expense tag and within the last 30 days
let recentExpenses = Array.from(
    dv.pages("#log/expense")
        .filter(page => page.date >= thirtyDaysAgo && page.date <= today) // Filter by date range
);

// Validate data integrity
recentExpenses.forEach((entry, index) => {
    if (!entry.date) {
        console.error(`Missing date at index ${index}:`, entry);
    }
});

// Sort expenses by date (descending)
recentExpenses = recentExpenses
    .filter(page => page.date) // Remove entries without a valid date
    .sort((a, b) => {
        if (!a.date || !b.date) {
            console.error("Invalid comparison:", { a, b });
            return 0; // Treat undefined dates as equal
        }
        return b.date.ts - a.date.ts; // Use .ts (timestamp) for comparison
    });

let total = 0; // Initialize total

// Render the expenses
let expenseRows = recentExpenses.map(p => {
    let expenseAmount = p.price ? parseFloat(p.price) : 0;
    total += expenseAmount;
    return [
        p.date.toLocaleString({ month: "long", day: "2-digit", year: "numeric" }), // Use luxon's toLocaleString
        p.file.link, 
        expenseAmount
    ];
});

// Display the table of expenses
dv.table(["Date", "Link", "Expense Amount"], expenseRows);

// Display the total expenses for the last 30 days
dv.paragraph(`Total expenses for the last 30 days: $${total.toFixed(2)}`);

```
