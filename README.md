// Task Manager Class
class TaskManager {
    constructor() {
        this.tasks = this.loadTasks();
        this.currentFilter = 'all';
        this.editingTaskId = null;
        this.init();
    }

    // Initialize the app
    init() {
        this.cacheDOM();
        this.setupEventListeners();
        this.render();
    }

    // Cache DOM elements
    cacheDOM() {
        this.taskInput = document.getElementById('taskInput');
        this.addBtn = document.getElementById('addBtn');
        this.taskList = document.getElementById('taskList');
        this.clearBtn = document.getElementById('clearBtn');
        this.deleteAllBtn = document.getElementById('deleteAllBtn');
        this.filterBtns = document.querySelectorAll('.filter-btn');
        this.totalCount = document.getElementById('totalCount');
        this.completedCount = document.getElementById('completedCount');
        this.remainingCount = document.getElementById('remainingCount');
    }

    // Setup event listeners
    setupEventListeners() {
        this.addBtn.addEventListener('click', () => this.addTask());
        this.taskInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') this.addTask();
        });
        this.clearBtn.addEventListener('click', () => this.clearCompleted());
        this.deleteAllBtn.addEventListener('click', () => this.deleteAll());
        this.filterBtns.forEach(btn => {
            btn.addEventListener('click', (e) => this.setFilter(e.target.dataset.filter));
        });
    }

    // Load tasks from local storage
    loadTasks() {
        const saved = localStorage.getItem('tasks');
        return saved ? JSON.parse(saved) : [];
    }

    // Save tasks to local storage
    saveTasks() {
        localStorage.setItem('tasks', JSON.stringify(this.tasks));
    }

    // Add a new task
    addTask() {
        const text = this.taskInput.value.trim();
        
        if (!text) {
            this.showToast('Please enter a task', 'error');
            return;
        }

        if (text.length > 100) {
            this.showToast('Task is too long (max 100 characters)', 'error');
            return;
        }

        const task = {
            id: Date.now(),
            text: text,
            completed: false,
            priority: 'medium',
            createdAt: new Date().toLocaleDateString(),
            dueDate: ''
        };

        this.tasks.unshift(task);
        this.saveTasks();
        this.taskInput.value = '';
        this.taskInput.focus();
        this.showToast('Task added successfully', 'success');
        this.render();
    }

    // Toggle task completion
    toggleTask(id) {
        const task = this.tasks.find(t => t.id === id);
        if (task) {
            task.completed = !task.completed;
            this.saveTasks();
            this.render();
        }
    }

    // Delete a task
    deleteTask(id) {
        this.tasks = this.tasks.filter(t => t.id !== id);
        this.saveTasks();
        this.showToast('Task deleted', 'info');
        this.render();
    }

    // Edit a task
    editTask(id) {
        const task = this.tasks.find(t => t.id === id);
        if (task) {
            this.editingTaskId = id;
            this.showEditModal(task);
        }
    }

    // Show edit modal
    showEditModal(task) {
        const modal = document.createElement('div');
        modal.className = 'modal active';
        modal.id = 'editModal';
        modal.innerHTML = `
            <div class="modal-content">
                <div class="modal-header">Edit Task</div>
                <div class="modal-body">
                    <div class="form-group">
                        <label for="editText">Task Description</label>
                        <input type="text" id="editText" value="${task.text}" maxlength="100">
                    </div>
                    <div class="form-group">
                        <label for="editPriority">Priority</label>
                        <select id="editPriority">
                            <option value="low" ${task.priority === 'low' ? 'selected' : ''}>Low</option>
                            <option value="medium" ${task.priority === 'medium' ? 'selected' : ''}>Medium</option>
                            <option value="high" ${task.priority === 'high' ? 'selected' : ''}>High</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label for="editDueDate">Due Date (Optional)</label>
                        <input type="date" id="editDueDate" value="${task.dueDate}">
                    </div>
                </div>
                <div class="modal-footer">
                    <button class="modal-btn modal-cancel" onclick="document.getElementById('editModal').remove()">Cancel</button>
                    <button class="modal-btn modal-save" id="saveEditBtn">Save</button>
                </div>
            </div>
        `;
        document.body.appendChild(modal);

        modal.addEventListener('click', (e) => {
            if (e.target === modal) modal.remove();
        });

        document.getElementById('saveEditBtn').addEventListener('click', () => {
            const newText = document.getElementById('editText').value.trim();
            const newPriority = document.getElementById('editPriority').value;
            const newDueDate = document.getElementById('editDueDate').value;

            if (!newText) {
                this.showToast('Task cannot be empty', 'error');
                return;
            }

            task.text = newText;
            task.priority = newPriority;
            task.dueDate = newDueDate;
            this.saveTasks();
            this.editingTaskId = null;
            modal.remove();
            this.showToast('Task updated', 'success');
            this.render();
        });
    }

    // Clear completed tasks
    clearCompleted() {
        const completedCount = this.tasks.filter(t => t.completed).length;
        if (completedCount === 0) {
            this.showToast('No completed tasks to clear', 'info');
            return;
        }

        if (confirm(`Delete ${completedCount} completed task(s)?`)) {
            this.tasks = this.tasks.filter(t => !t.completed);
            this.saveTasks();
            this.showToast('Completed tasks cleared', 'success');
            this.render();
        }
    }

    // Delete all tasks
    deleteAll() {
        if (this.tasks.length === 0) {
            this.showToast('No tasks to delete', 'info');
            return;
        }

        if (confirm('Delete ALL tasks? This cannot be undone.')) {
            this.tasks = [];
            this.saveTasks();
            this.showToast('All tasks deleted', 'success');
            this.render();
        }
    }

    // Set filter
    setFilter(filter) {
        this.currentFilter = filter;
        this.filterBtns.forEach(btn => {
            btn.classList.remove('active');
            if (btn.dataset.filter === filter) {
                btn.classList.add('active');
            }
        });
        this.render();
    }

    // Get filtered tasks
    getFilteredTasks() {
        switch (this.currentFilter) {
            case 'active':
                return this.tasks.filter(t => !t.completed);
            case 'completed':
                return this.tasks.filter(t => t.completed);
            default:
                return this.tasks;
        }
    }

    // Update stats
    updateStats() {
        const total = this.tasks.length;
        const completed = this.tasks.filter(t => t.completed).length;
        const remaining = total - completed;

        this.totalCount.textContent = total;
        this.completedCount.textContent = completed;
        this.remainingCount.textContent = remaining;
    }

    // Show/hide action buttons
    updateActionButtons() {
        const hasCompleted = this.tasks.some(t => t.completed);
        const hasAnyTask = this.tasks.length > 0;

        this.clearBtn.style.display = hasCompleted ? 'block' : 'none';
        this.deleteAllBtn.style.display = hasAnyTask ? 'block' : 'none';
    }

    // Render tasks
    render() {
        const filteredTasks = this.getFilteredTasks();
        this.updateStats();
        this.updateActionButtons();

        if (filteredTasks.length === 0) {
            this.taskList.innerHTML = `
                <div class="empty-state">
                    <div class="empty-icon">📝</div>
                    <p>${this.currentFilter === 'all' ? 'No tasks yet. Add one to get started!' : `No ${this.currentFilter} tasks.`}</p>
                </div>
            `;
            return;
        }

        this.taskList.innerHTML = filteredTasks.map(task => this.createTaskHTML(task)).join('');

        // Add event listeners to task items
        this.taskList.querySelectorAll('.task-checkbox').forEach(checkbox => {
            checkbox.addEventListener('change', (e) => {
                this.toggleTask(parseInt(e.target.dataset.id));
            });
        });

        this.taskList.querySelectorAll('.edit-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.editTask(parseInt(e.target.dataset.id));
            });
        });

        this.taskList.querySelectorAll('.delete-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.deleteTask(parseInt(e.target.dataset.id));
            });
        });
    }

    // Create task HTML
    createTaskHTML(task) {
        return `
            <div class="task-item ${task.completed ? 'completed' : ''}">
                <input 
                    type="checkbox" 
                    class="task-checkbox" 
                    data-id="${task.id}"
                    ${task.completed ? 'checked' : ''}
                >
                <div class="task-content">
                    <span class="task-text">${this.escapeHtml(task.text)}</span>
                    <span class="task-priority ${task.priority}">${task.priority}</span>
                </div>
                ${task.dueDate ? `<div class="task-date">Due: ${task.dueDate}</div>` : ''}
                <div class="task-actions">
                    <button class="task-btn edit-btn" data-id="${task.id}">Edit</button>
                    <button class="task-btn delete-btn" data-id="${task.id}">Delete</button>
                </div>
            </div>
        `;
    }

    // Escape HTML to prevent XSS
    escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }

    // Show toast notification
    showToast(message, type = 'info') {
        const toast = document.createElement('div');
        toast.className = `toast ${type}`;
        toast.textContent = message;
        document.body.appendChild(toast);

        setTimeout(() => {
            toast.style.animation = 'slideInRight 0.3s ease reverse';
            setTimeout(() => toast.remove(), 300);
        }, 3000);
    }
}

// Initialize app when DOM is loaded
document.addEventListener('DOMContentLoaded', () => {
    new TaskManager();
});

// Keyboard shortcuts
document.addEventListener('keydown', (e) => {
    // Ctrl/Cmd + L to focus input
    if ((e.ctrlKey || e.metaKey) && e.key === 'l') {
        e.preventDefault();
        document.getElementById('taskInput').focus();
    }
});

// Show a welcome message on first load
window.addEventListener('load', () => {
    const hasVisited = localStorage.getItem('hasVisited');
    if (!hasVisited) {
        localStorage.setItem('hasVisited', 'true');
        setTimeout(() => {
            const taskManager = window.taskManager || new TaskManager();
            if (taskManager.tasks.length === 0) {
                // Add welcome message via toast
                console.log('Welcome to TaskFlow! Add your first task to get started.');
            }
        }, 500);
    }
});

// Prevent accidental page close if there are unsaved changes
window.addEventListener('beforeunload', (e) => {
    const tasks = JSON.parse(localStorage.getItem('tasks') || '[]');
    if (tasks.length > 0 && document.getElementById('taskInput').value.trim()) {
        e.preventDefault();
        e.returnValue = '';
    }
});
