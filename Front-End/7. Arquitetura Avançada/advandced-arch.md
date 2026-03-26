# 🏛️ ARQUITETURAS DE SOFTWARE NO ANGULAR
## Guia Prático: Simplificado e Direto ao Ponto

**Versão:** 2025 - Simplificada  
**Objetivo:** Saber QUANDO e COMO usar cada arquitetura

---

## 📚 SUMÁRIO RÁPIDO

**PARTE 1: CONCEITOS BASE**
- [Por Que Arquitetura Importa](#parte1)
- [Princípios SOLID em 5 minutos](#solid)

**PARTE 2: ARQUITETURAS**
- [Clean Architecture](#clean)
- [DDD (Domain-Driven Design)](#ddd)
- [Hexagonal (Ports & Adapters)](#hexagonal)
- [Layered Architecture](#layered)
- [CQRS](#cqrs)
- [Event-Driven](#eventdriven)
- [Micro Frontends](#microfrontends)

**PARTE 3: NA PRÁTICA**
- [Tabela de Comparação](#comparacao)
- [Estruturas de Pasta](#estruturas)
- [Como Migrar](#migracao)
- [Checklist de Decisão](#checklist)

---

<a name="parte1"></a>
# PARTE 1: POR QUE ARQUITETURA IMPORTA

## O Problema Sem Arquitetura

**Exemplo: E-commerce simples**

```typescript
// ❌ TUDO MISTURADO - Component de 500 linhas
@Component({...})
export class ProductComponent {
  products: any[] = [];
  
  ngOnInit() {
    // HTTP direto no component
    this.http.get('https://api.com/products').subscribe(data => {
      this.products = data;
      localStorage.setItem('cache', JSON.stringify(data)); // LocalStorage aqui
    });
  }
  
  addToCart(product: any) {
    if (product.stock <= 0) { // Validação aqui
      alert('Sem estoque!'); // Alert aqui
      return;
    }
    
    let price = product.price;
    if (this.cart.length > 5) { // Regra de negócio aqui
      price = price * 0.9;
    }
    
    this.http.put('https://api.com/cart', { product }).subscribe(); // HTTP de novo
  }
}
```

**Problemas:**
- ✗ Impossível testar
- ✗ Código duplicado em 10 components
- ✗ Mudou API? Quebrou tudo
- ✗ Regra de desconto espalhada

## A Solução: Separar Responsabilidades

**Mesmo código, mas organizado:**

```typescript
// 1️⃣ DOMAIN - Regras de negócio
export class Product {
  constructor(
    public id: string,
    public name: string,
    public price: number,
    public stock: number
  ) {}
  
  hasStock(): boolean {
    return this.stock > 0;
  }
  
  applyDiscount(cartSize: number): number {
    return cartSize > 5 ? this.price * 0.9 : this.price;
  }
}

// 2️⃣ APPLICATION - Caso de uso
@Injectable()
export class AddToCartUseCase {
  constructor(private repo: ProductRepository) {}
  
  execute(productId: string): Observable<void> {
    return this.repo.getById(productId).pipe(
      tap(product => {
        if (!product.hasStock()) {
          throw new OutOfStockError();
        }
      }),
      switchMap(product => this.repo.addToCart(product))
    );
  }
}

// 3️⃣ INFRASTRUCTURE - Tecnologia
@Injectable()
export class ProductHttpRepository implements ProductRepository {
  getById(id: string): Observable<Product> {
    return this.http.get(`${this.api}/products/${id}`);
  }
}

// 4️⃣ UI - Só apresentação
@Component({...})
export class ProductComponent {
  constructor(private addToCart: AddToCartUseCase) {}
  
  onAddClick(id: string) {
    this.addToCart.execute(id).subscribe({
      next: () => this.toast.success('Adicionado!'),
      error: (err) => this.handleError(err)
    });
  }
}
```

**Ganhos:**
- ✓ Testável (cada parte isolada)
- ✓ Reutilizável (use case funciona em CLI, mobile, web)
- ✓ Manutenível (mudanças localizadas)
- ✓ Legível (cada arquivo faz uma coisa)

---

<a name="solid"></a>
# PRINCÍPIOS SOLID - 5 MINUTOS

## S - Single Responsibility (Uma Responsabilidade)

```typescript
// ❌ Component faz tudo
export class UserComponent {
  validateEmail() { ... }    // Validação
  formatDate() { ... }        // Formatação
  saveUser() { ... }          // HTTP
  calculateAge() { ... }      // Lógica
}

// ✅ Cada classe faz uma coisa
export class EmailValidator { ... }
export class DateFormatter { ... }
export class UserService { ... }
export class AgeCalculator { ... }
```

## O - Open/Closed (Aberto para extensão)

```typescript
// ❌ Precisa modificar código existente
if (type === 'regular') { ... }
else if (type === 'premium') { ... }
// Adicionar novo tipo? Modifica aqui!

// ✅ Adiciona sem modificar
interface DiscountStrategy {
  calculate(order: Order): number;
}

class RegularDiscount implements DiscountStrategy { ... }
class PremiumDiscount implements DiscountStrategy { ... }
// Novo tipo? Só cria nova classe!
```

## L - Liskov Substitution (Substituível)

```typescript
// ❌ Quebra quando substitui
abstract class Bird {
  abstract fly(): void;
}

class Penguin extends Bird {
  fly() { throw new Error('Cannot fly!'); } // Quebra!
}

// ✅ Substituível sem problemas
abstract class Bird {
  abstract move(): void;
}

class Sparrow extends Bird {
  move() { this.fly(); }
}

class Penguin extends Bird {
  move() { this.swim(); }
}
```

## I - Interface Segregation (Interfaces específicas)

```typescript
// ❌ Interface gorda
interface Repository {
  findAll(): Observable<T[]>;
  save(item: T): Observable<T>;
  delete(id: string): Observable<void>;
  search(term: string): Observable<T[]>;
  // ... 10 métodos que nem todos usam
}

// ✅ Interfaces específicas
interface Reader<T> {
  findAll(): Observable<T[]>;
}

interface Writer<T> {
  save(item: T): Observable<T>;
}

interface Searcher<T> {
  search(term: string): Observable<T[]>;
}

// Use só o que precisa
class ReadOnlyService {
  constructor(private repo: Reader<User>) {}
}
```

## D - Dependency Inversion (Dependa de abstrações)

```typescript
// ❌ Depende de implementação concreta
class OrderService {
  constructor(private email: EmailSmtpService) {} // Acoplado!
}

// ✅ Depende de abstração
abstract class EmailService {
  abstract send(to: string, message: string): Observable<void>;
}

class OrderService {
  constructor(private email: EmailService) {} // Desacoplado!
}

// DI injeta implementação
providers: [
  { provide: EmailService, useClass: EmailSmtpService }
]
```

---

<a name="clean"></a>
# CLEAN ARCHITECTURE

## Conceito em 1 Minuto

**Ideia:** Separar o que importa (negócio) do que pode mudar (tecnologia).

```
┌────────────────────────┐
│  UI (Components)       │ ← Pode trocar Angular por React
│  ↓ usa                 │
├────────────────────────┤
│  Use Cases             │ ← Orquestra a lógica
│  ↓ usa                 │
├────────────────────────┤
│  Domain (Entities)     │ ← Regras de negócio PURAS
│  ↑ não depende de nada │
└────────────────────────┘
```

## Exemplo Prático: TODO App

### 1. Domain (Regras de Negócio)

```typescript
// domain/todo.entity.ts
export class Todo {
  constructor(
    public id: string,
    public title: string,
    public completed: boolean = false,
    public createdAt: Date = new Date()
  ) {}
  
  // Regra de negócio: título deve ter 3+ caracteres
  static create(title: string): Todo {
    if (title.length < 3) {
      throw new Error('Título muito curto');
    }
    return new Todo(crypto.randomUUID(), title);
  }
  
  complete(): void {
    this.completed = true;
  }
  
  reopen(): void {
    this.completed = false;
  }
}
```

### 2. Application (Use Cases)

```typescript
// application/create-todo.usecase.ts
@Injectable()
export class CreateTodoUseCase {
  constructor(private repo: TodoRepository) {}
  
  execute(title: string): Observable<Todo> {
    const todo = Todo.create(title); // Validação na entity!
    return this.repo.save(todo);
  }
}

// application/toggle-todo.usecase.ts
@Injectable()
export class ToggleTodoUseCase {
  constructor(private repo: TodoRepository) {}
  
  execute(id: string): Observable<Todo> {
    return this.repo.findById(id).pipe(
      map(todo => {
        todo.completed ? todo.reopen() : todo.complete();
        return todo;
      }),
      switchMap(todo => this.repo.save(todo))
    );
  }
}
```

### 3. Infrastructure (Implementação)

```typescript
// infrastructure/todo-localstorage.repository.ts
@Injectable()
export class TodoLocalStorageRepository implements TodoRepository {
  private key = 'todos';
  
  findAll(): Observable<Todo[]> {
    const json = localStorage.getItem(this.key) || '[]';
    return of(JSON.parse(json));
  }
  
  save(todo: Todo): Observable<Todo> {
    return this.findAll().pipe(
      map(todos => {
        const index = todos.findIndex(t => t.id === todo.id);
        if (index >= 0) {
          todos[index] = todo;
        } else {
          todos.push(todo);
        }
        localStorage.setItem(this.key, JSON.stringify(todos));
        return todo;
      })
    );
  }
}
```

### 4. Presentation (UI)

```typescript
// presentation/todo-list.component.ts
@Component({
  selector: 'app-todo-list',
  template: `
    <input #input placeholder="Nova tarefa">
    <button (click)="add(input.value)">Adicionar</button>
    
    <ul>
      <li *ngFor="let todo of todos$ | async">
        <input 
          type="checkbox" 
          [checked]="todo.completed"
          (change)="toggle(todo.id)">
        {{ todo.title }}
      </li>
    </ul>
  `
})
export class TodoListComponent {
  todos$ = this.getTodos.execute();
  
  constructor(
    private getTodos: GetTodosUseCase,
    private createTodo: CreateTodoUseCase,
    private toggleTodo: ToggleTodoUseCase
  ) {}
  
  add(title: string): void {
    this.createTodo.execute(title).subscribe(() => {
      this.todos$ = this.getTodos.execute();
    });
  }
  
  toggle(id: string): void {
    this.toggleTodo.execute(id).subscribe();
  }
}
```

## Estrutura de Pastas

```
src/app/
├── core/
│   ├── domain/
│   │   ├── entities/
│   │   │   └── todo.entity.ts
│   │   └── repositories/
│   │       └── todo.repository.ts      # Interface
│   │
│   └── application/
│       └── use-cases/
│           ├── create-todo.usecase.ts
│           ├── get-todos.usecase.ts
│           └── toggle-todo.usecase.ts
│
├── infrastructure/
│   └── repositories/
│       ├── todo-localstorage.repository.ts
│       └── todo-http.repository.ts
│
└── presentation/
    └── todo-list/
        └── todo-list.component.ts
```

## Quando Usar

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Domínio complexo (muitas regras) | CRUD ultra simples |
| App vai durar 3+ anos | MVP descartável |
| Time de 10+ devs | 1-2 devs |
| Múltiplos clientes (web, mobile) | Só web |
| Precisa trocar framework | Projeto de 1 mês |

---

<a name="ddd"></a>
# DDD (DOMAIN-DRIVEN DESIGN)

## Conceito em 1 Minuto

**Ideia:** Modelar software igual ao mundo real.

**Princípio:** Fale a mesma língua do negócio (Ubiquitous Language).

## Conceitos-Chave

### 1. Bounded Context (Contexto Delimitado)

Divida domínio grande em pedaços menores.

**Exemplo: E-commerce**

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  CATALOGO       │  │  VENDAS         │  │  ENTREGA        │
│                 │  │                 │  │                 │
│  Product        │  │  Order          │  │  Shipment       │
│  Category       │  │  OrderItem      │  │  Tracking       │
│  Price          │  │  Payment        │  │  Address        │
└─────────────────┘  └─────────────────┘  └─────────────────┘
     ↓                      ↓                      ↓
  "Produto tem       "Pedido tem           "Envio tem
   foto e preço"      total e status"       código de rastreio"
```

**Cada contexto tem sua própria definição de "Product"!**

```typescript
// Contexto: CATALOGO
export class Product {
  id: string;
  name: string;
  description: string;
  price: number;
  images: string[];
}

// Contexto: VENDAS (Product é diferente!)
export class Product {
  id: string;
  name: string;
  price: number; // Só o que importa pra venda
}

// Contexto: ESTOQUE (Product é diferente de novo!)
export class Product {
  id: string;
  quantity: number;
  location: string;
}
```

### 2. Aggregate (Agregado)

**Ideia:** Grupo de objetos tratados como uma unidade.

**Exemplo: Order (Pedido)**

```typescript
// Aggregate Root
export class Order {
  private items: OrderItem[] = [];
  private _total: number = 0;
  
  // ✅ Adiciona item via Order (mantém consistência)
  addItem(product: Product, quantity: number): void {
    const item = new OrderItem(product, quantity);
    this.items.push(item);
    this.recalculateTotal(); // Total sempre correto!
  }
  
  // ❌ NUNCA deixa modificar items diretamente
  getItems(): readonly OrderItem[] {
    return this.items; // Readonly!
  }
  
  private recalculateTotal(): void {
    this._total = this.items.reduce((sum, item) => 
      sum + item.subtotal, 0
    );
  }
}

// Entity dentro do Aggregate
export class OrderItem {
  constructor(
    public product: Product,
    public quantity: number
  ) {}
  
  get subtotal(): number {
    return this.product.price * this.quantity;
  }
}

// Uso
const order = new Order();
order.addItem(product, 2); // ✅ Certo
order.getItems()[0].quantity = 5; // ❌ Erro! Readonly
```

### 3. Value Object

**Ideia:** Objeto sem identidade, definido por valores.

```typescript
// Value Object: Money
export class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {}
  
  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Moedas diferentes!');
    }
    return new Money(this.amount + other.amount, this.currency);
  }
  
  // Value Objects são IMUTÁVEIS
  equals(other: Money): boolean {
    return this.amount === other.amount && 
           this.currency === other.currency;
  }
}

// Value Object: Email
export class Email {
  private constructor(public readonly value: string) {}
  
  static create(email: string): Email {
    if (!email.includes('@')) {
      throw new Error('Email inválido');
    }
    return new Email(email.toLowerCase());
  }
  
  getDomain(): string {
    return this.value.split('@')[1];
  }
}

// Uso
const price1 = new Money(100, 'BRL');
const price2 = new Money(50, 'BRL');
const total = price1.add(price2); // Novo objeto! Imutável

const email = Email.create('USER@EXAMPLE.COM');
console.log(email.value); // user@example.com (normalizado)
```

### 4. Domain Service

**Ideia:** Lógica que não pertence a nenhuma entity.

```typescript
// Domain Service: Transfer entre contas
@Injectable()
export class MoneyTransferService {
  transfer(from: Account, to: Account, amount: Money): void {
    // Lógica que envolve DUAS entities
    from.withdraw(amount);
    to.deposit(amount);
    
    // Poderia ter regras complexas aqui
    if (amount.amount > 10000) {
      this.notifyCompliance(from, to, amount);
    }
  }
}
```

## Exemplo Completo: Blog

```typescript
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// BOUNDED CONTEXT: Publicação
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// Value Objects
export class Slug {
  private constructor(public readonly value: string) {}
  
  static fromTitle(title: string): Slug {
    const slug = title
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/^-|-$/g, '');
    return new Slug(slug);
  }
}

export class Content {
  constructor(public readonly text: string) {
    if (text.length < 100) {
      throw new Error('Post muito curto');
    }
  }
  
  wordCount(): number {
    return this.text.split(/\s+/).length;
  }
}

// Aggregate Root
export class Post {
  private _status: 'draft' | 'published' | 'archived' = 'draft';
  private _comments: Comment[] = [];
  
  constructor(
    public readonly id: string,
    public readonly authorId: string,
    public title: string,
    public content: Content,
    public slug: Slug,
    private publishedAt?: Date
  ) {}
  
  // Regra de negócio
  publish(): void {
    if (this._status === 'published') {
      throw new Error('Já publicado');
    }
    
    this._status = 'published';
    this.publishedAt = new Date();
  }
  
  // Aggregate controla seus filhos
  addComment(author: string, text: string): void {
    if (this._status !== 'published') {
      throw new Error('Só posts publicados aceitam comentários');
    }
    
    const comment = new Comment(
      crypto.randomUUID(),
      this.id,
      author,
      text
    );
    
    this._comments.push(comment);
  }
  
  getComments(): readonly Comment[] {
    return this._comments;
  }
}

// Entity (parte do Aggregate)
export class Comment {
  constructor(
    public readonly id: string,
    public readonly postId: string,
    public readonly author: string,
    public readonly text: string,
    public readonly createdAt: Date = new Date()
  ) {}
}

// Domain Service
@Injectable()
export class PublishingService {
  canUserPublish(user: User, post: Post): boolean {
    // Lógica de autorização
    return user.id === post.authorId || user.role === 'admin';
  }
}

// Repository Interface
export abstract class PostRepository {
  abstract save(post: Post): Observable<Post>;
  abstract findBySlug(slug: Slug): Observable<Post>;
  abstract findPublished(): Observable<Post[]>;
}
```

## Estrutura de Pastas DDD

```
src/app/
├── bounded-contexts/
│   ├── catalog/                    # Contexto: Catálogo
│   │   ├── domain/
│   │   │   ├── product.entity.ts
│   │   │   ├── category.vo.ts
│   │   │   └── product.repository.ts
│   │   ├── application/
│   │   │   └── list-products.usecase.ts
│   │   └── infrastructure/
│   │       └── product-http.repository.ts
│   │
│   ├── sales/                      # Contexto: Vendas
│   │   ├── domain/
│   │   │   ├── order.aggregate.ts
│   │   │   ├── order-item.entity.ts
│   │   │   └── money.vo.ts
│   │   └── ...
│   │
│   └── shipping/                   # Contexto: Entrega
│       └── ...
│
└── shared/                         # Compartilhado entre contextos
    ├── kernel/
    │   ├── entity.base.ts
    │   └── value-object.base.ts
    └── infrastructure/
        └── http.service.ts
```

## Quando Usar DDD

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Domínio complexo e rico | CRUD simples |
| Regras de negócio mudam muito | Lógica trivial |
| Time tem conhecimento do domínio | Desenvolvedor solo |
| Precisa de Ubiquitous Language | Deadline apertado |
| Múltiplos bounded contexts | App pequeno |

---

<a name="hexagonal"></a>
# HEXAGONAL ARCHITECTURE (PORTS & ADAPTERS)

## Conceito em 1 Minuto

**Ideia:** Core isolado, comunicação via "portas".

```
        ┌──────────────────────┐
        │   ADAPTERS (OUT)     │
        │   - HTTP Repository  │
        │   - Email Service    │
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │    PORTS (OUT)       │  ← Interfaces
        │    - IRepository     │
        │    - INotification   │
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │       CORE           │  ← Lógica de negócio
        │    (Use Cases)       │
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │    PORTS (IN)        │  ← Interfaces
        │    - IAddUser        │
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │   ADAPTERS (IN)      │
        │   - REST Controller  │
        │   - GraphQL Resolver │
        └──────────────────────┘
```

## Exemplo Prático: User Registration

### 1. Core (Lógica)

```typescript
// core/register-user.usecase.ts
export class RegisterUserUseCase {
  constructor(
    private userRepo: IUserRepository,        // Port OUT
    private emailService: IEmailService,      // Port OUT
    private passwordHasher: IPasswordHasher   // Port OUT
  ) {}
  
  async execute(input: RegisterUserInput): Promise<User> {
    // Valida
    if (await this.userRepo.existsByEmail(input.email)) {
      throw new Error('Email já cadastrado');
    }
    
    // Hash da senha
    const hashedPassword = await this.passwordHasher.hash(input.password);
    
    // Cria usuário
    const user = new User(
      crypto.randomUUID(),
      input.name,
      input.email,
      hashedPassword
    );
    
    // Salva
    await this.userRepo.save(user);
    
    // Envia email
    await this.emailService.sendWelcome(user.email, user.name);
    
    return user;
  }
}
```

### 2. Ports (Interfaces)

```typescript
// ports/out/user-repository.interface.ts
export interface IUserRepository {
  save(user: User): Promise<User>;
  findById(id: string): Promise<User | null>;
  existsByEmail(email: string): Promise<boolean>;
}

// ports/out/email-service.interface.ts
export interface IEmailService {
  sendWelcome(to: string, name: string): Promise<void>;
}

// ports/out/password-hasher.interface.ts
export interface IPasswordHasher {
  hash(password: string): Promise<string>;
  verify(password: string, hash: string): Promise<boolean>;
}

// ports/in/register-user.interface.ts
export interface IRegisterUser {
  execute(input: RegisterUserInput): Promise<User>;
}
```

### 3. Adapters OUT (Infraestrutura)

```typescript
// adapters/out/user-http.repository.ts
@Injectable()
export class UserHttpRepository implements IUserRepository {
  constructor(private http: HttpClient) {}
  
  save(user: User): Promise<User> {
    return firstValueFrom(
      this.http.post<User>('/api/users', user)
    );
  }
  
  findById(id: string): Promise<User | null> {
    return firstValueFrom(
      this.http.get<User>(`/api/users/${id}`).pipe(
        catchError(() => of(null))
      )
    );
  }
  
  existsByEmail(email: string): Promise<boolean> {
    return firstValueFrom(
      this.http.get<boolean>(`/api/users/exists?email=${email}`)
    );
  }
}

// adapters/out/email-smtp.service.ts
@Injectable()
export class EmailSmtpService implements IEmailService {
  async sendWelcome(to: string, name: string): Promise<void> {
    await fetch('/api/email/send', {
      method: 'POST',
      body: JSON.stringify({
        to,
        subject: `Bem-vindo, ${name}!`,
        template: 'welcome'
      })
    });
  }
}

// adapters/out/bcrypt-hasher.service.ts
@Injectable()
export class BcryptHasher implements IPasswordHasher {
  async hash(password: string): Promise<string> {
    const salt = await bcrypt.genSalt(10);
    return bcrypt.hash(password, salt);
  }
  
  async verify(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

### 4. Adapters IN (UI/Controllers)

```typescript
// adapters/in/register-user.component.ts
@Component({
  selector: 'app-register',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="name" placeholder="Nome">
      <input formControlName="email" placeholder="Email">
      <input formControlName="password" type="password">
      <button type="submit">Cadastrar</button>
    </form>
  `
})
export class RegisterUserComponent {
  form = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    password: ['', Validators.required]
  });
  
  constructor(
    private fb: FormBuilder,
    private registerUser: IRegisterUser  // Port IN!
  ) {}
  
  onSubmit(): void {
    this.registerUser.execute(this.form.value).then(
      user => alert(`Cadastrado: ${user.name}`),
      err => alert(`Erro: ${err.message}`)
    );
  }
}
```

### 5. DI (Conecta tudo)

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    // Ports IN
    { provide: IRegisterUser, useClass: RegisterUserUseCase },
    
    // Ports OUT
    { provide: IUserRepository, useClass: UserHttpRepository },
    { provide: IEmailService, useClass: EmailSmtpService },
    { provide: IPasswordHasher, useClass: BcryptHasher },
  ]
};
```

## Estrutura de Pastas

```
src/app/
├── core/
│   └── use-cases/
│       └── register-user.usecase.ts
│
├── ports/
│   ├── in/
│   │   └── register-user.interface.ts
│   └── out/
│       ├── user-repository.interface.ts
│       ├── email-service.interface.ts
│       └── password-hasher.interface.ts
│
├── adapters/
│   ├── in/
│   │   └── register-user.component.ts
│   └── out/
│       ├── user-http.repository.ts
│       ├── email-smtp.service.ts
│       └── bcrypt-hasher.service.ts
│
└── domain/
    └── user.entity.ts
```

## Vantagens

1. **Testável:** Troca adapters por mocks
2. **Flexível:** Muda implementação sem tocar no core
3. **Isolado:** Core não conhece Angular, HTTP, etc

## Quando Usar

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Precisa trocar infraestrutura | App simples |
| Múltiplas integrações (REST, GraphQL) | Só uma API |
| Testabilidade crítica | Projeto pequeno |

---

<a name="layered"></a>
# LAYERED ARCHITECTURE (ARQUITETURA EM CAMADAS)

## Conceito

**Ideia:** Camadas empilhadas, cada uma com responsabilidade específica.

```
┌─────────────────────────┐
│  PRESENTATION           │ ← Components, Pages
├─────────────────────────┤
│  APPLICATION            │ ← Services, Use Cases
├─────────────────────────┤
│  DOMAIN                 │ ← Entities, Business Logic
├─────────────────────────┤
│  INFRASTRUCTURE         │ ← HTTP, Database, APIs
└─────────────────────────┘

Regra: Camada só conhece camada abaixo
```

## Exemplo: E-commerce

```typescript
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// LAYER 1: DOMAIN
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

export class Product {
  constructor(
    public id: string,
    public name: string,
    public price: number
  ) {}
}

export class Cart {
  private items: CartItem[] = [];
  
  add(product: Product, quantity: number): void {
    this.items.push(new CartItem(product, quantity));
  }
  
  getTotal(): number {
    return this.items.reduce((sum, item) => 
      sum + (item.product.price * item.quantity), 0
    );
  }
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// LAYER 2: INFRASTRUCTURE
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Injectable()
export class ProductApiService {
  constructor(private http: HttpClient) {}
  
  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>('/api/products');
  }
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// LAYER 3: APPLICATION
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Injectable()
export class CartService {
  private cart = new Cart();
  
  constructor(private productApi: ProductApiService) {}
  
  addProduct(productId: string): Observable<void> {
    return this.productApi.getById(productId).pipe(
      tap(product => this.cart.add(product, 1))
    );
  }
  
  getCartTotal(): number {
    return this.cart.getTotal();
  }
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// LAYER 4: PRESENTATION
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Component({...})
export class ProductListComponent {
  products$ = this.productApi.getAll();
  
  constructor(
    private productApi: ProductApiService,
    private cart: CartService
  ) {}
  
  addToCart(id: string): void {
    this.cart.addProduct(id).subscribe();
  }
}
```

## Estrutura de Pastas

```
src/app/
├── presentation/
│   ├── pages/
│   └── components/
│
├── application/
│   └── services/
│
├── domain/
│   ├── entities/
│   └── interfaces/
│
└── infrastructure/
    ├── http/
    └── storage/
```

## Quando Usar

**Arquitetura mais simples**, boa para começar.

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Projeto médio | App muito complexo (prefira Clean/Hexagonal) |
| Time conhece MVC | Precisa isolar domínio completamente |
| Quer simplicidade | Múltiplos clientes (prefira Clean) |

---

<a name="cqrs"></a>
# CQRS (COMMAND QUERY RESPONSIBILITY SEGREGATION)

## Conceito em 1 Minuto

**Ideia:** Separar leitura (Query) de escrita (Command).

```
          WRITE                    READ
       (Commands)               (Queries)
            ↓                       ↓
    ┌──────────────┐       ┌──────────────┐
    │  Write Model │       │  Read Model  │
    │   (normalize)│       │  (denormalize)│
    └──────┬───────┘       └───────┬──────┘
           │                       │
           ↓                       ↓
    ┌──────────────┐       ┌──────────────┐
    │  Write DB    │──────>│   Read DB    │
    │  (PostgreSQL)│ sync  │  (MongoDB)   │
    └──────────────┘       └──────────────┘
```

## Exemplo: Blog

### Commands (Escrita)

```typescript
// commands/create-post.command.ts
export class CreatePostCommand {
  constructor(
    public title: string,
    public content: string,
    public authorId: string
  ) {}
}

@Injectable()
export class CreatePostHandler {
  constructor(
    private writeRepo: PostWriteRepository,
    private eventBus: EventBus
  ) {}
  
  async execute(cmd: CreatePostCommand): Promise<void> {
    const post = new Post(
      crypto.randomUUID(),
      cmd.title,
      cmd.content,
      cmd.authorId
    );
    
    await this.writeRepo.save(post);
    
    // Publica evento
    this.eventBus.publish(new PostCreatedEvent(post));
  }
}

// commands/publish-post.command.ts
export class PublishPostCommand {
  constructor(public postId: string) {}
}

@Injectable()
export class PublishPostHandler {
  async execute(cmd: PublishPostCommand): Promise<void> {
    const post = await this.writeRepo.findById(cmd.postId);
    post.publish();
    await this.writeRepo.save(post);
    
    this.eventBus.publish(new PostPublishedEvent(post));
  }
}
```

### Queries (Leitura)

```typescript
// queries/get-post.query.ts
export class GetPostQuery {
  constructor(public slug: string) {}
}

@Injectable()
export class GetPostHandler {
  constructor(private readRepo: PostReadRepository) {}
  
  async execute(query: GetPostQuery): Promise<PostDTO> {
    // Read Model já vem desnormalizado!
    return this.readRepo.findBySlug(query.slug);
  }
}

// queries/list-posts.query.ts
export class ListPostsQuery {
  constructor(
    public page: number,
    public pageSize: number
  ) {}
}

@Injectable()
export class ListPostsHandler {
  constructor(private readRepo: PostReadRepository) {}
  
  async execute(query: ListPostsQuery): Promise<PostDTO[]> {
    return this.readRepo.findAll(query.page, query.pageSize);
  }
}
```

### Models Diferentes

```typescript
// Write Model (normalizado)
export class Post {
  constructor(
    public id: string,
    public title: string,
    public content: string,
    public authorId: string,
    public status: 'draft' | 'published' = 'draft'
  ) {}
  
  publish(): void {
    if (this.status === 'published') {
      throw new Error('Já publicado');
    }
    this.status = 'published';
  }
}

// Read Model (desnormalizado)
export interface PostDTO {
  id: string;
  title: string;
  content: string;
  author: {              // Dados do autor já inclusos!
    id: string;
    name: string;
    avatar: string;
  };
  commentsCount: number; // Contador pré-calculado!
  publishedAt: Date;
}
```

### Event Sourcing (Opcional)

```typescript
// Sincroniza Write → Read
@Injectable()
export class PostEventHandler {
  constructor(private readRepo: PostReadRepository) {}
  
  @EventListener(PostCreatedEvent)
  async onPostCreated(event: PostCreatedEvent): Promise<void> {
    // Atualiza Read Model
    const dto: PostDTO = {
      id: event.post.id,
      title: event.post.title,
      content: event.post.content,
      author: await this.getAuthorData(event.post.authorId),
      commentsCount: 0,
      publishedAt: null
    };
    
    await this.readRepo.save(dto);
  }
  
  @EventListener(PostPublishedEvent)
  async onPostPublished(event: PostPublishedEvent): Promise<void> {
    await this.readRepo.updateStatus(event.post.id, 'published');
  }
}
```

## Estrutura de Pastas

```
src/app/
├── commands/
│   ├── create-post.command.ts
│   ├── publish-post.command.ts
│   └── handlers/
│
├── queries/
│   ├── get-post.query.ts
│   ├── list-posts.query.ts
│   └── handlers/
│
├── write-model/
│   ├── entities/
│   └── repositories/
│
├── read-model/
│   ├── dtos/
│   └── repositories/
│
└── events/
    ├── post-created.event.ts
    └── handlers/
```

## Quando Usar

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Leitura >> Escrita | CRUD equilibrado |
| Precisa performance de leitura | App simples |
| Modelos de leitura/escrita muito diferentes | Consistência imediata obrigatória |
| Auditoria/Event Sourcing | Complexidade não justifica |

---

<a name="eventdriven"></a>
# EVENT-DRIVEN ARCHITECTURE

## Conceito

**Ideia:** Componentes se comunicam via eventos, não chamadas diretas.

```
┌──────────────┐         ┌──────────────┐
│  Component A │ ──────> │  Event Bus   │
└──────────────┘ emite   └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │  Component B │ escuta
                         └──────────────┘
                         ┌──────────────┐
                         │  Component C │ escuta
                         └──────────────┘
```

## Exemplo Simples: Notificações

```typescript
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// EVENTS
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

export class UserRegisteredEvent {
  constructor(
    public userId: string,
    public email: string,
    public name: string
  ) {}
}

export class OrderPlacedEvent {
  constructor(
    public orderId: string,
    public userId: string,
    public total: number
  ) {}
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// EVENT BUS (RxJS)
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Injectable({ providedIn: 'root' })
export class EventBus {
  private subject = new Subject<any>();
  
  publish<T>(event: T): void {
    this.subject.next(event);
  }
  
  on<T>(eventType: new (...args: any[]) => T): Observable<T> {
    return this.subject.pipe(
      filter(event => event instanceof eventType)
    );
  }
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// PUBLISHERS (Quem emite eventos)
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Injectable()
export class AuthService {
  constructor(private eventBus: EventBus) {}
  
  register(name: string, email: string): void {
    const userId = crypto.randomUUID();
    
    // Salva usuário...
    
    // Publica evento
    this.eventBus.publish(new UserRegisteredEvent(
      userId,
      email,
      name
    ));
  }
}

@Injectable()
export class OrderService {
  constructor(private eventBus: EventBus) {}
  
  placeOrder(userId: string, items: any[]): void {
    const orderId = crypto.randomUUID();
    const total = this.calculateTotal(items);
    
    // Salva pedido...
    
    // Publica evento
    this.eventBus.publish(new OrderPlacedEvent(
      orderId,
      userId,
      total
    ));
  }
}

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// SUBSCRIBERS (Quem escuta eventos)
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

@Injectable()
export class EmailNotificationService {
  constructor(private eventBus: EventBus) {
    this.listen();
  }
  
  private listen(): void {
    // Escuta UserRegisteredEvent
    this.eventBus.on(UserRegisteredEvent).subscribe(event => {
      this.sendWelcomeEmail(event.email, event.name);
    });
    
    // Escuta OrderPlacedEvent
    this.eventBus.on(OrderPlacedEvent).subscribe(event => {
      this.sendOrderConfirmation(event.orderId);
    });
  }
  
  private sendWelcomeEmail(email: string, name: string): void {
    console.log(`Enviando email de boas-vindas para ${email}`);
  }
  
  private sendOrderConfirmation(orderId: string): void {
    console.log(`Enviando confirmação de pedido ${orderId}`);
  }
}

@Injectable()
export class AnalyticsService {
  constructor(private eventBus: EventBus) {
    this.listen();
  }
  
  private listen(): void {
    this.eventBus.on(UserRegisteredEvent).subscribe(event => {
      console.log('Analytics: Novo usuário registrado');
    });
    
    this.eventBus.on(OrderPlacedEvent).subscribe(event => {
      console.log(`Analytics: Pedido de R$ ${event.total}`);
    });
  }
}

@Injectable()
export class LoyaltyPointsService {
  constructor(private eventBus: EventBus) {
    this.listen();
  }
  
  private listen(): void {
    // Dá pontos quando faz pedido
    this.eventBus.on(OrderPlacedEvent).subscribe(event => {
      const points = Math.floor(event.total / 10);
      this.addPoints(event.userId, points);
    });
  }
  
  private addPoints(userId: string, points: number): void {
    console.log(`Adicionando ${points} pontos para usuário ${userId}`);
  }
}
```

## Vantagens

1. **Desacoplamento:** Serviços não se conhecem
2. **Escalável:** Adiciona subscribers sem modificar publishers
3. **Auditável:** Log de todos os eventos

## Quando Usar

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Múltiplos módulos reagindo ao mesmo evento | App simples com poucas integrações |
| Precisa audit trail | Debugging complexo não justifica |
| Microservices | Processamento síncrono obrigatório |

---

<a name="microfrontends"></a>
# MICRO FRONTENDS

## Conceito

**Ideia:** Divida frontend em apps independentes.

```
┌─────────────────────────────────────────────┐
│          Shell App (Container)              │
│  ┌────────┐  ┌────────┐  ┌────────┐        │
│  │ Header │  │Catalog │  │Checkout│        │
│  │ Team A │  │ Team B │  │ Team C │        │
│  └────────┘  └────────┘  └────────┘        │
└─────────────────────────────────────────────┘

Cada time tem:
- Repositório separado
- Deploy independente
- Stack próprio (pode ser Angular, React, Vue)
```

## Estratégias de Implementação

### 1. Module Federation (Webpack 5)

```typescript
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// APP 1: Shell (Container)
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// webpack.config.js (shell)
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        catalog: 'catalog@http://localhost:4201/remoteEntry.js',
        checkout: 'checkout@http://localhost:4202/remoteEntry.js'
      },
      shared: {
        '@angular/core': { singleton: true },
        '@angular/common': { singleton: true }
      }
    })
  ]
};

// app-routing.module.ts (shell)
const routes: Routes = [
  {
    path: 'catalog',
    loadChildren: () => import('catalog/Module').then(m => m.CatalogModule)
  },
  {
    path: 'checkout',
    loadChildren: () => import('checkout/Module').then(m => m.CheckoutModule)
  }
];

// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
// APP 2: Catalog (Remote)
// ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

// webpack.config.js (catalog)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'catalog',
      filename: 'remoteEntry.js',
      exposes: {
        './Module': './src/app/catalog/catalog.module.ts'
      },
      shared: {
        '@angular/core': { singleton: true },
        '@angular/common': { singleton: true }
      }
    })
  ]
};
```

### 2. iframes (Simples mas limitado)

```typescript
// shell.component.ts
@Component({
  selector: 'app-shell',
  template: `
    <nav>
      <a (click)="navigate('catalog')">Catálogo</a>
      <a (click)="navigate('checkout')">Checkout</a>
    </nav>
    
    <iframe 
      [src]="currentUrl | safe" 
      width="100%" 
      height="600">
    </iframe>
  `
})
export class ShellComponent {
  currentUrl = 'http://localhost:4201';
  
  navigate(app: string): void {
    const urls = {
      catalog: 'http://localhost:4201',
      checkout: 'http://localhost:4202'
    };
    this.currentUrl = urls[app];
  }
}
```

### 3. Web Components

```typescript
// catalog-app (web component)
@Component({
  selector: 'catalog-app',
  template: `<h1>Catálogo</h1>`
})
export class CatalogAppComponent {}

// main.ts (catalog)
import { createCustomElement } from '@angular/elements';

const CatalogElement = createCustomElement(CatalogAppComponent, {
  injector: this.injector
});
customElements.define('catalog-app', CatalogElement);

// shell.component.html
<catalog-app></catalog-app>
<checkout-app></checkout-app>
```

## Comunicação Entre Microfrontends

```typescript
// shared-events.service.ts
@Injectable({ providedIn: 'root' })
export class SharedEventsService {
  private subject = new Subject<any>();
  
  publish(event: any): void {
    // Também publica no window para comunicação cross-app
    window.postMessage(event, '*');
    this.subject.next(event);
  }
  
  on(): Observable<any> {
    return merge(
      this.subject.asObservable(),
      fromEvent(window, 'message').pipe(
        map((event: any) => event.data)
      )
    );
  }
}

// catalog-app usa
this.sharedEvents.publish({ type: 'PRODUCT_ADDED', productId: '123' });

// checkout-app escuta
this.sharedEvents.on().subscribe(event => {
  if (event.type === 'PRODUCT_ADDED') {
    this.addToCart(event.productId);
  }
});
```

## Estrutura de Monorepo

```
apps/
├── shell/                    # Container app
│   ├── src/
│   └── webpack.config.js
│
├── catalog/                  # Microfrontend 1
│   ├── src/
│   └── webpack.config.js
│
├── checkout/                 # Microfrontend 2
│   ├── src/
│   └── webpack.config.js
│
└── shared/                   # Libs compartilhadas
    ├── ui-components/
    ├── utils/
    └── types/
```

## Quando Usar

| ✅ Use quando | ❌ Evite quando |
|--------------|----------------|
| Times grandes (50+ devs) | Time pequeno (<10 devs) |
| Deploy independente necessário | Deploy monolítico ok |
| Times autônomos | Forte interdependência |
| Escala horizontal | App pequeno/médio |

---

<a name="comparacao"></a>
# COMPARAÇÃO DAS ARQUITETURAS

## Tabela Resumida

| Arquitetura | Complexidade | Testabilidade | Escalabilidade | Quando Usar |
|-------------|--------------|---------------|----------------|-------------|
| **Layered** | ⭐ Baixa | ⭐⭐ Média | ⭐⭐ Média | Apps médios, CRUD |
| **Clean** | ⭐⭐⭐ Alta | ⭐⭐⭐⭐⭐ Excelente | ⭐⭐⭐⭐ Alta | Domínio rico, longo prazo |
| **Hexagonal** | ⭐⭐⭐ Alta | ⭐⭐⭐⭐⭐ Excelente | ⭐⭐⭐⭐ Alta | Múltiplas integrações |
| **DDD** | ⭐⭐⭐⭐ Muito Alta | ⭐⭐⭐⭐ Alta | ⭐⭐⭐⭐⭐ Excelente | Domínio complexo |
| **CQRS** | ⭐⭐⭐⭐ Muito Alta | ⭐⭐⭐ Média | ⭐⭐⭐⭐⭐ Excelente | Leitura >> Escrita |
| **Event-Driven** | ⭐⭐⭐ Alta | ⭐⭐⭐ Média | ⭐⭐⭐⭐⭐ Excelente | Microservices, Audit |
| **Micro Frontends** | ⭐⭐⭐⭐⭐ Extrema | ⭐⭐ Baixa | ⭐⭐⭐⭐⭐ Excelente | Times grandes, autonomia |

## Por Tamanho de Projeto

```
Projeto Pequeno (1-5 devs, 1-6 meses)
└─> Layered ou Clean simplificado

Projeto Médio (5-15 devs, 6-24 meses)
└─> Clean ou Hexagonal

Projeto Grande (15-50 devs, 2+ anos)
└─> DDD + Clean ou Hexagonal

Projeto Enterprise (50+ devs, múltiplos times)
└─> DDD + CQRS + Event-Driven + Micro Frontends
```

---

<a name="estruturas"></a>
# ESTRUTURAS DE PASTAS

## 1. Clean Architecture

```
src/app/
├── core/
│   ├── domain/
│   │   ├── entities/
│   │   ├── value-objects/
│   │   ├── repositories/        # Interfaces
│   │   └── errors/
│   │
│   └── application/
│       └── use-cases/
│
├── infrastructure/
│   ├── repositories/            # Implementations
│   ├── http/
│   └── storage/
│
└── presentation/
    ├── pages/
    └── components/
```

## 2. DDD

```
src/app/
├── bounded-contexts/
│   ├── catalog/
│   │   ├── domain/
│   │   ├── application/
│   │   └── infrastructure/
│   │
│   ├── sales/
│   │   ├── domain/
│   │   ├── application/
│   │   └── infrastructure/
│   │
│   └── shipping/
│       └── ...
│
└── shared/
    ├── kernel/
    └── infrastructure/
```

## 3. Feature-Sliced

```
src/app/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── services/
│   │   ├── store/
│   │   └── auth.routes.ts
│   │
│   ├── products/
│   │   └── ...
│   │
│   └── cart/
│       └── ...
│
├── shared/
│   ├── ui/
│   ├── utils/
│   └── types/
│
└── core/
    ├── services/
    └── guards/
```

---

<a name="migracao"></a>
# COMO MIGRAR PARA CLEAN ARCHITECTURE

## Estratégia: Strangler Fig Pattern

**Não reescreva tudo! Migre gradualmente.**

```
┌──────────────────────────────────┐
│     APP LEGADO (Monolito)        │
│  ┌────────────────────────────┐  │
│  │  Features existentes       │  │
│  │  (código antigo)           │  │
│  └────────────────────────────┘  │
│                                   │
│  ┌────────────────────────────┐  │
│  │  Nova Feature              │  │ ← Nova feature já em Clean
│  │  (Clean Architecture)      │  │
│  └────────────────────────────┘  │
│                                   │
│  ┌────────────────────────────┐  │
│  │  Feature migrada           │  │ ← Feature antiga migrada
│  │  (Clean Architecture)      │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘

Aos poucos, Clean "estrangula" o legado
```

## Passo a Passo

### Passo 1: Crie estrutura de pastas

```bash
mkdir -p src/app/core/domain/entities
mkdir -p src/app/core/domain/repositories
mkdir -p src/app/core/application/use-cases
mkdir -p src/app/infrastructure/repositories
```

### Passo 2: Escolha UMA feature para migrar

Comece pela mais simples!

```typescript
// ANTES (Legado)
@Component({...})
export class UserListComponent {
  users: any[] = [];
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    this.http.get('/api/users').subscribe(data => {
      this.users = data;
    });
  }
}
```

### Passo 3: Extraia Entity

```typescript
// core/domain/entities/user.entity.ts
export class User {
  constructor(
    public id: string,
    public name: string,
    public email: string
  ) {}
  
  isActive(): boolean {
    return this.status === 'active';
  }
}
```

### Passo 4: Crie Repository Interface

```typescript
// core/domain/repositories/user.repository.ts
export abstract class UserRepository {
  abstract findAll(): Observable<User[]>;
  abstract findById(id: string): Observable<User>;
}
```

### Passo 5: Implemente Repository

```typescript
// infrastructure/repositories/user-http.repository.ts
@Injectable()
export class UserHttpRepository implements UserRepository {
  constructor(private http: HttpClient) {}
  
  findAll(): Observable<User[]> {
    return this.http.get<any[]>('/api/users').pipe(
      map(dtos => dtos.map(dto => new User(dto.id, dto.name, dto.email)))
    );
  }
}
```

### Passo 6: Crie Use Case

```typescript
// core/application/use-cases/get-users.usecase.ts
@Injectable()
export class GetUsersUseCase {
  constructor(private repo: UserRepository) {}
  
  execute(): Observable<User[]> {
    return this.repo.findAll();
  }
}
```

### Passo 7: Atualize Component

```typescript
// presentation/user-list.component.ts (MIGRADO)
@Component({...})
export class UserListComponent {
  users$ = this.getUsers.execute();
  
  constructor(private getUsers: GetUsersUseCase) {}
}
```

### Passo 8: Configure DI

```typescript
// app.config.ts
providers: [
  { provide: UserRepository, useClass: UserHttpRepository }
]
```

### Passo 9: Repita para outras features

Migre uma feature por sprint, não tudo de uma vez!

---

<a name="checklist"></a>
# CHECKLIST DE DECISÃO

## Perguntas para Escolher Arquitetura

### 1. Tamanho do Projeto

- [ ] 1-3 devs → **Layered**
- [ ] 3-10 devs → **Clean** ou **Hexagonal**
- [ ] 10-30 devs → **DDD + Clean**
- [ ] 30+ devs → **DDD + Micro Frontends**

### 2. Complexidade do Domínio

- [ ] CRUD simples → **Layered**
- [ ] Regras de negócio médias → **Clean**
- [ ] Domínio rico e complexo → **DDD**
- [ ] Múltiplos contextos → **DDD + Bounded Contexts**

### 3. Tempo de Vida

- [ ] 1-6 meses → **Layered** ou sem arquitetura
- [ ] 6 meses - 2 anos → **Clean**
- [ ] 2-5 anos → **DDD + Clean**
- [ ] 5+ anos → **DDD + CQRS + Event-Driven**

### 4. Necessidades Especiais

- [ ] Performance de leitura crítica → **CQRS**
- [ ] Auditoria obrigatória → **Event-Driven**
- [ ] Múltiplas integrações → **Hexagonal**
- [ ] Deploy independente → **Micro Frontends**
- [ ] Testabilidade máxima → **Clean** ou **Hexagonal**

### 5. Time

- [ ] Júnior → **Layered** (simples)
- [ ] Pleno → **Clean** (médio)
- [ ] Sênior → **DDD** (complexo)
- [ ] Múltiplos times → **Micro Frontends**

## Decisão Final

**Regra de Ouro:** Comece simples, refatore quando necessário.

```
MVP → Layered
  ↓ Cresce
Clean Architecture
  ↓ Fica complexo
DDD + Clean
  ↓ Times grandes
Micro Frontends
```

---

# 🎯 RESUMO EXECUTIVO

## Se tem 5 minutos, leia isso:

**Layered:** Bom para começar (⭐ simplicidade)  
**Clean:** Melhor opção para maioria dos casos (⭐⭐⭐ testável, manutenível)  
**Hexagonal:** Quando precisa trocar infraestrutura facilmente  
**DDD:** Quando domínio é rico e complexo  
**CQRS:** Quando leitura >> escrita  
**Event-Driven:** Quando precisa de audit trail e desacoplamento  
**Micro Frontends:** Quando tem 50+ devs em times autônomos  

**Recomendação Padrão:**
- Projeto pequeno/médio → **Clean Architecture**
- Projeto grande → **DDD + Clean Architecture**
- Empresa → **DDD + CQRS + Event-Driven + Micro Frontends**

**Migração:** Use Strangler Fig Pattern (migre gradualmente)

---

## 📚 RECURSOS EXTRAS

### Livros
- Clean Architecture (Robert C. Martin)
- Domain-Driven Design (Eric Evans)
- Building Microservices (Sam Newman)

### Links
- https://blog.cleancoder.com
- https://martinfowler.com/architecture/
- https://angular.io/guide/architecture

---

**FIM DO GUIA**

Este documento é um guia prático. Adapte conforme seu contexto!

**Boa arquitetura não é a mais complexa, é a adequada ao seu problema.**
