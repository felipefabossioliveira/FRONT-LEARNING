# 🅰️ GUIA COMPLETO ANGULAR - PREPARAÇÃO SENIOR 2025

**Versão:** Angular 18+ (compatível com 14-18)  
**Objetivo:** Preparação completa para entrevistas técnicas de nível Senior  
**Foco:** Conceitos essenciais + Boas práticas + Comparação de versões + Arquitetura

---

## 📚 ÍNDICE

1. [Components & Data Binding](#1-components)
2. [Comunicação entre Components](#2-comunicacao)
3. [Lifecycle Hooks](#3-lifecycle)
4. [Services & Dependency Injection](#4-services)
5. [RxJS & Observables](#5-rxjs)
6. [Pipes (Built-in & Custom)](#6-pipes)
7. [Directives Customizadas](#7-directives)
8. [Forms (Reactive & Template-Driven)](#8-forms)
9. [Routing Avançado](#9-routing)
10. [Change Detection & OnPush](#10-change-detection)
11. [Signals (Angular 16+)](#11-signals)
12. [Standalone Components](#12-standalone)
13. [HTTP & Interceptors](#13-http)
14. [Testing (Jasmine/Jest)](#14-testing)
15. [Performance Optimization](#15-performance)
16. [Content Projection](#16-content-projection)
17. [ViewChild & ContentChild](#17-viewchild)
18. [Dynamic Components](#18-dynamic-components)
19. [Animations](#19-animations)
20. [SSR (Server-Side Rendering)](#20-ssr)
21. [PWA (Progressive Web Apps)](#21-pwa)
22. [State Management](#22-state-management)
23. [Security](#23-security)
24. [Internacionalização (i18n)](#24-i18n)
25. [Boas Práticas & Design Patterns](#25-boas-praticas)
26. [Arquitetura de Projetos Grandes](#26-arquitetura)
27. [Comparação de Versões do Angular](#27-comparacao-versoes)
28. [Dicas para Entrevista Senior](#28-dicas-entrevista)

---

<a name="1-components"></a>
## 1. COMPONENTS & DATA BINDING

### 1.1 Anatomia de um Component

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-user',              // <app-user></app-user>
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush  // Performance!
})
export class UserComponent {
  // Propriedades
  name = 'João Silva';
  age = 30;
  isActive = true;
  
  // Computed property (getter)
  get displayName(): string {
    return `${this.name} (${this.age} anos)`;
  }
  
  // Métodos
  incrementAge(): void {
    this.age++;
  }
}
```

### 1.2 Component Metadata - Todas as Opções

```typescript
@Component({
  selector: 'app-user',                          // CSS selector
  templateUrl: './user.component.html',           // Template externo
  // template: `<div>Inline</div>`,              // OU inline
  styleUrls: ['./user.component.scss'],           // Estilos externos
  // styles: [`div { color: red; }`],            // OU inline
  changeDetection: ChangeDetectionStrategy.OnPush,// Estratégia CD
  encapsulation: ViewEncapsulation.Emulated,      // Default - scoped CSS
  // ViewEncapsulation.None    → CSS global
  // ViewEncapsulation.ShadowDom → Shadow DOM real
  providers: [LocalService],                      // DI local
  animations: [fadeInOut],                        // Animações
  host: {                                         // Host bindings
    '[class.active]': 'isActive',
    '(click)': 'onClick()',
    'role': 'button'
  },
  exportAs: 'appUser'                             // Permite #ref="appUser"
})
```

**Quando usar cada ViewEncapsulation:**
- `Emulated` (default) → 99% dos casos, isola CSS por component
- `None` → Estilos globais, útil para temas ou override de libs
- `ShadowDom` → Encapsulamento real do browser, limitação: não funciona em IE

### 1.3 Data Binding - 4 Tipos

| Tipo | Sintaxe | Direção | Exemplo | Quando Usar |
|------|---------|---------|---------|-------------|
| **Interpolação** | `{{ }}` | Component → View | `{{ name }}` | Exibir valores |
| **Property Binding** | `[property]` | Component → View | `[disabled]="isLoading"` | Bind de propriedades |
| **Event Binding** | `(event)` | View → Component | `(click)="save()"` | Escutar eventos |
| **Two-Way Binding** | `[(ngModel)]` | Ambas direções | `[(ngModel)]="name"` | Forms simples |

**Por baixo dos panos:** Two-way binding é açúcar sintático:
```html
<!-- Isto: -->
<input [(ngModel)]="name">

<!-- É equivalente a: -->
<input [ngModel]="name" (ngModelChange)="name = $event">
```

### 1.4 Property Binding - Exemplos

```html
<!-- Propriedades HTML padrão -->
<input [value]="username">
<img [src]="imageUrl" [alt]="imageAlt">
<button [disabled]="isLoading">Salvar</button>

<!-- Atributos (quando não há property correspondente) -->
<div [attr.data-id]="userId"></div>
<div [attr.aria-label]="description"></div>
<td [attr.colspan]="colSpan"></td>

<!-- Classes CSS -->
<div [class.active]="isActive"></div>
<div [ngClass]="{'active': isActive, 'disabled': !isEnabled}"></div>
<div [ngClass]="classArray"></div>   <!-- ['class1', 'class2'] -->

<!-- Estilos -->
<div [style.color]="textColor"></div>
<div [style.font-size.px]="fontSize"></div>
<div [ngStyle]="{'color': textColor, 'font-size.px': fontSize}"></div>
```

**Quando usar `[class.x]` vs `[ngClass]`:**
- `[class.active]="isActive"` → Toggle de UMA classe
- `[ngClass]="{...}"` → Múltiplas classes condicionais

**Property vs Attribute binding (pergunta de entrevista):**
```html
<!-- Property: altera a propriedade DOM do elemento -->
<input [value]="name">  <!-- input.value = name -->

<!-- Attribute: altera o atributo HTML -->
<input [attr.value]="name">  <!-- input.setAttribute('value', name) -->

<!-- Diferença prática: colspan não tem property DOM -->
<td [attr.colspan]="2"></td>  <!-- ✅ Funciona -->
<td [colspan]="2"></td>       <!-- ❌ Erro -->
```

### 1.5 Diretivas Estruturais

#### **\*ngIf**

```html
<!-- Simples -->
<div *ngIf="isLoggedIn">Bem-vindo!</div>

<!-- Com else -->
<div *ngIf="user; else loginTemplate">
  {{ user.name }}
</div>
<ng-template #loginTemplate>
  <button>Login</button>
</ng-template>

<!-- Com then/else -->
<div *ngIf="isLoading; then loadingTpl; else contentTpl"></div>
<ng-template #loadingTpl><spinner></spinner></ng-template>
<ng-template #contentTpl><app-content></app-content></ng-template>

<!-- Com variável local (evita async pipe duplo) -->
<div *ngIf="user$ | async as user">
  {{ user.name }}
  {{ user.email }}
</div>
```

#### **\*ngFor**

```html
<!-- Loop básico -->
<div *ngFor="let item of items">{{ item }}</div>

<!-- Com index e variáveis -->
<div *ngFor="let item of items; 
             let i = index;
             let first = first;
             let last = last;
             let even = even;
             let odd = odd;
             let count = count">
  <span [class.highlight]="first">{{ i }}. {{ item }}</span>
  <span *ngIf="last"> (Total: {{ count }})</span>
</div>

<!-- ⚠️ CRÍTICO: trackBy para performance -->
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>
```

```typescript
// No component
trackByUserId(index: number, user: User): number {
  return user.id;  // Angular usa ID, não recria tudo
}
```

**Por que trackBy é ESSENCIAL?**
```
Sem trackBy:
- Lista de 1000 usuários renderizada
- Adiciona 1 usuário
- Angular recria 1001 elementos DOM ❌

Com trackBy:
- Lista de 1000 usuários renderizada
- Adiciona 1 usuário
- Angular cria apenas 1 novo elemento DOM ✅
```

#### **\*ngSwitch**

```html
<div [ngSwitch]="status">
  <div *ngSwitchCase="'loading'">Carregando...</div>
  <div *ngSwitchCase="'success'">Sucesso!</div>
  <div *ngSwitchCase="'error'">Erro</div>
  <div *ngSwitchDefault>Desconhecido</div>
</div>
```

#### **Novo Control Flow (Angular 17+) - IMPORTANTE**

```html
<!-- @if substitui *ngIf -->
@if (user) {
  <div>{{ user.name }}</div>
} @else if (isLoading) {
  <spinner />
} @else {
  <div>Login</div>
}

<!-- @for substitui *ngFor (track é OBRIGATÓRIO) -->
@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
} @empty {
  <div>Nenhum item encontrado</div>
}

<!-- @switch substitui ngSwitch -->
@switch (status) {
  @case ('loading') { <spinner /> }
  @case ('error') { <error-message /> }
  @default { <app-content /> }
}
```

**Vantagens do novo control flow:**
- Syntax mais limpa e legível
- `@empty` block no @for (antes era manual)
- `track` obrigatório (força boa prática)
- Performance melhor (compilado diferente)
- Não precisa importar CommonModule

### 1.6 ng-container - Wrapper sem DOM

```html
<!-- ❌ Cria <div> desnecessária -->
<div *ngIf="show">
  <p>Item 1</p>
  <p>Item 2</p>
</div>

<!-- ✅ Não cria elemento extra -->
<ng-container *ngIf="show">
  <p>Item 1</p>
  <p>Item 2</p>
</ng-container>

<!-- ✅ Combinar múltiplas diretivas (não pode ter 2 structural directives no mesmo elemento) -->
<ng-container *ngIf="showList">
  <div *ngFor="let item of items">{{ item }}</div>
</ng-container>
```

### 1.7 ng-template - Template Reutilizável

```html
<!-- Definir template -->
<ng-template #myTemplate let-name let-age="age">
  <div>Nome: {{ name }}, Idade: {{ age }}</div>
</ng-template>

<!-- Usar com ngTemplateOutlet -->
<ng-container *ngTemplateOutlet="myTemplate; context: { $implicit: 'João', age: 30 }">
</ng-container>

<!-- Pattern: Template condicional reutilizável -->
<ng-container *ngTemplateOutlet="isAdmin ? adminTpl : userTpl; context: { $implicit: user }">
</ng-container>

<ng-template #adminTpl let-user>
  <admin-panel [user]="user"></admin-panel>
</ng-template>

<ng-template #userTpl let-user>
  <user-dashboard [user]="user"></user-dashboard>
</ng-template>
```

---

<a name="2-comunicacao"></a>
## 2. COMUNICAÇÃO ENTRE COMPONENTS

### 2.1 @Input - Pai → Filho

```typescript
// FILHO: user-card.component.ts
export class UserCardComponent {
  @Input() user!: User;              // Obrigatório (! = definite assignment)
  @Input() showEmail = true;         // Opcional com default
  @Input('userName') name?: string;  // Com alias
  
  // Angular 16+: required input
  @Input({ required: true }) userId!: number;
  
  // Angular 16+: transform
  @Input({ transform: booleanAttribute }) disabled = false;
  @Input({ transform: numberAttribute }) size = 16;
}
```

```html
<!-- PAI -->
<app-user-card 
  [user]="currentUser" 
  [showEmail]="false"
  [userId]="42"
  disabled>  <!-- booleanAttribute converte string vazia para true -->
</app-user-card>
```

**Input com setter (reagir a mudanças):**
```typescript
export class UserCardComponent {
  private _user!: User;
  
  @Input()
  set user(value: User) {
    this._user = value;
    this.processUser(value);  // Lógica quando Input muda
  }
  get user(): User { return this._user; }
}
```

### 2.2 @Output - Filho → Pai

```typescript
// FILHO
export class UserCardComponent {
  @Input() user!: User;
  @Output() deleted = new EventEmitter<number>();
  @Output() edited = new EventEmitter<User>();
  @Output('userSelected') selected = new EventEmitter<User>(); // Com alias
  
  onDelete() {
    this.deleted.emit(this.user.id);
  }
  
  onEdit() {
    this.edited.emit({ ...this.user, name: 'Editado' });
  }
}
```

```html
<!-- PAI -->
<app-user-card 
  [user]="user"
  (deleted)="handleDelete($event)"
  (edited)="handleEdit($event)"
  (userSelected)="handleSelect($event)">
</app-user-card>
```

### 2.3 Service Compartilhado (Siblings / Não relacionados)

```typescript
// Quando components NÃO têm relação pai-filho
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private notificationSubject = new Subject<Notification>();
  notifications$ = this.notificationSubject.asObservable();
  
  send(notification: Notification): void {
    this.notificationSubject.next(notification);
  }
}

// Component A (envia)
export class HeaderComponent {
  constructor(private notificationService: NotificationService) {}
  
  notify() {
    this.notificationService.send({ type: 'success', message: 'Salvo!' });
  }
}

// Component B (recebe)
export class ToastComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  notifications: Notification[] = [];
  
  constructor(private notificationService: NotificationService) {}
  
  ngOnInit() {
    this.notificationService.notifications$.pipe(
      takeUntil(this.destroy$)
    ).subscribe(n => this.notifications.push(n));
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### 2.4 Pattern: Smart vs Dumb Components

| Aspecto | Dumb (Presentational) | Smart (Container) |
|---------|----------------------|-------------------|
| **Responsabilidade** | Apenas UI | Lógica de negócio |
| **Dados** | Recebe via @Input | Chama services |
| **Comunicação** | Emite @Output | Orquestra components |
| **Services** | NÃO usa | Injeta e usa |
| **Reutilizável** | SIM | Geralmente não |
| **Testável** | Muito fácil | Mais complexo |
| **Change Detection** | OnPush (sempre) | Default ou OnPush |

**Exemplo Completo:**

```typescript
// ========== DUMB - Só apresenta dados ==========
@Component({
  selector: 'app-product-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="card">
      <img [src]="product.image" [alt]="product.name">
      <h3>{{ product.name }}</h3>
      <p>{{ product.price | currency:'BRL' }}</p>
      <button (click)="addToCart.emit(product)">Comprar</button>
    </div>
  `
})
export class ProductCardComponent {
  @Input({ required: true }) product!: Product;
  @Output() addToCart = new EventEmitter<Product>();
}

// ========== SMART - Orquestra lógica ==========
@Component({
  selector: 'app-product-list',
  template: `
    <div class="filters">
      <input (input)="onSearch($event)">
    </div>
    <app-product-card 
      *ngFor="let product of filteredProducts$ | async; trackBy: trackById"
      [product]="product"
      (addToCart)="onAddToCart($event)">
    </app-product-card>
    <app-loading *ngIf="loading$ | async"></app-loading>
    <app-empty-state *ngIf="(filteredProducts$ | async)?.length === 0"></app-empty-state>
  `
})
export class ProductListComponent {
  private searchTerm$ = new BehaviorSubject<string>('');
  loading$ = new BehaviorSubject<boolean>(true);
  
  filteredProducts$ = combineLatest([
    this.productService.getProducts().pipe(
      tap(() => this.loading$.next(false))
    ),
    this.searchTerm$
  ]).pipe(
    map(([products, term]) => 
      products.filter(p => p.name.toLowerCase().includes(term.toLowerCase()))
    )
  );
  
  constructor(
    private productService: ProductService,
    private cartService: CartService,
    private snackbar: MatSnackBar
  ) {}
  
  onSearch(event: Event) {
    this.searchTerm$.next((event.target as HTMLInputElement).value);
  }
  
  onAddToCart(product: Product) {
    this.cartService.add(product).subscribe({
      next: () => this.snackbar.open('Adicionado ao carrinho!'),
      error: (err) => this.snackbar.open('Erro ao adicionar')
    });
  }
  
  trackById(index: number, product: Product): number {
    return product.id;
  }
}
```

---

<a name="3-lifecycle"></a>
## 3. LIFECYCLE HOOKS

### 3.1 Ordem de Execução

| Hook | Quando Executa | Uso Típico |
|------|---------------|------------|
| `constructor` | 1º - Injeção de dependências | Inicializar propriedades |
| `ngOnChanges` | 2º - Quando @Input muda | Reagir a mudanças de Input |
| `ngOnInit` | 3º - Após 1º ngOnChanges | **Setup inicial, HTTP calls** |
| `ngDoCheck` | A cada change detection | Detecção customizada (raro) |
| `ngAfterContentInit` | Após content projection | Acessa @ContentChild |
| `ngAfterContentChecked` | Após checar content | Raramente usado |
| `ngAfterViewInit` | Após renderizar view | **Acessa @ViewChild** |
| `ngAfterViewChecked` | Após checar view | Raramente usado |
| `ngOnDestroy` | Antes de destruir component | **Cleanup (unsubscribe)** |

**Fluxo visual:**
```
constructor → ngOnChanges → ngOnInit → ngDoCheck
                                          ↓
ngAfterContentInit → ngAfterContentChecked → ngAfterViewInit → ngAfterViewChecked
                                                                      ↓
                         (repete a cada CD cycle) ←←←←←←←←←←←←←←←←←←←

                                     ngOnDestroy (quando component é removido)
```

### 3.2 Exemplo Prático Completo

```typescript
export class UserComponent implements OnInit, OnChanges, AfterViewInit, OnDestroy {
  @Input() userId!: number;
  @ViewChild('nameInput') nameInput!: ElementRef;
  
  private destroy$ = new Subject<void>();
  user?: User;
  
  constructor(private userService: UserService) {
    // ❌ NÃO acessa @Input aqui (ainda undefined)
    // ❌ NÃO acessa @ViewChild aqui (DOM não existe)
    // ✅ SÓ injeção de dependências
  }
  
  ngOnChanges(changes: SimpleChanges) {
    // Dispara quando @Input muda
    if (changes['userId']) {
      const change = changes['userId'];
      console.log(`User ID mudou: ${change.previousValue} → ${change.currentValue}`);
      console.log(`É primeira mudança? ${change.firstChange}`);
      
      // Recarrega dados quando Input muda
      if (!change.firstChange) {
        this.loadUser(change.currentValue);
      }
    }
  }
  
  ngOnInit() {
    // @Input já disponível aqui
    // MELHOR lugar para HTTP calls iniciais
    this.loadUser(this.userId);
    this.setupRealtimeUpdates();
  }
  
  ngAfterViewInit() {
    // DOM renderizado, @ViewChild disponível
    this.nameInput.nativeElement.focus();
    // ⚠️ Cuidado: mudar propriedades aqui pode causar ExpressionChangedAfterItHasBeenCheckedError
  }
  
  ngOnDestroy() {
    // ESSENCIAL: limpar subscriptions, timers, event listeners
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  private loadUser(id: number) {
    this.userService.getUser(id).pipe(
      takeUntil(this.destroy$)
    ).subscribe(user => this.user = user);
  }
  
  private setupRealtimeUpdates() {
    this.userService.onUserUpdate().pipe(
      takeUntil(this.destroy$)
    ).subscribe(update => {
      // Atualiza em tempo real
    });
  }
}
```

### 3.3 Armadilhas Comuns

❌ **Erro 1: Acessar @Input no constructor**
```typescript
constructor() {
  console.log(this.userId); // undefined!
}
```

❌ **Erro 2: Não fazer unsubscribe**
```typescript
ngOnInit() {
  this.service.getData().subscribe(data => {
    this.data = data;
  });
  // Memory leak! Subscription não é cancelada quando component é destruído
}
```

❌ **Erro 3: Modificar estado no AfterViewInit sem detectChanges**
```typescript
ngAfterViewInit() {
  this.title = 'Novo'; // ExpressionChangedAfterItHasBeenCheckedError!
  
  // ✅ Correto:
  setTimeout(() => this.title = 'Novo'); // Ou
  this.cdr.detectChanges(); // Ou usar OnPush
}
```

❌ **Erro 4: HTTP call no constructor**
```typescript
constructor(private http: HttpClient) {
  this.http.get('/api/data').subscribe(); // ❌ Difícil de testar
}
// ✅ Sempre no ngOnInit
```

✅ **Padrões Corretos de Unsubscribe (4 formas):**

```typescript
// 1. takeUntil (mais comum)
private destroy$ = new Subject<void>();

ngOnInit() {
  this.service.getData().pipe(
    takeUntil(this.destroy$)
  ).subscribe(data => this.data = data);
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}

// 2. async pipe (preferido no template)
// Faz unsubscribe automático!
<div *ngIf="data$ | async as data">{{ data }}</div>

// 3. takeUntilDestroyed (Angular 16+) - MAIS ELEGANTE
constructor() {
  this.service.getData().pipe(
    takeUntilDestroyed()  // Sem boilerplate!
  ).subscribe(data => this.data = data);
}

// 4. DestroyRef (Angular 16+)
constructor() {
  const destroyRef = inject(DestroyRef);
  
  const sub = this.service.getData().subscribe(data => this.data = data);
  destroyRef.onDestroy(() => sub.unsubscribe());
}
```

---

<a name="4-services"></a>
## 4. SERVICES & DEPENDENCY INJECTION

### 4.1 Service Básico

```typescript
@Injectable({
  providedIn: 'root'  // ← Singleton global, tree-shakeable
})
export class UserService {
  private apiUrl = environment.apiUrl;
  
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.apiUrl}/users`).pipe(
      retry(2),
      catchError(this.handleError)
    );
  }
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/users/${id}`);
  }
  
  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(`${this.apiUrl}/users`, user);
  }
  
  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/users/${id}`, user);
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/users/${id}`);
  }
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    let message = 'Erro desconhecido';
    
    if (error.error instanceof ErrorEvent) {
      // Erro client-side (network, etc)
      message = error.error.message;
    } else {
      // Erro server-side
      switch (error.status) {
        case 400: message = 'Dados inválidos'; break;
        case 401: message = 'Não autorizado'; break;
        case 403: message = 'Sem permissão'; break;
        case 404: message = 'Não encontrado'; break;
        case 500: message = 'Erro interno do servidor'; break;
      }
    }
    
    return throwError(() => new Error(message));
  }
}
```

### 4.2 Níveis de Injeção (Scopes)

| Scope | Sintaxe | Quando Usar | Instâncias |
|-------|---------|-------------|------------|
| **Root** | `providedIn: 'root'` | 95% dos casos | 1 (Singleton) |
| **Module** | `providedIn: MyModule` | Feature isolada | 1 por módulo lazy |
| **Component** | `providers: [Service]` | Estado local | 1 por component |
| **Directive** | `providers: [Service]` | Estado por diretiva | 1 por diretiva |

```typescript
// Root (recomendado - tree-shakeable)
@Injectable({ providedIn: 'root' })
export class AuthService {}

// Module (carregado com lazy module)
@Injectable({ providedIn: AdminModule })
export class AdminService {}

// Component (nova instância por component)
@Component({
  providers: [FormStateService]  // Cada instância tem seu estado
})
export class UserFormComponent {
  constructor(private formState: FormStateService) {}
}
```

**Quando usar Component-level provider (pergunta de entrevista):**
- Formulário com estado local
- Dialog/Modal com dados isolados
- Componente reutilizável que precisa de estado independente
- Exemplo: dois `<app-cart>` na mesma tela, cada um com seu CartService

### 4.3 Injection Tokens & Factories

```typescript
// InjectionToken para valores não-classe
export const API_URL = new InjectionToken<string>('API_URL');
export const IS_PRODUCTION = new InjectionToken<boolean>('IS_PRODUCTION');

// Configuração de interfaces (não pode injetar interface diretamente)
export const LOGGER = new InjectionToken<Logger>('LOGGER');

export interface Logger {
  log(message: string): void;
  error(message: string): void;
}

// Provider com factory
providers: [
  { provide: API_URL, useValue: environment.apiUrl },
  { provide: IS_PRODUCTION, useValue: environment.production },
  { 
    provide: LOGGER, 
    useFactory: (isProd: boolean) => {
      return isProd ? new ProductionLogger() : new ConsoleLogger();
    },
    deps: [IS_PRODUCTION]
  }
]

// Uso
constructor(@Inject(API_URL) private apiUrl: string) {}
```

### 4.4 Tipos de Providers

```typescript
// useClass - instancia a classe
{ provide: Logger, useClass: ConsoleLogger }

// useExisting - alias para outro provider
{ provide: AbstractLogger, useExisting: ConsoleLogger }

// useValue - valor estático
{ provide: API_URL, useValue: 'https://api.example.com' }

// useFactory - lógica customizada
{ 
  provide: CacheService,
  useFactory: (http: HttpClient) => new CacheService(http, 300),
  deps: [HttpClient]
}
```

### 4.5 Cache com shareReplay

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private usersCache$?: Observable<User[]>;
  
  getUsers(forceRefresh = false): Observable<User[]> {
    if (!this.usersCache$ || forceRefresh) {
      this.usersCache$ = this.http.get<User[]>('/api/users').pipe(
        shareReplay({ 
          bufferSize: 1,      // Guarda último valor
          refCount: true      // Libera cache quando não há subscribers
        })
      );
    }
    return this.usersCache$;
  }
  
  // Cache com TTL (Time to Live)
  private cacheTimestamp = 0;
  private readonly CACHE_TTL = 60000; // 1 minuto
  
  getUsersWithTTL(): Observable<User[]> {
    const now = Date.now();
    if (!this.usersCache$ || (now - this.cacheTimestamp) > this.CACHE_TTL) {
      this.cacheTimestamp = now;
      this.usersCache$ = this.http.get<User[]>('/api/users').pipe(
        shareReplay(1)
      );
    }
    return this.usersCache$;
  }
  
  invalidateCache(): void {
    this.usersCache$ = undefined;
  }
}
```

**Benefícios do shareReplay:**
- ✅ Múltiplos subscribers = 1 única requisição HTTP
- ✅ Cache automático em memória
- ✅ `refCount: true` libera memória quando não usado

### 4.6 Facade Pattern (Padrão Avançado)

```typescript
// Facade: simplifica interface complexa
@Injectable({ providedIn: 'root' })
export class UserFacade {
  // Estado
  private loadingSubject = new BehaviorSubject<boolean>(false);
  private usersSubject = new BehaviorSubject<User[]>([]);
  private errorSubject = new BehaviorSubject<string | null>(null);
  
  // Selectores públicos (read-only)
  loading$ = this.loadingSubject.asObservable();
  users$ = this.usersSubject.asObservable();
  error$ = this.errorSubject.asObservable();
  
  constructor(
    private userApi: UserApiService,
    private notificationService: NotificationService
  ) {}
  
  // Ações (o component chama isso)
  loadUsers(): void {
    this.loadingSubject.next(true);
    this.errorSubject.next(null);
    
    this.userApi.getUsers().pipe(
      finalize(() => this.loadingSubject.next(false))
    ).subscribe({
      next: users => this.usersSubject.next(users),
      error: err => {
        this.errorSubject.next('Erro ao carregar');
        this.notificationService.showError(err);
      }
    });
  }
  
  deleteUser(id: number): void {
    this.userApi.deleteUser(id).subscribe({
      next: () => {
        const updated = this.usersSubject.value.filter(u => u.id !== id);
        this.usersSubject.next(updated);
        this.notificationService.showSuccess('Usuário removido');
      },
      error: err => this.notificationService.showError(err)
    });
  }
}

// Component fica limpo
@Component({ ... })
export class UserListComponent implements OnInit {
  users$ = this.facade.users$;
  loading$ = this.facade.loading$;
  
  constructor(private facade: UserFacade) {}
  
  ngOnInit() { this.facade.loadUsers(); }
  onDelete(id: number) { this.facade.deleteUser(id); }
}
```

---

<a name="5-rxjs"></a>
## 5. RxJS & OBSERVABLES

### 5.1 Observable vs Promise

| Feature | Observable | Promise |
|---------|-----------|---------|
| **Execução** | Lazy (só quando subscribe) | Eager (executa imediato) |
| **Valores** | Múltiplos ao longo do tempo | Um único valor |
| **Cancelamento** | Sim (unsubscribe) | Não |
| **Operadores** | 100+ (map, filter, etc) | then, catch, finally |
| **Retry** | Built-in | Manual |

```typescript
// Promise - executa imediatamente
const promise = fetch('/api/users'); // Já está executando!

// Observable - só executa quando alguém subscribes
const obs$ = this.http.get('/api/users'); // Nada acontece
obs$.subscribe(data => ...); // Agora executa!
```

### 5.2 Subjects - Tipos e Quando Usar

```typescript
// Subject - Não tem valor inicial, emissor manual
const subject = new Subject<string>();
subject.subscribe(v => console.log(v)); // Esperando...
subject.next('A'); // 'A'

// BehaviorSubject - TEM valor inicial, novos subscribers recebem último valor
const behavior = new BehaviorSubject<string>('inicial');
behavior.subscribe(v => console.log(v)); // 'inicial' (imediato!)
behavior.next('A'); // 'A'
behavior.getValue(); // 'A' (acesso síncrono)

// ReplaySubject - Replay N últimos valores para novos subscribers
const replay = new ReplaySubject<string>(2); // Buffer de 2
replay.next('A');
replay.next('B');
replay.next('C');
replay.subscribe(v => console.log(v)); // 'B', 'C' (últimos 2)

// AsyncSubject - Só emite o ÚLTIMO valor, e só quando completa
const async$ = new AsyncSubject<string>();
async$.next('A');
async$.next('B');
async$.subscribe(v => console.log(v)); // Nada ainda...
async$.complete(); // 'B' (agora emite)
```

**Quando usar cada um:**
- `Subject` → Event bus, notificações (sem estado)
- `BehaviorSubject` → Estado reativo (filtros, user logado, tema)
- `ReplaySubject` → Quando novos subscribers precisam do histórico
- `AsyncSubject` → Resolver tipo Promise (um valor final)

### 5.3 Operators Essenciais para Senior

| Operator | O que faz | Quando Usar | Exemplo |
|----------|-----------|-------------|---------|
| **map** | Transforma valores | Extrair propriedades | `map(users => users.length)` |
| **filter** | Filtra valores | Só valores válidos | `filter(n => n > 0)` |
| **tap** | Side effects (não modifica) | Logging, debug | `tap(data => console.log(data))` |
| **switchMap** | Troca Observable (cancela anterior) | Search, autocomplete | Ver exemplo abaixo |
| **mergeMap** | Todos em paralelo | Batch uploads | Ver exemplo abaixo |
| **concatMap** | Um por vez, ordenado | Fila ordenada | Ver exemplo abaixo |
| **exhaustMap** | Ignora novos até terminar | Previne double-click | Ver exemplo abaixo |
| **debounceTime** | Espera silêncio | Input de busca | `debounceTime(300)` |
| **throttleTime** | Limita emissões | Scroll, resize | `throttleTime(200)` |
| **distinctUntilChanged** | Só emite se mudou | Evita duplicados | Combinar com debounce |
| **catchError** | Tratamento de erro | Fallback quando falha | `catchError(() => of([]))` |
| **retry** | Tenta novamente | Resiliência HTTP | `retry(2)` |
| **retryWhen** | Retry com lógica | Retry com delay | Ver exemplo |
| **shareReplay** | Cache Observable | HTTP compartilhado | Ver seção anterior |
| **takeUntil** | Cancela quando emite | Unsubscribe automático | Pattern destroy$ |
| **take** | Pega N valores e completa | Pegar só o primeiro | `take(1)` |
| **skip** | Pula N valores | Ignorar valor inicial | `skip(1)` |
| **startWith** | Emite valor inicial | Estado default | `startWith([])` |
| **withLatestFrom** | Combina com último valor | Pegar contexto atual | Ver exemplo |
| **combineLatest** | Combina múltiplos Observables | Filtros combinados | Ver exemplo abaixo |
| **forkJoin** | Promise.all para Observables | Múltiplos HTTP paralelos | Ver exemplo abaixo |
| **pairwise** | Valor atual + anterior | Comparar mudanças | `pairwise()` |
| **scan** | Acumulador (reduce reativo) | Contadores, estado | `scan((acc, val) => acc + val, 0)` |
| **finalize** | Executa ao completar/erro | Esconder loading | `finalize(() => loading = false)` |

### 5.4 switchMap vs mergeMap vs concatMap vs exhaustMap

**A pergunta MAIS COMUM em entrevistas:**

```
switchMap  → "Troque para o mais recente" (cancela anterior)
mergeMap   → "Execute todos em paralelo" (não cancela nada)
concatMap  → "Execute em fila" (um de cada vez, em ordem)
exhaustMap → "Ignore novos enquanto ocupado" (ignora durante execução)
```

```typescript
// switchMap - Cancela requisição anterior
// Use: Search, autocomplete, navegação
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  switchMap(term => this.api.search(term))
  // Se digitar "ang" e depois "angular":
  // Cancela a busca por "ang" e busca "angular"
).subscribe(results => this.results = results);

// mergeMap - Todas em paralelo (cuidado com ordem!)
// Use: Upload de múltiplos arquivos, fire-and-forget
from(files).pipe(
  mergeMap(file => this.uploadFile(file), 3)  // Máximo 3 paralelos
  //                                      ↑ concurrency limit
).subscribe(result => console.log(result));

// concatMap - Uma por vez, em ordem
// Use: Operações que dependem da anterior, processar fila
from([createUser, addAddress, addPayment]).pipe(
  concatMap(action => action())
  // Espera createUser terminar antes de addAddress
).subscribe();

// exhaustMap - Ignora novos até terminar
// Use: Submit de form, login button, prevenir double-click
fromEvent(saveBtn, 'click').pipe(
  exhaustMap(() => this.saveForm())
  // Se clicar 5x rápido, só executa 1 saveForm()
).subscribe();
```

**Resumo visual:**
```
switchMap:   --A----B----C-->   →   --a]---b]---c-->    (cancela anterior)
mergeMap:    --A----B----C-->   →   --a--a-b-b-c-c-->   (todos paralelos)
concatMap:   --A----B----C-->   →   --aaaa-bbbb-cccc->  (fila ordenada)
exhaustMap:  --A----B----C-->   →   --aaaa-------cccc-> (ignora B)
```

### 5.5 Autocomplete Perfeito (Exemplo Completo)

```typescript
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  results: Product[] = [];
  loading = false;
  error: string | null = null;
  
  ngOnInit() {
    this.searchControl.valueChanges.pipe(
      debounceTime(300),              // Espera 300ms sem digitação
      distinctUntilChanged(),         // Só emite se valor mudou
      tap(() => {
        this.loading = true;
        this.error = null;
      }),
      filter(term => term.length >= 2),  // Mínimo 2 caracteres
      switchMap(term => 
        this.searchService.search(term).pipe(
          catchError(err => {
            this.error = 'Erro na busca';
            return of([]);            // Retorna [] em caso de erro
          })
        )
      ),
      tap(() => this.loading = false)
    ).subscribe(results => this.results = results);
  }
}
```

### 5.6 combineLatest vs forkJoin vs withLatestFrom vs zip

```typescript
// combineLatest - Emite quando QUALQUER um emite (precisa que todos tenham emitido pelo menos 1x)
// Use: Filtros combinados, estado derivado
combineLatest([
  this.products$,       // Emite quando produtos mudam
  this.categoryFilter$, // Emite quando filtro muda
  this.sortOrder$       // Emite quando ordenação muda
]).pipe(
  map(([products, category, sort]) => {
    // Recalcula sempre que qualquer coisa muda
    return this.applyFiltersAndSort(products, category, sort);
  })
);

// forkJoin - Espera TODOS completarem (como Promise.all)
// Use: Múltiplos HTTP paralelos (HTTP completa automaticamente)
forkJoin({
  users: this.http.get<User[]>('/api/users'),
  posts: this.http.get<Post[]>('/api/posts'),
  config: this.http.get<Config>('/api/config')
}).subscribe(({ users, posts, config }) => {
  // Só executa quando TODOS terminarem
});

// withLatestFrom - Combina com ÚLTIMO valor do outro
// Use: Pegar contexto no momento de uma ação
this.saveButton$.pipe(
  withLatestFrom(this.form.valueChanges, this.currentUser$),
  map(([click, formValue, user]) => ({ ...formValue, updatedBy: user.id })),
  switchMap(data => this.api.save(data))
);

// zip - Combina por INDEX (1º com 1º, 2º com 2º...)
// Use: Sincronizar streams par a par
zip(
  this.http.get('/api/questions'),
  this.http.get('/api/answers')
).pipe(
  map(([questions, answers]) => this.merge(questions, answers))
);
```

### 5.7 Error Handling Patterns

```typescript
// 1. catchError com fallback
this.http.get('/api/users').pipe(
  catchError(err => {
    console.error(err);
    return of([]); // Valor padrão
  })
);

// 2. retry simples
this.http.get('/api/users').pipe(
  retry(3), // Tenta 3x antes de emitir erro
  catchError(err => of([]))
);

// 3. retry com delay exponencial (produção!)
this.http.get('/api/users').pipe(
  retry({
    count: 3,
    delay: (error, retryCount) => {
      const delayMs = Math.pow(2, retryCount) * 1000; // 1s, 2s, 4s
      console.log(`Retry ${retryCount} em ${delayMs}ms`);
      return timer(delayMs);
    }
  }),
  catchError(err => {
    this.notificationService.showError('Falha após 3 tentativas');
    return EMPTY;
  })
);

// 4. catchError que re-emite (não mata o Observable)
this.searchControl.valueChanges.pipe(
  switchMap(term =>
    this.api.search(term).pipe(
      catchError(() => of([]))  // ← Dentro do switchMap!
    )
  )
  // Se catchError fosse fora, mataria o stream de valueChanges
);
```

### 5.8 Custom Operators

```typescript
// Operator customizado reutilizável
function retryWithBackoff<T>(maxRetries: number, delayMs: number) {
  return (source: Observable<T>) => source.pipe(
    retry({
      count: maxRetries,
      delay: (_, retryCount) => timer(delayMs * Math.pow(2, retryCount - 1))
    })
  );
}

// Uso
this.http.get('/api/data').pipe(
  retryWithBackoff(3, 1000)
);

// Outro: indicador de loading
function withLoading<T>(loading$: BehaviorSubject<boolean>) {
  return (source: Observable<T>) => source.pipe(
    tap(() => loading$.next(true)),
    finalize(() => loading$.next(false))
  );
}
```

---

<a name="6-pipes"></a>
## 6. PIPES (BUILT-IN & CUSTOM)

### 6.1 Pipes Built-in Essenciais

```html
<!-- Date -->
<div>{{ today | date }}</div>                    <!-- Dec 25, 2024 -->
<div>{{ today | date:'dd/MM/yyyy' }}</div>       <!-- 25/12/2024 -->
<div>{{ today | date:'short' }}</div>            <!-- 12/25/24, 3:15 PM -->
<div>{{ today | date:'fullDate' }}</div>         <!-- Wednesday, December 25, 2024 -->
<div>{{ today | date:'HH:mm:ss' }}</div>         <!-- 15:30:45 -->

<!-- Currency -->
<div>{{ price | currency }}</div>                <!-- $100.00 -->
<div>{{ price | currency:'BRL' }}</div>          <!-- R$ 100,00 -->
<div>{{ price | currency:'EUR':'symbol':'1.2-2' }}</div>  <!-- €100.00 -->

<!-- Number/Decimal -->
<div>{{ 3.14159 | number:'1.2-4' }}</div>        <!-- 3.1416 -->
<!--                      ↑ ↑   ↑
          mínimo dígitos inteiros . mín-máx decimais -->
<div>{{ 1000000 | number:'1.0-0' }}</div>        <!-- 1,000,000 -->

<!-- Percent -->
<div>{{ 0.256 | percent }}</div>                 <!-- 26% -->
<div>{{ 0.256 | percent:'1.1-2' }}</div>         <!-- 25.6% -->

<!-- Async (IMPORTANTE - unsubscribe automático) -->
<div *ngIf="user$ | async as user">
  {{ user.name }}
</div>
<!-- Combinar pipes -->
<div>{{ (price$ | async) | currency:'BRL' }}</div>

<!-- JSON (debug) -->
<pre>{{ complexObject | json }}</pre>

<!-- Uppercase/Lowercase/Titlecase -->
<div>{{ 'hello' | uppercase }}</div>             <!-- HELLO -->
<div>{{ 'WORLD' | lowercase }}</div>             <!-- world -->
<div>{{ 'hello world' | titlecase }}</div>       <!-- Hello World -->

<!-- Slice (arrays e strings) -->
<div>{{ [1,2,3,4,5] | slice:1:3 }}</div>         <!-- [2,3] -->
<div>{{ 'Angular' | slice:0:3 }}</div>           <!-- Ang -->

<!-- KeyValue (iterar objects) -->
<div *ngFor="let item of map | keyvalue">
  {{ item.key }}: {{ item.value }}
</div>

<!-- I18nPlural / I18nSelect -->
<div>{{ messages.length }} {{ messages.length | i18nPlural: pluralMap }}</div>
<!-- pluralMap = { '=0': 'mensagens', '=1': 'mensagem', 'other': 'mensagens' } -->
```

### 6.2 Pipe Customizado

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'highlight',
  pure: true  // ← Só recalcula se inputs mudarem (performance)
})
export class HighlightPipe implements PipeTransform {
  transform(text: string, search: string): string {
    if (!search || !text) return text;
    
    const regex = new RegExp(`(${this.escapeRegex(search)})`, 'gi');
    return text.replace(regex, '<mark>$1</mark>');
  }
  
  private escapeRegex(str: string): string {
    return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  }
}
```

```html
<div [innerHTML]="'Angular é incrível' | highlight:'Angular'"></div>
<!-- Resultado: <mark>Angular</mark> é incrível -->
```

### 6.3 Pure vs Impure Pipes

| Aspecto | Pure (default) | Impure |
|---------|---------------|--------|
| **Recalcula** | Quando input muda (referência) | A CADA change detection |
| **Performance** | Excelente | Péssima |
| **Array mutation** | NÃO detecta `.push()` | Detecta |
| **Uso** | 99% dos casos | Quase nunca |

```typescript
// Pure (default) - só recalcula se referência do input mudar
@Pipe({ name: 'filterUsers', pure: true })
export class FilterUsersPipe implements PipeTransform {
  transform(users: User[], searchTerm: string): User[] {
    if (!searchTerm) return users;
    return users.filter(u => u.name.includes(searchTerm));
  }
}

// ⚠️ Com pure pipe: users.push(newUser) NÃO dispara recalcular
// ✅ Precisa: users = [...users, newUser] (nova referência)
```

**Quando usar impure?** Quase nunca. Melhor forçar imutabilidade.

### 6.4 Pipes Úteis para Projetos

```typescript
// Tempo relativo ("há 5 minutos")
@Pipe({ name: 'timeAgo', pure: true })
export class TimeAgoPipe implements PipeTransform {
  transform(date: Date | string): string {
    const now = new Date();
    const past = new Date(date);
    const diffMs = now.getTime() - past.getTime();
    const diffSec = Math.floor(diffMs / 1000);
    const diffMin = Math.floor(diffSec / 60);
    const diffHour = Math.floor(diffMin / 60);
    const diffDay = Math.floor(diffHour / 24);
    
    if (diffSec < 60) return 'agora mesmo';
    if (diffMin < 60) return `há ${diffMin} min`;
    if (diffHour < 24) return `há ${diffHour}h`;
    if (diffDay < 30) return `há ${diffDay} dias`;
    return past.toLocaleDateString('pt-BR');
  }
}

// Truncate com "..."
@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit = 50, ellipsis = '...'): string {
    if (!value || value.length <= limit) return value;
    return value.substring(0, limit).trimEnd() + ellipsis;
  }
}

// Safe URL (bypass sanitizer para iframes/vídeos)
@Pipe({ name: 'safeUrl' })
export class SafeUrlPipe implements PipeTransform {
  constructor(private sanitizer: DomSanitizer) {}
  
  transform(url: string): SafeResourceUrl {
    return this.sanitizer.bypassSecurityTrustResourceUrl(url);
  }
}

// Bytes para human readable
@Pipe({ name: 'fileSize' })
export class FileSizePipe implements PipeTransform {
  transform(bytes: number): string {
    if (bytes === 0) return '0 B';
    const units = ['B', 'KB', 'MB', 'GB', 'TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    return `${(bytes / Math.pow(1024, i)).toFixed(1)} ${units[i]}`;
  }
}
```

---

<a name="7-directives"></a>
## 7. DIRECTIVES CUSTOMIZADAS

### 7.1 Attribute Directive

```typescript
import { Directive, ElementRef, HostListener, Input, Renderer2 } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight = 'yellow';
  @Input() hoverColor = '';
  
  private originalColor = '';
  
  // ✅ Usar Renderer2 ao invés de ElementRef direto (segurança SSR)
  constructor(
    private el: ElementRef,
    private renderer: Renderer2
  ) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.originalColor = this.el.nativeElement.style.backgroundColor;
    this.setColor(this.hoverColor || this.appHighlight);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.setColor(this.originalColor);
  }
  
  private setColor(color: string) {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
  }
}
```

```html
<div appHighlight>Hover amarelo (default)</div>
<div [appHighlight]="'lightblue'" [hoverColor]="'blue'">Hover azul</div>
```

### 7.2 Structural Directive

```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;
  
  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
}
```

### 7.3 Directives Úteis para Projetos

```typescript
// Permission Directive - Mostrar/esconder baseado em permissão
@Directive({ selector: '[appHasPermission]' })
export class HasPermissionDirective implements OnInit {
  @Input('appHasPermission') permission!: string;
  
  private hasView = false;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private authService: AuthService
  ) {}
  
  ngOnInit() {
    if (this.authService.hasPermission(this.permission) && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (!this.authService.hasPermission(this.permission) && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
// Uso: <button *appHasPermission="'admin:delete'">Deletar</button>

// Auto-focus Directive
@Directive({ selector: '[appAutoFocus]' })
export class AutoFocusDirective implements AfterViewInit {
  constructor(private el: ElementRef) {}
  
  ngAfterViewInit() {
    setTimeout(() => this.el.nativeElement.focus());
  }
}

// Click Outside Directive (fechar dropdown)
@Directive({ selector: '[appClickOutside]' })
export class ClickOutsideDirective {
  @Output() appClickOutside = new EventEmitter<void>();
  
  constructor(private el: ElementRef) {}
  
  @HostListener('document:click', ['$event.target'])
  onClick(target: HTMLElement) {
    if (!this.el.nativeElement.contains(target)) {
      this.appClickOutside.emit();
    }
  }
}
// Uso: <div appClickOutside (appClickOutside)="closeDropdown()">

// Debounce Click (previne double click)
@Directive({ selector: '[appDebounceClick]' })
export class DebounceClickDirective implements OnInit, OnDestroy {
  @Input() debounceTime = 500;
  @Output() debounceClick = new EventEmitter<MouseEvent>();
  
  private clicks$ = new Subject<MouseEvent>();
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.clicks$.pipe(
      debounceTime(this.debounceTime),
      takeUntil(this.destroy$)
    ).subscribe(event => this.debounceClick.emit(event));
  }
  
  @HostListener('click', ['$event'])
  onClick(event: MouseEvent) {
    event.preventDefault();
    event.stopPropagation();
    this.clicks$.next(event);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
// Uso: <button appDebounceClick (debounceClick)="save()">Salvar</button>
```

### 7.4 Directive Composition API (Angular 15+)

```typescript
// Compor diretivas em um component
@Component({
  selector: 'app-button',
  hostDirectives: [
    { directive: TooltipDirective, inputs: ['tooltipText'] },
    { directive: RippleDirective },
    { directive: TrackClickDirective, outputs: ['tracked'] }
  ],
  template: `<ng-content></ng-content>`
})
export class ButtonComponent {}

// Uso: o component herda todos os inputs/outputs das directives
<app-button tooltipText="Clique aqui" (tracked)="onTrack($event)">
  Salvar
</app-button>
```

---

<a name="8-forms"></a>
## 8. FORMS (REACTIVE & TEMPLATE-DRIVEN)

### 8.1 Template-Driven vs Reactive

| Aspecto | Template-Driven | Reactive |
|---------|----------------|----------|
| **Setup** | Simples (ngModel) | Mais código |
| **Lógica** | No template | No component (TypeScript) |
| **Testabilidade** | Difícil | Fácil |
| **Validação** | Directives | Funções |
| **Dinâmico** | Limitado | Total controle |
| **Recomendado** | Forms simples | Senior, apps complexas |

### 8.2 Template-Driven (Simples)

```typescript
export class LoginComponent {
  user = { email: '', password: '' };
  
  onSubmit(form: NgForm) {
    if (form.valid) {
      console.log(this.user);
    }
  }
}
```

```html
<form #loginForm="ngForm" (ngSubmit)="onSubmit(loginForm)">
  <input name="email" [(ngModel)]="user.email" required email #emailField="ngModel">
  <div *ngIf="emailField.invalid && emailField.touched" class="error">
    Email inválido
  </div>
  
  <input name="password" [(ngModel)]="user.password" required minlength="6">
  
  <button [disabled]="!loginForm.valid">Login</button>
  
  <!-- Debug -->
  <pre>Valid: {{ loginForm.valid }}</pre>
  <pre>Values: {{ loginForm.value | json }}</pre>
</form>
```

### 8.3 Reactive Forms (Recomendado para Senior)

```typescript
export class RegistrationComponent implements OnInit {
  registrationForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.registrationForm = this.fb.group({
      // [valor_inicial, [validators_síncronos], [validators_assíncronos]]
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8), this.strongPassword]],
      confirmPassword: ['', Validators.required],
      
      // FormGroup aninhado
      address: this.fb.group({
        street: [''],
        city: ['', Validators.required],
        state: ['', Validators.required],
        zip: ['', [Validators.required, Validators.pattern(/^\d{5}-\d{3}$/)]],
      }),
      
      // FormArray
      phones: this.fb.array([]),
      
      // Checkbox
      acceptTerms: [false, Validators.requiredTrue],
    }, {
      validators: [this.passwordMatchValidator]  // Cross-field validation
    });
    
    // Reagir a mudanças
    this.registrationForm.get('state')?.valueChanges.pipe(
      distinctUntilChanged()
    ).subscribe(state => {
      this.loadCities(state);
    });
  }
  
  // Getters para template limpo
  get name() { return this.registrationForm.get('name'); }
  get email() { return this.registrationForm.get('email'); }
  get password() { return this.registrationForm.get('password'); }
  get address() { return this.registrationForm.get('address') as FormGroup; }
  get phones() { return this.registrationForm.get('phones') as FormArray; }
  
  // Validador customizado inline
  strongPassword(control: AbstractControl): ValidationErrors | null {
    const value = control.value;
    const hasUpper = /[A-Z]/.test(value);
    const hasLower = /[a-z]/.test(value);
    const hasNumber = /\d/.test(value);
    const hasSpecial = /[!@#$%^&*]/.test(value);
    
    const valid = hasUpper && hasLower && hasNumber && hasSpecial;
    return valid ? null : { weakPassword: true };
  }
  
  // Cross-field
  passwordMatchValidator(group: AbstractControl): ValidationErrors | null {
    const pass = group.get('password')?.value;
    const confirm = group.get('confirmPassword')?.value;
    return pass === confirm ? null : { passwordMismatch: true };
  }
  
  onSubmit() {
    if (this.registrationForm.valid) {
      console.log(this.registrationForm.value);
      // OU getRawValue() para incluir campos disabled
    } else {
      this.markAllAsTouched();
    }
  }
  
  private markAllAsTouched() {
    this.registrationForm.markAllAsTouched();
  }
}
```

```html
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">
  <!-- Name -->
  <input formControlName="name" placeholder="Nome">
  <div *ngIf="name?.invalid && name?.touched">
    <small *ngIf="name?.errors?.['required']">Nome obrigatório</small>
    <small *ngIf="name?.errors?.['minlength']">
      Mínimo {{ name?.errors?.['minlength'].requiredLength }} caracteres
    </small>
  </div>
  
  <!-- Email -->
  <input formControlName="email" placeholder="Email">
  
  <!-- Password -->
  <input formControlName="password" type="password">
  <div *ngIf="password?.invalid && password?.touched">
    <small *ngIf="password?.errors?.['weakPassword']">
      Senha precisa de maiúscula, minúscula, número e caractere especial
    </small>
  </div>
  
  <!-- Confirm Password -->
  <input formControlName="confirmPassword" type="password">
  <div *ngIf="registrationForm.errors?.['passwordMismatch'] && password?.touched">
    <small>Senhas não conferem</small>
  </div>
  
  <!-- Address (FormGroup aninhado) -->
  <div formGroupName="address">
    <input formControlName="street" placeholder="Rua">
    <input formControlName="city" placeholder="Cidade">
    <input formControlName="state" placeholder="Estado">
    <input formControlName="zip" placeholder="CEP">
  </div>
  
  <!-- Phones (FormArray) -->
  <div formArrayName="phones">
    <div *ngFor="let phone of phones.controls; let i = index">
      <input [formControlName]="i" placeholder="Telefone">
      <button type="button" (click)="removePhone(i)">X</button>
    </div>
  </div>
  <button type="button" (click)="addPhone()">+ Telefone</button>
  
  <label>
    <input formControlName="acceptTerms" type="checkbox">
    Aceito os termos
  </label>
  
  <button [disabled]="registrationForm.invalid">Registrar</button>
</form>
```

### 8.4 FormArray (Arrays Dinâmicos)

```typescript
export class SkillsComponent implements OnInit {
  form!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.form = this.fb.group({
      skills: this.fb.array([], [Validators.minLength(1)])
    });
  }
  
  get skills(): FormArray {
    return this.form.get('skills') as FormArray;
  }
  
  addSkill() {
    const skillGroup = this.fb.group({
      name: ['', Validators.required],
      level: ['beginner', Validators.required],
      years: [0, [Validators.required, Validators.min(0)]]
    });
    this.skills.push(skillGroup);
  }
  
  removeSkill(index: number) {
    this.skills.removeAt(index);
  }
  
  moveSkillUp(index: number) {
    if (index <= 0) return;
    const current = this.skills.at(index);
    this.skills.removeAt(index);
    this.skills.insert(index - 1, current);
  }
}
```

### 8.5 Validação Customizada

```typescript
// Validador síncrono reutilizável
export function cpfValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const cpf = control.value?.replace(/\D/g, '');
    if (!cpf) return null;
    if (cpf.length !== 11) return { cpfInvalido: { message: 'CPF deve ter 11 dígitos' } };
    
    // Validação dos dígitos
    const isValid = validateCPFDigits(cpf);
    return isValid ? null : { cpfInvalido: { message: 'CPF inválido' } };
  };
}

// Validador assíncrono (checa no servidor)
export function emailExistsValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) return of(null);
    
    return timer(500).pipe(  // Debounce de 500ms
      switchMap(() => userService.checkEmail(control.value)),
      map(exists => exists ? { emailJaExiste: true } : null),
      catchError(() => of(null))
    );
  };
}

// Validador parametrizado
export function rangeValidator(min: number, max: number): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value;
    if (value === null || value === undefined) return null;
    if (value < min || value > max) {
      return { range: { min, max, actual: value } };
    }
    return null;
  };
}
```

```typescript
// Uso
this.form = this.fb.group({
  cpf: ['', [Validators.required, cpfValidator()]],
  email: ['', 
    [Validators.required, Validators.email],       // Sync
    [emailExistsValidator(this.userService)]         // Async
  ],
  age: [0, [Validators.required, rangeValidator(18, 120)]]
});
```

### 8.6 Dynamic Forms (avançado)

```typescript
// Gerar formulário dinamicamente a partir de config
interface FieldConfig {
  key: string;
  label: string;
  type: 'text' | 'email' | 'number' | 'select' | 'checkbox';
  validators?: ValidatorFn[];
  options?: { value: any; label: string }[]; // Para select
}

@Component({ ... })
export class DynamicFormComponent implements OnInit {
  @Input() fields: FieldConfig[] = [];
  @Output() formSubmit = new EventEmitter<any>();
  
  form!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    const group: Record<string, any> = {};
    
    this.fields.forEach(field => {
      group[field.key] = ['', field.validators || []];
    });
    
    this.form = this.fb.group(group);
  }
  
  onSubmit() {
    if (this.form.valid) {
      this.formSubmit.emit(this.form.value);
    }
  }
}
```

---

<a name="9-routing"></a>
## 9. ROUTING AVANÇADO

### 9.1 Configuração Básica

```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: 'users/:id/edit', component: UserEditComponent },
  { path: '**', component: NotFoundComponent }  // 404 - SEMPRE último
];

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    scrollPositionRestoration: 'enabled',   // Scroll to top ao navegar
    anchorScrolling: 'enabled',             // Scroll para #anchors
    onSameUrlNavigation: 'reload',          // Permite recarregar mesma rota
    preloadingStrategy: PreloadAllModules   // Pre-carrega lazy modules
  })],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

### 9.2 Lazy Loading (ESSENCIAL)

```typescript
const routes: Routes = [
  // Lazy load de módulo inteiro
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  },
  
  // Lazy load de component standalone (Angular 14+)
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard.component').then(m => m.DashboardComponent)
  },
  
  // Lazy load com preload condicional
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.module').then(m => m.ReportsModule),
    data: { preload: true }  // Para custom preloading strategy
  }
];
```

**Custom Preloading Strategy:**
```typescript
// Pré-carregar apenas módulos marcados
@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}
```

### 9.3 Route Guards

```typescript
// CanActivate - Protege acesso à rota
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean | UrlTree {
    if (this.authService.isLoggedIn()) {
      // Checar roles se necessário
      const requiredRole = route.data['role'];
      if (requiredRole && !this.authService.hasRole(requiredRole)) {
        return this.router.createUrlTree(['/forbidden']);
      }
      return true;
    }
    // Salva URL para redirect após login
    return this.router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url }
    });
  }
}

// Functional Guard (Angular 15+ - preferido)
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isLoggedIn()) return true;
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Role Guard (funcional)
export const roleGuard: CanActivateFn = (route) => {
  const authService = inject(AuthService);
  const requiredRole = route.data['role'] as string;
  return authService.hasRole(requiredRole) || inject(Router).createUrlTree(['/forbidden']);
};

// CanDeactivate - Previne saída com form não salvo
export const unsavedChangesGuard: CanDeactivateFn<{ hasUnsavedChanges: () => boolean }> = 
  (component) => {
    if (component.hasUnsavedChanges()) {
      return confirm('Há alterações não salvas. Deseja sair?');
    }
    return true;
  };
```

```typescript
// Uso nas rotas
{
  path: 'admin',
  canActivate: [authGuard, roleGuard],
  data: { role: 'admin' },
  children: [
    {
      path: 'users/:id/edit',
      component: UserEditComponent,
      canDeactivate: [unsavedChangesGuard]
    }
  ]
}
```

### 9.4 Resolve - Pré-carregar Dados

```typescript
// Functional resolver (Angular 15+)
export const userResolver: ResolveFn<User> = (route) => {
  const userService = inject(UserService);
  const router = inject(Router);
  const id = Number(route.paramMap.get('id'));
  
  return userService.getUser(id).pipe(
    catchError(() => {
      router.navigate(['/not-found']);
      return EMPTY;
    })
  );
};

// Na rota
{
  path: 'users/:id',
  component: UserDetailComponent,
  resolve: { user: userResolver }
}

// No component - dados já carregados!
export class UserDetailComponent implements OnInit {
  user!: User;
  
  constructor(private route: ActivatedRoute) {}
  
  ngOnInit() {
    // Snapshot (valor estático)
    this.user = this.route.snapshot.data['user'];
    
    // OU Observable (reage se rota mudar)
    this.route.data.subscribe(data => {
      this.user = data['user'];
    });
  }
}
```

### 9.5 Nested Routes e Named Outlets

```typescript
// Nested routes (children)
const routes: Routes = [
  {
    path: 'admin',
    component: AdminLayoutComponent,
    canActivate: [authGuard],
    children: [
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: DashboardComponent },
      { path: 'users', component: UsersComponent },
      { path: 'settings', component: SettingsComponent }
    ]
  }
];

// AdminLayoutComponent template
<nav>
  <a routerLink="dashboard" routerLinkActive="active">Dashboard</a>
  <a routerLink="users" routerLinkActive="active">Users</a>
</nav>
<router-outlet></router-outlet>  <!-- Children renderizam aqui -->

// Named outlets (múltiplos outlets)
<router-outlet></router-outlet>           <!-- Primary -->
<router-outlet name="sidebar"></router-outlet>  <!-- Named -->

{
  path: 'users',
  component: UsersComponent,
  children: [
    { path: 'filter', component: FilterComponent, outlet: 'sidebar' }
  ]
}
// URL: /users(sidebar:filter)
```

### 9.6 Parâmetros de Rota

```typescript
// ===== Leitura de parâmetros =====

// Snapshot (valor atual, não reage a mudanças)
const id = this.route.snapshot.paramMap.get('id');
const page = this.route.snapshot.queryParamMap.get('page');

// Observable (reage quando parâmetro muda na mesma rota)
this.route.paramMap.pipe(
  map(params => Number(params.get('id'))),
  switchMap(id => this.userService.getUser(id))
).subscribe(user => this.user = user);

// ===== Navegação programática =====

// Navegação simples
this.router.navigate(['/users', userId]);
// URL: /users/42

// Com query params
this.router.navigate(['/users'], {
  queryParams: { page: 2, sort: 'name', order: 'asc' }
});
// URL: /users?page=2&sort=name&order=asc

// Preservar query params ao navegar
this.router.navigate(['/users/42'], {
  queryParamsHandling: 'preserve'  // Mantém params existentes
  // OU 'merge' para juntar com novos
});

// Navegação relativa
this.router.navigate(['edit'], { relativeTo: this.route });
// Se estiver em /users/42 → vai para /users/42/edit

// Com fragment
this.router.navigate(['/docs'], { fragment: 'section-2' });
// URL: /docs#section-2
```

### 9.7 Router Events (Monitorar Navegação)

```typescript
export class AppComponent implements OnInit {
  loading = false;
  
  constructor(private router: Router) {}
  
  ngOnInit() {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationStart) {
        this.loading = true;
      }
      if (event instanceof NavigationEnd) {
        this.loading = false;
        // Analytics
        this.analytics.trackPageView(event.urlAfterRedirects);
      }
      if (event instanceof NavigationError) {
        this.loading = false;
        console.error('Navigation error:', event.error);
      }
      if (event instanceof NavigationCancel) {
        this.loading = false;
      }
    });
  }
}
```

---

<a name="10-change-detection"></a>
## 10. CHANGE DETECTION & OnPush

### 10.1 Como Change Detection Funciona

```
Evento (click, HTTP, timer) → Zone.js intercepta → Angular checa toda árvore de components
                                                            ↓
                                    Component A (checado) → Component B (checado) → Component C (checado)
                                         ↓
                                    Component D (checado)
```

**Zone.js monitora:**
- Events DOM (click, input, etc)
- setTimeout, setInterval
- HTTP requests (XMLHttpRequest, fetch)
- Promises

### 10.2 Estratégias

| Estratégia | Como Funciona | Performance |
|-----------|---------------|-------------|
| **Default** | Checa toda árvore de cima pra baixo a cada evento | O(n) - todas as views |
| **OnPush** | Só checa quando: (1) @Input muda por referência, (2) evento originado no component, (3) async pipe emite, (4) `markForCheck()` chamado | O(1) - só affected |

### 10.3 OnPush em Detalhes

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class FastComponent {
  @Input() users!: User[];
  
  // ===== O que DISPARA change detection com OnPush =====
  
  // 1. Nova referência de @Input
  // No pai: this.users = [...this.users, newUser]  ✅
  
  // 2. Evento DOM dentro do component
  onClick() {
    // Angular checa este component automaticamente
  }
  
  // 3. Async pipe
  // <div>{{ data$ | async }}</div>  → marca automaticamente
  
  // 4. markForCheck() manual
  constructor(private cdr: ChangeDetectorRef) {}
  forceUpdate() {
    this.cdr.markForCheck();  // Marca este component E ancestors
  }
  
  // ===== O que NÃO dispara =====
  
  // ❌ Mutação de array
  addUser(user: User) {
    this.users.push(user);  // Mesma referência, OnPush ignora
  }
  
  // ❌ Mutação de objeto
  updateUser() {
    this.users[0].name = 'Novo';  // Não detecta!
  }
  
  // ===== Como fazer CERTO =====
  
  // ✅ Nova referência (imutabilidade)
  addUser(user: User) {
    this.users = [...this.users, user];
  }
  
  updateUser(index: number, name: string) {
    this.users = this.users.map((u, i) => 
      i === index ? { ...u, name } : u
    );
  }
}
```

### 10.4 OnPush + Async Pipe = Perfeito

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @if (loading$ | async) {
      <app-spinner />
    }
    
    @if (error$ | async; as error) {
      <app-error [message]="error" />
    }
    
    @for (user of users$ | async; track user.id) {
      <app-user-card [user]="user" (deleted)="onDelete($event)" />
    } @empty {
      <p>Nenhum usuário encontrado.</p>
    }
  `
})
export class UserListComponent {
  users$ = this.userService.getUsers();
  loading$ = new BehaviorSubject(false);
  error$ = new BehaviorSubject<string | null>(null);
  
  constructor(private userService: UserService) {}
  
  onDelete(id: number) { /* ... */ }
}
```

### 10.5 ChangeDetectorRef - Controle Manual

```typescript
import { ChangeDetectorRef } from '@angular/core';

export class MyComponent {
  constructor(private cdr: ChangeDetectorRef) {}
  
  // markForCheck - marca para próxima checagem (com OnPush)
  updateFromWebSocket(data: any) {
    this.data = data;
    this.cdr.markForCheck();  // "Na próxima CD cycle, cheque este component"
  }
  
  // detectChanges - força checagem imediata
  updateImmediately() {
    this.data = 'nova';
    this.cdr.detectChanges();  // Checa AGORA
  }
  
  // detach/reattach - desconecta completamente
  pauseUpdates() {
    this.cdr.detach();  // Para de checar este component
  }
  
  resumeUpdates() {
    this.cdr.reattach();  // Volta a checar
    this.cdr.markForCheck();
  }
}
```

### 10.6 Zoneless (Angular 18+ experimental)

```typescript
// Sem Zone.js - change detection 100% manual via Signals
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});

// Component usando Signals (funciona sem Zone.js)
@Component({
  template: `<div>{{ count() }}</div>`
})
export class CounterComponent {
  count = signal(0);
  
  increment() {
    this.count.update(v => v + 1);
    // Signal notifica Angular automaticamente, sem Zone.js
  }
}
```

---

<a name="11-signals"></a>
## 11. SIGNALS (Angular 16+)

### 11.1 O que são Signals?

Signals são **primitivos reativos** introduzidos no Angular 16. Alternativa granular aos Observables para gerenciar estado síncrono.

```typescript
import { Component, signal, computed, effect } from '@angular/core';

export class CounterComponent {
  // WritableSignal - leitura e escrita
  count = signal(0);
  
  // Computed - derivado, auto-recalcula (read-only)
  doubleCount = computed(() => this.count() * 2);
  isPositive = computed(() => this.count() > 0);
  
  // Múltiplos dependências
  message = computed(() => 
    `Count: ${this.count()}, Double: ${this.doubleCount()}, Positive: ${this.isPositive()}`
  );
  
  increment() {
    // set: substitui valor
    this.count.set(this.count() + 1);
    
    // update: baseado no valor atual (preferido)
    this.count.update(value => value + 1);
  }
  
  reset() {
    this.count.set(0);
  }
}
```

```html
<!-- Template - Note os () para ler -->
<div>Count: {{ count() }}</div>
<div>Double: {{ doubleCount() }}</div>
<button (click)="increment()">+1</button>
```

### 11.2 Signal vs BehaviorSubject

| Feature | Signal | BehaviorSubject |
|---------|--------|-----------------|
| **Leitura** | `count()` | `count$ \| async` ou `.value` |
| **Escrita** | `count.set(5)` / `.update()` | `count$.next(5)` |
| **Performance** | Melhor (granular) | Boa (Zone.js) |
| **Change Detection** | Só components afetados | Subárvore inteira |
| **Síncrono** | Sempre | `.value` sim, subscribe async |
| **RxJS Operators** | Não (mas interop) | Sim, todos |
| **Syntax** | Mais simples | Mais verbosa |
| **Memory leaks** | Não (auto-cleanup) | Sim (precisa unsubscribe) |
| **Melhor para** | UI state, derived state | Streams assíncronos, HTTP |

### 11.3 Signals com Arrays/Objects

```typescript
export class TodoComponent {
  todos = signal<Todo[]>([]);
  filter = signal<'all' | 'active' | 'done'>('all');
  
  // Computed derivado
  filteredTodos = computed(() => {
    const todos = this.todos();
    const filter = this.filter();
    
    switch (filter) {
      case 'active': return todos.filter(t => !t.done);
      case 'done': return todos.filter(t => t.done);
      default: return todos;
    }
  });
  
  activeCount = computed(() => this.todos().filter(t => !t.done).length);
  
  addTodo(title: string) {
    // ❌ ERRADO - mutação não dispara (mesma referência)
    this.todos().push({ id: Date.now(), title, done: false });
    
    // ✅ CERTO - nova referência
    this.todos.update(current => [
      ...current, 
      { id: Date.now(), title, done: false }
    ]);
  }
  
  toggleTodo(id: number) {
    this.todos.update(current =>
      current.map(t => t.id === id ? { ...t, done: !t.done } : t)
    );
  }
  
  removeTodo(id: number) {
    this.todos.update(current => current.filter(t => t.id !== id));
  }
}
```

### 11.4 toSignal / toObservable - Interoperabilidade

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

export class UserComponent {
  // ====== Observable → Signal ======
  
  // HTTP (completa automaticamente)
  users = toSignal(this.userService.getUsers(), {
    initialValue: [] as User[]  // Valor enquanto carrega
  });
  
  // Com loading state
  private usersResource = toSignal(this.userService.getUsers());
  users = computed(() => this.usersResource() ?? []);
  loading = computed(() => this.usersResource() === undefined);
  
  // ====== Signal → Observable (para usar RxJS) ======
  
  searchTerm = signal('');
  
  // Converte para Observable e aplica operadores RxJS
  results$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    filter(term => term.length >= 2),
    distinctUntilChanged(),
    switchMap(term => this.searchService.search(term))
  );
  
  // E pode converter de volta para Signal!
  results = toSignal(this.results$, { initialValue: [] });
}
```

### 11.5 Effect - Side Effects

```typescript
import { effect, untracked } from '@angular/core';

export class UserComponent {
  userId = signal(1);
  theme = signal<'light' | 'dark'>('light');
  
  constructor() {
    // Roda automaticamente quando signals que ele LÊ mudam
    effect(() => {
      console.log('User ID mudou:', this.userId());
      // ⚠️ Cuidado: NÃO altere outros signals dentro de effect (loop infinito)
    });
    
    // Effect com cleanup
    effect((onCleanup) => {
      const id = this.userId();
      const interval = setInterval(() => this.pollUser(id), 5000);
      
      onCleanup(() => clearInterval(interval));  // Limpa quando effect re-executa
    });
    
    // untracked - ler signal sem criar dependência
    effect(() => {
      const id = this.userId();  // Dependência (re-executa quando muda)
      const theme = untracked(() => this.theme());  // Não é dependência
      console.log(`User ${id}, theme: ${theme}`);
    });
  }
}
```

### 11.6 Signal Inputs (Angular 17.1+)

```typescript
// Novo: input como Signal (sem decorator)
import { input, output } from '@angular/core';

export class UserCardComponent {
  // Signal inputs
  name = input.required<string>();        // Obrigatório
  age = input(0);                         // Opcional com default
  role = input<string>('user');           // Tipado com default
  
  // Computed baseado em input
  displayName = computed(() => `${this.name()} (${this.age()})`);
  
  // Signal output
  deleted = output<number>();
  
  onDelete() {
    this.deleted.emit(42);
  }
}
```

### 11.7 Model Signals (Angular 17.2+ - Two-Way Binding)

```typescript
import { model } from '@angular/core';

// Filho
@Component({ selector: 'app-slider' })
export class SliderComponent {
  value = model(0);  // Two-way bindable signal
  
  increment() {
    this.value.update(v => v + 1);  // Propaga para pai
  }
}

// Pai
<app-slider [(value)]="myValue" />
```

---

<a name="12-standalone"></a>
## 12. STANDALONE COMPONENTS

### 12.1 Component Standalone

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-user',
  standalone: true,
  imports: [
    CommonModule,       // *ngIf, *ngFor, pipes
    FormsModule,        // ngModel
    RouterModule,       // routerLink
    UserCardComponent,  // Outros standalone components
    HighlightPipe,      // Standalone pipes
    TooltipDirective    // Standalone directives
  ],
  template: `
    <div *ngFor="let user of users">
      <app-user-card [user]="user" />
    </div>
  `
})
export class UserComponent {}
```

### 12.2 App Standalone (Sem AppModule)

```typescript
// main.ts - Bootstrap sem módulo
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimations } from '@angular/platform-browser/animations';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes, 
      withComponentInputBinding(),      // Bind route params para @Input
      withPreloading(PreloadAllModules)  // Preloading
    ),
    provideHttpClient(
      withInterceptors([authInterceptor, loggingInterceptor])
    ),
    provideAnimations(),
    // Outros providers...
  ]
});
```

### 12.3 Lazy Load Standalone

```typescript
const routes: Routes = [
  // Lazy load de component
  {
    path: 'admin',
    loadComponent: () => import('./admin.component').then(m => m.AdminComponent)
  },
  
  // Lazy load de rotas filhas
  {
    path: 'settings',
    loadChildren: () => import('./settings/settings.routes').then(m => m.SETTINGS_ROUTES)
  }
];

// settings.routes.ts
export const SETTINGS_ROUTES: Routes = [
  { path: '', component: SettingsLayoutComponent, children: [
    { path: 'profile', loadComponent: () => import('./profile.component').then(m => m.ProfileComponent) },
    { path: 'security', loadComponent: () => import('./security.component').then(m => m.SecurityComponent) },
  ]}
];
```

### 12.4 Migração Gradual (NgModule ↔ Standalone)

```typescript
// Usar standalone component em NgModule
@NgModule({
  imports: [
    StandaloneComponent  // ← Importa direto no imports!
  ]
})
export class LegacyModule {}

// Usar NgModule em standalone component
@Component({
  standalone: true,
  imports: [
    SharedModule  // ← Pode importar módulos inteiros
  ]
})
export class StandaloneComponent {}
```

---

<a name="13-http"></a>
## 13. HTTP & INTERCEPTORS

### 13.1 HttpClient Completo

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private apiUrl = `${environment.apiUrl}/users`;
  
  constructor(private http: HttpClient) {}
  
  // GET com tipagem
  getUsers(params?: { page?: number; limit?: number }): Observable<PaginatedResponse<User>> {
    let httpParams = new HttpParams();
    if (params?.page) httpParams = httpParams.set('page', params.page.toString());
    if (params?.limit) httpParams = httpParams.set('limit', params.limit.toString());
    
    return this.http.get<PaginatedResponse<User>>(this.apiUrl, { params: httpParams });
  }
  
  // GET com headers customizados
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`, {
      headers: new HttpHeaders({
        'X-Custom-Header': 'value'
      })
    });
  }
  
  // POST
  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
  
  // PUT (substituição total)
  replaceUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }
  
  // PATCH (atualização parcial)
  updateUser(id: number, changes: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, changes);
  }
  
  // DELETE
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
  
  // Upload com progress
  uploadAvatar(userId: number, file: File): Observable<HttpEvent<any>> {
    const formData = new FormData();
    formData.append('avatar', file);
    
    return this.http.post(`${this.apiUrl}/${userId}/avatar`, formData, {
      reportProgress: true,
      observe: 'events'  // Recebe eventos de progresso
    });
  }
  
  // Download blob
  downloadReport(id: number): Observable<Blob> {
    return this.http.get(`${this.apiUrl}/${id}/report`, {
      responseType: 'blob'
    });
  }
}
```

### 13.2 HTTP Interceptors

```typescript
// ===== Class-based Interceptor (legado mas ainda muito usado) =====

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    // Skip para URLs públicas
    if (req.url.includes('/public/')) {
      return next.handle(req);
    }
    
    const authReq = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
    
    return next.handle(authReq);
  }
}

// Registro (NgModule)
providers: [
  { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
  { provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true },
  // Ordem importa: Auth → Logging → ...
]

// ===== Functional Interceptor (Angular 15+ - preferido) =====

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();
  
  if (req.url.includes('/public/')) {
    return next(req);
  }
  
  const authReq = req.clone({
    setHeaders: { Authorization: `Bearer ${token}` }
  });
  
  return next(authReq);
};

// Error interceptor com retry e refresh token
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        // Tenta refresh token
        return authService.refreshToken().pipe(
          switchMap(newToken => {
            const retryReq = req.clone({
              setHeaders: { Authorization: `Bearer ${newToken}` }
            });
            return next(retryReq);
          }),
          catchError(() => {
            authService.logout();
            router.navigate(['/login']);
            return throwError(() => error);
          })
        );
      }
      return throwError(() => error);
    })
  );
};

// Loading interceptor
export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  const loadingService = inject(LoadingService);
  
  loadingService.show();
  return next(req).pipe(
    finalize(() => loadingService.hide())
  );
};

// Cache interceptor
export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  const cache = inject(HttpCacheService);
  
  // Só cacheia GET
  if (req.method !== 'GET') return next(req);
  
  const cached = cache.get(req.urlWithParams);
  if (cached) return of(cached);
  
  return next(req).pipe(
    tap(event => {
      if (event instanceof HttpResponse) {
        cache.set(req.urlWithParams, event, 60000); // TTL 1min
      }
    })
  );
};

// Registro (Standalone)
provideHttpClient(
  withInterceptors([authInterceptor, errorInterceptor, loadingInterceptor, cacheInterceptor])
)
```

---

<a name="14-testing"></a>
## 14. TESTING

### 14.1 Pirâmide de Testes

```
         /\
        /  \      E2E (Cypress/Playwright) - Poucos, lentos, alto custo
       /----\
      /      \    Integration - Médio, testa componentes juntos
     /--------\
    /          \  Unit - Muitos, rápidos, baixo custo
   /____________\
```

### 14.2 Teste de Service

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify();  // Garante que não há requests pendentes
  });
  
  it('should fetch users', () => {
    const mockUsers: User[] = [
      { id: 1, name: 'João', email: 'joao@test.com' }
    ];
    
    service.getUsers().subscribe(users => {
      expect(users.length).toBe(1);
      expect(users[0].name).toBe('João');
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);  // Simula resposta
  });
  
  it('should handle error', () => {
    service.getUsers().subscribe({
      error: err => {
        expect(err.message).toContain('Erro');
      }
    });
    
    const req = httpMock.expectOne('/api/users');
    req.flush('Error', { status: 500, statusText: 'Server Error' });
  });
  
  it('should create user with correct body', () => {
    const newUser = { name: 'Maria', email: 'maria@test.com' };
    
    service.createUser(newUser).subscribe();
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(newUser);
    req.flush({ id: 2, ...newUser });
  });
});
```

### 14.3 Teste de Component

```typescript
describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let mockUserService: jasmine.SpyObj<UserService>;
  
  const mockUsers: User[] = [
    { id: 1, name: 'João', email: 'joao@test.com' },
    { id: 2, name: 'Maria', email: 'maria@test.com' }
  ];
  
  beforeEach(async () => {
    mockUserService = jasmine.createSpyObj('UserService', ['getUsers', 'deleteUser']);
    mockUserService.getUsers.and.returnValue(of(mockUsers));
    mockUserService.deleteUser.and.returnValue(of(void 0));
    
    await TestBed.configureTestingModule({
      declarations: [UserListComponent, UserCardComponent],
      providers: [
        { provide: UserService, useValue: mockUserService }
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should load users on init', () => {
    fixture.detectChanges();  // Dispara ngOnInit
    
    expect(mockUserService.getUsers).toHaveBeenCalledTimes(1);
    expect(component.users.length).toBe(2);
  });
  
  it('should render user cards', () => {
    fixture.detectChanges();
    
    const cards = fixture.debugElement.queryAll(By.css('app-user-card'));
    expect(cards.length).toBe(2);
  });
  
  it('should display user names in the DOM', () => {
    fixture.detectChanges();
    
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.textContent).toContain('João');
    expect(compiled.textContent).toContain('Maria');
  });
  
  it('should call deleteUser when card emits delete event', () => {
    fixture.detectChanges();
    
    const card = fixture.debugElement.query(By.directive(UserCardComponent));
    card.triggerEventHandler('deleted', 1);
    
    expect(mockUserService.deleteUser).toHaveBeenCalledWith(1);
  });
  
  it('should show loading spinner while loading', () => {
    // Antes do subscribe
    const compiled = fixture.nativeElement as HTMLElement;
    fixture.detectChanges();
    
    // Teste de loading state depende da implementação
  });
  
  it('should show empty state when no users', () => {
    mockUserService.getUsers.and.returnValue(of([]));
    fixture.detectChanges();
    
    const emptyState = fixture.debugElement.query(By.css('.empty-state'));
    expect(emptyState).toBeTruthy();
  });
});
```

### 14.4 Teste de Pipe

```typescript
describe('TruncatePipe', () => {
  let pipe: TruncatePipe;
  
  beforeEach(() => {
    pipe = new TruncatePipe();
  });
  
  it('should not truncate short text', () => {
    expect(pipe.transform('Hello')).toBe('Hello');
  });
  
  it('should truncate at default limit', () => {
    const long = 'a'.repeat(60);
    const result = pipe.transform(long);
    expect(result.length).toBe(53);  // 50 + '...'
    expect(result.endsWith('...')).toBeTrue();
  });
  
  it('should use custom limit', () => {
    const result = pipe.transform('Hello World', 5);
    expect(result).toBe('Hello...');
  });
  
  it('should use custom ellipsis', () => {
    const result = pipe.transform('Hello World', 5, ' →');
    expect(result).toBe('Hello →');
  });
  
  it('should handle empty string', () => {
    expect(pipe.transform('')).toBe('');
  });
});
```

### 14.5 Teste de Directive

```typescript
@Component({
  template: `<div appHighlight [appHighlight]="color">Test</div>`
})
class TestHostComponent {
  color = 'yellow';
}

describe('HighlightDirective', () => {
  let fixture: ComponentFixture<TestHostComponent>;
  let divEl: DebugElement;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [HighlightDirective, TestHostComponent]
    });
    
    fixture = TestBed.createComponent(TestHostComponent);
    divEl = fixture.debugElement.query(By.directive(HighlightDirective));
    fixture.detectChanges();
  });
  
  it('should highlight on mouseenter', () => {
    divEl.triggerEventHandler('mouseenter', null);
    expect(divEl.nativeElement.style.backgroundColor).toBe('yellow');
  });
  
  it('should remove highlight on mouseleave', () => {
    divEl.triggerEventHandler('mouseenter', null);
    divEl.triggerEventHandler('mouseleave', null);
    expect(divEl.nativeElement.style.backgroundColor).toBe('');
  });
});
```

### 14.6 Teste de Guard

```typescript
describe('authGuard', () => {
  let authService: jasmine.SpyObj<AuthService>;
  let router: jasmine.SpyObj<Router>;
  
  beforeEach(() => {
    authService = jasmine.createSpyObj('AuthService', ['isLoggedIn']);
    router = jasmine.createSpyObj('Router', ['createUrlTree']);
    
    TestBed.configureTestingModule({
      providers: [
        { provide: AuthService, useValue: authService },
        { provide: Router, useValue: router }
      ]
    });
  });
  
  it('should allow access when logged in', () => {
    authService.isLoggedIn.and.returnValue(true);
    
    const result = TestBed.runInInjectionContext(() => 
      authGuard({} as any, { url: '/admin' } as any)
    );
    
    expect(result).toBeTrue();
  });
  
  it('should redirect to login when not logged in', () => {
    authService.isLoggedIn.and.returnValue(false);
    const urlTree = {} as any;
    router.createUrlTree.and.returnValue(urlTree);
    
    const result = TestBed.runInInjectionContext(() => 
      authGuard({} as any, { url: '/admin' } as any)
    );
    
    expect(result).toBe(urlTree);
    expect(router.createUrlTree).toHaveBeenCalledWith(['/login'], jasmine.any(Object));
  });
});
```

### 14.7 Dicas de Testing para Senior

```typescript
// Padrão: AAA (Arrange, Act, Assert)
it('should add product to cart', () => {
  // Arrange
  const product = { id: 1, name: 'Test', price: 10 };
  
  // Act
  component.addToCart(product);
  
  // Assert
  expect(cartService.add).toHaveBeenCalledWith(product);
});

// Teste async com fakeAsync
it('should debounce search', fakeAsync(() => {
  component.searchControl.setValue('ang');
  tick(100);  // Ainda no debounce
  expect(searchService.search).not.toHaveBeenCalled();
  
  tick(200);  // 300ms total - debounce passou
  expect(searchService.search).toHaveBeenCalledWith('ang');
}));

// Teste com Observable que não completa
it('should handle real-time updates', () => {
  const updates$ = new Subject<User>();
  mockService.onUpdate.and.returnValue(updates$);
  
  fixture.detectChanges();
  
  updates$.next({ id: 1, name: 'Updated' });
  fixture.detectChanges();
  
  expect(component.user.name).toBe('Updated');
});
```

---

<a name="15-performance"></a>
## 15. PERFORMANCE OPTIMIZATION

### 15.1 Checklist de Performance

```
□ OnPush em todos os components que puder
□ trackBy em todo *ngFor / track no @for
□ Lazy loading de módulos/components
□ Pure pipes ao invés de métodos no template
□ Virtual scrolling para listas grandes (100+)
□ Async pipe ao invés de subscribe manual
□ Preloading strategy para módulos críticos
□ Bundle analysis (webpack-bundle-analyzer)
□ Image optimization (NgOptimizedImage)
□ @defer para components pesados
□ SSR/Hydration para first paint rápido
```

### 15.2 Erros de Performance Mais Comuns

```typescript
// ❌ RUIM: Método no template (executa a cada change detection!)
<div *ngFor="let user of filterUsers(users, searchTerm)">
// Pode executar 100+ vezes por segundo!

// ✅ BOM: Pure pipe (executa só quando inputs mudam)
<div *ngFor="let user of users | filterUsers:searchTerm">

// ❌ RUIM: Getter complexo com Default change detection
get filteredUsers() {
  return this.users.filter(u => u.name.includes(this.term));
  // Recalcula a CADA change detection cycle
}

// ✅ BOM: Computed signal OU Observable com combineLatest
filteredUsers = computed(() => 
  this.users().filter(u => u.name.includes(this.term()))
);
```

### 15.3 Virtual Scrolling

```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  imports: [ScrollingModule],
  template: `
    <cdk-virtual-scroll-viewport 
      itemSize="72" 
      class="viewport"
      minBufferPx="400"
      maxBufferPx="800">
      
      <div *cdkVirtualFor="let item of items; trackBy: trackById" class="item">
        <app-user-card [user]="item" />
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport { height: 600px; }
    .item { height: 72px; }
  `]
})
export class LargeListComponent {
  items: User[] = [];  // Pode ter 100k items!
  
  trackById(index: number, item: User): number {
    return item.id;
  }
}
```

**Virtual scrolling renderiza apenas os itens visíveis na viewport. Lista de 100k items? Renderiza ~20.**

### 15.4 @defer - Lazy Loading de Components (Angular 17+)

```html
<!-- Carrega quando entra na viewport -->
@defer (on viewport) {
  <app-heavy-chart [data]="chartData" />
} @placeholder {
  <div class="skeleton-chart">Carregando gráfico...</div>
} @loading (minimum 500ms) {
  <app-spinner />
} @error {
  <p>Erro ao carregar gráfico</p>
}

<!-- Carrega quando idle (após initial render) -->
@defer (on idle) {
  <app-recommendations />
}

<!-- Carrega após interação -->
@defer (on interaction) {
  <app-comments [postId]="post.id" />
} @placeholder {
  <button>Carregar comentários</button>
}

<!-- Carrega após delay -->
@defer (on timer(3s)) {
  <app-analytics />
}

<!-- Combinar triggers -->
@defer (on viewport; on timer(5s)) {
  <app-footer-content />
}

<!-- Prefetch (baixa JS antes, renderiza depois) -->
@defer (on viewport; prefetch on idle) {
  <app-heavy-widget />
}
```

### 15.5 NgOptimizedImage (Angular 15+)

```typescript
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  template: `
    <!-- LCP image (prioritária) -->
    <img ngSrc="hero.jpg" width="1200" height="600" priority>
    
    <!-- Lazy (default) -->
    <img ngSrc="product.jpg" width="400" height="300">
    
    <!-- Fill mode (responsivo) -->
    <div class="image-container" style="position: relative;">
      <img ngSrc="background.jpg" fill>
    </div>
    
    <!-- Com loader customizado (CDN) -->
    <img ngSrc="photo.jpg" width="800" height="600">
  `
})
```

### 15.6 Bundle Analysis

```bash
# Analisar tamanho do bundle
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json

# Verificar o que está no bundle
# Procurar:
# - Bibliotecas grandes sendo importadas inteiras (moment.js, lodash)
# - Código duplicado
# - Módulos que deveriam ser lazy
```

### 15.7 Web Workers

```typescript
// Processamento pesado fora da thread principal
if (typeof Worker !== 'undefined') {
  const worker = new Worker(new URL('./heavy.worker', import.meta.url));
  
  worker.postMessage({ data: largeDataset, operation: 'sort' });
  
  worker.onmessage = ({ data }) => {
    this.sortedData = data.result;
  };
}

// heavy.worker.ts
addEventListener('message', ({ data }) => {
  const result = heavySort(data.data);
  postMessage({ result });
});
```

---

<a name="16-content-projection"></a>
## 16. CONTENT PROJECTION

### 16.1 Single Slot Projection

```typescript
// card.component.ts
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content></ng-content>  <!-- Tudo que estiver dentro de <app-card> -->
    </div>
  `
})
export class CardComponent {}

// Uso:
<app-card>
  <h2>Título</h2>
  <p>Conteúdo qualquer</p>
</app-card>
```

### 16.2 Multi-Slot Projection (Named Slots)

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="card-header">
        <ng-content select="[card-header]"></ng-content>
      </div>
      <div class="card-body">
        <ng-content></ng-content>  <!-- Default slot -->
      </div>
      <div class="card-footer">
        <ng-content select="[card-footer]"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent {}

// Uso:
<app-card>
  <div card-header>
    <h2>Título do Card</h2>
    <span class="badge">Novo</span>
  </div>
  
  <p>Este conteúdo vai pro slot default (body)</p>
  <p>Também vai pro body</p>
  
  <div card-footer>
    <button>Cancelar</button>
    <button>Salvar</button>
  </div>
</app-card>
```

### 16.3 Selectors Avançados

```typescript
@Component({
  selector: 'app-layout',
  template: `
    <!-- Por atributo -->
    <ng-content select="[slot='header']"></ng-content>
    
    <!-- Por tag de component -->
    <ng-content select="app-sidebar"></ng-content>
    
    <!-- Por classe CSS -->
    <ng-content select=".main-content"></ng-content>
    
    <!-- Por tag HTML -->
    <ng-content select="footer"></ng-content>
    
    <!-- Default (tudo que não combina com nenhum seletor) -->
    <ng-content></ng-content>
  `
})
```

### 16.4 Conditional Projection com @ContentChild

```typescript
@Component({
  selector: 'app-expandable',
  template: `
    <div class="header" (click)="toggle()">
      <ng-content select="[title]"></ng-content>
      <span>{{ isExpanded ? '▼' : '▶' }}</span>
    </div>
    
    @if (isExpanded) {
      <div class="body" @fadeIn>
        <ng-content select="[body]"></ng-content>
      </div>
    }
    
    <!-- Mostrar footer só se projetado -->
    @if (hasFooter) {
      <div class="footer">
        <ng-content select="[footer]"></ng-content>
      </div>
    }
  `
})
export class ExpandableComponent {
  @ContentChild('footer') footerContent?: ElementRef;
  
  isExpanded = false;
  
  get hasFooter(): boolean {
    return !!this.footerContent;
  }
  
  toggle() { this.isExpanded = !this.isExpanded; }
}
```

### 16.5 ngProjectAs

```typescript
// Quando o component pai usa um seletor específico mas você quer projetar algo diferente
<app-card>
  <!-- Sem ngProjectAs: ng-container não tem tag, então select="[card-header]" não pega -->
  <!-- Com ngProjectAs: finge ser um elemento com atributo card-header -->
  <ng-container ngProjectAs="[card-header]">
    <h2 *ngIf="showTitle">{{ title }}</h2>
  </ng-container>
</app-card>
```

---

<a name="17-viewchild"></a>
## 17. ViewChild & ContentChild

### 17.1 @ViewChild - Referência a Elementos/Components na View

```typescript
@Component({
  template: `
    <input #searchInput>
    <app-child-component></app-child-component>
    <div #container></div>
  `
})
export class ParentComponent implements AfterViewInit {
  // Referência a elemento DOM
  @ViewChild('searchInput') searchInput!: ElementRef<HTMLInputElement>;
  
  // Referência a component filho
  @ViewChild(ChildComponent) child!: ChildComponent;
  
  // Referência a ViewContainerRef (para components dinâmicos)
  @ViewChild('container', { read: ViewContainerRef }) container!: ViewContainerRef;
  
  // Static: true = disponível no ngOnInit (se não depender de *ngIf/*ngFor)
  @ViewChild('alwaysPresent', { static: true }) alwaysPresent!: ElementRef;
  
  // ⚠️ Disponível a partir de ngAfterViewInit (não no constructor/ngOnInit)
  ngAfterViewInit() {
    this.searchInput.nativeElement.focus();
    this.child.someMethod();
  }
}
```

### 17.2 @ViewChildren - Múltiplas Referências

```typescript
@Component({
  template: `
    <app-tab *ngFor="let tab of tabs" [data]="tab"></app-tab>
  `
})
export class TabsComponent implements AfterViewInit {
  @ViewChildren(TabComponent) tabComponents!: QueryList<TabComponent>;
  
  ngAfterViewInit() {
    // Iterar sobre todos
    this.tabComponents.forEach(tab => tab.deactivate());
    this.tabComponents.first.activate();
    
    // Reagir quando lista muda (add/remove tabs)
    this.tabComponents.changes.subscribe((tabs: QueryList<TabComponent>) => {
      console.log('Tabs mudaram:', tabs.length);
    });
  }
  
  activateTab(index: number) {
    const tabs = this.tabComponents.toArray();
    tabs.forEach(t => t.deactivate());
    tabs[index]?.activate();
  }
}
```

### 17.3 @ContentChild / @ContentChildren - Conteúdo Projetado

```typescript
// Container que acessa conteúdo projetado via <ng-content>
@Component({
  selector: 'app-accordion',
  template: `
    <div class="accordion">
      <ng-content select="app-accordion-item"></ng-content>
    </div>
  `
})
export class AccordionComponent implements AfterContentInit {
  @ContentChildren(AccordionItemComponent) items!: QueryList<AccordionItemComponent>;
  
  ngAfterContentInit() {
    // Disponível aqui (não no AfterViewInit)
    this.items.forEach((item, index) => {
      item.toggle.subscribe(() => this.closeOthers(index));
    });
    
    // Quando items mudam dinamicamente
    this.items.changes.subscribe(() => {
      // Re-setup
    });
  }
  
  closeOthers(openIndex: number) {
    this.items.forEach((item, i) => {
      if (i !== openIndex) item.close();
    });
  }
}

// Uso:
<app-accordion>
  <app-accordion-item *ngFor="let faq of faqs" [title]="faq.title">
    {{ faq.answer }}
  </app-accordion-item>
</app-accordion>
```

### 17.4 Diferença entre ViewChild e ContentChild

```
@ViewChild:   Elementos definidos no TEMPLATE do component
@ContentChild: Elementos PROJETADOS via <ng-content>

Template:     O que está no templateUrl/template do @Component
Content:      O que o PAI coloca entre <app-meu-component>AQUI</app-meu-component>
```

---

<a name="18-dynamic-components"></a>
## 18. DYNAMIC COMPONENTS

### 18.1 Criação Dinâmica com ViewContainerRef

```typescript
@Component({
  template: `
    <div #dynamicContainer></div>
    <button (click)="loadAlert()">Mostrar Alert</button>
    <button (click)="loadDialog()">Mostrar Dialog</button>
  `
})
export class DynamicHostComponent {
  @ViewChild('dynamicContainer', { read: ViewContainerRef }) container!: ViewContainerRef;
  
  loadAlert() {
    this.container.clear();  // Remove component anterior
    const componentRef = this.container.createComponent(AlertComponent);
    
    // Setar @Input
    componentRef.instance.message = 'Operação concluída!';
    componentRef.instance.type = 'success';
    
    // Escutar @Output
    componentRef.instance.closed.subscribe(() => {
      componentRef.destroy();
    });
  }
  
  loadDialog() {
    this.container.clear();
    const ref = this.container.createComponent(DialogComponent);
    ref.instance.title = 'Confirmar';
    ref.instance.confirmed.subscribe(() => this.onConfirm());
  }
}
```

### 18.2 Lazy Load de Component Dinâmico

```typescript
async loadHeavyComponent() {
  this.container.clear();
  
  // Importação dinâmica - só baixa quando necessário
  const { HeavyChartComponent } = await import('./heavy-chart.component');
  const ref = this.container.createComponent(HeavyChartComponent);
  ref.instance.data = this.chartData;
}
```

### 18.3 Component Factory Pattern

```typescript
// Registry de components
@Injectable({ providedIn: 'root' })
export class WidgetRegistry {
  private widgets = new Map<string, Type<any>>();
  
  register(type: string, component: Type<any>) {
    this.widgets.set(type, component);
  }
  
  get(type: string): Type<any> | undefined {
    return this.widgets.get(type);
  }
}

// Host que renderiza baseado em config
@Component({
  selector: 'app-widget-host',
  template: `<ng-container #widgetContainer></ng-container>`
})
export class WidgetHostComponent implements OnInit, OnChanges {
  @Input() config!: WidgetConfig;
  @ViewChild('widgetContainer', { read: ViewContainerRef, static: true }) 
  container!: ViewContainerRef;
  
  constructor(private registry: WidgetRegistry) {}
  
  ngOnChanges() {
    this.loadWidget();
  }
  
  private loadWidget() {
    this.container.clear();
    const componentType = this.registry.get(this.config.type);
    
    if (componentType) {
      const ref = this.container.createComponent(componentType);
      Object.assign(ref.instance, this.config.props);
    }
  }
}

// Uso: Dashboard com widgets configuráveis
<app-widget-host 
  *ngFor="let widget of dashboardConfig"
  [config]="widget">
</app-widget-host>
```

---

<a name="19-animations"></a>
## 19. ANIMATIONS

### 19.1 Setup

```typescript
// Importar no módulo ou providers
import { provideAnimations } from '@angular/platform-browser/animations';
// OU para lazy: provideAnimationsAsync()

bootstrapApplication(AppComponent, {
  providers: [provideAnimations()]
});
```

### 19.2 Animações Básicas

```typescript
import { trigger, state, style, transition, animate, query, stagger, group } from '@angular/animations';

@Component({
  animations: [
    // Fade in/out
    trigger('fadeInOut', [
      transition(':enter', [
        style({ opacity: 0 }),
        animate('300ms ease-in', style({ opacity: 1 }))
      ]),
      transition(':leave', [
        animate('300ms ease-out', style({ opacity: 0 }))
      ])
    ]),
    
    // Slide
    trigger('slideIn', [
      transition(':enter', [
        style({ transform: 'translateX(-100%)', opacity: 0 }),
        animate('400ms ease-out', style({ transform: 'translateX(0)', opacity: 1 }))
      ]),
      transition(':leave', [
        animate('400ms ease-in', style({ transform: 'translateX(100%)', opacity: 0 }))
      ])
    ]),
    
    // State-based
    trigger('expandCollapse', [
      state('collapsed', style({ height: '0px', overflow: 'hidden', opacity: 0 })),
      state('expanded', style({ height: '*', opacity: 1 })),
      transition('collapsed <=> expanded', animate('300ms ease-in-out'))
    ]),
    
    // Staggered list animation
    trigger('listAnimation', [
      transition('* => *', [
        query(':enter', [
          style({ opacity: 0, transform: 'translateY(20px)' }),
          stagger(50, [
            animate('300ms ease-out', style({ opacity: 1, transform: 'translateY(0)' }))
          ])
        ], { optional: true })
      ])
    ])
  ],
  template: `
    <div @fadeInOut *ngIf="showContent">Fade content</div>
    
    <div [@expandCollapse]="isExpanded ? 'expanded' : 'collapsed'">
      Expandable content
    </div>
    
    <div [@listAnimation]="items.length">
      <div *ngFor="let item of items" @slideIn>{{ item }}</div>
    </div>
  `
})
export class AnimatedComponent {
  showContent = true;
  isExpanded = false;
  items: string[] = [];
}
```

### 19.3 Animações Reutilizáveis

```typescript
// animations.ts - Arquivo separado
import { trigger, transition, style, animate } from '@angular/animations';

export const fadeIn = trigger('fadeIn', [
  transition(':enter', [
    style({ opacity: 0 }),
    animate('300ms ease-in', style({ opacity: 1 }))
  ])
]);

export const slideInFromLeft = trigger('slideInFromLeft', [
  transition(':enter', [
    style({ transform: 'translateX(-100%)' }),
    animate('400ms ease-out')
  ])
]);

// Uso em qualquer component
@Component({
  animations: [fadeIn, slideInFromLeft]
})
```

### 19.4 Route Animations

```typescript
// Animar transições de rota
const routeAnimation = trigger('routeAnimation', [
  transition('* <=> *', [
    query(':enter, :leave', style({ position: 'absolute', width: '100%' }), { optional: true }),
    group([
      query(':leave', [
        animate('300ms ease-out', style({ opacity: 0, transform: 'translateX(-50px)' }))
      ], { optional: true }),
      query(':enter', [
        style({ opacity: 0, transform: 'translateX(50px)' }),
        animate('300ms ease-out', style({ opacity: 1, transform: 'translateX(0)' }))
      ], { optional: true })
    ])
  ])
]);

@Component({
  animations: [routeAnimation],
  template: `
    <div [@routeAnimation]="getRouteAnimationData()">
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent {
  getRouteAnimationData() {
    return this.route.snapshot.firstChild?.data?.['animation'];
  }
}

// Nas rotas
{ path: 'home', component: HomeComponent, data: { animation: 'Home' } },
{ path: 'about', component: AboutComponent, data: { animation: 'About' } }
```

---

<a name="20-ssr"></a>
## 20. SSR (Server-Side Rendering)

### 20.1 O que é SSR? (Explicação Profunda)

**SSR (Server-Side Rendering)** é uma técnica em que o HTML da página é gerado no **servidor** (Node.js) a cada requisição, ao invés de ser montado no browser pelo JavaScript. O servidor executa o código Angular, renderiza os components em HTML puro, e envia esse HTML pronto para o browser.

**O problema que SSR resolve:**

Em uma SPA (Single Page Application) tradicional, quando o browser faz uma requisição, o servidor retorna um HTML quase vazio — apenas um `<app-root></app-root>` e tags `<script>`. O browser precisa então baixar todo o JavaScript, parsear, executar o framework Angular, montar a árvore de components, fazer chamadas HTTP para buscar dados, e só então renderizar o conteúdo visível. Até tudo isso acontecer, o usuário vê uma tela em branco (ou um spinner).

Com SSR, o servidor já faz todo esse trabalho. O HTML que chega no browser já contém o conteúdo renderizado — textos, listas, imagens, tudo visível. O usuário vê o conteúdo imediatamente.

### 20.2 Como Funciona por Dentro (Fluxo Completo)

**Fluxo SPA (sem SSR):**
```
1. Browser faz GET /products
2. Servidor retorna HTML mínimo:
   <html>
     <body>
       <app-root></app-root>      ← Vazio!
       <script src="main.js">     ← 500KB+ de JS
     </body>
   </html>
3. Browser baixa main.js (500KB-2MB)
4. Browser parseia e executa JavaScript
5. Angular inicializa (bootstrap)
6. Angular monta a árvore de components
7. ProductListComponent faz HTTP GET /api/products
8. API retorna dados
9. Angular renderiza a lista no DOM
10. Usuário finalmente VÊ o conteúdo       ← 3-8 segundos depois!

Timeline:
[---download JS---][--parse--][--execute--][--HTTP--][render]
                                                          ↑ FCP (First Contentful Paint)
```

**Fluxo SSR:**
```
1. Browser faz GET /products
2. Servidor Node.js recebe a requisição
3. Servidor executa Angular em memória (sem browser real)
4. Angular no servidor monta a árvore de components
5. ProductListComponent faz HTTP GET /api/products (servidor → API, rede local = rápido!)
6. Angular renderiza tudo em HTML string
7. Servidor retorna HTML COMPLETO:
   <html>
     <body>
       <app-root>
         <h1>Produtos</h1>         ← Conteúdo já pronto!
         <div class="product">
           <h2>Notebook Dell</h2>
           <p>R$ 4.500,00</p>
         </div>
         <div class="product">
           <h2>Mouse Logitech</h2>
           <p>R$ 150,00</p>
         </div>
       </app-root>
       <script src="main.js">
     </body>
   </html>
8. Browser renderiza HTML imediatamente    ← Usuário VÊ o conteúdo em <1 segundo!
9. Em paralelo, browser baixa e executa JS
10. Angular "assume" o HTML (hydration)    ← Agora tem interatividade

Timeline:
[server render][---HTML pronto + visível---]
               [---download JS em paralelo---][hydration]
               ↑ FCP (muito mais rápido!)              ↑ TTI (Time to Interactive)
```

**Ponto-chave para entrevista:** Com SSR, o FCP (First Contentful Paint) é muito mais rápido porque o HTML já vem pronto. Porém, o TTI (Time to Interactive) pode ser similar ao SPA, porque o JavaScript ainda precisa ser baixado e executado para que botões, forms e interações funcionem.

### 20.3 O que Acontece no Servidor (Angular Universal/SSR Engine)

O Angular SSR usa o **Angular Universal** (até Angular 16) ou o **@angular/ssr** (Angular 17+) para executar components no servidor. O mecanismo funciona assim:

```
┌─────────────────────────────────────────────────────────┐
│                   Servidor Node.js                       │
│                                                          │
│  1. Express recebe requisição (GET /products)            │
│                    ↓                                     │
│  2. CommonEngine cria um "browser virtual" em memória    │
│     (usa Domino - uma implementação de DOM para Node.js) │
│                    ↓                                     │
│  3. Angular bootstrapa a app nesse DOM virtual           │
│     - Cria AppComponent                                  │
│     - Resolve rotas (Router)                             │
│     - Instancia ProductListComponent                     │
│     - Executa ngOnInit → chama HttpClient                │
│                    ↓                                     │
│  4. HttpClient faz requisição real para a API            │
│     (servidor → API = rede interna, muito rápido)        │
│                    ↓                                     │
│  5. Dados chegam, Angular renderiza no DOM virtual       │
│                    ↓                                     │
│  6. Angular serializa o DOM virtual em string HTML       │
│                    ↓                                     │
│  7. Express envia essa string como response              │
│                                                          │
└─────────────────────────────────────────────────────────┘
                           ↓
                    HTML completo vai
                    para o browser
```

```typescript
// server.ts - Como o servidor processa cada requisição
import { CommonEngine } from '@angular/ssr';
import express from 'express';
import { dirname, join, resolve } from 'node:path';
import { fileURLToPath } from 'node:url';
import bootstrap from './src/main.server';

const app = express();
const serverDistFolder = dirname(fileURLToPath(import.meta.url));
const browserDistFolder = resolve(serverDistFolder, '../browser');
const indexHtml = join(serverDistFolder, 'index.server.html');
const commonEngine = new CommonEngine();

// Servir arquivos estáticos (JS, CSS, imagens)
app.use(express.static(browserDistFolder, { maxAge: '1y', index: false }));

// Todas as rotas passam pelo Angular SSR
app.get('**', (req, res, next) => {
  commonEngine
    .render({
      bootstrap,                    // Entry point do Angular
      documentFilePath: indexHtml,  // Template HTML base
      url: req.originalUrl,         // URL que o user acessou
      publicPath: browserDistFolder,
      providers: [
        { provide: 'REQUEST', useValue: req },    // Acesso ao request no Angular
        { provide: 'RESPONSE', useValue: res },
      ],
    })
    .then((html) => res.send(html))    // Envia HTML renderizado
    .catch((err) => next(err));
});

app.listen(4000, () => {
  console.log('SSR server rodando em http://localhost:4000');
});
```

### 20.4 Diferença Entre CSR, SSR, SSG e ISR

| Estratégia | Quando Renderiza | Onde Renderiza | Melhor Para |
|-----------|-----------------|---------------|-------------|
| **CSR** (Client-Side) | No browser, após JS carregar | Browser | Dashboards, apps internas |
| **SSR** (Server-Side) | A cada requisição | Servidor Node.js | E-commerce, conteúdo dinâmico |
| **SSG** (Static Generation) | No build (1x) | Build pipeline | Blog, docs, landing pages |
| **ISR** (Incremental Static) | No build + revalida | Servidor + cache | Catálogos com atualização periódica |

```
CSR:  Build → HTML vazio → Browser monta tudo
SSR:  Cada request → Servidor renderiza → HTML pronto → Browser hydrate
SSG:  Build → Gera HTML de cada rota → Deploy estático → Sem servidor
ISR:  Build → Gera HTML → Serve do cache → Revalida a cada N segundos
```

### 20.5 Comparação Detalhada

| Aspecto | SPA/CSR | SSR | SSG |
|---------|---------|-----|-----|
| **FCP (First Contentful Paint)** | Lento (3-8s) | Rápido (<1s) | Muito rápido (CDN) |
| **TTI (Time to Interactive)** | = FCP | FCP + hydration | FCP + hydration |
| **TTFB (Time to First Byte)** | Rápido (HTML pequeno) | Mais lento (servidor renderiza) | Muito rápido (estático) |
| **SEO** | Ruim (HTML vazio) | Excelente | Excelente |
| **Social Sharing (OG tags)** | Não funciona | Funciona | Funciona |
| **Dados dinâmicos** | Sim (API calls) | Sim (fresh a cada request) | Não (dados do build) |
| **Infraestrutura** | Estática (CDN, S3) | Node.js server | Estática (CDN, S3) |
| **Custo** | Baixo | Médio-Alto (servidor) | Baixo |
| **Escalabilidade** | Alta (CDN) | Precisa escalar servidor | Alta (CDN) |
| **Cache** | Só assets | Pode cachear HTML | 100% cacheável |

### 20.6 Setup (Angular 17+)

```bash
# Novo projeto já com SSR
ng new my-app --ssr

# Adicionar SSR em projeto existente
ng add @angular/ssr
```

O `ng add @angular/ssr` gera automaticamente:
- `server.ts` — servidor Express com Angular SSR
- `src/main.server.ts` — entry point para server-side
- `src/app/app.config.server.ts` — config do Angular para servidor
- Atualiza `angular.json` com configurações de build SSR

```typescript
// src/app/app.config.server.ts
import { mergeApplicationConfig, ApplicationConfig } from '@angular/core';
import { provideServerRendering } from '@angular/platform-server';
import { appConfig } from './app.config';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering()
  ]
};

export const config = mergeApplicationConfig(appConfig, serverConfig);
```

```typescript
// src/main.server.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { config } from './app/app.config.server';

const bootstrap = () => bootstrapApplication(AppComponent, config);
export default bootstrap;
```

### 20.7 Cuidados com SSR (APIs do Browser)

No servidor Node.js, **não existem** `window`, `document`, `localStorage`, `navigator`, `alert`, `setTimeout` (parcial) e outras APIs de browser. Acessar essas APIs no servidor causa crash.

```typescript
import { isPlatformBrowser, isPlatformServer, DOCUMENT } from '@angular/common';
import { PLATFORM_ID, inject } from '@angular/core';

export class MyComponent implements OnInit {
  private platformId = inject(PLATFORM_ID);
  private document = inject(DOCUMENT);  // Abstração segura para document
  
  ngOnInit() {
    // ❌ CRASH no servidor! Essas APIs não existem no Node.js:
    window.scrollTo(0, 0);
    localStorage.setItem('key', 'value');
    document.getElementById('el');
    navigator.geolocation.getCurrentPosition(...);
    
    // ✅ Verificar plataforma antes
    if (isPlatformBrowser(this.platformId)) {
      window.scrollTo(0, 0);
      localStorage.setItem('key', 'value');
      
      // Inicializar libs que dependem do browser
      this.initGoogleMaps();
      this.initAnalytics();
    }
    
    // ✅ Usar DOCUMENT token (funciona em ambos ambientes)
    const title = this.document.title;
  }
}
```

**afterNextRender / afterRender (Angular 16+) — A forma moderna:**

```typescript
import { afterNextRender, afterRender } from '@angular/core';

export class ChartComponent {
  constructor() {
    // Roda 1x após o PRIMEIRO render no browser
    // NUNCA roda no servidor — sem necessidade de if(isPlatformBrowser)
    afterNextRender(() => {
      // Seguro para acessar DOM, window, localStorage, etc
      this.initChart();
      this.setupResizeObserver();
    });
  }
}

export class AutoResizeComponent {
  constructor() {
    // Roda após CADA render no browser (como AfterViewChecked, mas só no browser)
    afterRender(() => {
      this.adjustHeight();
    });
  }
}
```

**Lista completa de APIs que NÃO existem no servidor:**

```
❌ window         → Use isPlatformBrowser ou afterNextRender
❌ document       → Use inject(DOCUMENT) ou Renderer2
❌ localStorage   → Use TransferState ou cookie
❌ sessionStorage → Use TransferState
❌ navigator      → Verificar plataforma
❌ alert/confirm  → Não usar
❌ requestAnimationFrame → Use afterRender
❌ IntersectionObserver  → Use afterNextRender
❌ ResizeObserver → Use afterNextRender
❌ WebSocket      → Inicializar só no browser
❌ fetch (global) → Use HttpClient (funciona em ambos)
```

### 20.8 TransferState — Evitar Requisições Duplicadas

**O problema:** Sem TransferState, a API é chamada 2 vezes — uma no servidor (para gerar HTML) e outra no browser (quando Angular re-inicializa). TransferState permite que o servidor passe dados para o browser, evitando a duplicação.

```
SEM TransferState:
Servidor: Angular executa → HTTP GET /api/products → renderiza HTML com dados
Browser:  Angular re-inicializa → HTTP GET /api/products DE NOVO → re-renderiza
                                   ↑ Duplicado e desnecessário!

COM TransferState:
Servidor: Angular executa → HTTP GET /api/products → renderiza HTML → salva dados no HTML
Browser:  Angular re-inicializa → encontra dados no HTML → usa direto, SEM novo HTTP
```

```typescript
// Angular 16+ com provideClientHydration: TransferState é AUTOMÁTICO para HttpClient!
// Basta ativar hydration:

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(),  // ← Ativa hydration + TransferState automático
    provideHttpClient()
  ]
});

// O Angular automaticamente:
// 1. No servidor: faz o HTTP call e embute o resultado no HTML como <script type="application/json">
// 2. No browser: intercepta o HTTP call e retorna os dados do HTML, sem fazer nova requisição

// =============================================================
// Para dados NÃO-HTTP (custom), usar TransferState manualmente:
// =============================================================

import { TransferState, makeStateKey } from '@angular/core';

const PRODUCTS_KEY = makeStateKey<Product[]>('products');

@Injectable({ providedIn: 'root' })
export class ProductService {
  constructor(
    private http: HttpClient,
    private transferState: TransferState,
    @Inject(PLATFORM_ID) private platformId: Object
  ) {}
  
  getProducts(): Observable<Product[]> {
    // 1. Verificar se dados já vieram do servidor
    if (this.transferState.hasKey(PRODUCTS_KEY)) {
      const products = this.transferState.get(PRODUCTS_KEY, []);
      this.transferState.remove(PRODUCTS_KEY);  // Limpar para não ficar stale
      return of(products);
    }
    
    // 2. Se não tem (ou é o servidor), buscar da API
    return this.http.get<Product[]>('/api/products').pipe(
      tap(products => {
        // 3. Se é o servidor, salvar para o browser
        if (isPlatformServer(this.platformId)) {
          this.transferState.set(PRODUCTS_KEY, products);
        }
      })
    );
  }
}
```

**Como os dados são transferidos fisicamente:**
```html
<!-- O Angular SSR embute os dados em um script tag no HTML -->
<html>
  <body>
    <app-root>
      <!-- ... HTML renderizado ... -->
    </app-root>
    
    <!-- TransferState: dados serializados como JSON -->
    <script id="serverApp-state" type="application/json">
      {"products": [{"id":1,"name":"Notebook","price":4500}, ...]}
    </script>
    
    <script src="main.js"></script>
  </body>
</html>
<!-- O Angular no browser lê esse JSON antes de fazer qualquer HTTP call -->
```

### 20.9 Hydration (Angular 16+) — Explicação Detalhada

**O problema antes do Hydration:**

Antes do Angular 16, quando o browser recebia o HTML do SSR e o JavaScript carregava, o Angular **destruía todo o DOM renderizado pelo servidor** e re-criava do zero. Isso causava:
- Flash/flicker visível (conteúdo sumia e reaparecia)
- Perda de estado do DOM (scroll position, foco de input, seleção de texto)
- Re-trigger de animações CSS
- Desperdício de trabalho (renderizou 2 vezes)

**Como Hydration resolve:**

Com Hydration, o Angular no browser **não destrói o HTML do servidor**. Ao invés disso, ele "caminha" pelo DOM existente, reconhece cada element como correspondente a um component, e **anexa os event listeners e a lógica** ao DOM que já existe.

```
SEM Hydration (Angular <16):
Servidor: renderiza HTML → envia
Browser:  recebe HTML → mostra → carrega JS → DESTRÓI todo o DOM → recria tudo do zero
                                                ↑ Flash! Usuário vê conteúdo sumir e voltar

COM Hydration (Angular 16+):
Servidor: renderiza HTML + marca nodes → envia
Browser:  recebe HTML → mostra → carrega JS → PRESERVA o DOM → anexa listeners
                                                ↑ Sem flash! Transição invisível
```

```typescript
// Ativar hydration
bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(
      withEventReplay(),           // Angular 18+: replay de eventos durante hydration
      withIncrementalHydration()   // Angular 19+: hydration incremental com @defer
    )
  ]
});
```

**Event Replay (Angular 18+):**

Se o usuário clicar em um botão ANTES do JavaScript carregar e hydration completar, o clique normalmente seria perdido. Com `withEventReplay()`, o Angular grava esses eventos e os "replaya" assim que hydration termina.

```
Sem Event Replay:
Usuário vê botão "Comprar" → clica → nada acontece (JS não carregou) → frustração

Com Event Replay:
Usuário vê botão "Comprar" → clica → evento é gravado → JS carrega → hydration → evento é replayado → compra funciona!
```

**Como Hydration funciona por dentro:**

```
1. Servidor renderiza HTML e adiciona atributos especiais:
   <div ngh="0">
     <app-product-list ngh="1">
       <div ngh="2" ngSkipHydration>...</div>
     </app-product-list>
   </div>
   
   ngh = "Angular Hydration Node" — identifica qual component corresponde a qual DOM node

2. Servidor embute um mapa de serialização no HTML:
   <script type="application/json">
     { "0": { "type": "AppComponent", "children": [1] },
       "1": { "type": "ProductListComponent", "children": [2] } }
   </script>

3. Browser carrega JS, Angular lê o mapa:
   - Encontra <div ngh="0"> → associa ao AppComponent (SEM recriar)
   - Encontra <app-product-list ngh="1"> → associa ao ProductListComponent (SEM recriar)
   - Adiciona event listeners (click, input, etc) nos elements existentes
   
4. Resultado: DOM permanece intacto, mas agora tem interatividade
```

**ngSkipHydration — Pular hydration para partes específicas:**

```html
<!-- Quando um component third-party manipula DOM diretamente (ex: lib de gráfico) -->
<div ngSkipHydration>
  <third-party-chart [data]="chartData"></third-party-chart>
</div>
<!-- Angular vai destruir e recriar essa parte normalmente -->
```

### 20.10 SSG (Static Site Generation) — Pre-Rendering

SSG gera HTML estático no momento do build. Diferente do SSR, não precisa de servidor Node.js em runtime — os arquivos HTML são servidos diretamente de um CDN.

```typescript
// angular.json — configurar pre-rendering
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "prerender": {
                "discoverRoutes": true,        // Descobre rotas automaticamente
                "routesFile": "prerender-routes.txt"  // OU lista manual
              }
            }
          }
        }
      }
    }
  }
}
```

```
// prerender-routes.txt — rotas para gerar estaticamente
/
/about
/contact
/blog/post-1
/blog/post-2
/products/1
/products/2
```

```bash
# Build gera HTML para cada rota
ng build

# Resultado em dist/:
dist/my-app/browser/
├── index.html          ← HTML de /
├── about/index.html    ← HTML de /about
├── blog/
│   ├── post-1/index.html
│   └── post-2/index.html
└── products/
    ├── 1/index.html
    └── 2/index.html

# Cada arquivo é HTML completo com conteúdo renderizado
# Pode ser servido por qualquer CDN/servidor estático (Nginx, S3, Cloudflare)
```

**Quando usar SSG vs SSR:**

```
SSG (Static):
✅ Conteúdo que muda raramente (blog, docs, landing pages, FAQ)
✅ Pode ser servido em CDN (máxima performance e escala)
✅ Não precisa de Node.js em produção
❌ Dados ficam estáticos até próximo build
❌ Não serve para conteúdo personalizado por usuário

SSR (Dynamic):
✅ Conteúdo que muda a cada request (e-commerce, dashboard, feed)
✅ Pode ter dados personalizados por usuário/sessão
✅ SEO com dados sempre frescos
❌ Precisa de servidor Node.js rodando
❌ Cada request consome CPU do servidor
❌ Mais complexo de escalar (precisa load balancer, caching)
```

### 20.11 Patterns de SSR para Produção

**Cache de SSR responses:**
```typescript
// Cache HTML renderizado para evitar re-render a cada request
import NodeCache from 'node-cache';

const ssrCache = new NodeCache({ stdTTL: 60 }); // 60 segundos

app.get('**', (req, res, next) => {
  const cacheKey = req.originalUrl;
  const cached = ssrCache.get<string>(cacheKey);
  
  if (cached) {
    return res.send(cached);  // Serve do cache
  }
  
  commonEngine.render({ ... }).then(html => {
    // Não cachear páginas que dependem de auth
    if (!req.headers.cookie?.includes('auth_token')) {
      ssrCache.set(cacheKey, html);
    }
    res.send(html);
  });
});
```

**SEO: Meta tags dinâmicas com SSR:**
```typescript
import { Meta, Title } from '@angular/platform-browser';

@Component({ ... })
export class ProductDetailComponent implements OnInit {
  constructor(
    private meta: Meta,
    private title: Title,
    private route: ActivatedRoute
  ) {}
  
  ngOnInit() {
    this.route.data.subscribe(({ product }) => {
      // Essas tags são renderizadas no HTML do servidor!
      // Crawlers e social media bots as lêem corretamente
      this.title.setTitle(`${product.name} | Minha Loja`);
      this.meta.updateTag({ name: 'description', content: product.description });
      this.meta.updateTag({ property: 'og:title', content: product.name });
      this.meta.updateTag({ property: 'og:description', content: product.description });
      this.meta.updateTag({ property: 'og:image', content: product.image });
      this.meta.updateTag({ property: 'og:type', content: 'product' });
    });
  }
}
```

**Lidar com timeouts no servidor:**
```typescript
// Se uma API demora muito, o SSR pode travar
// Implementar timeout para não bloquear o servidor

app.get('**', (req, res, next) => {
  const renderTimeout = setTimeout(() => {
    // Fallback: retorna o HTML do CSR (sem dados)
    res.sendFile(join(browserDistFolder, 'index.html'));
  }, 5000); // 5 segundos máximo
  
  commonEngine.render({ ... }).then(html => {
    clearTimeout(renderTimeout);
    res.send(html);
  }).catch(err => {
    clearTimeout(renderTimeout);
    // Fallback para CSR em caso de erro
    res.sendFile(join(browserDistFolder, 'index.html'));
  });
});
```

### 20.12 Quando NÃO Usar SSR

```
❌ Apps internas/administrativas (não precisa de SEO)
❌ Apps atrás de login (crawlers não vão ver o conteúdo de qualquer forma)
❌ Dashboards com dados 100% dinâmicos/personalizados
❌ Quando o time não tem experiência com Node.js em produção
❌ Quando o custo de infraestrutura não justifica
❌ Apps simples onde performance de CSR já é aceitável

✅ QUANDO USAR SSR:
✅ E-commerce (SEO é crítico, social sharing de produtos)
✅ Blogs, portais de conteúdo (SEO + social sharing)
✅ Landing pages de marketing (performance do first paint)
✅ Apps que precisam funcionar com JavaScript desabilitado
✅ Quando Core Web Vitals (LCP, FID, CLS) são KPIs do projeto
```

### 20.13 Perguntas de Entrevista sobre SSR

**P:** "Explique como SSR funciona no Angular."  
**R:** "O servidor Node.js executa o código Angular em memória usando um DOM virtual (Domino). A cada requisição, ele bootstrapa a app, resolve a rota, executa os components (incluindo chamadas HTTP), renderiza tudo em uma string HTML, e retorna esse HTML pronto. O browser exibe o conteúdo imediatamente. Depois, o JavaScript carrega e o Angular faz hydration — associa a lógica aos elements DOM que já existem, sem recriar."

**P:** "O que é Hydration?"  
**R:** "Hydration é o processo onde o Angular no browser 'assume' o HTML gerado pelo servidor. Ao invés de destruir e recriar o DOM (como fazia antes do Angular 16), ele caminha pelo DOM existente, reconhece cada element, e anexa event listeners e reatividade. O resultado é uma transição invisível de HTML estático para app interativa, sem flash ou flicker."

**P:** "O que é TransferState?"  
**R:** "É o mecanismo que evita requisições HTTP duplicadas. Sem ele, a API seria chamada no servidor (para gerar HTML) e novamente no browser (quando Angular reinicializa). TransferState serializa os dados do servidor em um script tag no HTML, e o Angular no browser lê esses dados ao invés de fazer um novo HTTP call. No Angular 16+, isso é automático quando hydration está ativo."

**P:** "SSR vs SSG, quando usar cada um?"  
**R:** "SSG gera HTML no build, ideal para conteúdo estático como blog e docs — sem servidor em runtime, servido em CDN. SSR renderiza a cada request, ideal para conteúdo dinâmico como e-commerce — precisa de servidor Node.js mas os dados são sempre frescos. Pode combinar os dois: SSG para páginas estáticas e SSR para páginas dinâmicas."

---

<a name="21-pwa"></a>
## 21. PWA (Progressive Web Apps)

### 21.1 O que é uma PWA?

Uma PWA é uma aplicação web que usa tecnologias modernas do browser para oferecer uma experiência próxima de um app nativo. As três tecnologias-chave são:

- **Service Worker** — Script que roda em background, separado da thread principal. Intercepta requisições de rede, gerencia cache, e permite funcionamento offline.
- **Web App Manifest** — Arquivo JSON que descreve o app (nome, ícones, cores, orientação). Permite que o browser ofereça "Adicionar à tela inicial".
- **HTTPS** — Obrigatório. Service Workers só funcionam em conexões seguras (exceto localhost para dev).

**O que uma PWA permite que uma SPA comum não permite:**
```
SPA comum:
- Funciona só com internet
- Não aparece na tela inicial do celular
- Não recebe push notifications
- Não atualiza em background
- Não cacheia assets inteligentemente

PWA:
✅ Funciona offline (ou com conexão ruim)
✅ Instalável na tela inicial (sem app store)
✅ Pode receber push notifications
✅ Atualiza em background automaticamente
✅ Cache inteligente de assets e dados de API
✅ Splash screen customizada ao abrir
✅ Pode rodar em tela cheia (sem barra do browser)
```

### 21.2 Como o Service Worker Funciona por Dentro

```
Fluxo NORMAL (sem Service Worker):
Browser → Request HTTP → Internet → Servidor → Response → Browser renderiza

Fluxo com Service Worker:
Browser → Request HTTP → Service Worker INTERCEPTA
                              ↓
                    Tem no cache? ──── SIM → Retorna do cache (instantâneo!)
                         │
                        NÃO
                         │
                         ↓
                    Vai para Internet → Servidor → Response
                                                      ↓
                                        Salva no cache + retorna para browser
```

O Service Worker atua como um **proxy** entre o browser e a rede. Ele intercepta TODA requisição que o app faz e pode decidir: servir do cache, ir para a rede, ou uma combinação dos dois.

```
┌──────────┐     ┌──────────────────┐     ┌──────────┐
│  Browser  │ ──→ │  Service Worker   │ ──→ │  Rede /  │
│  (App)    │ ←── │  (proxy/cache)    │ ←── │ Servidor │
└──────────┘     └──────────────────┘     └──────────┘
                        │     ↑
                        ↓     │
                  ┌──────────────┐
                  │  Cache API    │
                  │  (no browser) │
                  └──────────────┘
```

**Ciclo de vida do Service Worker:**

```
1. INSTALL  → Browser baixa o SW pela primeira vez
              → SW faz cache dos assets essenciais (app shell)
              
2. ACTIVATE → SW antigo é removido, novo assume
              → Limpa caches antigos
              
3. FETCH    → SW intercepta requisições
              → Decide: cache, rede, ou ambos
              
4. UPDATE   → Browser detecta nova versão do SW
              → Baixa nova versão em background
              → Aguarda todas as tabs fecharem (ou force update)
              → Novo SW assume
```

### 21.3 Setup no Angular

```bash
ng add @angular/pwa
```

Isso gera automaticamente:
- `ngsw-config.json` — Configuração de cache do Service Worker
- `manifest.webmanifest` — Metadata do app (nome, ícones, cores)
- Ícones padrão em `assets/icons/`
- Registra o Service Worker no `app.module.ts` ou `app.config.ts`
- Atualiza `index.html` com link para manifest e meta tags

```typescript
// O que o ng add faz no app.config.ts (standalone):
import { provideServiceWorker } from '@angular/service-worker';

bootstrapApplication(AppComponent, {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),           // Só ativa em produção
      registrationStrategy: 'registerWhenStable:30000'
      // Registra o SW após app estabilizar ou após 30s (o que vier primeiro)
    })
  ]
});

// OU no AppModule (com módulos):
ServiceWorkerModule.register('ngsw-worker.js', {
  enabled: !isDevMode(),
  registrationStrategy: 'registerWhenStable:30000'
})
```

**Estratégias de registro:**
```
'registerWhenStable:30000'  → Espera app estabilizar OU 30s (recomendado)
'registerImmediately'       → Registra no bootstrap (pode atrasar o app)
```

### 21.4 manifest.webmanifest — Configuração do App

```json
{
  "name": "Minha Aplicação Angular",
  "short_name": "MeuApp",
  "description": "Descrição do meu app para SEO e instalação",
  "theme_color": "#1976d2",
  "background_color": "#fafafa",
  "display": "standalone",
  "scope": "/",
  "start_url": "/",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "assets/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "assets/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "assets/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "assets/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "assets/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "shortcuts": [
    {
      "name": "Novo Pedido",
      "short_name": "Pedido",
      "url": "/orders/new",
      "icons": [{ "src": "assets/icons/shortcut-order.png", "sizes": "96x96" }]
    }
  ]
}
```

**Opções de `display`:**
```
"standalone"   → Parece app nativo (sem barra de URL) — mais usado
"fullscreen"   → Tela cheia total (sem status bar) — jogos
"minimal-ui"   → Com barra mínima de navegação
"browser"      → Abre no browser normal (não faz muito sentido para PWA)
```

**`purpose: "maskable any"`** — O ícone funciona tanto como ícone normal quanto como "maskable" (recortável em formato adaptativo pelo sistema operacional — círculo no Android, quadrado arredondado no iOS, etc).

### 21.5 ngsw-config.json — Estratégias de Cache (O Mais Importante)

O Angular Service Worker usa dois conceitos de cache: **assetGroups** (arquivos do build) e **dataGroups** (respostas de API).

```json
{
  "$schema": "./node_modules/@angular/service-worker/config/schema.json",
  "index": "/index.html",
  
  "assetGroups": [
    {
      "name": "app-shell",
      "installMode": "prefetch",
      "updateMode": "prefetch",
      "resources": {
        "files": [
          "/favicon.ico",
          "/index.html",
          "/manifest.webmanifest",
          "/*.css",
          "/*.js"
        ]
      }
    },
    {
      "name": "assets",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "resources": {
        "files": [
          "/assets/**",
          "/*.(png|jpg|jpeg|svg|gif|webp|ico)"
        ],
        "urls": [
          "https://fonts.googleapis.com/**",
          "https://fonts.gstatic.com/**"
        ]
      }
    }
  ],
  
  "dataGroups": [
    {
      "name": "api-freshness",
      "urls": ["/api/users", "/api/notifications"],
      "cacheConfig": {
        "maxSize": 50,
        "maxAge": "10m",
        "timeout": "5s",
        "strategy": "freshness"
      }
    },
    {
      "name": "api-performance",
      "urls": ["/api/products", "/api/categories"],
      "cacheConfig": {
        "maxSize": 200,
        "maxAge": "1h",
        "strategy": "performance"
      }
    },
    {
      "name": "api-long-cache",
      "urls": ["/api/config", "/api/static-data"],
      "cacheConfig": {
        "maxSize": 20,
        "maxAge": "7d",
        "strategy": "performance"
      }
    }
  ]
}
```

**Explicação de cada campo:**

**assetGroups** (arquivos estáticos do build):

| Campo | Significado |
|-------|------------|
| `installMode: "prefetch"` | Baixa TODOS os arquivos quando SW instala (app shell — garante offline imediato) |
| `installMode: "lazy"` | Só baixa quando o browser realmente requisitar (imagens, fontes — não bloqueia install) |
| `updateMode: "prefetch"` | Quando SW atualiza, baixa versões novas proativamente |
| `updateMode: "lazy"` | Quando SW atualiza, só baixa quando requisitado |
| `resources.files` | Padrões glob para arquivos locais |
| `resources.urls` | URLs externas para cachear (CDNs, fontes do Google, etc) |

**dataGroups** (respostas de API):

| Campo | Significado |
|-------|------------|
| `maxSize` | Máximo de entradas no cache |
| `maxAge` | Tempo de vida no cache (`10m`, `1h`, `7d`) |
| `timeout` | Tempo máximo de espera pela rede antes de usar cache (só para freshness) |
| `strategy: "freshness"` | **Rede primeiro** — tenta rede, se falhar ou demorar mais que `timeout`, usa cache |
| `strategy: "performance"` | **Cache primeiro** — serve do cache se existir e não estiver expirado, senão vai para rede |

**Quando usar cada estratégia:**

```
freshness (rede primeiro):
✅ Dados que mudam frequentemente (notificações, perfil do usuário, feed)
✅ Quando dados frescos são mais importantes que velocidade
✅ Com timeout: se rede demorar >5s, serve cache (melhor que nada)

performance (cache primeiro):
✅ Dados que mudam raramente (catálogo de produtos, categorias, config)
✅ Quando velocidade é mais importante que dados em tempo real
✅ maxAge controla quando o cache expira e força ida à rede
```

**Fluxo visual:**
```
FRESHNESS (timeout: 5s):
Request → Tenta rede ──── responde em <5s → Usa rede + atualiza cache
                    └──── demora >5s → Usa cache (se tiver)
                    └──── offline → Usa cache (se tiver)
                    └──── offline + sem cache → Erro

PERFORMANCE (maxAge: 1h):
Request → Tem cache válido (<1h)? ──── SIM → Serve do cache (instantâneo!)
                                  └──── NÃO → Vai para rede → Atualiza cache
```

### 21.6 Notificação de Atualização (Versão Nova Disponível)

Quando você faz deploy de uma nova versão, o Service Worker detecta que os arquivos mudaram. O Angular fornece o `SwUpdate` service para gerenciar isso.

```typescript
import { SwUpdate, VersionReadyEvent } from '@angular/service-worker';

@Injectable({ providedIn: 'root' })
export class PwaUpdateService {
  constructor(
    private swUpdate: SwUpdate,
    private snackbar: MatSnackBar
  ) {
    if (!this.swUpdate.isEnabled) return; // SW não ativo (dev mode)
    
    this.checkForUpdates();
    this.listenForUpdates();
  }
  
  // 1. Verificar atualizações periodicamente
  private checkForUpdates() {
    // Checa a cada 6 horas
    interval(6 * 60 * 60 * 1000).pipe(
      switchMap(() => this.swUpdate.checkForUpdate())
    ).subscribe(updateFound => {
      if (updateFound) {
        console.log('Nova versão encontrada!');
      }
    });
  }
  
  // 2. Reagir quando atualização está pronta
  private listenForUpdates() {
    this.swUpdate.versionUpdates.pipe(
      filter((event): event is VersionReadyEvent => event.type === 'VERSION_READY')
    ).subscribe(event => {
      console.log(`Versão atual: ${event.currentVersion.hash}`);
      console.log(`Nova versão:  ${event.latestVersion.hash}`);
      
      this.promptUpdate();
    });
  }
  
  // 3. Perguntar ao usuário se quer atualizar
  private promptUpdate() {
    const snack = this.snackbar.open(
      'Nova versão disponível!', 
      'Atualizar',
      { duration: 0 }  // Não fechar automaticamente
    );
    
    snack.onAction().subscribe(() => {
      // Ativa a nova versão e recarrega
      this.swUpdate.activateUpdate().then(() => {
        window.location.reload();
      });
    });
  }
  
  // 4. Forçar atualização (sem perguntar)
  forceUpdate() {
    this.swUpdate.activateUpdate().then(() => window.location.reload());
  }
}

// Injetar no AppComponent para inicializar
@Component({ ... })
export class AppComponent {
  constructor(private pwaUpdate: PwaUpdateService) {} // Basta injetar
}
```

**Fluxo de atualização:**
```
1. Deploy nova versão no servidor
2. Usuário abre o app (ou está com app aberto)
3. SW checa ngsw.json (manifesto de versão gerado no build)
4. Detecta que os hashes mudaram → nova versão!
5. Baixa novos arquivos em BACKGROUND (usuário não percebe)
6. Emite evento VERSION_READY
7. App mostra notificação "Nova versão disponível"
8. Usuário clica "Atualizar" → activateUpdate() → reload
```

### 21.7 Push Notifications

```typescript
import { SwPush } from '@angular/service-worker';

@Injectable({ providedIn: 'root' })
export class PushNotificationService {
  readonly VAPID_PUBLIC_KEY = 'sua-chave-publica-aqui'; // Gerada no backend
  
  constructor(
    private swPush: SwPush,
    private http: HttpClient
  ) {}
  
  // 1. Pedir permissão e registrar
  subscribeToNotifications(): Observable<void> {
    return from(this.swPush.requestSubscription({
      serverPublicKey: this.VAPID_PUBLIC_KEY
    })).pipe(
      switchMap(subscription => {
        // Envia a subscription para o backend salvar
        return this.http.post<void>('/api/push-subscriptions', subscription);
      }),
      tap(() => console.log('Push notification ativada!')),
      catchError(err => {
        if (Notification.permission === 'denied') {
          console.error('Usuário bloqueou notificações');
        }
        return throwError(() => err);
      })
    );
  }
  
  // 2. Escutar notificações recebidas
  listenForNotifications() {
    // Quando notificação chega e app está aberto
    this.swPush.messages.subscribe((message: any) => {
      console.log('Push recebida:', message);
    });
    
    // Quando usuário CLICA na notificação
    this.swPush.notificationClicks.subscribe(({ action, notification }) => {
      console.log('Clicou na notificação:', notification);
      
      // Navegar baseado na ação/payload
      if (notification.data?.url) {
        window.open(notification.data.url);
      }
    });
  }
  
  // 3. Cancelar inscrição
  unsubscribe() {
    this.swPush.unsubscribe().then(() => {
      console.log('Push notification desativada');
    });
  }
}
```

**Fluxo completo de push:**
```
1. Frontend: swPush.requestSubscription() → browser gera subscription (endpoint + keys)
2. Frontend: envia subscription para backend (POST /api/push-subscriptions)
3. Backend: salva subscription no banco
4. Quando quiser notificar:
   Backend: usa web-push library + subscription + VAPID keys → envia para push service
5. Push service (Google/Mozilla/Apple) → entrega para o Service Worker do browser
6. Service Worker: mostra notificação nativa do sistema
7. Usuário clica → swPush.notificationClicks emite → app navega
```

### 21.8 Detectar Status Online/Offline

```typescript
@Injectable({ providedIn: 'root' })
export class ConnectivityService {
  private onlineSubject = new BehaviorSubject<boolean>(navigator.onLine);
  
  isOnline$ = this.onlineSubject.asObservable();
  
  constructor() {
    fromEvent(window, 'online').subscribe(() => this.onlineSubject.next(true));
    fromEvent(window, 'offline').subscribe(() => this.onlineSubject.next(false));
  }
  
  get isOnline(): boolean {
    return this.onlineSubject.value;
  }
}

// No component
@Component({
  template: `
    @if (!(connectivity.isOnline$ | async)) {
      <div class="offline-banner" @slideIn>
        <span>⚠️ Você está offline. Alguns recursos podem não funcionar.</span>
      </div>
    }
    
    <router-outlet />
  `
})
export class AppComponent {
  constructor(public connectivity: ConnectivityService) {}
}
```

### 21.9 Instalação do PWA (Add to Home Screen)

```typescript
@Injectable({ providedIn: 'root' })
export class PwaInstallService {
  private deferredPrompt: any = null;
  private installableSubject = new BehaviorSubject<boolean>(false);
  
  isInstallable$ = this.installableSubject.asObservable();
  
  constructor() {
    // O browser emite este evento quando o app é elegível para instalação
    window.addEventListener('beforeinstallprompt', (event: Event) => {
      event.preventDefault();  // Previne o mini-infobar automático
      this.deferredPrompt = event;
      this.installableSubject.next(true);
    });
    
    // Detectar se já foi instalado
    window.addEventListener('appinstalled', () => {
      this.deferredPrompt = null;
      this.installableSubject.next(false);
      console.log('PWA instalado com sucesso!');
    });
  }
  
  // Mostrar prompt de instalação customizado
  async promptInstall(): Promise<boolean> {
    if (!this.deferredPrompt) return false;
    
    this.deferredPrompt.prompt();  // Mostra dialog nativo do browser
    const result = await this.deferredPrompt.userChoice;
    
    this.deferredPrompt = null;
    this.installableSubject.next(false);
    
    return result.outcome === 'accepted';
  }
  
  // Checar se já está rodando como PWA instalada
  isRunningAsInstalledApp(): boolean {
    return window.matchMedia('(display-mode: standalone)').matches
        || (window.navigator as any).standalone === true;  // Safari iOS
  }
}

// No component: botão customizado de instalação
@Component({
  template: `
    @if (installService.isInstallable$ | async) {
      <button class="install-btn" (click)="onInstall()">
        📲 Instalar App
      </button>
    }
  `
})
export class InstallBannerComponent {
  constructor(public installService: PwaInstallService) {}
  
  async onInstall() {
    const accepted = await this.installService.promptInstall();
    if (accepted) {
      this.analytics.track('pwa_installed');
    }
  }
}
```

**Critérios para o browser oferecer instalação:**
```
✅ HTTPS (ou localhost)
✅ manifest.webmanifest válido com name, icons (192px e 512px), start_url, display
✅ Service Worker registrado com evento fetch
✅ Usuário engajado (visitou o site mais de 1x, passou tempo na página)
```

### 21.10 Strategies de Cache Offline Avançadas

```typescript
// Pattern: Queue de operações offline (sync quando voltar online)
@Injectable({ providedIn: 'root' })
export class OfflineQueueService {
  private queue: PendingRequest[] = [];
  private readonly STORAGE_KEY = 'offline_queue';
  
  constructor(
    private http: HttpClient,
    private connectivity: ConnectivityService
  ) {
    // Carregar fila persistida
    this.loadQueue();
    
    // Quando voltar online, processar a fila
    this.connectivity.isOnline$.pipe(
      filter(online => online),
      switchMap(() => this.processQueue())
    ).subscribe();
  }
  
  // Encolar operação quando offline
  enqueue(request: PendingRequest): void {
    this.queue.push({ ...request, timestamp: Date.now() });
    this.saveQueue();
  }
  
  // Processar fila quando online
  private processQueue(): Observable<void> {
    if (this.queue.length === 0) return of(void 0);
    
    const requests = [...this.queue];
    this.queue = [];
    this.saveQueue();
    
    return from(requests).pipe(
      concatMap(req => this.executeRequest(req).pipe(
        catchError(err => {
          // Se falhar de novo, reencola
          this.enqueue(req);
          return of(null);
        })
      )),
      finalize(() => console.log(`Processadas ${requests.length} operações offline`)),
      map(() => void 0)
    );
  }
  
  private executeRequest(req: PendingRequest): Observable<any> {
    switch (req.method) {
      case 'POST': return this.http.post(req.url, req.body);
      case 'PUT': return this.http.put(req.url, req.body);
      case 'DELETE': return this.http.delete(req.url);
      default: return of(null);
    }
  }
  
  private saveQueue() {
    localStorage.setItem(this.STORAGE_KEY, JSON.stringify(this.queue));
  }
  
  private loadQueue() {
    const saved = localStorage.getItem(this.STORAGE_KEY);
    this.queue = saved ? JSON.parse(saved) : [];
  }
}

// Uso: Service que funciona offline
@Injectable({ providedIn: 'root' })
export class OrderService {
  constructor(
    private http: HttpClient,
    private offlineQueue: OfflineQueueService,
    private connectivity: ConnectivityService
  ) {}
  
  createOrder(order: Order): Observable<Order> {
    if (this.connectivity.isOnline) {
      return this.http.post<Order>('/api/orders', order);
    }
    
    // Offline: salvar na fila
    this.offlineQueue.enqueue({
      method: 'POST',
      url: '/api/orders',
      body: order
    });
    
    // Retorna otimisticamente
    return of({ ...order, id: `temp-${Date.now()}`, status: 'pending_sync' });
  }
}
```

### 21.11 Testando PWA

```bash
# 1. Build de produção (SW só funciona em prod)
ng build --configuration production

# 2. Servir com http-server (simula servidor estático)
npx http-server -p 8080 -c-1 dist/my-app/browser

# 3. Abrir Chrome DevTools:
#    - Application tab → Service Workers (ver status, forçar update)
#    - Application tab → Cache Storage (ver o que está cacheado)
#    - Application tab → Manifest (ver configuração e testar install)
#    - Network tab → marcar "Offline" (testar comportamento offline)

# 4. Lighthouse audit (Chrome DevTools → Lighthouse)
#    - Roda checklist completa de PWA
#    - Mostra score e o que falta
```

**Checklist de PWA para produção:**
```
□ manifest.webmanifest com todos os ícones (72 a 512px)
□ Service Worker registrado e funcionando
□ App funciona offline (pelo menos tela de fallback)
□ HTTPS em produção
□ Estratégias de cache configuradas para API
□ Notificação de atualização implementada
□ Splash screen e theme_color definidos
□ Performance: Lighthouse PWA score > 90
□ Meta tags para iOS: <meta name="apple-mobile-web-app-capable" content="yes">
□ Push notifications (se aplicável)
□ Offline queue para operações de escrita (se aplicável)
```

### 21.12 Limitações e Cuidados

```
⚠️ iOS Safari:
- Push notifications só a partir do iOS 16.4 e só para PWAs instaladas
- Service Worker cache limitado a ~50MB
- Cache pode ser limpo pelo sistema após ~7 dias sem uso
- Sem background sync
- Sem badging API

⚠️ Geral:
- Service Worker atualiza no máximo a cada 24h (browser limita)
- Cache Storage tem limite de espaço por browser (~5-10% do disco)
- Não confundir Service Worker com Web Worker (thread de processamento)
- SW não tem acesso ao DOM diretamente
- Primeira visita é sempre online (SW instala após primeira carga)
```

### 21.13 Perguntas de Entrevista sobre PWA

**P:** "O que é uma PWA e como funciona?"  
**R:** "É uma web app que usa Service Worker, Web Manifest e HTTPS para oferecer experiência de app nativo. O Service Worker atua como proxy entre o browser e a rede, interceptando requisições e servindo do cache quando offline. O manifest permite instalação na tela inicial. O Angular fornece `@angular/pwa` que gera toda a configuração, e o `ngsw-config.json` define estratégias de cache para assets e dados de API."

**P:** "Qual a diferença entre as estratégias freshness e performance?"  
**R:** "Freshness (rede primeiro) tenta a rede e só usa cache se falhar ou demorar mais que o timeout — ideal para dados que mudam frequentemente. Performance (cache primeiro) serve do cache se existir e estiver dentro do maxAge — ideal para dados estáveis onde velocidade é prioridade."

**P:** "Como gerencia atualizações do app?"  
**R:** "O Angular SW compara o hash dos arquivos do build. Quando detecta mudança, baixa a nova versão em background. Usando `SwUpdate.versionUpdates`, escuto o evento `VERSION_READY` e mostro uma notificação ao usuário com opção de atualizar, que chama `activateUpdate()` seguido de `location.reload()`."

**P:** "Como funciona offline?"  
**R:** "Assets são cacheados na instalação do SW (prefetch) ou no primeiro acesso (lazy). Dados de API são cacheados conforme configuração do dataGroups. Para operações de escrita offline, implemento uma fila de pending requests que é processada com `concatMap` quando a conectividade retorna."

---

<a name="22-state-management"></a>
## 22. STATE MANAGEMENT

### 22.1 Quando Usar Cada Abordagem

| Abordagem | Complexidade | Quando Usar |
|-----------|-------------|-------------|
| **Component State** | Baixa | Dados locais de um component |
| **Service + BehaviorSubject** | Média | Feature-level state |
| **Signals** | Média | Angular 16+, state síncrono |
| **Facade Pattern** | Média-Alta | Organizar services complexos |
| **NgRx/NGXS** | Alta | App enterprise, muitos devs, estado global complexo |

### 22.2 Service + BehaviorSubject (Mais Comum)

```typescript
@Injectable({ providedIn: 'root' })
export class CartStore {
  private itemsSubject = new BehaviorSubject<CartItem[]>([]);
  
  items$ = this.itemsSubject.asObservable();
  total$ = this.items$.pipe(
    map(items => items.reduce((sum, item) => sum + item.price * item.qty, 0))
  );
  count$ = this.items$.pipe(
    map(items => items.reduce((sum, item) => sum + item.qty, 0))
  );
  
  addItem(product: Product) {
    const items = this.itemsSubject.value;
    const existing = items.find(i => i.id === product.id);
    
    if (existing) {
      this.itemsSubject.next(
        items.map(i => i.id === product.id ? { ...i, qty: i.qty + 1 } : i)
      );
    } else {
      this.itemsSubject.next([...items, { ...product, qty: 1 }]);
    }
  }
  
  removeItem(id: number) {
    this.itemsSubject.next(this.itemsSubject.value.filter(i => i.id !== id));
  }
  
  clear() {
    this.itemsSubject.next([]);
  }
}
```

### 22.3 Signal Store (Angular 16+)

```typescript
@Injectable({ providedIn: 'root' })
export class CartSignalStore {
  // Estado
  items = signal<CartItem[]>([]);
  
  // Derivados
  total = computed(() => 
    this.items().reduce((sum, item) => sum + item.price * item.qty, 0)
  );
  count = computed(() => 
    this.items().reduce((sum, item) => sum + item.qty, 0)
  );
  isEmpty = computed(() => this.items().length === 0);
  
  // Ações
  addItem(product: Product) {
    this.items.update(items => {
      const existing = items.find(i => i.id === product.id);
      if (existing) {
        return items.map(i => i.id === product.id ? { ...i, qty: i.qty + 1 } : i);
      }
      return [...items, { ...product, qty: 1 }];
    });
  }
  
  removeItem(id: number) {
    this.items.update(items => items.filter(i => i.id !== id));
  }
  
  clear() {
    this.items.set([]);
  }
}
```

### 22.4 NgRx (Resumo para Entrevista)

```
Fluxo: Component → Action → Reducer → Store → Selector → Component

Store: Estado global imutável
Action: Descreve o que aconteceu { type: '[Cart] Add Item', product }
Reducer: Função pura (state, action) => newState
Selector: Consulta derivada do estado
Effect: Side effects (HTTP, localStorage, etc)
```

```typescript
// Saber explicar quando usar:
// ✅ Usar NgRx quando:
// - App grande com muitos devs
// - Estado compartilhado complexo entre muitos components
// - Precisa de undo/redo, time travel debug
// - Precisa de histórico de ações

// ❌ NÃO usar quando:
// - App pequena/média
// - Estado pode ser gerenciado por services
// - Equipe não conhece Redux pattern
// - Over-engineering desnecessário
```

---

<a name="23-security"></a>
## 23. SECURITY

### 23.1 XSS Protection

```typescript
// Angular sanitiza automaticamente
// ❌ Isso NÃO executa script:
<div>{{ userInput }}</div>
// Se userInput = '<script>alert("xss")</script>'
// Renderiza como texto, não como HTML

// Quando PRECISA renderizar HTML:
<div [innerHTML]="trustedHtml"></div>
// Angular sanitiza automaticamente, remove scripts

// Bypass manual (USE COM CUIDADO)
import { DomSanitizer } from '@angular/platform-browser';

constructor(private sanitizer: DomSanitizer) {}

trustedUrl = this.sanitizer.bypassSecurityTrustResourceUrl('https://youtube.com/embed/...');
trustedHtml = this.sanitizer.bypassSecurityTrustHtml('<b>Safe HTML</b>');
```

### 23.2 CSRF Protection

```typescript
// Angular HttpClient envia cookie XSRF-TOKEN automaticamente
// Basta o backend setar o cookie

// Customizar se necessário
provideHttpClient(
  withXsrfConfiguration({
    cookieName: 'MY-XSRF-TOKEN',
    headerName: 'X-MY-XSRF-TOKEN'
  })
)
```

### 23.3 Route Guards para Autorização

```typescript
// Guard baseado em roles
export const roleGuard: CanActivateFn = (route) => {
  const auth = inject(AuthService);
  const requiredRoles = route.data['roles'] as string[];
  
  if (requiredRoles.some(role => auth.hasRole(role))) {
    return true;
  }
  return inject(Router).createUrlTree(['/forbidden']);
};

// Nas rotas
{ 
  path: 'admin', 
  canActivate: [roleGuard], 
  data: { roles: ['admin', 'superadmin'] } 
}
```

### 23.4 JWT Handling

```typescript
// NUNCA armazenar token em localStorage (vulnerável a XSS)
// Preferir: httpOnly cookie (setado pelo backend)
// Se precisar de localStorage, garantir sanitização rigorosa

@Injectable({ providedIn: 'root' })
export class TokenService {
  private tokenKey = 'auth_token';
  
  getToken(): string | null {
    return localStorage.getItem(this.tokenKey);
  }
  
  isTokenExpired(): boolean {
    const token = this.getToken();
    if (!token) return true;
    
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      return payload.exp * 1000 < Date.now();
    } catch {
      return true;
    }
  }
}
```

---

<a name="24-i18n"></a>
## 24. INTERNACIONALIZAÇÃO (i18n)

### 24.1 Angular Built-in i18n

```html
<!-- Marcar para tradução -->
<h1 i18n="site header|Welcome message for the user">Welcome</h1>

<!-- Com id customizado -->
<p i18n="@@welcomeMessage">Welcome to our app</p>

<!-- Pluralização -->
<span i18n>{count, plural, 
  =0 {Nenhuma mensagem}
  =1 {1 mensagem}
  other {{{count}} mensagens}
}</span>

<!-- Select -->
<span i18n>{gender, select,
  male {Ele}
  female {Ela}
  other {Pessoa}
} está online.</span>
```

```bash
# Extrair mensagens
ng extract-i18n --output-path src/locale

# Build para idioma específico
ng build --localize
```

### 24.2 ngx-translate (Runtime i18n)

```typescript
// Mais flexível que built-in (troca idioma sem rebuild)
import { TranslateModule, TranslateService } from '@ngx-translate/core';

// No component
constructor(private translate: TranslateService) {
  translate.setDefaultLang('pt');
  translate.use('pt');
}

// No template
<h1>{{ 'HOME.TITLE' | translate }}</h1>
<p>{{ 'HOME.WELCOME' | translate:{ name: userName } }}</p>

// Trocar idioma em runtime
switchLang(lang: string) {
  this.translate.use(lang);
}
```

---

<a name="25-boas-praticas"></a>
## 25. BOAS PRÁTICAS & DESIGN PATTERNS

### 25.1 SOLID no Angular

```
S - Single Responsibility
    - 1 component = 1 responsabilidade
    - Services separados para API, estado, lógica

O - Open/Closed
    - Use @Input/@Output para customizar components
    - Content projection para extensibilidade

L - Liskov Substitution
    - Interfaces para services (pode trocar implementação)

I - Interface Segregation
    - Interfaces pequenas e específicas
    - Não force component a implementar métodos que não usa

D - Dependency Inversion
    - Injete services via DI
    - Use InjectionTokens para abstrações
```

### 25.2 Estrutura de Pastas (Feature-based)

```
src/app/
├── core/                    # Singleton services, guards, interceptors
│   ├── services/
│   │   ├── auth.service.ts
│   │   └── api.service.ts
│   ├── guards/
│   │   └── auth.guard.ts
│   ├── interceptors/
│   │   ├── auth.interceptor.ts
│   │   └── error.interceptor.ts
│   └── core.module.ts       # Importado 1x no AppModule
│
├── shared/                  # Components, pipes, directives reutilizáveis
│   ├── components/
│   │   ├── button/
│   │   ├── modal/
│   │   └── spinner/
│   ├── pipes/
│   │   ├── truncate.pipe.ts
│   │   └── time-ago.pipe.ts
│   ├── directives/
│   │   └── click-outside.directive.ts
│   └── shared.module.ts     # Exporta tudo reutilizável
│
├── features/                # Módulos por feature (lazy loaded)
│   ├── users/
│   │   ├── components/
│   │   │   ├── user-list/
│   │   │   ├── user-card/
│   │   │   └── user-form/
│   │   ├── services/
│   │   │   └── user.service.ts
│   │   ├── models/
│   │   │   └── user.model.ts
│   │   ├── users-routing.module.ts
│   │   └── users.module.ts
│   │
│   └── products/
│       ├── ...
│
├── layouts/                 # Layouts de página
│   ├── main-layout/
│   └── auth-layout/
│
└── app.module.ts
```

### 25.3 Naming Conventions

```
Components:   user-list.component.ts     → UserListComponent
Services:     user.service.ts            → UserService
Directives:   highlight.directive.ts     → HighlightDirective
Pipes:        truncate.pipe.ts           → TruncatePipe
Guards:       auth.guard.ts              → AuthGuard / authGuard (functional)
Interceptors: auth.interceptor.ts        → AuthInterceptor / authInterceptor
Models:       user.model.ts              → User (interface)
Enums:        user-role.enum.ts          → UserRole
```

### 25.4 Patterns Importantes

```typescript
// 1. Facade Pattern - já mostrado na seção 4.6

// 2. Adapter Pattern - Normalizar APIs de terceiros
@Injectable({ providedIn: 'root' })
export class PaymentAdapter {
  constructor(
    private stripe: StripeService,
    private paypal: PayPalService
  ) {}
  
  pay(method: 'stripe' | 'paypal', amount: number): Observable<PaymentResult> {
    if (method === 'stripe') {
      return this.stripe.charge(amount).pipe(
        map(r => ({ success: r.status === 'succeeded', id: r.id }))
      );
    }
    return this.paypal.createPayment(amount).pipe(
      map(r => ({ success: r.approved, id: r.orderId }))
    );
  }
}

// 3. Strategy Pattern - Estratégias intercambiáveis
interface SortStrategy<T> {
  sort(items: T[]): T[];
}

class NameSortStrategy implements SortStrategy<User> {
  sort(items: User[]) { return [...items].sort((a, b) => a.name.localeCompare(b.name)); }
}

class DateSortStrategy implements SortStrategy<User> {
  sort(items: User[]) { return [...items].sort((a, b) => b.createdAt - a.createdAt); }
}

// 4. Mediator Pattern - Services como mediador entre components
@Injectable({ providedIn: 'root' })
export class DashboardMediator {
  private filterChange$ = new Subject<Filters>();
  private refreshRequest$ = new Subject<void>();
  
  onFilterChange$ = this.filterChange$.asObservable();
  onRefreshRequest$ = this.refreshRequest$.asObservable();
  
  changeFilter(filters: Filters) { this.filterChange$.next(filters); }
  requestRefresh() { this.refreshRequest$.next(); }
}
```

### 25.5 Regras de Ouro

```
✅ SEMPRE:
- Usar tipagem forte (evitar `any`)
- Unsubscribe de Observables (takeUntil, async pipe, takeUntilDestroyed)
- OnPush quando possível
- trackBy em loops
- Lazy loading de features
- Error handling em HTTP calls
- Loading, error, e empty states no UI

❌ NUNCA:
- Lógica de negócio no component (use Service)
- HTTP call direta no component (use Service)
- Mutação direta com OnPush
- Console.log em produção
- Ignorar memory leaks
- any sem justificativa
- Subscriptions dentro de subscriptions (use switchMap)
```

---

<a name="26-arquitetura"></a>
## 26. ARQUITETURA DE PROJETOS GRANDES

### 26.1 Monorepo com Nx

```
workspace/
├── apps/
│   ├── web-app/          # App principal
│   ├── admin-app/        # App admin
│   └── mobile-app/       # App mobile
│
├── libs/
│   ├── shared/
│   │   ├── ui/           # Components compartilhados
│   │   ├── utils/        # Funções utilitárias
│   │   └── models/       # Interfaces/tipos
│   ├── features/
│   │   ├── auth/         # Feature de auth
│   │   └── dashboard/    # Feature de dashboard
│   └── data-access/
│       ├── user/         # Service + state de user
│       └── product/      # Service + state de product
```

### 26.2 Micro-frontends com Module Federation

```typescript
// Shell (host)
const routes: Routes = [
  {
    path: 'products',
    loadRemoteModule({
      type: 'module',
      remoteEntry: 'http://localhost:4201/remoteEntry.js',
      exposedModule: './ProductModule'
    }).then(m => m.ProductModule)
  }
];

// Remote (micro-frontend)
// webpack.config.js
new ModuleFederationPlugin({
  name: 'products',
  filename: 'remoteEntry.js',
  exposes: {
    './ProductModule': './src/app/products/products.module.ts'
  }
});
```

### 26.3 API Layer Pattern

```typescript
// Camada de abstração para APIs
@Injectable({ providedIn: 'root' })
export abstract class BaseApiService {
  protected abstract baseUrl: string;
  
  constructor(protected http: HttpClient) {}
  
  protected getAll<T>(endpoint: string, params?: HttpParams): Observable<T[]> {
    return this.http.get<T[]>(`${this.baseUrl}/${endpoint}`, { params });
  }
  
  protected getOne<T>(endpoint: string, id: number | string): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${endpoint}/${id}`);
  }
  
  protected create<T>(endpoint: string, body: Partial<T>): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}/${endpoint}`, body);
  }
  
  protected update<T>(endpoint: string, id: number | string, body: Partial<T>): Observable<T> {
    return this.http.patch<T>(`${this.baseUrl}/${endpoint}/${id}`, body);
  }
  
  protected remove(endpoint: string, id: number | string): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${endpoint}/${id}`);
  }
}

@Injectable({ providedIn: 'root' })
export class UserApiService extends BaseApiService {
  protected baseUrl = environment.apiUrl;
  
  getUsers(params?: HttpParams) { return this.getAll<User>('users', params); }
  getUser(id: number) { return this.getOne<User>('users', id); }
  createUser(user: Partial<User>) { return this.create<User>('users', user); }
  updateUser(id: number, user: Partial<User>) { return this.update<User>('users', id, user); }
  deleteUser(id: number) { return this.remove('users', id); }
}
```

---

<a name="27-comparacao-versoes"></a>
## 27. COMPARAÇÃO DE VERSÕES DO ANGULAR

### 27.1 Timeline de Versões

| Versão | Data | Principais Features |
|--------|------|---------------------|
| **Angular 14** | Jun 2022 | Standalone Components (preview), Typed Forms |
| **Angular 15** | Nov 2022 | Standalone APIs stable, Directive Composition, Functional Guards |
| **Angular 16** | Mai 2023 | **Signals**, Required inputs, DestroyRef, takeUntilDestroyed |
| **Angular 17** | Nov 2023 | Novo Control Flow (@if, @for), @defer, esbuild, novo logo |
| **Angular 18** | Mai 2024 | Zoneless (experimental), Material 3, Hydration melhorado |

### 27.2 Angular 14

**Principais Features:**
- Standalone Components (developer preview)
- Typed Forms (FormControl<string>)
- inject() function
- CLI auto-completion

```typescript
// Typed Forms (grande mudança!)
// Antes (Angular <14): FormControl value é `any`
const name = new FormControl('');
name.value; // any

// Angular 14+: tipado
const name = new FormControl<string>('');
name.value; // string | null
name.setValue(42); // ❌ Erro de tipo!

// Strict typed form group
const form = new FormGroup({
  name: new FormControl<string>('', { nonNullable: true }),
  age: new FormControl<number>(0, { nonNullable: true })
});
form.value; // { name?: string; age?: number }
form.getRawValue(); // { name: string; age: number }
```

### 27.3 Angular 15

**Principais Features:**
- Standalone APIs stable
- Directive Composition API
- NgOptimizedImage (stable)
- Functional router guards
- Stack traces melhoradas

```typescript
// Functional guards (substituem classes)
export const authGuard = () => {
  return inject(AuthService).isLoggedIn() || inject(Router).createUrlTree(['/login']);
};

// Directive Composition
@Component({
  hostDirectives: [
    { directive: TooltipDirective, inputs: ['tooltipText'] },
    { directive: RippleDirective }
  ]
})

// NgOptimizedImage
<img ngSrc="hero.jpg" width="1200" height="600" priority>
```

### 27.4 Angular 16 ⭐

**Principais Features:**
- **Signals** (reatividade granular)
- Required inputs
- DestroyRef + takeUntilDestroyed
- toSignal/toObservable interop
- Hydration (developer preview)

```typescript
// Signals
count = signal(0);
doubleCount = computed(() => this.count() * 2);

// Required inputs
@Input({ required: true }) userId!: number;

// DestroyRef (alternativa ao ngOnDestroy)
constructor() {
  inject(DestroyRef).onDestroy(() => console.log('Cleanup'));
}

// takeUntilDestroyed (game changer!)
constructor() {
  this.service.getData().pipe(
    takeUntilDestroyed()  // Sem boilerplate de destroy$!
  ).subscribe();
}

// toSignal/toObservable
users = toSignal(this.http.get<User[]>('/api/users'), { initialValue: [] });
```

### 27.5 Angular 17 ⭐⭐

**Principais Features:**
- **Novo Control Flow** (@if, @for, @switch) - compilado, mais performante
- **Deferrable Views** (@defer) - lazy loading granular
- Novo builder (esbuild + vite) - build muito mais rápido
- SSR/SSG melhorados
- Novo branding/logo

```html
<!-- Novo control flow -->
@if (user) {
  <div>{{ user.name }}</div>
} @else {
  <div>Login</div>
}

@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
} @empty {
  <div>Lista vazia</div>
}

@switch (status) {
  @case ('loading') { <spinner /> }
  @case ('error') { <error /> }
  @default { <content /> }
}

<!-- Deferrable views -->
@defer (on viewport) {
  <heavy-component />
} @placeholder {
  <skeleton />
} @loading (minimum 200ms) {
  <spinner />
} @error {
  <error-message />
}
```

### 27.6 Angular 18

**Principais Features:**
- **Zoneless** change detection (experimental)
- Material 3 (stable)
- Improved hydration
- Signal-based forms (preview)
- Route redirects with functions

```typescript
// Zoneless
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});

// Route redirect function
{
  path: 'old',
  redirectTo: (route) => {
    const param = route.queryParamMap.get('id');
    return param ? `/new/${param}` : '/new';
  }
}
```

### 27.7 Tabela Comparativa

| Feature | 14 | 15 | 16 | 17 | 18 |
|---------|:--:|:--:|:--:|:--:|:--:|
| Standalone | Preview | Stable | ✅ | ✅ | ✅ |
| Typed Forms | ✅ | ✅ | ✅ | ✅ | ✅ |
| Functional Guards | ❌ | ✅ | ✅ | ✅ | ✅ |
| Directive Composition | ❌ | ✅ | ✅ | ✅ | ✅ |
| Signals | ❌ | ❌ | ✅ | ✅ | ✅ |
| Required Inputs | ❌ | ❌ | ✅ | ✅ | ✅ |
| DestroyRef | ❌ | ❌ | ✅ | ✅ | ✅ |
| Control Flow Novo | ❌ | ❌ | ❌ | ✅ | ✅ |
| @defer | ❌ | ❌ | ❌ | ✅ | ✅ |
| Zoneless | ❌ | ❌ | ❌ | ❌ | Exp. |
| Signal Inputs | ❌ | ❌ | ❌ | Preview | ✅ |

---

<a name="28-dicas-entrevista"></a>
## 28. DICAS PARA ENTREVISTA SENIOR

### 28.1 ❌ Red Flags que te Eliminam

- Não mencionar **memory leaks** (subscriptions) quando fala de Observables
- Não separar lógica em **Services** (tudo no component)
- Usar **`any`** sem questionar ou justificar
- Não pensar em **error states** (loading, empty, error)
- Não conhecer **OnPush** e **trackBy**
- Não conhecer diferença entre **switchMap, mergeMap, concatMap, exhaustMap**
- Não saber explicar **lazy loading** e por que importa
- Não mencionar **testes** quando perguntam como implementaria algo
- Dizer que NgRx é obrigatório para qualquer app (over-engineering)
- Não conhecer **Signals** se a vaga pede Angular 16+

### 28.2 ✅ O que Mencionar ao Refatorar

1. "Vou separar concerns: **model/interface, service, component**"
2. "Preciso de **tipagem forte** (interfaces, sem any)"
3. "Garantir **cleanup** com takeUntil/async pipe/takeUntilDestroyed"
4. "**Error handling** estruturado com catchError, não console.log"
5. "**Cache** faz sentido aqui (shareReplay)"
6. "**OnPush** para performance"
7. "**trackBy** no ngFor para não recriar DOM"
8. "Vou extrair isso para um **pipe puro** ao invés de método no template"
9. "**Lazy loading** para essa feature"
10. "Testes unitários cobrindo os cenários principais"

### 28.3 🎯 Perguntas Comuns e Respostas Ideais

**P:** "Por que não usar NgRx aqui?"  
**R:** "NgRx adiciona complexidade e boilerplate. Para estado local de feature, Service + BehaviorSubject ou Signals são suficientes e mais simples. NgRx faz sentido quando há estado global complexo compartilhado entre muitas features com muitos devs."

**P:** "Como você testaria isso?"  
**R:** "Service: mockar HttpClient com HttpClientTestingModule. Component: mockar service com jasmine.createSpyObj, testar que chama o service corretamente e que o template renderiza conforme os dados. Guard: testar cenário autorizado e não-autorizado."

**P:** "Performance com 10k items na lista?"  
**R:** "Virtual scrolling com CDK, paginação no backend, OnPush change detection, trackBy no ngFor. Se filtros: usar pure pipe. Se Angular 17+: @defer para components pesados abaixo do fold."

**P:** "Diferença entre switchMap e mergeMap?"  
**R:** "switchMap cancela a requisição anterior — ideal para search/autocomplete. mergeMap executa todas em paralelo — ideal para batch uploads. concatMap mantém ordem — ideal para filas. exhaustMap ignora novos — ideal para prevenir double-click."

**P:** "Como evitar memory leaks?"  
**R:** "Três formas: (1) async pipe no template — unsubscribe automático, (2) takeUntilDestroyed() no Angular 16+, (3) pattern destroy$ com takeUntil. Prefiro async pipe + OnPush quando possível."

**P:** "Standalone vs NgModules?"  
**R:** "Standalone é o futuro e recomendado pelo Angular team. Elimina boilerplate de módulos, melhora tree-shaking, e simplifica lazy loading. Migração pode ser gradual — standalone e módulos coexistem."

**P:** "O que são Signals e por que usar?"  
**R:** "Signals são primitivos reativos síncronos. Vantagens: (1) sem memory leaks, (2) change detection granular, (3) syntax mais simples que BehaviorSubject. Não substituem RxJS — são complementares. Signals para estado síncrono, Observables para streams assíncronos."

**P:** "Explique Change Detection do Angular."  
**R:** "Zone.js intercepta eventos assíncronos e dispara change detection. Com Default, checa toda a árvore de components. Com OnPush, só checa quando: @Input muda por referência, evento interno, async pipe emite, ou markForCheck() é chamado. Angular 18 introduz Zoneless experimental, onde Signals disparam CD diretamente sem Zone.js."

**P:** "Como estruturaria um projeto enterprise?"  
**R:** "Feature-based: core/ para singleton services e interceptors, shared/ para componentes reutilizáveis, features/ para módulos lazy-loaded por domínio. Smart/dumb components, services para lógica, facade pattern se necessário. Monorepo com Nx se múltiplos apps."

### 28.4 💡 Conceitos que SEMPRE Caem

- ✅ Diferença entre **switchMap, mergeMap, concatMap, exhaustMap**
- ✅ **OnPush** change detection e quando/como usar
- ✅ **Memory leaks** e como evitar (3+ formas)
- ✅ **Smart vs Dumb** components
- ✅ **Lazy loading** e code splitting
- ✅ **Signals vs Observables** (Angular 16+)
- ✅ **trackBy** no ngFor / track no @for
- ✅ **shareReplay** para cache
- ✅ **Reactive Forms** (validação customizada, FormArray)
- ✅ **Route Guards** (funcional e class-based)
- ✅ **Interceptors** (auth, error handling, refresh token)
- ✅ **Testing** (service, component, pipe, guard)
- ✅ **Dependency Injection** (scopes, tokens, factories)
- ✅ **Content Projection** e quando usar
- ✅ **@defer** e estratégias de lazy loading (Angular 17+)

### 28.5 🚀 Durante o Live Coding

**Verbalize seu raciocínio:**
- "Vou criar uma interface para tipar esses dados"
- "Vou criar um service para isolar a lógica de API"
- "Preciso garantir que não vaze memória, então vou usar takeUntil"
- "OnPush aqui vai melhorar performance porque os dados vêm via @Input"
- "trackBy é essencial para não recriar DOM elements"
- "Esse filtro no template deveria ser um pure pipe"
- "Vou adicionar error handling e loading state"

**Mostre que você pensa em produção:**
- Error handling estruturado
- Loading states
- Empty states
- Edge cases (null, undefined, array vazio)
- Accessibility (aria-labels, roles, keyboard navigation)
- Performance (OnPush, trackBy, lazy loading)
- Testes (mencionar como testaria)
- Segurança (sanitização, guards)

### 28.6 📚 Estude Antes da Entrevista

1. **RxJS Operators** - switchMap, mergeMap, concatMap, exhaustMap, combineLatest, forkJoin
2. **Change Detection** - OnPush vs Default, Zone.js, Zoneless
3. **Forms** - Reactive Forms completo (validação customizada, FormArray, cross-field)
4. **Routing** - Guards (funcional), Lazy Loading, Resolve, nested routes
5. **Testing** - Service com HTTP, Component com mocks, Pipes, Guards
6. **Performance** - trackBy, Virtual Scroll, OnPush, @defer, pure pipes
7. **Signals** - signal, computed, effect, toSignal, toObservable
8. **Standalone** - Migração, bootstrap, lazy loading
9. **HTTP** - Interceptors (auth, error, cache), retry com backoff
10. **Arquitetura** - Smart/Dumb, Facade, feature-based structure
11. **Segurança** - XSS, CSRF, JWT, sanitização
12. **SSR** - Quando usar, cuidados (isPlatformBrowser), hydration

---

## 🎉 BOA SORTE NA SUA ENTREVISTA!

Este guia cobre **tudo que você precisa** para uma entrevista de nível Senior Angular. Continue praticando com:

1. ✅ Implementar features completas com todos os patterns (Smart/Dumb, Service, Pipe, Guard)
2. ✅ Refatorar código ruim aplicando OnPush, trackBy, pipes, tipagem
3. ✅ Praticar live coding verbalizando decisões
4. ✅ Code reviews (olhar código e identificar memory leaks, performance issues, bad patterns)
5. ✅ Criar mini-projetos: CRUD completo com auth, guards, interceptors, forms, testes

**Lembre-se:** Na entrevista, **verbalizar seu raciocínio** é TÃO importante quanto escrever código correto. Mostre que você pensa em produção, não só em fazer funcionar.

**Dica final:** Se não sabe a resposta, não invente. Diga "Não tenho certeza sobre isso, mas investigaria na documentação oficial." Honestidade é valorizada.
