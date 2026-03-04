# 🏛️ ARQUITETURAS DE SOFTWARE NO ANGULAR
## Guia Completo: Do Desenvolvedor ao Arquiteto

**Autor:** Guia prático para entrevistas Senior e arquitetura de aplicações escaláveis  
**Versão:** 2025  
**Objetivo:** Ensinar QUANDO e COMO usar cada arquitetura no Angular

---

## 📚 SUMÁRIO

### PARTE 1: FUNDAMENTOS
1. [Por Que Arquitetura Importa](#parte1-1)
2. [Evolução Arquitetural: Monolito → Micro Frontends](#parte1-2)
3. [Princípios SOLID Aplicados](#parte1-3)

### PARTE 2: ARQUITETURAS PRINCIPAIS
4. [Clean Architecture (Arquitetura Limpa)](#parte2-1)
5. [DDD (Domain-Driven Design)](#parte2-2)
6. [Hexagonal Architecture (Ports & Adapters)](#parte2-3)
7. [Layered Architecture (Arquitetura em Camadas)](#parte2-4)
8. [CQRS (Command Query Responsibility Segregation)](#parte2-5)
9. [Event-Driven Architecture](#parte2-6)

### PARTE 3: ARQUITETURAS FRONTEND
10. [Feature-Sliced Design](#parte3-1)
11. [Micro Frontends](#parte3-2)
12. [Module Federation](#parte3-3)

### PARTE 4: PRÁTICA
13. [Comparação: Quando Usar Cada Arquitetura](#parte4-1)
14. [Estrutura de Pastas - Apps Reais](#parte4-2)
15. [Migração Gradual (Strangler Fig Pattern)](#parte4-3)
16. [Anti-Patterns e Erros Comuns](#parte4-4)
17. [Checklist do Arquiteto](#parte4-5)

---

<a name="parte1-1"></a>
# PARTE 1: FUNDAMENTOS

## 1. POR QUE ARQUITETURA IMPORTA

### 1.1 O Cenário Sem Arquitetura

Imagine um e-commerce que começou simples:

```typescript
// ❌ SEM ARQUITETURA - Tudo misturado
@Component({
  selector: 'app-product-list',
  template: `
    <div *ngFor="let product of products">
      <h3>{{ product.name }}</h3>
      <p>{{ product.price }}</p>
      <button (click)="addToCart(product)">Adicionar</button>
    </div>
  `
})
export class ProductListComponent implements OnInit {
  products: any[] = [];
  cart: any[] = [];
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    // Lógica HTTP no component
    this.http.get('https://api.example.com/products').subscribe(data => {
      this.products = data;
    });
    
    // Lê carrinho do localStorage
    const savedCart = localStorage.getItem('cart');
    if (savedCart) {
      this.cart = JSON.parse(savedCart);
    }
  }
  
  addToCart(product: any) {
    // Validação no component
    if (product.stock <= 0) {
      alert('Produto sem estoque!');
      return;
    }
    
    // Lógica de desconto no component
    let price = product.price;
    if (this.cart.length > 5) {
      price = price * 0.9; // 10% desconto
    }
    
    // Adiciona no carrinho
    this.cart.push({ ...product, finalPrice: price });
    
    // Salva no localStorage
    localStorage.setItem('cart', JSON.stringify(this.cart));
    
    // Atualiza estoque via HTTP
    this.http.put(`https://api.example.com/products/${product.id}`, {
      ...product,
      stock: product.stock - 1
    }).subscribe();
    
    // Analytics
    console.log('Product added:', product.id);
  }
}
```

**Problemas identificados:**

| Problema | Impacto | Exemplo |
|----------|---------|---------|
| **Alto Acoplamento** | Difícil testar | Component depende de HTTP, localStorage, alert |
| **Lógica Duplicada** | Código repetido | Validação de estoque copiada em 5 components |
| **Sem Tipagem** | Bugs em runtime | `any[]` aceita qualquer coisa |
| **Lógica de Negócio no UI** | Impossível reutilizar | Regra de desconto só funciona nesse component |
| **Dependência de Detalhes** | Difícil trocar backend | URL hardcoded em 20 lugares |
| **Violação SRP** | Component gigante | 1 component faz 10 coisas diferentes |

**O que acontece depois de 6 meses:**

```typescript
// O mesmo component agora tem 800 linhas...
export class ProductListComponent {
  // 50 propriedades
  products: any[];
  cart: any[];
  filters: any;
  sorting: any;
  pagination: any;
  // ...
  
  // 30 métodos
  ngOnInit() { /* 100 linhas */ }
  addToCart() { /* 80 linhas */ }
  removeFromCart() { /* 60 linhas */ }
  applyFilters() { /* 120 linhas */ }
  // ...
  
  // Ninguém mais entende esse código
  // Qualquer mudança quebra algo
  // Testes? Impossível.
}
```

### 1.2 Com Arquitetura - A Transformação

Agora, a mesma funcionalidade com **Clean Architecture**:

```typescript
// ✅ COM ARQUITETURA

// 1. DOMAIN - Regras de negócio puras
export class Product {
  constructor(
    public readonly id: string,
    public name: string,
    public price: Money,
    public stock: number
  ) {}
  
  hasStock(): boolean {
    return this.stock > 0;
  }
  
  decreaseStock(quantity: number): void {
    if (quantity > this.stock) {
      throw new InsufficientStockError(this.id, this.stock, quantity);
    }
    this.stock -= quantity;
  }
}

export class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {}
  
  applyDiscount(percentage: number): Money {
    return new Money(
      this.amount * (1 - percentage),
      this.currency
    );
  }
}

// 2. APPLICATION - Use Case (orquestra lógica)
@Injectable()
export class AddProductToCartUseCase {
  constructor(
    private productRepo: ProductRepository,
    private cartRepo: CartRepository,
    private discountService: DiscountService
  ) {}
  
  execute(productId: string, quantity: number): Observable<Cart> {
    return this.productRepo.findById(productId).pipe(
      switchMap(product => {
        // Valida estoque
        if (!product.hasStock()) {
          throw new OutOfStockError(productId);
        }
        
        // Busca carrinho atual
        return this.cartRepo.getCurrent().pipe(
          map(cart => ({ product, cart }))
        );
      }),
      switchMap(({ product, cart }) => {
        // Aplica desconto se aplicável
        const discount = this.discountService.calculate(cart);
        const finalPrice = product.price.applyDiscount(discount);
        
        // Adiciona item no carrinho
        const item = new CartItem(product, quantity, finalPrice);
        cart.addItem(item);
        
        // Salva carrinho
        return this.cartRepo.save(cart);
      })
    );
  }
}

// 3. INFRASTRUCTURE - Implementação técnica
@Injectable()
export class ProductHttpRepository implements ProductRepository {
  constructor(private http: HttpClient) {}
  
  findById(id: string): Observable<Product> {
    return this.http.get<ProductDTO>(`${this.apiUrl}/products/${id}`).pipe(
      map(dto => this.mapper.toDomain(dto))
    );
  }
}

// 4. PRESENTATION - UI simples
@Component({
  selector: 'app-product-card',
  template: `
    <div class="product-card">
      <h3>{{ product.name }}</h3>
      <p>{{ product.price.amount | currency }}</p>
      <button 
        (click)="onAddToCart()"
        [disabled]="!product.hasStock() || loading">
        {{ product.hasStock() ? 'Adicionar' : 'Sem Estoque' }}
      </button>
    </div>
  `
})
export class ProductCardComponent {
  @Input() product!: Product;
  loading = false;
  
  constructor(
    private addToCart: AddProductToCartUseCase,
    private toastr: ToastrService
  ) {}
  
  onAddToCart() {
    this.loading = true;
    
    this.addToCart.execute(this.product.id, 1).subscribe({
      next: () => {
        this.toastr.success('Produto adicionado!');
        this.loading = false;
      },
      error: (err) => {
        if (err instanceof OutOfStockError) {
          this.toastr.error('Produto sem estoque');
        } else {
          this.toastr.error('Erro ao adicionar produto');
        }
        this.loading = false;
      }
    });
  }
}
```

**Benefícios alcançados:**

| Antes | Depois |
|-------|--------|
| Component de 800 linhas | Component de 30 linhas |
| Impossível testar | Cada camada testável isoladamente |
| Lógica duplicada | Use Case reutilizável em CLI, Mobile, Web |
| Mudança quebra tudo | Mudanças localizadas |
| Sem tipagem | Type-safe end-to-end |
| Acoplado ao Angular | Domain independente de framework |

### 1.3 Arquitetura = Investimento de Longo Prazo

```
              CUSTO vs BENEFÍCIO ao longo do tempo

Custo
  ↑
  │                       ╱ Sem Arquitetura
  │                     ╱   (custo explode)
  │                   ╱
  │                 ╱
  │               ╱
  │             ╱
  │           ╱_____________ Com Arquitetura
  │         ╱                (custo controlado)
  │       ╱
  │     ╱
  │   ╱
  │ ╱
  └──────────────────────────────────→ Tempo
  0   3m   6m   1a   2a   3a   5a
  
  Ponto de Break-even: ~6 meses
  Após 6 meses: arquitetura compensa
```

**Quando vale o investimento:**

✅ **SIM, se:**
- App vai durar 1+ ano
- Time com 3+ desenvolvedores
- Domínio complexo (regras de negócio)
- Alta criticidade (financeiro, saúde)
- Múltiplos clientes (Web, Mobile, Desktop)

❌ **NÃO, se:**
- MVP descartável
- CRUD ultra simples
- 1 desenvolvedor sozinho
- Deadline de 2 semanas
- Protótipo para validação

---

<a name="parte1-2"></a>
## 2. EVOLUÇÃO ARQUITETURAL

### 2.1 Da Bagunça ao Micro Frontend

```
EVOLUÇÃO TÍPICA DE UMA STARTUP

┌─────────────────────────────────────────────────────┐
│  FASE 1: MVP (0-6 meses, 1-2 devs)                 │
│  ┌─────────────┐                                    │
│  │  Monolito   │  ← Tudo em um AppModule gigante   │
│  │  Angular    │     Velocidade > Qualidade         │
│  └─────────────┘                                    │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  FASE 2: Crescimento (6m-2a, 3-10 devs)            │
│  ┌─────────────────────────────────┐                │
│  │  Feature Modules                │                │
│  │  ┌────────┐  ┌────────┐        │                │
│  │  │ Users  │  │ Orders │  ...   │                │
│  │  └────────┘  └────────┘        │                │
│  └─────────────────────────────────┘                │
│  ↑ Módulos lazy-loaded                              │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  FASE 3: Maturidade (2-5a, 10-30 devs)             │
│  ┌─────────────────────────────────┐                │
│  │  Clean Architecture + DDD       │                │
│  │  ┌──────────────────┐           │                │
│  │  │  Bounded Context │           │                │
│  │  │  ┌────┐  ┌────┐ │           │                │
│  │  │  │Ctx1│  │Ctx2│ │ ...       │                │
│  │  │  └────┘  └────┘ │           │                │
│  │  └──────────────────┘           │                │
│  └─────────────────────────────────┘                │
│  ↑ Domínio rico, regras complexas                   │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│  FASE 4: Scale (5a+, 30+ devs, múltiplos times)    │
│  ┌──────┐  ┌──────┐  ┌──────┐                      │
│  │App 1 │  │App 2 │  │App 3 │                      │
│  │Team A│  │Team B│  │Team C│                      │
│  └──────┘  └──────┘  └──────┘                      │
│      ↓         ↓         ↓                          │
│  ┌────────────────────────────┐                     │
│  │   Shell App (Module Fed)  │                     │
│  └────────────────────────────┘                     │
│  ↑ Micro Frontends, autonomia total                 │
└─────────────────────────────────────────────────────┘
```

### 2.2 Tabela de Decisão por Tamanho

| Métrica | Monolito Simples | Modular | Clean/Hexagonal | Micro Frontends |
|---------|-----------------|---------|-----------------|-----------------|
| **Devs** | 1-2 | 3-10 | 10-30 | 30+ |
| **Features** | 1-5 | 5-20 | 20-50 | 50+ |
| **LOC** | <10k | 10k-50k | 50k-200k | 200k+ |
| **Complexidade Domínio** | Baixa | Média | Alta | Muito Alta |
| **Times** | 1 | 1-2 | 2-5 | 5+ |
| **Deploy** | Monolítico | Monolítico | Monolítico | Independente |
| **Custo Setup** | Baixo | Médio | Alto | Muito Alto |
| **Manutenibilidade** | Baixa | Média | Alta | Muito Alta |

---

<a name="parte1-3"></a>
## 3. PRINCÍPIOS SOLID APLICADOS AO ANGULAR

Antes de arquiteturas, precisa dominar SOLID:

### 3.1 S - Single Responsibility Principle

**Um component/service deve ter UMA razão para mudar.**

```typescript
// ❌ VIOLAÇÃO SRP - Component faz TUDO
@Component({ /* ... */ })
export class UserComponent {
  users: User[] = [];
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    // Responsabilidade 1: HTTP
    this.http.get('/api/users').subscribe();
  }
  
  validateEmail(email: string): boolean {
    // Responsabilidade 2: Validação
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
  
  formatDate(date: Date): string {
    // Responsabilidade 3: Formatação
    return date.toLocaleDateString();
  }
  
  calculateAge(birthDate: Date): number {
    // Responsabilidade 4: Cálculo
    const diff = Date.now() - birthDate.getTime();
    return Math.floor(diff / (1000 * 60 * 60 * 24 * 365));
  }
}
```

```typescript
// ✅ SRP APLICADO - Cada classe faz UMA coisa

// Service: só HTTP
@Injectable()
export class UserHttpService {
  getUsers(): Observable<User[]> { /* ... */ }
}

// Validator: só validação
export class EmailValidator {
  static isValid(email: string): boolean { /* ... */ }
}

// Pipe: só formatação
@Pipe({ name: 'appDate' })
export class DateFormatPipe {
  transform(date: Date): string { /* ... */ }
}

// Domain service: só lógica de negócio
export class AgeCalculator {
  calculate(birthDate: Date): number { /* ... */ }
}

// Component: só orquestração de UI
@Component({ /* ... */ })
export class UserComponent {
  users$ = this.userService.getUsers();
  
  constructor(private userService: UserHttpService) {}
}
```

**Por que isso importa na arquitetura:**
- Clean Architecture: cada **camada** tem responsabilidade única
- DDD: cada **bounded context** tem responsabilidade única
- Hexagonal: cada **port** tem responsabilidade única

### 3.2 O - Open/Closed Principle

**Aberto para extensão, fechado para modificação.**

```typescript
// ❌ VIOLAÇÃO OCP - Precisa modificar código existente
export class DiscountCalculator {
  calculate(order: Order): number {
    if (order.customerType === 'regular') {
      return order.total * 0.05;
    } else if (order.customerType === 'premium') {
      return order.total * 0.10;
    } else if (order.customerType === 'vip') {
      return order.total * 0.20;
    }
    // A cada novo tipo, precisa modificar AQUI
    return 0;
  }
}
```

```typescript
// ✅ OCP APLICADO - Strategy Pattern

// Interface (contrato)
export interface DiscountStrategy {
  calculate(order: Order): number;
}

// Implementações (extensões)
export class RegularDiscount implements DiscountStrategy {
  calculate(order: Order): number {
    return order.total * 0.05;
  }
}

export class PremiumDiscount implements DiscountStrategy {
  calculate(order: Order): number {
    return order.total * 0.10;
  }
}

export class VipDiscount implements DiscountStrategy {
  calculate(order: Order): number {
    return order.total * 0.20;
  }
}

// Context
export class DiscountCalculator {
  constructor(private strategy: DiscountStrategy) {}
  
  calculate(order: Order): number {
    return this.strategy.calculate(order);
  }
}

// Uso
const calculator = new DiscountCalculator(new PremiumDiscount());
const discount = calculator.calculate(order);

// Adicionar novo tipo? Só criar nova classe, não modifica nada existente!
export class GoldDiscount implements DiscountStrategy {
  calculate(order: Order): number {
    return order.total * 0.15;
  }
}
```

**Aplicação em arquiteturas:**
- Hexagonal: novos **adapters** sem modificar ports
- Clean: novos **use cases** sem modificar domain

### 3.3 L - Liskov Substitution Principle

**Subtipos devem ser substituíveis por seus tipos base.**

```typescript
// ❌ VIOLAÇÃO LSP
abstract class Bird {
  abstract fly(): void;
}

class Sparrow extends Bird {
  fly(): void {
    console.log('Flying!');
  }
}

class Penguin extends Bird {
  fly(): void {
    throw new Error('Penguins cannot fly!'); // Viola contrato!
  }
}

// Quebra quando substitui
function makeBirdFly(bird: Bird) {
  bird.fly(); // Pode lançar exception!
}
```

```typescript
// ✅ LSP APLICADO
abstract class Bird {
  abstract move(): void;
}

class Sparrow extends Bird {
  move(): void {
    this.fly();
  }
  
  private fly(): void {
    console.log('Flying!');
  }
}

class Penguin extends Bird {
  move(): void {
    this.swim();
  }
  
  private swim(): void {
    console.log('Swimming!');
  }
}

// Sempre funciona
function makeBirdMove(bird: Bird) {
  bird.move(); // Sempre válido!
}
```

**Aplicação no Angular:**

```typescript
// ✅ Repository abstrato
export abstract class UserRepository {
  abstract findById(id: string): Observable<User>;
}

// Implementações são substituíveis
export class UserHttpRepository extends UserRepository {
  findById(id: string): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
}

export class UserMockRepository extends UserRepository {
  findById(id: string): Observable<User> {
    return of({ id, name: 'Mock User', email: 'mock@test.com' });
  }
}

// Uso: QUALQUER implementação funciona
@Injectable()
export class GetUserUseCase {
  constructor(private repo: UserRepository) {}
  
  execute(id: string): Observable<User> {
    return this.repo.findById(id); // Sempre funciona!
  }
}
```

### 3.4 I - Interface Segregation Principle

**Muitas interfaces específicas > Uma interface genérica.**

```typescript
// ❌ VIOLAÇÃO ISP - Interface "gorda"
export interface UserRepository {
  // CRUD completo forçado
  findById(id: string): Observable<User>;
  findAll(): Observable<User[]>;
  save(user: User): Observable<User>;
  delete(id: string): Observable<void>;
  
  // Métodos específicos que nem todos usam
  findByEmail(email: string): Observable<User>;
  findByRole(role: string): Observable<User[]>;
  searchByName(name: string): Observable<User[]>;
  updatePassword(id: string, password: string): Observable<void>;
  sendWelcomeEmail(id: string): Observable<void>;
}

// Implementação forçada a ter tudo, mesmo que não use
export class UserReadOnlyRepository implements UserRepository {
  // Precisa implementar métodos que não usa!
  save(user: User): Observable<User> {
    throw new Error('Read-only repository!');
  }
  
  delete(id: string): Observable<void> {
    throw new Error('Read-only repository!');
  }
  
  // ... implementações vazias
}
```

```typescript
// ✅ ISP APLICADO - Interfaces segregadas

export interface UserReader {
  findById(id: string): Observable<User>;
  findAll(): Observable<User[]>;
}

export interface UserWriter {
  save(user: User): Observable<User>;
  delete(id: string): Observable<void>;
}

export interface UserSearcher {
  searchByName(name: string): Observable<User[]>;
  findByEmail(email: string): Observable<User>;
}

// Implementações específicas
export class UserReadRepository implements UserReader {
  findById(id: string): Observable<User> { /* ... */ }
  findAll(): Observable<User[]> { /* ... */ }
  // Só implementa o que precisa!
}

export class UserFullRepository implements UserReader, UserWriter, UserSearcher {
  // Implementa tudo porque precisa
}

// Use cases dependem só do que precisam
@Injectable()
export class GetUserUseCase {
  constructor(private repo: UserReader) {} // Só leitura!
  
  execute(id: string): Observable<User> {
    return this.repo.findById(id);
  }
}

@Injectable()
export class CreateUserUseCase {
  constructor(private repo: UserWriter) {} // Só escrita!
  
  execute(user: User): Observable<User> {
    return this.repo.save(user);
  }
}
```

### 3.5 D - Dependency Inversion Principle

**Dependa de abstrações, não de implementações concretas.**

```typescript
// ❌ VIOLAÇÃO DIP - Depende de implementação concreta
@Injectable()
export class OrderService {
  // Acoplado a EmailSmtpService específico
  constructor(private emailService: EmailSmtpService) {}
  
  createOrder(order: Order): Observable<Order> {
    return this.saveOrder(order).pipe(
      tap(() => this.emailService.sendOrderConfirmation(order))
    );
  }
}

// Problemas:
// - Não pode trocar implementação facilmente
// - Difícil testar (precisa mockar EmailSmtpService)
// - Acoplamento alto
```

```typescript
// ✅ DIP APLICADO - Depende de abstração

// Abstração (interface/abstract class)
export abstract class EmailService {
  abstract sendOrderConfirmation(order: Order): Observable<void>;
}

// Implementações concretas
export class EmailSmtpService extends EmailService {
  sendOrderConfirmation(order: Order): Observable<void> {
    return this.smtpClient.send(/* ... */);
  }
}

export class EmailMockService extends EmailService {
  sendOrderConfirmation(order: Order): Observable<void> {
    console.log('Mock: enviaria email para', order.customerEmail);
    return of(void 0);
  }
}

// Service depende de ABSTRAÇÃO
@Injectable()
export class OrderService {
  constructor(private emailService: EmailService) {} // Abstração!
  
  createOrder(order: Order): Observable<Order> {
    return this.saveOrder(order).pipe(
      tap(() => this.emailService.sendOrderConfirmation(order))
    );
  }
}

// DI injeta implementação
providers: [
  { provide: EmailService, useClass: EmailSmtpService }
  // Fácil trocar: useClass: EmailMockService
]
```

**Aplicação nas arquiteturas:**
- **Clean Architecture:** Domain depende de interfaces, Infrastructure implementa
- **Hexagonal:** Ports são abstrações, Adapters são implementações
- **DDD:** Domain services dependem de repository interfaces

---

<a name="parte2-1"></a>
# PARTE 2: ARQUITETURAS PRINCIPAIS

## 4. CLEAN ARCHITECTURE (ARQUITETURA LIMPA)

### 4.1 Conceito e História

**Criador:** Robert C. Martin (Uncle Bob), 2012  
**Livro:** "Clean Architecture: A Craftsman's Guide to Software Structure and Design"

**Princípio Central:** **Dependency Rule**

> Dependências de código devem apontar APENAS PARA DENTRO, em direção às políticas de alto nível.

```
┌─────────────────────────────────────────────────────────┐
│  FRAMEWORKS & DRIVERS (Camada 4 - Mais Externa)       │
│  Angular, HTTP, Database, UI, Devices                  │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │  INTERFACE ADAPTERS (Camada 3)                   │ │
│  │  Controllers, Gateways, Presenters               │ │
│  │                                                   │ │
│  │  ┌─────────────────────────────────────────────┐ │ │
│  │  │  APPLICATION BUSINESS RULES (Camada 2)     │ │ │
│  │  │  Use Cases, Interactors                    │ │ │
│  │  │                                             │ │ │
│  │  │  ┌───────────────────────────────────────┐ │ │ │
│  │  │  │ ENTERPRISE BUSINESS RULES (Camada 1) │ │ │ │
│  │  │  │ Entities, Domain Models               │ │ │ │
│  │  │  │           (CORE)                      │ │ │ │
│  │  │  └───────────────────────────────────────┘ │ │ │
│  │  │               ↑                             │ │ │
│  │  │               │  Dependências               │ │ │
│  │  │               │  apontam                    │ │ │
│  │  │               │  para dentro                │ │ │
│  │  └─────────────────────────────────────────────┘ │ │
│  │                                                   │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Características:**

1. **Independência de Frameworks:** Domain não conhece Angular, React, Vue
2. **Testabilidade:** Lógica de negócio testável sem UI, DB, frameworks
3. **Independência de UI:** UI é detail, pode trocar facilmente
4. **Independência de Database:** MySQL, MongoDB, PostgreSQL são details
5. **Independência de Agentes Externos:** Domain não conhece APIs externas

### 4.2 Camadas Detalhadas

#### **Camada 1: Enterprise Business Rules (Entities)**

**O que é:** Regras de negócio críticas da empresa, válidas em QUALQUER aplicação.

**Características:**
- Não muda se mudar UI ou DB
- Lógica pura, sem dependências externas
- Modela conceitos do domínio

**Exemplo: Sistema Bancário**

```typescript
// core/domain/entities/account.entity.ts

export enum AccountType {
  CHECKING = 'CHECKING',
  SAVINGS = 'SAVINGS',
  INVESTMENT = 'INVESTMENT'
}

export enum AccountStatus {
  ACTIVE = 'ACTIVE',
  BLOCKED = 'BLOCKED',
  CLOSED = 'CLOSED'
}

export class Account {
  constructor(
    public readonly id: AccountId,
    public readonly customerId: CustomerId,
    public readonly accountNumber: string,
    public readonly type: AccountType,
    private _balance: Money,
    private _status: AccountStatus,
    private _transactions: Transaction[] = []
  ) {}
  
  // ═══════════════════════════════════════════════════════════
  // REGRAS DE NEGÓCIO (INVARIANTS)
  // Essas regras SÃO a empresa, não podem ser violadas
  // ═══════════════════════════════════════════════════════════
  
  /**
   * Regra: Conta precisa de saldo mínimo para transferência
   */
  transfer(amount: Money, toAccount: Account): void {
    this.ensureIsActive();
    this.ensureHasSufficientFunds(amount);
    
    // Saque da conta origem
    this.withdraw(amount, `Transfer to ${toAccount.accountNumber}`);
    
    // Depósito na conta destino  
    toAccount.deposit(amount, `Transfer from ${this.accountNumber}`);
  }
  
  /**
   * Regra: Saque não pode exceder saldo disponível
   */
  withdraw(amount: Money, description: string): void {
    this.ensureIsActive();
    this.ensureHasSufficientFunds(amount);
    
    const transaction = new Transaction(
      TransactionId.generate(),
      TransactionType.WITHDRAWAL,
      amount,
      description,
      new Date()
    );
    
    this._balance = this._balance.subtract(amount);
    this._transactions.push(transaction);
  }
  
  /**
   * Regra: Depósito não pode ser negativo
   */
  deposit(amount: Money, description: string): void {
    this.ensureIsActive();
    
    if (amount.amount <= 0) {
      throw new InvalidAmountError('Deposit amount must be positive');
    }
    
    const transaction = new Transaction(
      TransactionId.generate(),
      TransactionType.DEPOSIT,
      amount,
      description,
      new Date()
    );
    
    this._balance = this._balance.add(amount);
    this._transactions.push(transaction);
  }
  
  /**
   * Regra: Conta poupança tem rendimento automático
   */
  applyInterest(rate: number): void {
    if (this.type !== AccountType.SAVINGS) {
      throw new InvalidOperationError('Only savings accounts earn interest');
    }
    
    const interest = this._balance.multiply(rate);
    this.deposit(interest, `Interest ${rate * 100}%`);
  }
  
  /**
   * Regra: Conta bloqueada não permite operações
   */
  block(reason: string): void {
    if (this._status === AccountStatus.CLOSED) {
      throw new InvalidOperationError('Cannot block closed account');
    }
    
    this._status = AccountStatus.BLOCKED;
    // Poderia emitir domain event aqui
  }
  
  close(): void {
    if (this._balance.amount !== 0) {
      throw new InvalidOperationError('Cannot close account with balance');
    }
    
    this._status = AccountStatus.CLOSED;
  }
  
  // ═══════════════════════════════════════════════════════════
  // GETTERS (encapsulamento)
  // ═══════════════════════════════════════════════════════════
  
  get balance(): Money {
    return this._balance; // Retorna cópia, não referência mutável
  }
  
  get status(): AccountStatus {
    return this._status;
  }
  
  get transactions(): ReadonlyArray<Transaction> {
    return this._transactions; // ReadonlyArray previne mutação externa
  }
  
  isActive(): boolean {
    return this._status === AccountStatus.ACTIVE;
  }
  
  // ═══════════════════════════════════════════════════════════
  // VALIDAÇÕES PRIVADAS (helpers)
  // ═══════════════════════════════════════════════════════════
  
  private ensureIsActive(): void {
    if (!this.isActive()) {
      throw new AccountNotActiveError(this.id.value, this._status);
    }
  }
  
  private ensureHasSufficientFunds(amount: Money): void {
    if (this._balance.amount < amount.amount) {
      throw new InsufficientFundsError(
        this.id.value,
        this._balance.amount,
        amount.amount
      );
    }
  }
}

// Value Objects relacionados
export class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string = 'BRL'
  ) {
    if (amount < 0) {
      throw new InvalidAmountError('Amount cannot be negative');
    }
  }
  
  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }
  
  subtract(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount - other.amount, this.currency);
  }
  
  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }
  
  private ensureSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new CurrencyMismatchError(this.currency, other.currency);
    }
  }
  
  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
}

// Domain Errors
export class DomainError extends Error {
  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class InsufficientFundsError extends DomainError {
  constructor(
    public readonly accountId: string,
    public readonly currentBalance: number,
    public readonly requestedAmount: number
  ) {
    super(
      `Insufficient funds in account ${accountId}. ` +
      `Balance: ${currentBalance}, Requested: ${requestedAmount}`
    );
  }
}

export class AccountNotActiveError extends DomainError {
  constructor(
    public readonly accountId: string,
    public readonly status: AccountStatus
  ) {
    super(`Account ${accountId} is ${status}, not ACTIVE`);
  }
}
```

**Por que esta camada é especial:**
- ✅ **Zero dependências:** Não importa Angular, RxJS, nada
- ✅ **Testável 100%:** Testes unitários puros, sem mocks
- ✅ **Reutilizável:** Pode usar em Node.js, React, Vue
- ✅ **Duradoura:** Não muda quando framework muda

#### **Camada 2: Application Business Rules (Use Cases)**

**O que é:** Orquestração de lógica de negócio específica da aplicação.

**Características:**
- Implementa casos de uso específicos do sistema
- Orquestra entities e services
- Depende de interfaces (repositories), não implementações
- Independente de frameworks

**Exemplo: Transfer Money Use Case**

```typescript
// application/use-cases/transfer-money.usecase.ts

import { Injectable } from '@angular/core';
import { Observable, forkJoin } from 'rxjs';
import { map, switchMap } from 'rxjs/operators';
import { AccountRepository } from '@core/domain/repositories/account.repository';
import { AccountId } from '@core/domain/value-objects/account-id.vo';
import { Money } from '@core/domain/value-objects/money.vo';
import { TransferMoneyCommand, TransferMoneyResult } from './transfer-money.dto';

/**
 * Use Case: Transferir dinheiro entre contas
 * 
 * Fluxo:
 * 1. Busca conta origem e destino
 * 2. Valida se ambas existem e estão ativas
 * 3. Executa transferência (lógica na entity)
 * 4. Persiste ambas as contas
 * 5. Retorna resultado
 */
@Injectable({
  providedIn: 'root'
})
export class TransferMoneyUseCase {
  constructor(
    private accountRepository: AccountRepository
    // Depende de INTERFACE, não implementação
  ) {}
  
  execute(command: TransferMoneyCommand): Observable<TransferMoneyResult> {
    const fromAccountId = AccountId.create(command.fromAccountId);
    const toAccountId = AccountId.create(command.toAccountId);
    const amount = new Money(command.amount, command.currency);
    
    // 1. Busca ambas as contas em paralelo
    return forkJoin({
      fromAccount: this.accountRepository.findById(fromAccountId),
      toAccount: this.accountRepository.findById(toAccountId)
    }).pipe(
      // 2. Valida existência
      map(({ fromAccount, toAccount }) => {
        if (!fromAccount) {
          throw new AccountNotFoundError(fromAccountId.value);
        }
        if (!toAccount) {
          throw new AccountNotFoundError(toAccountId.value);
        }
        return { fromAccount, toAccount };
      }),
      
      // 3. Executa transferência (lógica na entity!)
      map(({ fromAccount, toAccount }) => {
        fromAccount.transfer(amount, toAccount);
        // Validações acontecem DENTRO da entity
        // Se der erro, exception é lançada
        return { fromAccount, toAccount };
      }),
      
      // 4. Persiste ambas as contas
      switchMap(({ fromAccount, toAccount }) => 
        forkJoin({
          savedFrom: this.accountRepository.save(fromAccount),
          savedTo: this.accountRepository.save(toAccount)
        })
      ),
      
      // 5. Retorna resultado
      map(({ savedFrom, savedTo }) => ({
        success: true,
        fromAccountBalance: savedFrom.balance.amount,
        toAccountBalance: savedTo.balance.amount,
        transactionDate: new Date()
      }))
    );
  }
}

// DTOs (Data Transfer Objects)
export interface TransferMoneyCommand {
  fromAccountId: string;
  toAccountId: string;
  amount: number;
  currency: string;
}

export interface TransferMoneyResult {
  success: boolean;
  fromAccountBalance: number;
  toAccountBalance: number;
  transactionDate: Date;
}
```

**Outro Use Case: Create Account**

```typescript
// application/use-cases/create-account.usecase.ts

@Injectable({
  providedIn: 'root'
})
export class CreateAccountUseCase {
  constructor(
    private accountRepository: AccountRepository,
    private customerRepository: CustomerRepository,
    private accountNumberGenerator: AccountNumberGenerator
  ) {}
  
  execute(command: CreateAccountCommand): Observable<CreateAccountResult> {
    const customerId = CustomerId.create(command.customerId);
    
    return this.customerRepository.findById(customerId).pipe(
      // Valida que customer existe
      map(customer => {
        if (!customer) {
          throw new CustomerNotFoundError(customerId.value);
        }
        return customer;
      }),
      
      // Gera número de conta único
      switchMap(customer => 
        this.accountNumberGenerator.generate().pipe(
          map(accountNumber => ({ customer, accountNumber }))
        )
      ),
      
      // Cria entity Account
      map(({ customer, accountNumber }) => {
        const account = new Account(
          AccountId.generate(),
          customer.id,
          accountNumber,
          command.type,
          Money.zero(),
          AccountStatus.ACTIVE
        );
        return account;
      }),
      
      // Persiste
      switchMap(account => this.accountRepository.save(account)),
      
      // Retorna DTO
      map(account => ({
        accountId: account.id.value,
        accountNumber: account.accountNumber,
        type: account.type,
        initialBalance: account.balance.amount
      }))
    );
  }
}
```

**Características dos Use Cases:**
- ✅ **Uma responsabilidade:** Cada use case faz UMA coisa
- ✅ **Orquestração:** Coordena entities, repositories, domain services
- ✅ **Independente de framework:** Usa RxJS, mas não Angular específico
- ✅ **Testável:** Mocka repositories, testa lógica isolada

#### **Camada 3: Interface Adapters**

**O que é:** Converte dados entre formato do domain e formato externo (API, UI).

**Componentes:**
- **Controllers:** Recebem requests (no Angular, são os components)
- **Presenters:** Formatam dados para UI
- **Gateways:** Implementam repositories (HTTP, LocalStorage)
- **Mappers:** Convertem DTOs ↔ Entities

**Exemplo: Repository Implementation**

```typescript
// infrastructure/repositories/account-http.repository.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { AccountRepository } from '@core/domain/repositories/account.repository';
import { Account, AccountType, AccountStatus } from '@core/domain/entities/account.entity';
import { AccountId } from '@core/domain/value-objects/account-id.vo';
import { Money } from '@core/domain/value-objects/money.vo';
import { environment } from '@environments/environment';

// DTO - estrutura que vem do backend
interface AccountDTO {
  id: string;
  customer_id: string;
  account_number: string;
  type: 'CHECKING' | 'SAVINGS' | 'INVESTMENT';
  balance: number;
  currency: string;
  status: 'ACTIVE' | 'BLOCKED' | 'CLOSED';
  transactions: TransactionDTO[];
}

interface TransactionDTO {
  id: string;
  type: 'DEPOSIT' | 'WITHDRAWAL';
  amount: number;
  description: string;
  created_at: string;
}

@Injectable({
  providedIn: 'root'
})
export class AccountHttpRepository implements AccountRepository {
  private apiUrl = `${environment.apiUrl}/accounts`;
  
  constructor(private http: HttpClient) {}
  
  findById(id: AccountId): Observable<Account> {
    return this.http.get<AccountDTO>(`${this.apiUrl}/${id.value}`).pipe(
      map(dto => this.mapDtoToEntity(dto))
    );
  }
  
  findByCustomerId(customerId: CustomerId): Observable<Account[]> {
    return this.http.get<AccountDTO[]>(`${this.apiUrl}?customer=${customerId.value}`).pipe(
      map(dtos => dtos.map(dto => this.mapDtoToEntity(dto)))
    );
  }
  
  save(account: Account): Observable<Account> {
    const dto = this.mapEntityToDto(account);
    
    if (account.id.value) {
      // Update
      return this.http.put<AccountDTO>(`${this.apiUrl}/${account.id.value}`, dto).pipe(
        map(dto => this.mapDtoToEntity(dto))
      );
    } else {
      // Create
      return this.http.post<AccountDTO>(this.apiUrl, dto).pipe(
        map(dto => this.mapDtoToEntity(dto))
      );
    }
  }
  
  delete(id: AccountId): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id.value}`);
  }
  
  // ═══════════════════════════════════════════════════════════
  // MAPPERS - Convertem DTO ↔ Entity
  // ═══════════════════════════════════════════════════════════
  
  /**
   * Converte DTO do backend para Entity de domínio
   */
  private mapDtoToEntity(dto: AccountDTO): Account {
    const transactions = dto.transactions.map(t => new Transaction(
      TransactionId.create(t.id),
      t.type as TransactionType,
      new Money(t.amount, dto.currency),
      t.description,
      new Date(t.created_at)
    ));
    
    return new Account(
      AccountId.create(dto.id),
      CustomerId.create(dto.customer_id),
      dto.account_number,
      dto.type as AccountType,
      new Money(dto.balance, dto.currency),
      dto.status as AccountStatus,
      transactions
    );
  }
  
  /**
   * Converte Entity de domínio para DTO do backend
   */
  private mapEntityToDto(account: Account): AccountDTO {
    return {
      id: account.id.value,
      customer_id: account.customerId.value,
      account_number: account.accountNumber,
      type: account.type,
      balance: account.balance.amount,
      currency: account.balance.currency,
      status: account.status,
      transactions: account.transactions.map(t => ({
        id: t.id.value,
        type: t.type,
        amount: t.amount.amount,
        description: t.description,
        created_at: t.createdAt.toISOString()
      }))
    };
  }
}
```

**Por que Mappers são essenciais:**
- ✅ **Desacoplamento:** Domain não conhece formato do backend
- ✅ **Flexibilidade:** Backend muda? Só atualiza mapper
- ✅ **Type Safety:** Conversão explícita e tipada

#### **Camada 4: Frameworks & Drivers (UI)**

**O que é:** Angular components, routing, templates.

**Exemplo: Component (Presentation)**

```typescript
// presentation/pages/accounts/transfer/transfer.component.ts

@Component({
  selector: 'app-transfer',
  template: `
    <div class="transfer-page">
      <h2>Transferência entre Contas</h2>
      
      <form [formGroup]="form" (ngSubmit)="onSubmit()">
        <div class="form-group">
          <label>Conta Origem</label>
          <select formControlName="fromAccountId">
            <option *ngFor="let account of accounts$ | async" [value]="account.id">
              {{ account.accountNumber }} - {{ account.balance | currency }}
            </option>
          </select>
        </div>
        
        <div class="form-group">
          <label>Conta Destino</label>
          <input formControlName="toAccountId" placeholder="Número da conta">
        </div>
        
        <div class="form-group">
          <label>Valor</label>
          <input formControlName="amount" type="number" step="0.01">
        </div>
        
        <button 
          type="submit" 
          [disabled]="form.invalid || loading">
          {{ loading ? 'Processando...' : 'Transferir' }}
        </button>
      </form>
      
      <div *ngIf="result" class="result success">
        <h3>✓ Transferência realizada!</h3>
        <p>Novo saldo: {{ result.fromAccountBalance | currency }}</p>
      </div>
      
      <div *ngIf="error" class="result error">
        <h3>✗ Erro na transferência</h3>
        <p>{{ error }}</p>
      </div>
    </div>
  `
})
export class TransferComponent implements OnInit {
  form = this.fb.group({
    fromAccountId: ['', Validators.required],
    toAccountId: ['', Validators.required],
    amount: [0, [Validators.required, Validators.min(0.01)]]
  });
  
  accounts$!: Observable<Account[]>;
  loading = false;
  result?: TransferMoneyResult;
  error?: string;
  
  constructor(
    private fb: FormBuilder,
    private transferMoney: TransferMoneyUseCase,
    private getAccounts: GetCustomerAccountsUseCase,
    private customerId: string // Injetado de algum lugar
  ) {}
  
  ngOnInit() {
    this.accounts$ = this.getAccounts.execute(this.customerId);
  }
  
  onSubmit() {
    if (this.form.invalid) return;
    
    this.loading = true;
    this.result = undefined;
    this.error = undefined;
    
    const command: TransferMoneyCommand = {
      fromAccountId: this.form.value.fromAccountId!,
      toAccountId: this.form.value.toAccountId!,
      amount: this.form.value.amount!,
      currency: 'BRL'
    };
    
    this.transferMoney.execute(command).subscribe({
      next: (result) => {
        this.result = result;
        this.loading = false;
        this.form.reset();
      },
      error: (err) => {
        if (err instanceof InsufficientFundsError) {
          this.error = `Saldo insuficiente. Saldo atual: R$ ${err.currentBalance}`;
        } else if (err instanceof AccountNotActiveError) {
          this.error = `Conta está ${err.status}. Apenas contas ativas podem transferir.`;
        } else {
          this.error = 'Erro inesperado. Tente novamente.';
        }
        this.loading = false;
      }
    });
  }
}
```

**Component é apenas apresentação:**
- ✅ **Delega lógica:** Use case faz o trabalho
- ✅ **Trata erros:** Apresenta mensagens user-friendly
- ✅ **Sem lógica de negócio:** Só orquestra UI

### 4.3 Dependency Injection - Conectando as Camadas

```typescript
// app.config.ts (Angular 15+ standalone)

import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

// Domain interfaces
import { AccountRepository } from '@core/domain/repositories/account.repository';
import { CustomerRepository } from '@core/domain/repositories/customer.repository';
import { AccountNumberGenerator } from '@core/domain/services/account-number-generator.service';

// Infrastructure implementations
import { AccountHttpRepository } from '@infrastructure/repositories/account-http.repository';
import { CustomerHttpRepository } from '@infrastructure/repositories/customer-http.repository';
import { AccountNumberGeneratorImpl } from '@infrastructure/services/account-number-generator.impl';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    
    // Injeta implementações quando alguém pede interfaces
    { provide: AccountRepository, useClass: AccountHttpRepository },
    { provide: CustomerRepository, useClass: CustomerHttpRepository },
    { provide: AccountNumberGenerator, useClass: AccountNumberGeneratorImpl },
    
    // Use Cases são auto-injetados (providedIn: 'root')
  ]
};
```

**Para trocar implementação (ex: testes):**

```typescript
// app.config.test.ts

export const testConfig: ApplicationConfig = {
  providers: [
    // Usa mocks em vez de HTTP
    { provide: AccountRepository, useClass: AccountMockRepository },
    { provide: CustomerRepository, useClass: CustomerMockRepository },
    { provide: AccountNumberGenerator, useClass: AccountNumberGeneratorMock },
  ]
};
```

### 4.4 Testando Clean Architecture

#### **Teste de Entity (Domain - Mais Fácil)**

```typescript
// account.entity.spec.ts

describe('Account Entity', () => {
  let account: Account;
  
  beforeEach(() => {
    account = new Account(
      AccountId.generate(),
      CustomerId.generate(),
      '12345-6',
      AccountType.CHECKING,
      new Money(1000, 'BRL'),
      AccountStatus.ACTIVE
    );
  });
  
  it('should withdraw money successfully', () => {
    account.withdraw(new Money(500, 'BRL'), 'ATM withdrawal');
    
    expect(account.balance.amount).toBe(500);
    expect(account.transactions.length).toBe(1);
  });
  
  it('should throw error when withdrawing more than balance', () => {
    expect(() => {
      account.withdraw(new Money(1500, 'BRL'), 'ATM withdrawal');
    }).toThrow(InsufficientFundsError);
  });
  
  it('should not allow operations on blocked account', () => {
    account.block('Suspicious activity');
    
    expect(() => {
      account.withdraw(new Money(100, 'BRL'), 'ATM');
    }).toThrow(AccountNotActiveError);
  });
});
```

**Por que é fácil:**
- ✅ Sem dependências externas
- ✅ Sem mocks necessários
- ✅ Testa lógica pura

#### **Teste de Use Case**

```typescript
// transfer-money.usecase.spec.ts

describe('TransferMoneyUseCase', () => {
  let useCase: TransferMoneyUseCase;
  let mockRepo: jasmine.SpyObj<AccountRepository>;
  
  beforeEach(() => {
    mockRepo = jasmine.createSpyObj('AccountRepository', ['findById', 'save']);
    useCase = new TransferMoneyUseCase(mockRepo);
  });
  
  it('should transfer money between accounts', (done) => {
    const fromAccount = new Account(/* ... */);
    const toAccount = new Account(/* ... */);
    
    mockRepo.findById.and.returnValues(of(fromAccount), of(toAccount));
    mockRepo.save.and.returnValue(of(fromAccount));
    
    const command: TransferMoneyCommand = {
      fromAccountId: fromAccount.id.value,
      toAccountId: toAccount.id.value,
      amount: 500,
      currency: 'BRL'
    };
    
    useCase.execute(command).subscribe(result => {
      expect(result.success).toBe(true);
      expect(mockRepo.save).toHaveBeenCalledTimes(2);
      done();
    });
  });
  
  it('should throw error when insufficient funds', (done) => {
    const fromAccount = new Account(/* saldo: 100 */);
    const toAccount = new Account(/* ... */);
    
    mockRepo.findById.and.returnValues(of(fromAccount), of(toAccount));
    
    const command: TransferMoneyCommand = {
      fromAccountId: fromAccount.id.value,
      toAccountId: toAccount.id.value,
      amount: 500, // Mais que o saldo!
      currency: 'BRL'
    };
    
    useCase.execute(command).subscribe(
      () => fail('Should have thrown error'),
      err => {
        expect(err).toBeInstanceOf(InsufficientFundsError);
        done();
      }
    );
  });
});
```

### 4.5 Estrutura de Pastas Completa

```
src/
├── app/
│   ├── core/                           # Enterprise Business Rules
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   ├── account.entity.ts
│   │   │   │   ├── customer.entity.ts
│   │   │   │   └── transaction.entity.ts
│   │   │   ├── value-objects/
│   │   │   │   ├── account-id.vo.ts
│   │   │   │   ├── money.vo.ts
│   │   │   │   └── email.vo.ts
│   │   │   ├── repositories/          # Interfaces
│   │   │   │   ├── account.repository.ts
│   │   │   │   └── customer.repository.ts
│   │   │   ├── services/              # Domain Services
│   │   │   │   └── account-number-generator.service.ts
│   │   │   └── errors/
│   │   │       ├── domain.error.ts
│   │   │       ├── insufficient-funds.error.ts
│   │   │       └── account-not-active.error.ts
│   │   │
│   │   └── application/               # Application Business Rules
│   │       └── use-cases/
│   │           ├── transfer-money.usecase.ts
│   │           ├── create-account.usecase.ts
│   │           ├── get-account.usecase.ts
│   │           └── close-account.usecase.ts
│   │
│   ├── infrastructure/                # Interface Adapters
│   │   ├── repositories/              # Implementations
│   │   │   ├── account-http.repository.ts
│   │   │   ├── account-localstorage.repository.ts
│   │   │   └── customer-http.repository.ts
│   │   ├── services/
│   │   │   └── account-number-generator.impl.ts
│   │   ├── http/
│   │   │   ├── interceptors/
│   │   │   │   ├── auth.interceptor.ts
│   │   │   │   └── error.interceptor.ts
│   │   │   └── api.service.ts
│   │   └── mappers/
│   │       ├── account.mapper.ts
│   │       └── customer.mapper.ts
│   │
│   └── presentation/                  # Frameworks & Drivers
│       ├── pages/
│       │   ├── accounts/
│       │   │   ├── account-list/
│       │   │   ├── account-detail/
│       │   │   ├── transfer/
│       │   │   └── create-account/
│       │   └── customers/
│       │       └── customer-detail/
│       ├── components/                # Dumb components
│       │   ├── account-card/
│       │   ├── transaction-list/
│       │   └── balance-display/
│       └── shared/
│           ├── pipes/
│           ├── directives/
│           └── validators/
│
├── environments/
└── assets/
```

### 4.6 Quando Usar Clean Architecture

✅ **Use quando:**

| Critério | Descrição |
|----------|-----------|
| **Domínio Complexo** | Muitas regras de negócio, validações, invariants |
| **Longo Prazo** | App vai durar 3+ anos |
| **Múltiplos Clientes** | Web + Mobile + Desktop compartilham domínio |
| **Time Grande** | 10+ desenvolvedores |
| **Criticidade Alta** | Banco, saúde, seguros, financeiro |
| **Testabilidade** | Cobertura de testes >80% necessária |
| **Independência** | Quer trocar framework futuramente |

❌ **NÃO use quando:**

| Critério | Descrição |
|----------|-----------|
| **CRUD Simples** | Só formulários básicos, sem lógica |
| **MVP/Protótipo** | App descartável, validação de ideia |
| **Time Pequeno** | 1-2 devs, overhead não compensa |
| **Deadline Apertado** | Precisa entregar em 2 semanas |
| **Sem Complexidade** | Todo-list, blog pessoal |

### 4.7 Clean Architecture - Resumo Visual

```
┌────────────────────────────────────────────────────┐
│                   BENEFÍCIOS                        │
├────────────────────────────────────────────────────┤
│ ✅ Testabilidade máxima                            │
│ ✅ Independência de framework                      │
│ ✅ Fácil trocar infraestrutura (DB, API)          │
│ ✅ Regras de negócio protegidas                   │
│ ✅ Escalável para times grandes                   │
│ ✅ Manutenível a longo prazo                      │
└────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│                   CUSTOS                            │
├────────────────────────────────────────────────────┤
│ ❌ Mais arquivos/pastas (complexidade inicial)     │
│ ❌ Curva de aprendizado                            │
│ ❌ Overhead para apps simples                      │
│ ❌ Mais código boilerplate                         │
│ ❌ Time precisa conhecer arquitetura               │
└────────────────────────────────────────────────────┘

                PONTO DE EQUILÍBRIO
                       ↓
    Pequeno  ←──────────────────→  Grande
    Simples  ←──────────────────→  Complexo
    Curto prazo ←────────────────→ Longo prazo
    
         NÃO VALE          VALE MUITO
```

---

_(Continua com DDD, Hexagonal, CQRS, Event-Driven, Micro Frontends, Comparações, Migração, etc...)_

**Documento está com ~25% completo. Quer que eu:**
1. Continue expandindo as outras arquiteturas?
2. Foque em alguma arquitetura específica?
3. Vá direto pra parte de comparação e quando usar?
4. Crie exemplos mais práticos de migração?

Qual você prefere?
