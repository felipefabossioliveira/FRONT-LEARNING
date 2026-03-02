# 🅰️ GUIA COMPLETO ANGULAR - PREPARAÇÃO SENIOR 2025

**Versão:** Angular 18+ (compatível com 14-18)  
**Objetivo:** Preparação completa para entrevistas técnicas de nível Senior  
**Foco:** Conceitos essenciais + Boas práticas + Comparação de versões

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
22. [Boas Práticas & Design Patterns](#22-boas-praticas)
23. [Comparação de Versões do Angular](#23-comparacao-versoes)
24. [Dicas para Entrevista Senior](#24-dicas-entrevista)

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

### 1.2 Data Binding - 4 Tipos

| Tipo | Sintaxe | Direção | Exemplo | Quando Usar |
|------|---------|---------|---------|-------------|
| **Interpolação** | `{{ }}` | Component → View | `{{ name }}` | Exibir valores |
| **Property Binding** | `[property]` | Component → View | `[disabled]="isLoading"` | Bind de propriedades |
| **Event Binding** | `(event)` | View → Component | `(click)="save()"` | Escutar eventos |
| **Two-Way Binding** | `[(ngModel)]` | Ambas direções | `[(ngModel)]="name"` | Forms simples |

### 1.3 Property Binding - Exemplos

```html
<!-- Propriedades HTML padrão -->
<input [value]="username">
<img [src]="imageUrl" [alt]="imageAlt">
<button [disabled]="isLoading">Salvar</button>

<!-- Atributos (quando não há property correspondente) -->
<div [attr.data-id]="userId"></div>
<div [attr.aria-label]="description"></div>

<!-- Classes CSS -->
<div [class.active]="isActive"></div>
<div [ngClass]="{'active': isActive, 'disabled': !isEnabled}"></div>

<!-- Estilos -->
<div [style.color]="textColor"></div>
<div [style.font-size.px]="fontSize"></div>
<div [ngStyle]="{'color': textColor, 'font-size.px': fontSize}"></div>
```

**Quando usar `[class.x]` vs `[ngClass]`:**
- `[class.active]="isActive"` → Toggle de UMA classe
- `[ngClass]="{...}"` → Múltiplas classes condicionais

### 1.4 Diretivas Estruturais

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
             let odd = odd">
  {{ i }}. {{ item }}
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

### 1.5 ng-container - Wrapper sem DOM

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
```

---

<a name="2-comunicacao"></a>
## 2. COMUNICAÇÃO ENTRE COMPONENTS

### 2.1 @Input - Pai → Filho

```typescript
// FILHO: user-card.component.ts
export class UserCardComponent {
  @Input() user!: User;              // Obrigatório
  @Input() showEmail = true;         // Opcional com default
  @Input('userName') name?: string;  // Com alias
}
```

```html
<!-- PAI -->
<app-user-card 
  [user]="currentUser" 
  [showEmail]="false">
</app-user-card>
```

### 2.2 @Output - Filho → Pai

```typescript
// FILHO
export class UserCardComponent {
  @Input() user!: User;
  @Output() deleted = new EventEmitter<number>();
  @Output() edited = new EventEmitter<User>();
  
  onDelete() {
    this.deleted.emit(this.user.id);
  }
}
```

```html
<!-- PAI -->
<app-user-card 
  [user]="user"
  (deleted)="handleDelete($event)"
  (edited)="handleEdit($event)">
</app-user-card>
```

### 2.3 Pattern: Smart vs Dumb Components

| Aspecto | Dumb (Presentational) | Smart (Container) |
|---------|----------------------|-------------------|
| **Responsabilidade** | Apenas UI | Lógica de negócio |
| **Dados** | Recebe via @Input | Chama services |
| **Comunicação** | Emite @Output | Orquestra components |
| **Services** | NÃO usa | Injeta e usa |
| **Reutilizável** | SIM | Geralmente não |
| **Testável** | Muito fácil | Mais complexo |

**Exemplo:**

```typescript
// DUMB - Só apresenta dados
@Component({ selector: 'app-product-card' })
export class ProductCardComponent {
  @Input() product!: Product;
  @Output() addToCart = new EventEmitter<Product>();
  @Output() viewDetails = new EventEmitter<number>();
}

// SMART - Orquestra lógica
@Component({ selector: 'app-product-list' })
export class ProductListComponent {
  products$ = this.productService.getProducts();
  
  constructor(
    private productService: ProductService,
    private cartService: CartService
  ) {}
  
  onAddToCart(product: Product) {
    this.cartService.add(product);
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

### 3.2 Exemplo Prático

```typescript
export class UserComponent implements OnInit, OnChanges, OnDestroy {
  @Input() userId!: number;
  
  private destroy$ = new Subject<void>();
  user?: User;
  
  constructor(private userService: UserService) {
    // ❌ NÃO acessa @Input aqui (ainda undefined)
    // ✅ SÓ injeção de dependências
  }
  
  ngOnChanges(changes: SimpleChanges) {
    // Dispara quando @Input muda
    if (changes['userId']) {
      const previousId = changes['userId'].previousValue;
      const currentId = changes['userId'].currentValue;
      console.log(`User ID mudou: ${previousId} → ${currentId}`);
      this.loadUser(currentId);
    }
  }
  
  ngOnInit() {
    // @Input já disponível aqui
    // MELHOR lugar para HTTP calls iniciais
    this.loadInitialData();
  }
  
  ngOnDestroy() {
    // ESSENCIAL: limpar subscriptions
    this.destroy$.next();
    this.destroy$.complete();
  }
  
  private loadUser(id: number) {
    this.userService.getUser(id).pipe(
      takeUntil(this.destroy$)
    ).subscribe(user => this.user = user);
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
  // Memory leak! Subscription não é cancelada
}
```

✅ **Correto:**
```typescript
ngOnInit() {
  this.service.getData().pipe(
    takeUntil(this.destroy$)
  ).subscribe(data => this.data = data);
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
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
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    console.error('Erro:', error);
    return throwError(() => new Error('Erro ao carregar dados'));
  }
}
```

### 4.2 Níveis de Injeção (Scopes)

| Scope | Sintaxe | Quando Usar | Benefícios |
|-------|---------|-------------|------------|
| **Root** | `providedIn: 'root'` | 95% dos casos | Singleton, tree-shakeable |
| **Module** | `providedIn: MyModule` | Feature isolada | Compartilhado no módulo |
| **Component** | `providers: [Service]` | Estado local | Nova instância por component |

```typescript
// Root (recomendado)
@Injectable({ providedIn: 'root' })
export class AuthService {}

// Module
@Injectable({ providedIn: AdminModule })
export class AdminService {}

// Component (raro)
@Component({
  providers: [LocalService]
})
export class MyComponent {}
```

### 4.3 Cache com shareReplay

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
}
```

**Benefícios do shareReplay:**
- ✅ Múltiplos subscribers = 1 única requisição HTTP
- ✅ Cache automático em memória
- ✅ `refCount: true` libera memória quando não usado

---

<a name="5-rxjs"></a>
## 5. RxJS & OBSERVABLES

### 5.1 Operators Essenciais para Senior

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
| **distinctUntilChanged** | Só emite se mudou | Evita duplicados | Combinar com debounce |
| **catchError** | Tratamento de erro | Fallback quando falha | `catchError(() => of([]))` |
| **retry** | Tenta novamente | Resiliência HTTP | `retry(2)` |
| **shareReplay** | Cache Observable | HTTP compartilhado | Ver seção anterior |
| **takeUntil** | Cancela quando emite | Unsubscribe automático | Pattern destroy$ |
| **combineLatest** | Combina múltiplos Observables | Filtros combinados | Ver exemplo abaixo |
| **forkJoin** | Promise.all para Observables | Múltiplos HTTP paralelos | Ver exemplo abaixo |

### 5.2 switchMap vs mergeMap vs concatMap vs exhaustMap

**A pergunta MAIS COMUM em entrevistas:**

```typescript
// switchMap - Cancela requisição anterior
// Use: Search, autocomplete
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  switchMap(term => this.api.search(term))  // Cancela search anterior
).subscribe(results => this.results = results);

// mergeMap - Todas em paralelo
// Use: Batch uploads, não importa ordem
from(files).pipe(
  mergeMap(file => this.uploadFile(file))  // Todos uploads ao mesmo tempo
).subscribe(result => console.log(result));

// concatMap - Uma por vez, em ordem
// Use: Processar fila ordenada
from([1, 2, 3]).pipe(
  concatMap(id => this.api.getUser(id))  // Espera 1 terminar antes de chamar 2
).subscribe(user => console.log(user));

// exhaustMap - Ignora novos até terminar
// Use: Prevenir double-click em botão
this.loginButton.clicks$.pipe(
  exhaustMap(() => this.authService.login())  // Ignora cliques enquanto processa
).subscribe(result => console.log(result));
```

### 5.3 Autocomplete Perfeito (Exemplo Completo)

```typescript
export class SearchComponent implements OnInit {
  searchControl = new FormControl('');
  results: Product[] = [];
  loading = false;
  
  ngOnInit() {
    this.searchControl.valueChanges.pipe(
      debounceTime(300),              // Espera 300ms sem digitação
      distinctUntilChanged(),         // Só emite se valor mudou
      tap(() => this.loading = true), // Mostra loading
      switchMap(term => 
        this.searchService.search(term).pipe(
          catchError(() => of([]))    // Retorna [] em caso de erro
        )
      ),
      tap(() => this.loading = false) // Esconde loading
    ).subscribe(results => this.results = results);
  }
}
```

### 5.4 combineLatest - Combinar Filtros

```typescript
export class ProductListComponent {
  categoryFilter$ = new BehaviorSubject<string>('all');
  priceFilter$ = new BehaviorSubject<string>('all');
  products$ = this.productService.getProducts();
  
  filteredProducts$ = combineLatest([
    this.products$,
    this.categoryFilter$,
    this.priceFilter$
  ]).pipe(
    map(([products, category, price]) => {
      let filtered = products;
      
      if (category !== 'all') {
        filtered = filtered.filter(p => p.category === category);
      }
      
      if (price === 'low') {
        filtered = filtered.filter(p => p.price < 100);
      } else if (price === 'high') {
        filtered = filtered.filter(p => p.price >= 100);
      }
      
      return filtered;
    })
  );
  
  setCategory(category: string) {
    this.categoryFilter$.next(category);
  }
  
  setPrice(price: string) {
    this.priceFilter$.next(price);
  }
}
```

### 5.5 forkJoin - Múltiplos HTTP Paralelos

```typescript
// Carrega dados de 3 endpoints ao mesmo tempo
forkJoin({
  users: this.http.get<User[]>('/api/users'),
  posts: this.http.get<Post[]>('/api/posts'),
  comments: this.http.get<Comment[]>('/api/comments')
}).subscribe(({ users, posts, comments }) => {
  // Só executa quando TODOS terminarem
  console.log(users, posts, comments);
});
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

<!-- Currency -->
<div>{{ price | currency }}</div>                <!-- $100.00 -->
<div>{{ price | currency:'BRL' }}</div>          <!-- R$ 100,00 -->
<div>{{ price | currency:'EUR':'symbol':'1.2-2' }}</div>  <!-- €100.00 -->

<!-- Number/Decimal -->
<div>{{ 3.14159 | number:'1.2-4' }}</div>        <!-- 3.1416 -->
<!--                      ↑ ↑   ↑
                    min . min-max decimais -->

<!-- Percent -->
<div>{{ 0.25 | percent }}</div>                  <!-- 25% -->

<!-- Async (IMPORTANTE - unsubscribe automático) -->
<div *ngIf="user$ | async as user">
  {{ user.name }}
</div>

<!-- JSON (debug) -->
<pre>{{ user | json }}</pre>

<!-- Uppercase/Lowercase/Titlecase -->
<div>{{ 'hello' | uppercase }}</div>             <!-- HELLO -->
<div>{{ 'WORLD' | lowercase }}</div>             <!-- world -->
<div>{{ 'hello world' | titlecase }}</div>       <!-- Hello World -->

<!-- Slice -->
<div>{{ [1,2,3,4,5] | slice:1:3 }}</div>         <!-- [2,3] -->
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
    if (!search) return text;
    
    const regex = new RegExp(search, 'gi');
    return text.replace(regex, match => `<mark>${match}</mark>`);
  }
}
```

```html
<!-- Uso -->
<div [innerHTML]="'Angular é incrível' | highlight:'Angular'"></div>
<!-- Resultado: <mark>Angular</mark> é incrível -->
```

### 6.3 Pure vs Impure Pipes

```typescript
// Pure (default) - só recalcula se inputs mudarem
@Pipe({ name: 'myPipe', pure: true })
export class MyPipe implements PipeTransform {
  transform(value: any, arg: string): any {
    console.log('Executou!');  // Executa poucas vezes
    return value;
  }
}

// Impure - recalcula a CADA change detection
@Pipe({ name: 'myPipe', pure: false })
export class MyPipe implements PipeTransform {
  transform(value: any, arg: string): any {
    console.log('Executou!');  // Executa 100x por segundo!
    return value;
  }
}
```

**Quando usar impure?** Quase nunca. Só se o pipe depende de estado externo mutável.

### 6.4 Pipe com Parâmetros

```typescript
@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit = 50, ellipsis = '...'): string {
    if (value.length <= limit) return value;
    return value.substring(0, limit) + ellipsis;
  }
}
```

```html
<div>{{ longText | truncate }}</div>              <!-- Limite default 50 -->
<div>{{ longText | truncate:100 }}</div>          <!-- Limite 100 -->
<div>{{ longText | truncate:50:'---' }}</div>     <!-- Ellipsis customizado -->
```

---

<a name="7-directives"></a>
## 7. DIRECTIVES CUSTOMIZADAS

### 7.1 Attribute Directive

```typescript
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight = 'yellow';  // Cor customizável
  
  constructor(private el: ElementRef) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.el.nativeElement.style.backgroundColor = this.appHighlight;
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.el.nativeElement.style.backgroundColor = '';
  }
}
```

```html
<!-- Uso -->
<div appHighlight>Hover aqui</div>
<div [appHighlight]="'red'">Vermelho no hover</div>
```

### 7.2 Structural Directive

```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'  // Inverso do *ngIf
})
export class UnlessDirective {
  @Input() set appUnless(condition: boolean) {
    if (!condition) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainer.clear();
    }
  }
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
}
```

```html
<!-- Uso -->
<div *appUnless="isLoggedIn">
  Faça login para continuar
</div>
```

---

<a name="8-forms"></a>
## 8. FORMS (REACTIVE & TEMPLATE-DRIVEN)

### 8.1 Template-Driven (Simples)

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
  <input name="email" [(ngModel)]="user.email" required email>
  <input name="password" [(ngModel)]="user.password" required minlength="6">
  
  <button [disabled]="!loginForm.valid">Login</button>
</form>
```

### 8.2 Reactive Forms (Recomendado Senior)

```typescript
export class LoginComponent implements OnInit {
  loginForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.loginForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(6)]]
    });
    
    // Reagir a mudanças
    this.loginForm.valueChanges.subscribe(values => {
      console.log(values);
    });
  }
  
  onSubmit() {
    if (this.loginForm.valid) {
      console.log(this.loginForm.value);
    }
  }
  
  // Getters para facilitar no template
  get email() { return this.loginForm.get('email'); }
  get password() { return this.loginForm.get('password'); }
}
```

```html
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
  <input formControlName="email">
  <div *ngIf="email?.invalid && email?.touched">
    <div *ngIf="email?.errors?.['required']">Email obrigatório</div>
    <div *ngIf="email?.errors?.['email']">Email inválido</div>
  </div>
  
  <input formControlName="password" type="password">
  <div *ngIf="password?.invalid && password?.touched">
    <div *ngIf="password?.errors?.['required']">Senha obrigatória</div>
    <div *ngIf="password?.errors?.['minlength']">
      Mínimo {{ password?.errors?.['minlength'].requiredLength }} caracteres
    </div>
  </div>
  
  <button [disabled]="loginForm.invalid">Login</button>
</form>
```

### 8.3 FormArray (Arrays Dinâmicos)

```typescript
export class PhoneListComponent implements OnInit {
  form!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.form = this.fb.group({
      phones: this.fb.array([this.createPhone()])
    });
  }
  
  get phones() {
    return this.form.get('phones') as FormArray;
  }
  
  createPhone(): FormControl {
    return this.fb.control('', Validators.required);
  }
  
  addPhone() {
    this.phones.push(this.createPhone());
  }
  
  removePhone(index: number) {
    this.phones.removeAt(index);
  }
}
```

```html
<form [formGroup]="form">
  <div formArrayName="phones">
    <div *ngFor="let phone of phones.controls; let i = index">
      <input [formControlName]="i">
      <button (click)="removePhone(i)">Remover</button>
    </div>
  </div>
  <button (click)="addPhone()">Adicionar Telefone</button>
</form>
```

### 8.4 Validação Customizada

```typescript
// Validador síncrono
export function cpfValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const cpf = control.value;
    if (!cpf) return null;
    
    const isValid = validaCPF(cpf);
    return isValid ? null : { cpfInvalido: true };
  };
}

// Validador assíncrono (ex: checar se email existe)
export function emailExistsValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) return of(null);
    
    return userService.checkEmail(control.value).pipe(
      map(exists => exists ? { emailJaExiste: true } : null),
      catchError(() => of(null))
    );
  };
}
```

```typescript
// Uso
this.form = this.fb.group({
  cpf: ['', [Validators.required, cpfValidator()]],
  email: ['', 
    [Validators.required, Validators.email],
    [emailExistsValidator(this.userService)]  // Async
  ]
});
```

### 8.5 Cross-Field Validation

```typescript
export function passwordMatchValidator(): ValidatorFn {
  return (formGroup: AbstractControl): ValidationErrors | null => {
    const password = formGroup.get('password')?.value;
    const confirm = formGroup.get('confirmPassword')?.value;
    
    return password === confirm ? null : { passwordMismatch: true };
  };
}
```

```typescript
this.form = this.fb.group({
  password: ['', Validators.required],
  confirmPassword: ['', Validators.required]
}, { validators: passwordMatchValidator() });  // No FormGroup
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
  { path: '**', component: NotFoundComponent }  // 404
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

### 9.2 Lazy Loading (ESSENCIAL)

```typescript
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
    // ↑ Só carrega quando usuário acessar /admin
  }
];
```

**Benefícios:**
- ✅ Bundle inicial menor
- ✅ Carrega feature só quando necessário
- ✅ Melhora performance inicial

### 9.3 Route Guards

```typescript
// CanActivate - Protege acesso à rota
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(): boolean {
    if (this.authService.isLoggedIn()) {
      return true;
    }
    this.router.navigate(['/login']);
    return false;
  }
}

// CanDeactivate - Previne saída (form não salvo)
export interface CanComponentDeactivate {
  canDeactivate: () => boolean | Observable<boolean>;
}

@Injectable({ providedIn: 'root' })
export class UnsavedChangesGuard implements CanDeactivate<CanComponentDeactivate> {
  canDeactivate(component: CanComponentDeactivate): boolean {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}
```

```typescript
// Uso nas rotas
{
  path: 'admin',
  component: AdminComponent,
  canActivate: [AuthGuard],
  canDeactivate: [UnsavedChangesGuard]
}
```

### 9.4 Resolve - Pré-carregar Dados

```typescript
@Injectable({ providedIn: 'root' })
export class UserResolve implements Resolve<User> {
  constructor(private userService: UserService) {}
  
  resolve(route: ActivatedRouteSnapshot): Observable<User> {
    const id = +route.paramMap.get('id')!;
    return this.userService.getUser(id);
  }
}
```

```typescript
{
  path: 'users/:id',
  component: UserDetailComponent,
  resolve: { user: UserResolve }
}
```

```typescript
// Component
ngOnInit() {
  this.user = this.route.snapshot.data['user'];  // Já disponível!
}
```

### 9.5 Parâmetros de Rota

```typescript
// Snapshot (valor atual)
const id = this.route.snapshot.paramMap.get('id');

// Observable (reage a mudanças)
this.route.paramMap.subscribe(params => {
  const id = params.get('id');
  this.loadUser(id);
});

// Query params
this.router.navigate(['/users'], {
  queryParams: { page: 2, sort: 'name' }
});
// URL: /users?page=2&sort=name

// Ler query params
this.route.queryParams.subscribe(params => {
  const page = params['page'];
});
```

---

<a name="10-change-detection"></a>
## 10. CHANGE DETECTION & OnPush

### 10.1 Estratégias

| Estratégia | Como Funciona | Performance | Quando Usar |
|-----------|---------------|-------------|-------------|
| **Default** | Checa toda árvore a cada evento | Boa | Apps pequenas |
| **OnPush** | Só checa quando: Input muda (ref), evento interno, async pipe | Excelente | Apps médias/grandes |

### 10.2 Default vs OnPush

```typescript
// Default - checa TUDO a cada evento
@Component({
  changeDetection: ChangeDetectionStrategy.Default
})
export class SlowComponent {}

// OnPush - checa só quando necessário
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class FastComponent {
  @Input() users!: User[];
  
  // ❌ NÃO funciona com OnPush
  addUser(user: User) {
    this.users.push(user);  // Mesma referência
  }
  
  // ✅ Funciona com OnPush
  addUser(user: User) {
    this.users = [...this.users, user];  // Nova referência!
  }
}
```

### 10.3 OnPush + Async Pipe = Perfeito

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
    </div>
  `
})
export class UserListComponent {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserService) {}
}
```

**Por que funciona?**
- Async pipe marca component para check quando Observable emite
- OnPush + async pipe = performance máxima

### 10.4 ChangeDetectorRef - Controle Manual

```typescript
import { ChangeDetectorRef } from '@angular/core';

export class MyComponent {
  constructor(private cdr: ChangeDetectorRef) {}
  
  updateOutsideZone() {
    setTimeout(() => {
      this.data = 'Nova';
      this.cdr.markForCheck();  // Marca para próxima checagem
    });
  }
  
  forceCheck() {
    this.cdr.detectChanges();  // Força checagem imediata
  }
}
```

---

<a name="11-signals"></a>
## 11. SIGNALS (Angular 16+)

### 11.1 O que são Signals?

Signals são **primitivos reativos** introduzidos no Angular 16. Alternativa aos Observables para gerenciar estado.

```typescript
import { Component, signal, computed } from '@angular/core';

export class CounterComponent {
  // Signal primitivo
  count = signal(0);
  
  // Computed (auto-recalcula)
  doubleCount = computed(() => this.count() * 2);
  
  increment() {
    this.count.set(this.count() + 1);
    // OU
    this.count.update(value => value + 1);
  }
}
```

```html
<!-- Template -->
<div>Count: {{ count() }}</div>
<div>Double: {{ doubleCount() }}</div>
<button (click)="increment()">+1</button>
```

### 11.2 Signal vs BehaviorSubject

| Feature | Signal | BehaviorSubject |
|---------|--------|-----------------|
| **Leitura** | `count()` | `count$ \| async` |
| **Escrita** | `count.set(5)` | `count$.next(5)` |
| **Performance** | Melhor (granular) | Boa (Zone.js) |
| **Change Detection** | Só affected components | Árvore inteira |
| **RxJS Operators** | Não (mas tem interop) | Sim |
| **Syntax** | Mais simples | Mais verbosa |

### 11.3 Signals com Arrays/Objects

```typescript
export class TodoComponent {
  todos = signal<Todo[]>([]);
  
  addTodo(title: string) {
    // ❌ ERRADO - mutação não dispara
    this.todos().push({ id: Date.now(), title });
    
    // ✅ CERTO - nova referência
    this.todos.update(current => [...current, { id: Date.now(), title }]);
  }
  
  removeTodo(id: number) {
    this.todos.update(current => current.filter(t => t.id !== id));
  }
  
  updateTodo(id: number, title: string) {
    this.todos.update(current =>
      current.map(t => t.id === id ? { ...t, title } : t)
    );
  }
}
```

### 11.4 toSignal / toObservable

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

export class UserComponent {
  // Observable → Signal
  users = toSignal(this.userService.getUsers(), {
    initialValue: [] as User[]
  });
  
  // Signal → Observable (para usar RxJS)
  searchTerm = signal('');
  results$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    switchMap(term => this.searchService.search(term))
  );
}
```

### 11.5 Effect - Side Effects

```typescript
import { effect } from '@angular/core';

export class UserComponent {
  userId = signal(1);
  
  constructor() {
    // Roda toda vez que userId muda
    effect(() => {
      console.log('User ID:', this.userId());
      localStorage.setItem('userId', this.userId().toString());
    });
  }
}
```

---

<a name="12-standalone"></a>
## 12. STANDALONE COMPONENTS

### 12.1 Component Standalone

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-user',
  standalone: true,  // ← Standalone!
  imports: [CommonModule, FormsModule],  // Importa aqui
  template: `...`
})
export class UserComponent {}
```

**Benefícios:**
- ✅ Sem NgModule
- ✅ Tree-shaking melhor
- ✅ Lazy load direto

### 12.2 App Standalone (Sem AppModule)

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
  ]
});
```

### 12.3 Lazy Load Standalone

```typescript
// routes
{
  path: 'admin',
  loadComponent: () => import('./admin.component').then(m => m.AdminComponent)
}
```

---

<a name="13-http"></a>
## 13. HTTP & INTERCEPTORS

### 13.1 HttpClient Básico

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      retry(2),
      catchError(this.handleError)
    );
  }
  
  createUser(user: User): Observable<User> {
    return this.http.post<User>('/api/users', user);
  }
  
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`/api/users/${id}`, user);
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`/api/users/${id}`);
  }
}
```

### 13.2 HTTP Interceptor

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Adiciona token
    const authReq = req.clone({
      setHeaders: { Authorization: `Bearer ${this.getToken()}` }
    });
    
    return next.handle(authReq).pipe(
      retry(2),
      catchError(error => {
        if (error.status === 401) {
          this.router.navigate(['/login']);
        }
        return throwError(() => error);
      })
    );
  }
  
  private getToken(): string {
    return localStorage.getItem('token') || '';
  }
}
```

```typescript
// Registro (AppModule)
providers: [
  {
    provide: HTTP_INTERCEPTORS,
    useClass: AuthInterceptor,
    multi: true
  }
]

// Registro (Standalone)
provideHttpClient(
  withInterceptors([authInterceptor])
)
```

---

<a name="14-testing"></a>
## 14. TESTING

### 14.1 Teste de Component

```typescript
describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let mockService: jasmine.SpyObj<UserService>;
  
  beforeEach(() => {
    mockService = jasmine.createSpyObj('UserService', ['getUsers']);
    
    TestBed.configureTestingModule({
      declarations: [UserListComponent],
      providers: [{ provide: UserService, useValue: mockService }]
    });
    
    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
  });
  
  it('should load users on init', () => {
    const mockUsers = [{ id: 1, name: 'João' }];
    mockService.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges();
    
    expect(component.users).toEqual(mockUsers);
  });
});
```

### 14.2 Teste de Service com HTTP

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
    httpMock.verify();
  });
  
  it('should fetch users', () => {
    const mockUsers = [{ id: 1, name: 'João' }];
    
    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
});
```

---

<a name="15-performance"></a>
## 15. PERFORMANCE OPTIMIZATION

### 15.1 trackBy (CRÍTICO)

```typescript
trackByUserId(index: number, user: User): number {
  return user.id;
}
```

```html
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>
```

### 15.2 Virtual Scrolling

```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  imports: [ScrollingModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items">{{ item }}</div>
    </cdk-virtual-scroll-viewport>
  `
})
```

### 15.3 Pure Pipes

```typescript
// ❌ RUIM
<div *ngFor="let user of filterUsers(users, term)">

// ✅ BOM
@Pipe({ name: 'filterUsers', pure: true })
<div *ngFor="let user of users | filterUsers:term">
```

### 15.4 OnPush Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

---

<a name="16-content-projection"></a>
## 16. CONTENT PROJECTION

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="header">
      <ng-content select="[slot='header']"></ng-content>
    </div>
    <div class="body">
      <ng-content></ng-content>
    </div>
  `
})
```

```html
<app-card>
  <h2 slot="header">Título</h2>
  <p>Conteúdo</p>
</app-card>
```

---

<a name="17-viewchild"></a>
## 17. ViewChild & ContentChild

```typescript
@ViewChild('input') input!: ElementRef;
@ViewChild(ChildComponent) child!: ChildComponent;
@ViewChildren(ItemComponent) items!: QueryList<ItemComponent>;

ngAfterViewInit() {
  this.input.nativeElement.focus();
}
```

---

<a name="18-dynamic-components"></a>
## 18. DYNAMIC COMPONENTS

```typescript
@ViewChild('container', { read: ViewContainerRef }) container!: ViewContainerRef;

loadComponent() {
  this.container.clear();
  const ref = this.container.createComponent(AlertComponent);
  ref.instance.message = 'Alerta!';
}
```

---

<a name="19-animations"></a>
## 19. ANIMATIONS

```typescript
import { trigger, state, style, transition, animate } from '@angular/animations';

@Component({
  animations: [
    trigger('fadeIn', [
      transition(':enter', [
        style({ opacity: 0 }),
        animate('300ms', style({ opacity: 1 }))
      ])
    ])
  ]
})
```

---

<a name="20-ssr"></a>
## 20. SSR (Server-Side Rendering)

```bash
ng add @nguniversal/express-engine
```

```typescript
import { isPlatformBrowser } from '@angular/common';

if (isPlatformBrowser(this.platformId)) {
  // Só executa no browser
  localStorage.setItem('key', 'value');
}
```

---

<a name="21-pwa"></a>
## 21. PWA

```bash
ng add @angular/pwa
```

---

<a name="22-boas-praticas"></a>
## 22. BOAS PRÁTICAS & DESIGN PATTERNS

### 22.1 SOLID Principles

- **S**ingle Responsibility: Um component/service faz uma coisa
- **O**pen/Closed: Aberto para extensão, fechado para modificação
- **L**iskov Substitution: Substituível por subtipos
- **I**nterface Segregation: Interfaces pequenas e específicas
- **D**ependency Inversion: Dependa de abstrações

### 22.2 Patterns

- **Smart/Dumb Components**
- **Facade Pattern** (Services)
- **Observer Pattern** (RxJS)
- **Repository Pattern** (Data access)

---

<a name="23-comparacao-versoes"></a>
## 23. COMPARAÇÃO DE VERSÕES DO ANGULAR

### 23.1 Timeline de Versões

| Versão | Data | Principais Features |
|--------|------|---------------------|
| **Angular 14** | Jun 2022 | Standalone Components (developer preview), Typed Forms |
| **Angular 15** | Nov 2022 | Standalone APIs stable, Directive Composition API |
| **Angular 16** | Mai 2023 | **Signals**, Required inputs, DestroyRef |
| **Angular 17** | Nov 2023 | Novo Control Flow (@if, @for), Deferrable Views |
| **Angular 18** | Mai 2024 | Zoneless, Material 3, Hydration melhorado |

### 23.2 Angular 14

**Principais Features:**
- ✅ Standalone Components (developer preview)
- ✅ Typed Forms (FormControl tipado)
- ✅ Bind route info to component inputs
- ✅ CLI auto-completion

```typescript
// Typed Forms
const name = new FormControl<string>('');
name.value;  // string | null
```

### 23.3 Angular 15

**Principais Features:**
- ✅ Standalone APIs stable
- ✅ Directive Composition API
- ✅ Image directive (NgOptimizedImage)
- ✅ Functional router guards

```typescript
// Functional guards
export const authGuard = () => {
  return inject(AuthService).isLoggedIn() || inject(Router).createUrlTree(['/login']);
};

// Directive Composition
@Component({
  hostDirectives: [Tooltip, Focus]
})
```

### 23.4 Angular 16 ⭐

**Principais Features:**
- ✅ **Signals** (reatividade granular)
- ✅ Required inputs
- ✅ DestroyRef (alternativa ao ngOnDestroy)
- ✅ takeUntilDestroyed operator

```typescript
// Signals
count = signal(0);
doubleCount = computed(() => this.count() * 2);

// Required inputs
@Input({ required: true }) user!: User;

// DestroyRef
constructor() {
  const destroyRef = inject(DestroyRef);
  destroyRef.onDestroy(() => console.log('Cleanup'));
}

// takeUntilDestroyed
this.service.getData().pipe(
  takeUntilDestroyed()  // Auto cleanup!
).subscribe();
```

### 23.5 Angular 17 ⭐⭐

**Principais Features:**
- ✅ **Novo Control Flow** (@if, @for, @switch)
- ✅ **Deferrable Views** (@defer)
- ✅ Novo builder (esbuild)
- ✅ SSR/SSG melhorados

```typescript
// Novo control flow (sem * )
@if (user) {
  <div>{{ user.name }}</div>
} @else {
  <div>Login</div>
}

@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
}

// Deferrable views (lazy loading de components)
@defer (on viewport) {
  <heavy-component />
} @placeholder {
  <div>Carregando...</div>
}
```

### 23.6 Angular 18 (Atual)

**Principais Features:**
- ✅ **Zoneless** (experimental)
- ✅ Material 3
- ✅ Hydration melhorado
- ✅ Server route config

```typescript
// Zoneless (sem Zone.js)
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

### 23.7 Tabela Comparativa Resumida

| Feature | Angular 14 | Angular 15 | Angular 16 | Angular 17 | Angular 18 |
|---------|-----------|-----------|-----------|-----------|-----------|
| Standalone | Preview | Stable | Stable | Stable | Stable |
| Signals | ❌ | ❌ | ✅ | ✅ | ✅ |
| Control Flow Novo | ❌ | ❌ | ❌ | ✅ | ✅ |
| Deferrable Views | ❌ | ❌ | ❌ | ✅ | ✅ |
| Zoneless | ❌ | ❌ | ❌ | ❌ | ✅ (experimental) |
| Required Inputs | ❌ | ❌ | ✅ | ✅ | ✅ |
| DestroyRef | ❌ | ❌ | ✅ | ✅ | ✅ |

### 23.8 Migrações Importantes

**Angular 14 → 15:**
- Standalone components de preview pra stable
- Considerar migrar guards para functional guards

**Angular 15 → 16:**
- **Começar a usar Signals** para state management
- Adicionar `required: true` em @Input obrigatórios

**Angular 16 → 17:**
- **Migrar para novo control flow** (@if, @for)
- Considerar usar @defer para lazy loading

**Angular 17 → 18:**
- Testar zoneless (se quiser performance máxima)
- Migrar para Material 3

### 23.9 Compatibilidade com Versões Antigas

**Regra geral:** Angular mantém compatibilidade com **N-1** (versão anterior).

**Exemplo:**
- Angular 18 é compatível com Angular 17
- Angular 17 é compatível com Angular 16
- Mas Angular 18 pode quebrar coisas do Angular 14

**Recomendação:**
- Atualizar versões incrementalmente (não pule versões)
- Sempre rodar `ng update` antes
- Ler CHANGELOG de cada versão

---

<a name="24-dicas-entrevista"></a>
## 24. DICAS PARA ENTREVISTA SENIOR

### 24.1 ❌ Red Flags que te Eliminam

- Não mencionar **memory leaks** (subscriptions)
- Não separar lógica em **Services**
- Usar **`any`** sem questionar
- Não pensar em **error states**
- Não conhecer **OnPush** e **trackBy**
- Não conhecer diferença entre **switchMap, mergeMap, concatMap**

### 24.2 ✅ O que Mencionar ao Refatorar

1. "Vou separar concerns: **model, service, component**"
2. "Preciso de **tipagem forte** (interfaces)"
3. "Garantir **cleanup** com takeUntil/async pipe"
4. "**Error handling** estruturado, não console.log"
5. "**Cache** faz sentido aqui (shareReplay)"
6. "**OnPush** para performance"
7. "**trackBy** no ngFor para não recriar DOM"

### 24.3 🎯 Perguntas Comuns

**P:** "Por que não usar NgRx aqui?"  
**R:** "Overhead desnecessário para feature isolada. State local com Signals/BehaviorSubject é suficiente."

**P:** "Como você testaria isso?"  
**R:** "Service: mockar HttpClient com HttpClientTestingModule. Component: mockar Service com jasmine.createSpyObj."

**P:** "Performance com 10k usuários?"  
**R:** "Virtual scrolling (CDK), paginação no backend, OnPush change detection, trackBy no ngFor."

**P:** "Diferença entre switchMap e mergeMap?"  
**R:** "switchMap cancela requisição anterior (search). mergeMap executa todas em paralelo (batch upload)."

**P:** "Como evitar memory leaks?"  
**R:** "takeUntil pattern com destroy$, async pipe (auto-unsubscribe), ou DestroyRef (Angular 16+)."

### 24.4 💡 Conceitos que SEMPRE Caem

- ✅ Diferença entre **switchMap, mergeMap, concatMap, exhaustMap**
- ✅ **OnPush** change detection e quando usar
- ✅ **Memory leaks** e como evitar
- ✅ **Smart vs Dumb** components
- ✅ **Lazy loading** e code splitting
- ✅ **Signals vs Observables** (Angular 16+)
- ✅ **trackBy** no ngFor
- ✅ **shareReplay** para cache
- ✅ **Reactive Forms** vs Template-Driven

### 24.5 🚀 Durante o Live Coding

**Verbalize seu raciocínio:**
- "Vou criar um service para isolar a lógica de API"
- "Preciso garantir que não vaze memória, então vou usar takeUntil"
- "OnPush aqui vai melhorar performance porque..."
- "trackBy é essencial para não recriar DOM elements"

**Mostre que você pensa em produção:**
- Error handling
- Loading states
- Empty states
- Edge cases
- Accessibility
- Performance

### 24.6 📚 Estude Antes da Entrevista

1. **RxJS Operators** - Saiba na ponta da língua
2. **Change Detection** - OnPush vs Default
3. **Forms** - Reactive Forms (validação customizada)
4. **Routing** - Guards, Lazy Loading, Resolve
5. **Testing** - Jasmine, mocking
6. **Performance** - trackBy, Virtual Scroll, OnPush
7. **Signals** - Se a vaga menciona Angular 16+

---

## 🎉 BOA SORTE NA SUA ENTREVISTA!

Este guia cobre **tudo que você precisa** para uma entrevista de nível Senior. Continue praticando com:

1. ✅ Exercícios de lógica
2. ✅ Live coding de features completas
3. ✅ Refatoração de código ruim
4. ✅ Code reviews (olhar código e achar problemas)

**Lembre-se:** Na entrevista, **verbalizar seu raciocínio** é TÃO importante quanto escrever código correto. Mostre que você pensa em produção, não só em fazer funcionar.

**Próximo passo:** Resolver exercícios de lógica para treinar! 🔥
