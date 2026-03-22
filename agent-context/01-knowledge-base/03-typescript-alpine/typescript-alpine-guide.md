# 🚀 TypeScript & Alpine.js voor CMS

## Overzicht
Modern JavaScript development met TypeScript voor type safety en Alpine.js voor reactive UI components - de perfecte balans tussen performance en developer experience.

## 🎯 Kernprincipes

### 1. Progressive Enhancement met Alpine.js
```html
<!-- HTML met Alpine.js directives -->
<div x-data="{ menuOpen: false }" class="header">
    <button @click="menuOpen = !menuOpen" 
            :class="{ 'active': menuOpen }" 
            class="menu-toggle"
            aria-label="Toggle menu">
        <svg class="hamburger" viewBox="0 0 24 24">
            <path d="M3 12h18M3 6h18M3 18h18"/>
        </svg>
    </button>

    <nav x-show="menuOpen" 
         x-transition
         class="main-menu">
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
        </ul>
    </nav>
</div>
```

```typescript
// TypeScript enhancement voor complexere logica
// types/menu.ts
export interface MenuState {
    isOpen: boolean;
    activeItem: string | null;
    items: MenuItem[];
}

export interface MenuItem {
    id: string;
    label: string;
    url: string;
    children?: MenuItem[];
}

// components/menu.ts
export class MenuComponent {
    private state: MenuState;
    private element: HTMLElement;

    constructor(element: HTMLElement, initialState: Partial<MenuState> = {}) {
        this.element = element;
        this.state = {
            isOpen: false,
            activeItem: null,
            items: [],
            ...initialState
        };
        this.init();
    }

    private init(): void {
        this.bindEvents();
        this.render();
    }

    private bindEvents(): void {
        // Event delegation voor menu items
        this.element.addEventListener('click', (e: Event) => {
            const target = e.target as HTMLElement;
            const menuItem = target.closest('[data-menu-item]');
            
            if (menuItem) {
                const itemId = menuItem.getAttribute('data-menu-item');
                this.handleItemClick(itemId);
            }
        });

        // Close op escape key
        document.addEventListener('keydown', (e: KeyboardEvent) => {
            if (e.key === 'Escape' && this.state.isOpen) {
                this.close();
            }
        });
    }

    private handleItemClick(itemId: string | null): void {
        if (itemId) {
            this.state.activeItem = itemId;
            this.dispatchEvent('menu:item-selected', { itemId });
        }
    }

    public open(): void {
        this.state.isOpen = true;
        this.render();
        this.dispatchEvent('menu:opened');
    }

    public close(): void {
        this.state.isOpen = false;
        this.render();
        this.dispatchEvent('menu:closed');
    }

    private render(): void {
        this.element.classList.toggle('open', this.state.isOpen);
    }

    private dispatchEvent(eventName: string, detail?: any): void {
        this.element.dispatchEvent(new CustomEvent(eventName, { detail }));
    }
}
```

### 2. TypeScript met Modern ES6+ Features
```typescript
// components/modal.ts
export interface ModalOptions {
    closeOnEscape?: boolean;
    closeOnBackdrop?: boolean;
    showCloseButton?: boolean;
}

export interface ModalState {
    isOpen: boolean;
    title: string;
    content: string;
}

export class Modal {
    private element: HTMLElement;
    private state: ModalState;
    private options: Required<ModalOptions>;

    constructor(element: HTMLElement, options: ModalOptions = {}) {
        this.element = element;
        this.options = {
            closeOnEscape: true,
            closeOnBackdrop: true,
            showCloseButton: true,
            ...options
        };
        
        this.state = {
            isOpen: false,
            title: '',
            content: ''
        };

        this.init();
    }

    private init(): void {
        this.setupShadowDOM();
        this.bindEvents();
    }

    private setupShadowDOM(): void {
        const shadow = this.element.attachShadow({ mode: 'open' });
        shadow.innerHTML = this.getTemplate();
    }

    private getTemplate(): string {
        return `
            <style>
                :host {
                    display: none;
                    position: fixed;
                    top: 0;
                    left: 0;
                    width: 100%;
                    height: 100%;
                    background: rgba(0, 0, 0, 0.5);
                    z-index: 1000;
                    align-items: center;
                    justify-content: center;
                }
                :host([open]) {
                    display: flex;
                }
                .modal-content {
                    background: white;
                    border-radius: 8px;
                    padding: 2rem;
                    max-width: 500px;
                    width: 90%;
                    max-height: 80vh;
                    overflow-y: auto;
                }
                .modal-header {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    margin-bottom: 1rem;
                }
                .close-button {
                    background: none;
                    border: none;
                    font-size: 1.5rem;
                    cursor: pointer;
                    padding: 0;
                    width: 2rem;
                    height: 2rem;
                }
            </style>
            <div class="modal-content">
                <div class="modal-header">
                    <h2 id="modal-title"></h2>
                    <button class="close-button" aria-label="Close">&times;</button>
                </div>
                <div id="modal-body"></div>
            </div>
        `;
    }

    private bindEvents(): void {
        const shadow = this.element.shadowRoot!;
        const closeButton = shadow.querySelector('.close-button') as HTMLButtonElement;
        const modalContent = shadow.querySelector('.modal-content') as HTMLElement;

        closeButton?.addEventListener('click', () => this.close());

        if (this.options.closeOnBackdrop) {
            modalContent?.addEventListener('click', (e: Event) => {
                if (e.target === modalContent) {
                    this.close();
                }
            });
        }

        if (this.options.closeOnEscape) {
            document.addEventListener('keydown', (e: KeyboardEvent) => {
                if (e.key === 'Escape' && this.state.isOpen) {
                    this.close();
                }
            });
        }
    }

    public open(title: string, content: string): void {
        this.state.title = title;
        this.state.content = content;
        this.state.isOpen = true;
        
        this.updateContent();
        this.element.setAttribute('open', '');
        document.body.style.overflow = 'hidden';
        
        this.dispatchEvent('modal:opened', { title, content });
    }

    public close(): void {
        this.state.isOpen = false;
        this.element.removeAttribute('open');
        document.body.style.overflow = '';
        
        this.dispatchEvent('modal:closed');
    }

    private updateContent(): void {
        const shadow = this.element.shadowRoot!;
        const titleElement = shadow.querySelector('#modal-title') as HTMLElement;
        const bodyElement = shadow.querySelector('#modal-body') as HTMLElement;
        
        if (titleElement) titleElement.textContent = this.state.title;
        if (bodyElement) bodyElement.innerHTML = this.state.content;
    }

    private dispatchEvent(eventName: string, detail?: any): void {
        this.element.dispatchEvent(new CustomEvent(eventName, { detail }));
    }
}

// Usage
import { Modal } from './components/modal';

const modal = new Modal(document.getElementById('modal') as HTMLElement);
modal.open('Welcome', 'Hello from TypeScript!');
```

### 3. Alpine.js Reactive State Management
```typescript
// types/state.ts
export interface AppState {
    user: User | null;
    theme: 'light' | 'dark';
    sidebarOpen: boolean;
    notifications: Notification[];
}

export interface User {
    id: string;
    name: string;
    email: string;
    role: string;
}

export interface Notification {
    id: string;
    message: string;
    type: 'info' | 'success' | 'warning' | 'error';
    timestamp: Date;
}

// stores/app-store.ts
import { AppState, User, Notification } from '../types/state';

class AppStore {
    private state: AppState = {
        user: null,
        theme: 'light',
        sidebarOpen: false,
        notifications: []
    };

    private listeners: Set<() => void> = new Set();

    getState(): AppState {
        return { ...this.state };
    }

    setState(updates: Partial<AppState>): void {
        this.state = { ...this.state, ...updates };
        this.notify();
    }

    setUser(user: User | null): void {
        this.setState({ user });
    }

    setTheme(theme: 'light' | 'dark'): void {
        this.setState({ theme });
        document.documentElement.setAttribute('data-theme', theme);
    }

    toggleSidebar(): void {
        this.setState({ sidebarOpen: !this.state.sidebarOpen });
    }

    addNotification(notification: Omit<Notification, 'id' | 'timestamp'>): void {
        const newNotification: Notification = {
            ...notification,
            id: Date.now().toString(),
            timestamp: new Date()
        };
        
        this.setState({
            notifications: [...this.state.notifications, newNotification]
        });

        // Auto remove na 5 seconden
        setTimeout(() => {
            this.removeNotification(newNotification.id);
        }, 5000);
    }

    removeNotification(id: string): void {
        this.setState({
            notifications: this.state.notifications.filter(n => n.id !== id)
        });
    }

    subscribe(listener: () => void): () => void {
        this.listeners.add(listener);
        return () => this.listeners.delete(listener);
    }

    private notify(): void {
        this.listeners.forEach(listener => listener());
    }
}

export const appStore = new AppStore();

// Alpine.js integration
document.addEventListener('alpine:init', () => {
    Alpine.store('app', {
        user: appStore.getState().user,
        theme: appStore.getState().theme,
        sidebarOpen: appStore.getState().sidebarOpen,
        notifications: appStore.getState().notifications,

        setUser(user: User | null) {
            appStore.setUser(user);
            this.user = user;
        },

        setTheme(theme: 'light' | 'dark') {
            appStore.setTheme(theme);
            this.theme = theme;
        },

        toggleSidebar() {
            appStore.toggleSidebar();
            this.sidebarOpen = !this.sidebarOpen;
        },

        addNotification(notification: Omit<Notification, 'id' | 'timestamp'>) {
            appStore.addNotification(notification);
            this.notifications = appStore.getState().notifications;
        }
    });
});
```

## 🧩 Component Patterns met Alpine.js

### 1. Alpine.js Component
```html
<!-- components/content-editor.html -->
<div x-data="contentEditor({
    content: '',
    autoSave: true,
    autoSaveDelay: 2000
})" 
     x-init="init()"
     class="content-editor">

    <div class="toolbar">
        <template x-for="command in commands" :key="command.action">
            <button @click="executeCommand(command.action)"
                    :class="{ 'active': command.isActive }"
                    :disabled="!command.isEnabled"
                    x-text="command.label"
                    class="toolbar-button">
            </button>
        </template>
    </div>

    <div class="editor-wrapper">
        <div @input="handleInput"
             @keydown="handleKeydown"
             contenteditable="true"
             class="editor"
             x-text="content"></div>
        
        <div x-show="isSaving" class="saving-indicator">
            Saving...
        </div>
        
        <div x-show="lastSaved" class="last-saved">
            Last saved: <span x-text="lastSaved"></span>
        </div>
    </div>
</div>

<script>
function contentEditor(options) {
    return {
        content: options.content || '',
        autoSave: options.autoSave || false,
        autoSaveDelay: options.autoSaveDelay || 2000,
        isSaving: false,
        lastSaved: null,
        saveTimeout: null,

        commands: [
            { action: 'bold', label: 'B', isActive: false, isEnabled: true },
            { action: 'italic', label: 'I', isActive: false, isEnabled: true },
            { action: 'underline', label: 'U', isActive: false, isEnabled: true }
        ],

        init() {
            this.setupAutoSave();
            this.updateCommandStates();
        },

        setupAutoSave() {
            if (this.autoSave) {
                this.$watch('content', () => {
                    clearTimeout(this.saveTimeout);
                    this.saveTimeout = setTimeout(() => {
                        this.save();
                    }, this.autoSaveDelay);
                });
            }
        },

        handleInput(event) {
            this.content = event.target.innerText;
            this.updateCommandStates();
        },

        handleKeydown(event) {
            // Auto-save shortcuts
            if (event.ctrlKey && event.key === 's') {
                event.preventDefault();
                this.save();
            }
        },

        executeCommand(command) {
            document.execCommand(command, false, null);
            this.updateCommandStates();
            this.$el.querySelector('.editor').focus();
        },

        updateCommandStates() {
            this.commands.forEach(command => {
                command.isActive = document.queryCommandState(command.action);
            });
        },

        async save() {
            if (!this.autoSave) return;
            
            this.isSaving = true;
            
            try {
                const response = await fetch('/api/content/save', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        content: this.content,
                        timestamp: new Date().toISOString()
                    })
                });

                if (response.ok) {
                    this.lastSaved = new Date().toLocaleTimeString();
                    this.$dispatch('content-saved', { content: this.content });
                }
            } catch (error) {
                console.error('Save failed:', error);
                this.$dispatch('content-save-error', { error });
            } finally {
                this.isSaving = false;
            }
        }
    }
}
</script>
```

### 2. TypeScript Form Validator met Alpine.js
```typescript
// types/forms.ts
export interface ValidationRule {
    validate: (value: string) => boolean;
    message: string;
}

export interface FieldRules {
    [fieldName: string]: ValidationRule[];
}

export interface FormState {
    data: Record<string, any>;
    errors: Record<string, string>;
    touched: Record<string, boolean>;
    isValid: boolean;
    isSubmitting: boolean;
}

// components/form-validator.ts
export class FormValidator {
    private form: HTMLFormElement;
    private rules: FieldRules;
    private state: FormState;

    constructor(form: HTMLFormElement, rules: FieldRules = {}) {
        this.form = form;
        this.rules = rules;
        this.state = {
            data: this.getFormData(),
            errors: {},
            touched: {},
            isValid: false,
            isSubmitting: false
        };

        this.init();
    }

    private init(): void {
        this.bindEvents();
        this.validate();
    }

    private getFormData(): Record<string, any> {
        const formData = new FormData(this.form);
        const data: Record<string, any> = {};
        
        formData.forEach((value, key) => {
            data[key] = value;
        });
        
        return data;
    }

    private bindEvents(): void {
        // Real-time validation
        this.form.addEventListener('input', (e: Event) => {
            const target = e.target as HTMLInputElement;
            if (target.matches('input, textarea, select')) {
                this.updateField(target.name, target.value);
                this.markTouched(target.name);
                this.validateField(target.name);
            }
        });

        // Submit handling
        this.form.addEventListener('submit', async (e: Event) => {
            e.preventDefault();
            
            if (this.validate()) {
                await this.submit();
            }
        });
    }

    private updateField(name: string, value: any): void {
        this.state.data[name] = value;
    }

    private markTouched(name: string): void {
        this.state.touched[name] = true;
    }

    private validateField(fieldName: string): boolean {
        const value = this.state.data[fieldName];
        const fieldRules = this.rules[fieldName] || [];
        
        let isValid = true;
        
        for (const rule of fieldRules) {
            if (!rule.validate(String(value))) {
                this.state.errors[fieldName] = rule.message;
                isValid = false;
                break;
            }
        }

        if (isValid) {
            delete this.state.errors[fieldName];
        }

        this.updateFieldUI(fieldName);
        return isValid;
    }

    private validate(): boolean {
        let isValid = true;
        
        Object.keys(this.rules).forEach(fieldName => {
            const fieldValid = this.validateField(fieldName);
            if (!fieldValid) {
                isValid = false;
            }
        });

        this.state.isValid = isValid;
        return isValid;
    }

    private updateFieldUI(fieldName: string): void {
        const field = this.form.querySelector(`[name="${fieldName}"]`) as HTMLElement;
        if (!field) return;

        const hasError = !!this.state.errors[fieldName];
        const isTouched = this.state.touched[fieldName];

        field.classList.toggle('error', hasError && isTouched);
        field.classList.toggle('valid', !hasError && isTouched);

        // Update error message
        let errorElement = field.parentNode?.querySelector('.error-message') as HTMLElement;
        
        if (hasError && isTouched) {
            if (!errorElement) {
                errorElement = document.createElement('div');
                errorElement.className = 'error-message';
                field.parentNode?.appendChild(errorElement);
            }
            errorElement.textContent = this.state.errors[fieldName];
        } else if (errorElement) {
            errorElement.remove();
        }
    }

    private async submit(): Promise<void> {
        this.state.isSubmitting = true;
        this.updateSubmitUI();

        try {
            const response = await fetch(this.form.action, {
                method: this.form.method,
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(this.state.data)
            });

            if (response.ok) {
                this.form.dispatchEvent(new CustomEvent('form:success', {
                    detail: await response.json()
                }));
                this.reset();
            } else {
                throw new Error('Form submission failed');
            }
        } catch (error) {
            this.form.dispatchEvent(new CustomEvent('form:error', {
                detail: { error }
            }));
        } finally {
            this.state.isSubmitting = false;
            this.updateSubmitUI();
        }
    }

    private updateSubmitUI(): void {
        const submitButton = this.form.querySelector('button[type="submit"]') as HTMLButtonElement;
        if (submitButton) {
            submitButton.disabled = this.state.isSubmitting;
            submitButton.textContent = this.state.isSubmitting ? 'Submitting...' : 'Submit';
        }
    }

    private reset(): void {
        this.form.reset();
        this.state.data = this.getFormData();
        this.state.errors = {};
        this.state.touched = {};
        this.state.isValid = false;
    }
}

// Validation rules
export const validationRules = {
    required: {
        validate: (value: string) => value.trim().length > 0,
        message: 'This field is required'
    },
    email: {
        validate: (value: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
        message: 'Please enter a valid email address'
    },
    minLength: (min: number) => ({
        validate: (value: string) => value.length >= min,
        message: `Must be at least ${min} characters long`
    }),
    pattern: (regex: RegExp, message: string) => ({
        validate: (value: string) => regex.test(value),
        message
    })
};
```

## 🔄 Async Patterns met TypeScript

### 1. Typed API Client
```typescript
// types/api.ts
export interface ApiResponse<T> {
    data: T;
    status: number;
    message?: string;
}

export interface PaginatedResponse<T> {
    data: T[];
    pagination: {
        page: number;
        limit: number;
        total: number;
        totalPages: number;
    };
}

export interface ContentItem {
    id: string;
    title: string;
    slug: string;
    content: string;
    category: string;
    publishedAt: string;
    updatedAt: string;
}

export interface SearchParams {
    query?: string;
    category?: string;
    page?: number;
    limit?: number;
    sortBy?: string;
    sortOrder?: 'asc' | 'desc';
}

// utils/api-client.ts
export class ApiClient {
    private baseURL: string;
    private defaultHeaders: Record<string, string>;

    constructor(baseURL: string = '/api') {
        this.baseURL = baseURL;
        this.defaultHeaders = {
            'Content-Type': 'application/json',
            'X-Requested-With': 'XMLHttpRequest'
        };
    }

    private async request<T>(
        endpoint: string, 
        options: RequestInit = {}
    ): Promise<ApiResponse<T>> {
        const url = `${this.baseURL}${endpoint}`;
        const config: RequestInit = {
            headers: { ...this.defaultHeaders, ...options.headers },
            ...options
        };

        try {
            const response = await fetch(url, config);

            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }

            const contentType = response.headers.get('content-type');
            if (contentType && contentType.includes('application/json')) {
                const data = await response.json();
                return {
                    data,
                    status: response.status,
                    message: data.message
                };
            }

            const text = await response.text();
            return {
                data: text as unknown as T,
                status: response.status
            };
        } catch (error) {
            console.error('API request failed:', error);
            throw error;
        }
    }

    async get<T>(endpoint: string, params: Record<string, any> = {}): Promise<ApiResponse<T>> {
        const url = new URL(`${this.baseURL}${endpoint}`);
        
        Object.keys(params).forEach(key => {
            const value = params[key];
            if (value !== null && value !== undefined) {
                url.searchParams.append(key, String(value));
            }
        });

        return this.request<T>(url.pathname + url.search);
    }

    async post<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
        return this.request<T>(endpoint, {
            method: 'POST',
            body: JSON.stringify(data)
        });
    }

    async put<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
        return this.request<T>(endpoint, {
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }

    async delete<T>(endpoint: string): Promise<ApiResponse<T>> {
        return this.request<T>(endpoint, {
            method: 'DELETE'
        });
    }

    // Content-specific methods
    async getContent(params: SearchParams = {}): Promise<ApiResponse<PaginatedResponse<ContentItem>>> {
        return this.get<PaginatedResponse<ContentItem>>('/content', params);
    }

    async getContentById(id: string): Promise<ApiResponse<ContentItem>> {
        return this.get<ContentItem>(`/content/${id}`);
    }

    async createContent(data: Partial<ContentItem>): Promise<ApiResponse<ContentItem>> {
        return this.post<ContentItem>('/content', data);
    }

    async updateContent(id: string, data: Partial<ContentItem>): Promise<ApiResponse<ContentItem>> {
        return this.put<ContentItem>(`/content/${id}`, data);
    }

    async deleteContent(id: string): Promise<ApiResponse<void>> {
        return this.delete<void>(`/content/${id}`);
    }
}

// Usage
const api = new ApiClient();

// Get content with TypeScript typing
const contentResponse = await api.getContent({
    category: 'news',
    page: 1,
    limit: 10,
    sortBy: 'publishedAt',
    sortOrder: 'desc'
});

const contentItems = contentResponse.data.data;
const pagination = contentResponse.data.pagination;
```

### 2. Alpine.js Search Component met TypeScript
```typescript
// types/search.ts
export interface SearchResult {
    id: string;
    title: string;
    excerpt: string;
    url: string;
    category: string;
    score: number;
}

export interface SearchState {
    query: string;
    results: SearchResult[];
    loading: boolean;
    error: string | null;
    selectedIndex: number;
}

// components/search-manager.ts
export class SearchManager {
    private input: HTMLInputElement;
    private resultsContainer: HTMLElement;
    private state: SearchState;
    private debounceTimeout: number | null = null;
    private abortController: AbortController | null = null;

    constructor(input: HTMLInputElement, resultsContainer: HTMLElement) {
        this.input = input;
        this.resultsContainer = resultsContainer;
        
        this.state = {
            query: '',
            results: [],
            loading: false,
            error: null,
            selectedIndex: -1
        };

        this.init();
    }

    private init(): void {
        this.bindEvents();
        this.setupAlpineIntegration();
    }

    private bindEvents(): void {
        this.input.addEventListener('input', (e: Event) => {
            const target = e.target as HTMLInputElement;
            this.handleInput(target.value);
        });

        this.input.addEventListener('keydown', (e: KeyboardEvent) => {
            this.handleKeydown(e);
        });

        // Close on click outside
        document.addEventListener('click', (e: Event) => {
            if (!this.input.contains(e.target as Node) && 
                !this.resultsContainer.contains(e.target as Node)) {
                this.clearResults();
            }
        });
    }

    private handleInput(query: string): void {
        this.state.query = query;
        
        // Cancel previous search
        if (this.abortController) {
            this.abortController.abort();
        }

        // Debounce
        if (this.debounceTimeout) {
            clearTimeout(this.debounceTimeout);
        }

        if (query.length < 2) {
            this.clearResults();
            return;
        }

        this.debounceTimeout = window.setTimeout(() => {
            this.performSearch(query);
        }, 300);
    }

    private handleKeydown(e: KeyboardEvent): void {
        const results = this.state.results;
        
        switch (e.key) {
            case 'ArrowDown':
                e.preventDefault();
                this.state.selectedIndex = Math.min(this.state.selectedIndex + 1, results.length - 1);
                this.updateSelection();
                break;
                
            case 'ArrowUp':
                e.preventDefault();
                this.state.selectedIndex = Math.max(this.state.selectedIndex - 1, -1);
                this.updateSelection();
                break;
                
            case 'Enter':
                e.preventDefault();
                if (this.state.selectedIndex >= 0) {
                    this.selectResult(this.state.selectedIndex);
                }
                break;
                
            case 'Escape':
                this.clearResults();
                break;
        }
    }

    private async performSearch(query: string): Promise<void> {
        this.state.loading = true;
        this.state.error = null;
        this.state.results = [];
        this.state.selectedIndex = -1;
        
        this.updateUI();

        this.abortController = new AbortController();

        try {
            const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`, {
                signal: this.abortController.signal
            });

            if (!response.ok) {
                throw new Error('Search failed');
            }

            const data = await response.json();
            this.state.results = data.results || [];
        } catch (error) {
            if (error instanceof Error && error.name !== 'AbortError') {
                this.state.error = error.message;
            }
        } finally {
            this.state.loading = false;
            this.abortController = null;
            this.updateUI();
        }
    }

    private updateSelection(): void {
        const items = this.resultsContainer.querySelectorAll('.search-result');
        items.forEach((item, index) => {
            item.classList.toggle('selected', index === this.state.selectedIndex);
        });
    }

    private selectResult(index: number): void {
        const result = this.state.results[index];
        if (result) {
            window.location.href = result.url;
        }
    }

    private updateUI(): void {
        if (this.state.loading) {
            this.resultsContainer.innerHTML = '<div class="search-loading">Searching...</div>';
            return;
        }

        if (this.state.error) {
            this.resultsContainer.innerHTML = `<div class="search-error">${this.escapeHtml(this.state.error)}</div>`;
            return;
        }

        if (this.state.results.length === 0) {
            this.resultsContainer.innerHTML = '<div class="search-empty">No results found</div>';
            return;
        }

        this.resultsContainer.innerHTML = this.state.results.map((result, index) => `
            <div class="search-result ${index === this.state.selectedIndex ? 'selected' : ''}" 
                 data-index="${index}"
                 data-url="${result.url}">
                <h3>${this.escapeHtml(result.title)}</h3>
                <p>${this.escapeHtml(result.excerpt)}</p>
                <small>${result.category}</small>
            </div>
        `).join('');

        // Bind click events
        this.resultsContainer.querySelectorAll('.search-result').forEach((item, index) => {
            item.addEventListener('click', () => this.selectResult(index));
        });
    }

    private clearResults(): void {
        this.state.results = [];
        this.state.selectedIndex = -1;
        this.resultsContainer.innerHTML = '';
    }

    private setupAlpineIntegration(): void {
        // Maak state beschikbaar voor Alpine.js
        if (window.Alpine) {
            window.Alpine.effect(() => {
                window.Alpine.store('search', {
                    query: this.state.query,
                    results: this.state.results,
                    loading: this.state.loading,
                    error: this.state.error
                });
            });
        }
    }

    private escapeHtml(text: string): string {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }
}

// Alpine.js search component
document.addEventListener('alpine:init', () => {
    Alpine.data('search', (initialQuery = '') => ({
        query: initialQuery,
        results: [],
        loading: false,
        error: null,
        selectedIndex: -1,

        init() {
            this.$watch('query', (value) => {
                if (value.length >= 2) {
                    this.search(value);
                } else {
                    this.clearResults();
                }
            });
        },

        async search(query: string) {
            this.loading = true;
            this.error = null;

            try {
                const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
                const data = await response.json();
                this.results = data.results || [];
            } catch (error) {
                this.error = 'Search failed';
                this.results = [];
            } finally {
                this.loading = false;
            }
        },

        clearResults() {
            this.results = [];
            this.error = null;
            this.selectedIndex = -1;
        },

        selectResult(result: any) {
            window.location.href = result.url;
        }
    }));
});
```

## 📋 Best Practices Checklist

- [ ] Gebruik TypeScript voor type safety
- [ ] Implementeer Alpine.js voor reactive UI
- [ ] Gebruik progressive enhancement
- [ ] Implementeer proper error handling met try-catch
- [ ] Gebruik modern TypeScript features (generics, interfaces)
- [ ] Implementeer accessibility features (ARIA, keyboard navigation)
- [ ] Test met TypeScript compiler
- [ ] Gebruik Alpine.js directives (x-data, x-init, x-show, etc.)
- [ ] Implementeer proper state management
- [ ] Test TypeScript en Alpine.js functionaliteit

## 🚀 Performance Tips

1. **Use Alpine.js** voor lightweight reactivity
2. **Implement lazy loading** voor images en content
3. **Use TypeScript generics** voor type-safe API calls
4. **Debounce user input** voor search en forms
5. **Use Web Workers** voor heavy computations
6. **Implement proper caching** strategies
7. **Minimize DOM manipulation** met Alpine.js
8. **Use TypeScript tree-shaking** voor kleinere bundles
9. **Implement virtual scrolling** voor grote lijsten
10. **Use Alpine.js morphing** voor smooth transitions
