```dataviewjs
// Initialize today's date
let today = dv.date("today");

// Fetch all todos
let todos = Array.from(
    dv.pages("#todo")
        .filter(page => !page.file.tags.includes("#done")) // Exclude completed todos
        .filter(page => !page.file.tags.includes("#dnf"))  // Exclude "dnf" todos
        .filter(page => !page.activation_date || dv.date(page.activation_date) <= today) // Only include if activation_date is in the past or today
);

// Function to check if all blockers are completed or dnf
function isUnblocked(todo) {
    if (!todo.blocker || todo.blocker.length === 0) return true; // No blockers, so it's unblocked
    
    return todo.blocker.every(blocker => {
        let blockerPage = dv.page(blocker.path);
        return blockerPage && (blockerPage.file.tags.includes("#done") || blockerPage.file.tags.includes("#dnf"));
    });
}

// Filter out blocked todos
let unblockedTodos = todos.filter(todo => isUnblocked(todo));

// Group todos by priority
let groupedTodos = {
    "High Priority": unblockedTodos.filter(page => page.file.tags.includes("#priority/high_priority")),
    "Medium Priority": unblockedTodos.filter(page => page.file.tags.includes("#priority/medium_priority")),
    "Low Priority": unblockedTodos.filter(page => page.file.tags.includes("#priority/low_priority")),
    "Unprioritized": unblockedTodos.filter(page => 
        !page.file.tags.includes("#priority/high_priority") &&
        !page.file.tags.includes("#priority/medium_priority") &&
        !page.file.tags.includes("#priority/low_priority")
    )
};

// Render todos for each priority group
Object.entries(groupedTodos).forEach(([priority, items]) => {
    if (items.length > 0) {
        // Render priority heading
        dv.header(2, priority);

        // Render todos as a list
        dv.list(items.map(todo => todo.file.link));
    }
});
```
