# 🅰️ GUIA COMPLETO ANGULAR - PREPARAÇÃO SENIOR 2025

**Versão:** Angular 18+ (compatível com 14-18)  
**Objetivo:** Preparação completa para entrevistas técnicas de nível Senior  
**Foco:** Conceitos essenciais + Teoria + Boas práticas + Comparação de versões

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

### 1.0 O que é um Component?

Um **Component** é o bloco fundamental de construção de aplicações Angular. Ele combina três responsabilidades: **template** (o que é renderizado), **classe** (lógica e estado) e **estilos** (aparência).

O Angular segue a arquitetura **MVC (Model-View-Controller)** adaptada como **MVVM (Model-View-ViewModel)**:
- **Model:** os dados (interfaces, classes de domínio)
- **View:** o template HTML
- **ViewModel:** a classe do component (a "cola" entre os dois)

Cada component cria seu próprio **escopo de encapsulamento** — por padrão, o Angular usa **View Encapsulation** para isolar os estilos, de forma que o CSS de um component não vaze para outros.

**Modos de View Encapsulation:**

| Modo | Comportamento |
|------|--------------|
| `Emulated` (padrão) | Estilos isolados via atributos gerados automaticamente |
| `ShadowDom` | Usa Shadow DOM nativo do browser |
| `None` | Sem isolamento — estilos viram globais |

```typescript
@Component({
  encapsulation: ViewEncapsulation.None  // use com cuidado
})
```

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

O **Data Binding** é o mecanismo que sincroniza dados entre a classe do component e o template. É a base da reatividade no Angular.

| Tipo | Sintaxe | Direção | Exemplo | Quando Usar |
|------|---------|---------|---------|-------------|
| **Interpolação** | `{{ }}` | Component → View | `{{ name }}` | Exibir valores |
| **Property Binding** | `[property]` | Component → View | `[disabled]="isLoading"` | Bind de propriedades |
| **Event Binding** | `(event)` | View → Component | `(click)="save()"` | Escutar eventos |
| **Two-Way Binding** | `[(ngModel)]` | Ambas direções | `[(ngModel)]="name"` | Forms simples |

> **Conceito importante:** `[(ngModel)]` é açúcar sintático para `[ngModel]="name" (ngModelChange)="name=$event"`. O Angular chama isso de **banana-in-a-box** pela forma dos colchetes e parênteses.

### 1.3 Property Binding - Exemplos

```html
<!-- Propriedades HTML padrão -->
<input [value]="username">
<img [src]="imageUrl" [alt]="imageAlt">
<button [disabled]="isLoading">Salvar</button>

<!-- Atributos (quando não há property correspondente) -->
<!-- Use [attr.x] quando o HTML não tem uma DOM property equivalente -->
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

> **Atenção: diferença entre Property e Attribute**  
> - **Property** é uma propriedade do objeto DOM (JavaScript): `element.disabled`  
> - **Attribute** é o atributo do HTML: `<div aria-label="...">`  
> O Angular faz binding de **properties** por padrão. Use `[attr.x]` quando a property DOM não existe.

### 1.4 Diretivas Estruturais

**Diretivas estruturais** modificam a estrutura do DOM — adicionam, removem ou reorganizam elementos. São reconhecidas pelo prefixo `*`.

O `*` é açúcar sintático para `<ng-template>`. Por exemplo:
```html
<!-- Sintaxe curta -->
<div *ngIf="show">Conteúdo</div>

<!-- Equivalente expandido -->
<ng-template [ngIf]="show">
  <div>Conteúdo</div>
</ng-template>
```

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

> **Como o Angular identifica mudanças sem trackBy?**  
> O Angular usa **identidade de referência** dos objetos JavaScript. Se você receber um novo array da API (mesmo com os mesmos dados), todos os objetos são "novos" do ponto de vista do Angular — então ele destrói e recria todos os elementos DOM. Com `trackBy`, você diz ao Angular "use o ID como identidade estável", e ele consegue reaproveitar elementos DOM existentes.

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

> **Por que isso importa?**  
> Em tabelas, por exemplo, adicionar um `<div>` dentro de `<tr>` ou `<tbody>` quebra o HTML semântico. O `<ng-container>` é uma construção puramente Angular — não gera nenhum elemento no DOM.

---

<a name="2-comunicacao"></a>
## 2. COMUNICAÇÃO ENTRE COMPONENTS

### 2.0 Visão Geral das Estratégias

No Angular existem várias formas de components se comunicarem. A escolha depende do **relacionamento** entre eles:

| Relação | Estratégia recomendada |
|---------|----------------------|
| Pai → Filho | `@Input()` |
| Filho → Pai | `@Output() + EventEmitter` |
| Irmãos / distantes | Service compartilhado (com Subject/Signal) |
| Qualquer → Qualquer (global) | Store (NgRx, Signal Store) |

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

> **Como o Angular detecta mudança em @Input?**  
> Por padrão, o Angular compara por **referência** (`===`), não por valor profundo. Isso é fundamental para entender o OnPush. Se você passar um objeto e mutar uma propriedade interna sem trocar a referência, o Angular (com OnPush) não detecta a mudança.

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

> **EventEmitter vs Subject:**  
> O `EventEmitter` é internamente um `Subject` do RxJS, mas é idiomaticamente destinado ao `@Output`. Para comunicação entre services, prefira `Subject` ou `BehaviorSubject` diretamente.

### 2.3 Comunicação via Service (Irmãos/Distantes)

Quando components não têm relação pai-filho direta, use um **Service como mediador**:

```typescript
// shared-state.service.ts
@Injectable({ providedIn: 'root' })
export class SharedStateService {
  private selectedUser$ = new BehaviorSubject<User | null>(null);
  
  // Expõe como Observable (readonly para os consumers)
  selectedUser = this.selectedUser$.asObservable();
  
  selectUser(user: User) {
    this.selectedUser$.next(user);
  }
  
  clearUser() {
    this.selectedUser$.next(null);
  }
}
```

```typescript
// Component A (emite)
export class UserListComponent {
  constructor(private state: SharedStateService) {}
  
  onSelect(user: User) {
    this.state.selectUser(user);
  }
}

// Component B (recebe)
export class UserDetailComponent implements OnInit {
  user$ = this.state.selectedUser;
  
  constructor(private state: SharedStateService) {}
}
```

> **BehaviorSubject vs Subject:**
> - `Subject`: emite apenas para subscribers **ativos no momento** da emissão. Novos subscribers perdem eventos anteriores.
> - `BehaviorSubject`: guarda o **último valor emitido** e o entrega imediatamente a qualquer novo subscriber. Ideal para estado.
> - `ReplaySubject(n)`: guarda os últimos `n` valores e os replaya para novos subscribers.

### 2.4 Pattern: Smart vs Dumb Components

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

> **Por que esse padrão é valioso?**  
> - **Reutilização:** components Dumb podem ser usados em qualquer contexto, pois não dependem de services específicos.  
> - **Testabilidade:** testar um Dumb component é trivial — você só passa @Inputs e verifica @Outputs.  
> - **Manutenibilidade:** a lógica de negócio fica concentrada nos Smart components e services, não espalhada pelo template.

---

<a name="3-lifecycle"></a>
## 3. LIFECYCLE HOOKS

### 3.0 O que é o Lifecycle?

Todo component Angular passa por um **ciclo de vida** gerenciado pelo framework: é criado, renderizado, atualizado quando os dados mudam e, por fim, destruído. O Angular oferece **hooks** — métodos que você implementa para executar código em momentos específicos desse ciclo.

Para usar um hook, basta implementar a interface correspondente e definir o método:

```typescript
export class MyComponent implements OnInit, OnDestroy {
  ngOnInit() { ... }
  ngOnDestroy() { ... }
}
```

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

> **Por que não fazer HTTP calls no constructor?**  
> No constructor, o Angular ainda está injetando dependências e o component não está completamente inicializado. Os `@Input`s ainda não foram populados. O `ngOnInit` é o momento certo porque: (1) todos os `@Input`s já foram recebidos, (2) o component foi completamente inicializado, (3) é mais testável (você pode criar a instância sem disparar efeitos colaterais).

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
    // SimpleChanges: objeto com a mudança de cada @Input
    if (changes['userId']) {
      const previousId = changes['userId'].previousValue;
      const currentId = changes['userId'].currentValue;
      const isFirstChange = changes['userId'].firstChange;
      console.log(`User ID mudou: ${previousId} → ${currentId}`);
      if (!isFirstChange) {
        // Evita lógica desnecessária na primeira renderização
        this.loadUser(currentId);
      }
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
  
  private loadInitialData() {
    this.loadUser(this.userId);
  }
}
```

### 3.3 ngAfterViewInit — Quando Usar

O `ngAfterViewInit` é o único lugar seguro para acessar elementos do DOM ou filhos do component via `@ViewChild`:

```typescript
export class FocusComponent implements AfterViewInit {
  @ViewChild('input') input!: ElementRef;
  
  ngAfterViewInit() {
    // ✅ Aqui o elemento já existe no DOM
    this.input.nativeElement.focus();
  }
  
  ngOnInit() {
    // ❌ ERRO: input ainda não foi renderizado
    // this.input.nativeElement.focus();
  }
}
```

### 3.4 Armadilhas Comuns

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

❌ **Erro 3: Modificar o DOM em ngAfterViewChecked**
```typescript
ngAfterViewChecked() {
  // Cuidado: dispara MUITAS vezes.
  // Modificar estado aqui causa loop de detecção!
  this.title = 'novo'; // Pode causar ExpressionChangedAfterItHasBeenCheckedError
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

### 4.0 O que é Dependency Injection?

**Dependency Injection (DI)** é um padrão de design onde uma classe recebe suas dependências externamente em vez de criá-las. O Angular tem um sistema de DI embutido baseado em um **Injector** hierárquico.

**Sem DI (acoplado):**
```typescript
export class UserComponent {
  private userService = new UserService(); // Cria direto — impossível testar/trocar
}
```

**Com DI (desacoplado):**
```typescript
export class UserComponent {
  constructor(private userService: UserService) {} // Angular injeta automaticamente
}
```

**Vantagens:**
- ✅ Testabilidade: você pode injetar mocks nos testes
- ✅ Reutilização: o mesmo serviço é compartilhado (singleton)
- ✅ Manutenibilidade: troca de implementação sem mudar consumers
- ✅ Separação de responsabilidades

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

O sistema de DI do Angular é **hierárquico**: cada nível (root, module, component) tem seu próprio Injector. Quando um component pede uma dependência, o Angular sobe na hierarquia de injectors até encontrar um provider.

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

// Component (raro) — cada instância do component tem sua PRÓPRIA instância do service
@Component({
  providers: [LocalService]
})
export class MyComponent {}
```

> **Tree-shakeable providers:**  
> Quando você usa `providedIn: 'root'`, o Angular só inclui o service no bundle final se ele for realmente usado em algum lugar. Se você registrar no `providers: []` do NgModule, ele sempre será incluído, mesmo que ninguém use.

### 4.3 Injeção com `inject()` (Angular 14+)

A função `inject()` permite injetar dependências fora do constructor — útil em funções standalone, guards funcionais e factories:

```typescript
// Em vez de constructor injection:
export class UserService {
  private http = inject(HttpClient);         // ✅ Mais conciso
  private auth = inject(AuthService);
  
  // Funciona também em contexts sem constructor (guards, interceptors funcionais)
}

// Guard funcional (Angular 15+)
export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);
  return authService.isLoggedIn() || router.createUrlTree(['/login']);
};
```

### 4.4 Cache com shareReplay

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

> **`refCount: true` vs `refCount: false`:**  
> Com `refCount: true`, quando o último subscriber cancela, o Observable fonte é cancelado e o cache é limpo. Com `false`, o cache persiste mesmo sem subscribers (útil para dados que mudam raramente, como configurações da aplicação).

---

<a name="5-rxjs"></a>
## 5. RxJS & OBSERVABLES

### 5.0 O que é RxJS e o Padrão Observer?

**RxJS (Reactive Extensions for JavaScript)** é uma biblioteca para programação reativa usando Observables. O Angular integra RxJS profundamente — HTTP, formulários, roteamento, change detection: tudo usa Observables.

**Conceitos fundamentais:**

| Conceito | Analogia | Descrição |
|----------|---------|-----------|
| **Observable** | Vídeo no YouTube | Sequência de valores ao longo do tempo |
| **Observer** | Você assistindo | Consome os valores (next, error, complete) |
| **Subscription** | Inscrição no canal | Conexão entre Observable e Observer |
| **Operator** | Filtro/efeito no vídeo | Transforma a sequência de valores |
| **Subject** | Live stream | Observable + Observer ao mesmo tempo |

**Observable vs Promise:**

| | Observable | Promise |
|--|-----------|---------|
| Valores | Múltiplos ao longo do tempo | Apenas um |
| Execução | Lazy (só executa quando subscrito) | Eager (executa imediatamente) |
| Cancelamento | Sim (unsubscribe) | Não |
| Operadores | Centenas (RxJS) | Limitados (.then, .catch) |
| Retry | `retry(n)` | Manual |

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

Todos os quatro são **higher-order mapping operators** — recebem um valor, criam um novo Observable internamente, e controlam como lidar com sobreposição de emissões. A diferença está em **o que fazer quando chega um novo valor antes do anterior terminar**:

```typescript
// switchMap - Cancela requisição anterior
// Use: Search, autocomplete
// Semântica: "só me interessa o resultado MAIS RECENTE"
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  switchMap(term => this.api.search(term))  // Cancela search anterior
).subscribe(results => this.results = results);

// mergeMap - Todas em paralelo
// Use: Batch uploads, não importa ordem
// Semântica: "processa TUDO ao mesmo tempo, não me importo com ordem"
from(files).pipe(
  mergeMap(file => this.uploadFile(file))  // Todos uploads ao mesmo tempo
).subscribe(result => console.log(result));

// concatMap - Uma por vez, em ordem
// Use: Processar fila ordenada
// Semântica: "processa um por vez, RESPEITANDO a ordem de chegada"
from([1, 2, 3]).pipe(
  concatMap(id => this.api.getUser(id))  // Espera 1 terminar antes de chamar 2
).subscribe(user => console.log(user));

// exhaustMap - Ignora novos até terminar
// Use: Prevenir double-click em botão
// Semântica: "enquanto estou processando, IGNORA novos pedidos"
this.loginButton.clicks$.pipe(
  exhaustMap(() => this.authService.login())  // Ignora cliques enquanto processa
).subscribe(result => console.log(result));
```

**Resumo visual:**
```
Chega A, depois B antes de A terminar:

switchMap:   A → cancelado, B → ✅ (só o último)
mergeMap:    A → ✅, B → ✅ (ambos, em paralelo)
concatMap:   A → ✅, B espera → ✅ (sequencial)
exhaustMap:  A → ✅, B → ignorado (primeiro ganha)
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

> **Como funciona o `combineLatest`?**  
> Ele só começa a emitir quando **todos** os Observables combinados emitiram pelo menos um valor. Depois, emite sempre que **qualquer um** deles emitir um novo valor, combinando com os valores mais recentes dos outros.

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

> **Como funciona o `forkJoin`?**  
> Similar ao `Promise.all`: aguarda **todos** os Observables completarem e emite um único array com os resultados de cada um. Se qualquer Observable emitir um erro, o `forkJoin` propaga esse erro.  
> **Importante:** `forkJoin` só funciona com Observables que **completam** (como requisições HTTP). Não use com Observables infinitos como `BehaviorSubject`.

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

### 5.6 Subjects — Tipos e Quando Usar

| Subject | Comportamento | Quando Usar |
|---------|--------------|-------------|
| `Subject` | Sem memória | Eventos pontuais (click, submit) |
| `BehaviorSubject(val)` | Lembra o último valor | Estado (usuário logado, filtro ativo) |
| `ReplaySubject(n)` | Lembra os últimos N valores | Cache de eventos recentes |
| `AsyncSubject` | Só emite o último valor ao completar | Resultado de uma operação |

```typescript
// BehaviorSubject — padrão mais comum para estado
private isLoggedIn$ = new BehaviorSubject<boolean>(false);

// Expondo como Observable (encapsulamento)
isLoggedIn = this.isLoggedIn$.asObservable();

login() { this.isLoggedIn$.next(true); }
logout() { this.isLoggedIn$.next(false); }
```

---

<a name="6-pipes"></a>
## 6. PIPES (BUILT-IN & CUSTOM)

### 6.0 O que são Pipes?

Pipes são **transformadores de valores** no template. Eles recebem um valor de entrada, aplicam uma transformação e retornam o valor transformado — sem modificar o original. São semelhantes a filtros em outras frameworks.

```
valor | nomePipe:arg1:arg2
```

**Por que usar Pipes em vez de métodos no template?**  
Pipes puros são memoizados — o Angular só recalcula quando os argumentos mudam. Um método no template é chamado a **cada** ciclo de change detection:

```html
<!-- ❌ Chamado em CADA change detection -->
<div>{{ formatDate(date) }}</div>

<!-- ✅ Só recalculado quando 'date' mudar -->
<div>{{ date | date:'dd/MM/yyyy' }}</div>
```

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

> **Async Pipe em detalhe:**  
> O `async` pipe faz três coisas automaticamente:  
> 1. Faz subscribe no Observable (ou resolve a Promise)  
> 2. Atualiza a view quando um novo valor é emitido  
> 3. Faz **unsubscribe automático** quando o component é destruído  
> É a forma mais segura de consumir Observables no template — elimina o risco de memory leaks.

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
// Pure (default) - só recalcula se inputs mudarem (por referência)
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

> **Por que Pipes puros não detectam mudanças internas em arrays/objetos?**  
> Um Pipe puro compara os argumentos por **referência** (`===`). Se você fizer `array.push(item)`, a referência do array não muda — o Pipe não sabe que o conteúdo mudou. Soluções: use `[...array, item]` para criar nova referência, ou torne o Pipe impuro (com custo de performance).

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

### 7.0 O que são Directives?

Directives são **instruções ao DOM** — elas modificam o comportamento ou a aparência de elementos existentes. Diferente de Components (que têm template próprio), Directives se aplicam sobre elementos já existentes.

**Tipos:**
- **Component Directive:** um component é uma directive com template
- **Attribute Directive:** modifica aparência ou comportamento de um elemento (`[appHighlight]`)
- **Structural Directive:** modifica a estrutura do DOM (`*ngIf`, `*ngFor`)

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

> **`@HostListener` vs `addEventListener`:**  
> `@HostListener` é a forma Angular de escutar eventos no elemento host da directive. Ele gerencia automaticamente o `removeEventListener` quando a directive é destruída, evitando memory leaks. Evite usar `addEventListener` manualmente.

> **`ElementRef` e segurança:**  
> Acessar `nativeElement` diretamente funciona, mas tem limitações em SSR (não há DOM) e pode abrir brechas de XSS se você inserir HTML não sanitizado. Para manipulações mais complexas, use o `Renderer2`:

```typescript
constructor(
  private el: ElementRef,
  private renderer: Renderer2
) {}

onMouseEnter() {
  this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', this.appHighlight);
}
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
    private templateRef: TemplateRef<any>,       // O template dentro da directive
    private viewContainer: ViewContainerRef      // Onde renderizar o template
  ) {}
}
```

```html
<!-- Uso -->
<div *appUnless="isLoggedIn">
  Faça login para continue
</div>
```

> **`TemplateRef` vs `ViewContainerRef`:**  
> - `TemplateRef`: representa o template delimitado pela directive (o `<ng-template>` implícito criado pelo `*`). É o "molde".  
> - `ViewContainerRef`: representa o "slot" no DOM onde o template será inserido. É o "local de inserção".  
> Juntos, eles permitem criar e destruir views dinamicamente.

---

<a name="8-forms"></a>
## 8. FORMS (REACTIVE & TEMPLATE-DRIVEN)

### 8.0 Reactive vs Template-Driven — Qual a diferença?

Angular oferece duas abordagens para formulários:

| Aspecto | Template-Driven | Reactive Forms |
|---------|----------------|---------------|
| **Definição do form** | No template (HTML) | Na classe TypeScript |
| **Modelo de dados** | Implícito (ngModel) | Explícito (FormControl) |
| **Validação** | Diretivas no HTML | Funções no TypeScript |
| **Testabilidade** | Difícil (precisa do DOM) | Fácil (puro TypeScript) |
| **Tipagem** | Fraca | Forte (Angular 14+) |
| **Reatividade** | Limitada | Total (valueChanges, statusChanges) |
| **Complexidade** | Simples | Mais verboso, porém escalável |
| **Quando usar** | Forms simples, login | Forms complexos, dinâmicos |

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
    
    // Reagir a mudanças — Observable de mudanças de valor
    this.loginForm.valueChanges.subscribe(values => {
      console.log(values);
    });
    
    // Reagir apenas a mudanças de status (VALID, INVALID, PENDING, DISABLED)
    this.loginForm.statusChanges.subscribe(status => {
      console.log(status);
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

> **Estados de um FormControl:**  
> - `pristine` / `dirty`: o usuário ainda não alterou / já alterou  
> - `untouched` / `touched`: o campo ainda não perdeu o foco / já perdeu  
> - `valid` / `invalid` / `pending` / `disabled`: status de validação  
> A combinação `invalid && touched` é padrão para mostrar erros somente após o usuário interagir.

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
// O status do control fica "PENDING" durante a verificação
export function emailExistsValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) return of(null);
    
    return userService.checkEmail(control.value).pipe(
      debounceTime(400),           // Evita requests a cada keystroke
      map(exists => exists ? { emailJaExiste: true } : null),
      catchError(() => of(null))   // Em caso de erro de rede, não bloqueia
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
    [emailExistsValidator(this.userService)]  // Async (terceiro parâmetro)
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

```html
<!-- Exibir erro do grupo, não do control individual -->
<div *ngIf="form.errors?.['passwordMismatch'] && form.get('confirmPassword')?.touched">
  As senhas não coincidem
</div>
```

---

<a name="9-routing"></a>
## 9. ROUTING AVANÇADO

### 9.0 Como o Roteamento Funciona

O Angular usa o **Router** para mapear URLs a components. O processo quando o usuário navega:

1. O Router intercepta a mudança de URL
2. Percorre a lista de rotas buscando um match
3. Executa os Guards (CanActivate, etc.)
4. Executa os Resolvers (pre-fetch de dados)
5. Ativa o component correspondente no `<router-outlet>`

### 9.1 Configuração Básica

```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: '**', component: NotFoundComponent }  // 404 — sempre por último!
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

> **Ordem das rotas importa!**  
> O Angular usa a **primeira rota que casar** (first-match strategy). Rotas mais específicas devem vir antes das genéricas. O `**` (wildcard) deve ser sempre o último.

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
- ✅ Melhora performance inicial (Time to Interactive)
- ✅ O build gera chunks separados para cada módulo lazy

> **Preloading Strategy:**  
> Por padrão, módulos lazy são carregados apenas quando o usuário acessa a rota. Mas você pode usar `PreloadAllModules` para que o Angular pré-carregue os módulos no background após o carregamento inicial:

```typescript
RouterModule.forRoot(routes, {
  preloadingStrategy: PreloadAllModules
})
```

### 9.3 Route Guards

Guards são **interceptadores de navegação** — decidem se a navegação pode prosseguir.

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

**Tipos de Guards:**

| Guard | Pergunta | Retorno |
|-------|----------|---------|
| `CanActivate` | Posso entrar nesta rota? | `boolean \| UrlTree` |
| `CanActivateChild` | Posso entrar em rotas filhas? | `boolean \| UrlTree` |
| `CanDeactivate` | Posso sair desta rota? | `boolean \| Observable<boolean>` |
| `CanLoad` | Posso carregar este módulo lazy? | `boolean \| UrlTree` |
| `Resolve` | Quais dados pré-carregar? | `Observable<T>` |

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

> **Por que usar Resolve?**  
> Sem Resolve, o component carrega e exibe um estado de loading enquanto busca os dados. Com Resolve, o Angular aguarda os dados antes de ativar o component — o component recebe os dados já prontos no `ngOnInit`. Melhora a UX evitando "flash" de loading.

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
// Snapshot (valor atual — não reage a mudanças futuras na mesma instância)
const id = this.route.snapshot.paramMap.get('id');

// Observable (reage a mudanças — use quando o mesmo component pode ser reusado)
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

> **Quando usar snapshot vs Observable?**  
> Se o usuário pode navegar de `/users/1` para `/users/2` sem destruir e recriar o component (o Angular reusa a instância), `snapshot` fica desatualizado. Use o Observable de `paramMap` nesses casos para reagir à mudança do parâmetro.

---

<a name="10-change-detection"></a>
## 10. CHANGE DETECTION & OnPush

### 10.0 O que é Change Detection?

**Change Detection** é o mecanismo que o Angular usa para sincronizar o estado do component (classe TypeScript) com a view (template HTML). Sempre que algo pode ter mudado (evento do usuário, timer, resposta HTTP), o Angular verifica se o estado mudou e, se sim, atualiza o DOM.

**Como o Angular sabe quando algo mudou?**  
Por padrão, o Angular usa o **Zone.js** — uma biblioteca que "patcha" (substitui) funções assíncronas nativas do browser (`setTimeout`, `addEventListener`, `Promise.then`, etc.) para notificar o Angular quando operações assíncronas completam. Isso dispara o ciclo de change detection.

### 10.1 Estratégias

| Estratégia | Como Funciona | Performance | Quando Usar |
|-----------|---------------|-------------|-------------|
| **Default** | Checa toda árvore a cada evento | Boa | Apps pequenas |
| **OnPush** | Só checa quando: Input muda (ref), evento interno, async pipe | Excelente | Apps médias/grandes |

**O ciclo de Default Change Detection:**
```
Evento (click, HTTP, timer)
    → Zone.js notifica Angular
        → Angular percorre TODA a árvore de components
            → Verifica cada binding em cada component
                → Atualiza o DOM onde necessário
```

**Com OnPush, o Angular pula o component** a menos que:
1. Um `@Input` receba uma nova **referência** (não apenas mudança interna)
2. Um evento DOM dispare dentro do próprio component
3. O `async` pipe emita um novo valor
4. `ChangeDetectorRef.markForCheck()` seja chamado manualmente

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
    this.users.push(user);  // Mesma referência — Angular não detecta
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
- Zero memory leaks (async pipe faz unsubscribe automático)

### 10.4 ChangeDetectorRef - Controle Manual

```typescript
import { ChangeDetectorRef } from '@angular/core';

export class MyComponent {
  constructor(private cdr: ChangeDetectorRef) {}
  
  // markForCheck: agenda uma checagem no próximo ciclo
  updateFromExternalLibrary() {
    // Ex: callback de biblioteca third-party (fora do Zone.js)
    externalLib.onUpdate((data) => {
      this.data = data;
      this.cdr.markForCheck();  // "Angular, me cheque no próximo ciclo"
    });
  }
  
  // detectChanges: força checagem IMEDIATA (do component para baixo)
  forceUpdate() {
    this.data = 'Nova';
    this.cdr.detectChanges();  // Útil em casos de timing específico
  }
  
  // detach/reattach: remove o component do ciclo de detection completamente
  // Útil para componentes pesados que você controla manualmente
  optimizeHeavyList() {
    this.cdr.detach();  // Angular não verifica mais este component automaticamente
    // Você chama detectChanges() manualmente quando sabe que mudou
  }
}
```

---

<a name="11-signals"></a>
## 11. SIGNALS (Angular 16+)

### 11.0 Por que Signals foram criados?

O Angular historicamente dependia do **Zone.js** para detectar mudanças. Problemas:
- Zone.js tem overhead de performance (patcha TODAS as APIs assíncronas)
- Change detection é acionada de forma ampla, não granular
- Difícil saber "exatamente o que mudou"

**Signals** introduzem **reatividade granular**: o Angular sabe exatamente qual Signal mudou e atualiza **apenas** as partes do DOM que dependem daquele Signal — sem checar a árvore inteira.

> **Analogia:** Zone.js é como um alarme que dispara em toda a casa quando algo muda. Signals são sensores individuais em cada cômodo.

### 11.1 O que são Signals?

Signals são **primitivos reativos** introduzidos no Angular 16. Alternativa aos Observables para gerenciar estado.

```typescript
import { Component, signal, computed } from '@angular/core';

export class CounterComponent {
  // Signal primitivo
  count = signal(0);
  
  // Computed (auto-recalcula quando dependências mudam)
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
<div>Count: {{ count() }}</div>       <!-- Chame o signal como função -->
<div>Double: {{ doubleCount() }}</div>
<button (click)="increment()">+1</button>
```

> **Signals são síncronos e sempre têm um valor inicial** — diferente de Observables, que podem não emitir nada imediatamente. Isso os torna mais simples para representar estado.

### 11.2 Signal vs BehaviorSubject

| Feature | Signal | BehaviorSubject |
|---------|--------|-----------------|
| **Leitura** | `count()` | `count$ \| async` |
| **Escrita** | `count.set(5)` | `count$.next(5)` |
| **Performance** | Melhor (granular) | Boa (Zone.js) |
| **Change Detection** | Só affected components | Árvore inteira |
| **RxJS Operators** | Não (mas tem interop) | Sim |
| **Syntax** | Mais simples | Mais verbosa |
| **Valor inicial** | Obrigatório | Obrigatório |
| **Valor atual síncrono** | `count()` | `count$.getValue()` |

### 11.3 Signals com Arrays/Objects

```typescript
export class TodoComponent {
  todos = signal<Todo[]>([]);
  
  addTodo(title: string) {
    // ❌ ERRADO - mutação não dispara reatividade
    this.todos().push({ id: Date.now(), title });
    
    // ✅ CERTO - nova referência dispara reatividade
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
  // Observable → Signal (para consumir no template sem async pipe)
  users = toSignal(this.userService.getUsers(), {
    initialValue: [] as User[]
  });
  
  // Signal → Observable (para usar operadores RxJS)
  searchTerm = signal('');
  results$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    switchMap(term => this.searchService.search(term))
  );
  
  // Ou converta de volta para Signal
  results = toSignal(this.results$, { initialValue: [] });
}
```

> **Quando usar `toSignal`?**  
> Quando você tem um Observable (ex: HTTP) mas quer consumi-lo no template com a sintaxe de Signal (`signal()`) em vez do `async` pipe. O `toSignal` faz subscribe automaticamente e faz unsubscribe quando o component é destruído.

### 11.5 Effect - Side Effects

```typescript
import { effect } from '@angular/core';

export class UserComponent {
  userId = signal(1);
  theme = signal<'light' | 'dark'>('light');
  
  constructor() {
    // Roda imediatamente e toda vez que userId mudar
    effect(() => {
      console.log('User ID:', this.userId());
      localStorage.setItem('userId', this.userId().toString());
    });
    
    // Effect com cleanup
    effect((onCleanup) => {
      const interval = setInterval(() => console.log('tick'), 1000);
      onCleanup(() => clearInterval(interval));  // Limpa ao re-executar
    });
  }
}
```

> **Cuidado:** Evite modificar outros Signals dentro de um `effect` — isso pode causar ciclos infinitos. Use `computed` para derivar valores. O `effect` é para side effects (DOM, localStorage, logs).

---

<a name="12-standalone"></a>
## 12. STANDALONE COMPONENTS

### 12.0 O problema com NgModules

Antes do Angular 14, toda peça de código precisava ser declarada em um `NgModule`. Isso criava overhead de boilerplate e dificultava entender onde cada component estava disponível.

**Standalone Components** eliminam a necessidade de NgModules — cada component declara suas próprias dependências.

### 12.1 Component Standalone

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-user',
  standalone: true,  // ← Standalone!
  imports: [CommonModule, FormsModule],  // Importa aqui, não no NgModule
  template: `...`
})
export class UserComponent {}
```

**Benefícios:**
- ✅ Sem NgModule — menos boilerplate
- ✅ Tree-shaking melhor (Angular vê exatamente o que cada component usa)
- ✅ Lazy load direto por component (não só por módulo)
- ✅ Mais fácil de entender o que o component precisa

### 12.2 App Standalone (Sem AppModule)

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideAnimations } from '@angular/platform-browser/animations';

bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimations(),
  ]
});
```

### 12.3 Lazy Load Standalone

```typescript
// routes — lazy load por component, não por módulo
{
  path: 'admin',
  loadComponent: () => import('./admin.component').then(m => m.AdminComponent)
}

// Ou com grupo de rotas standalone
{
  path: 'admin',
  loadChildren: () => import('./admin.routes').then(r => r.ADMIN_ROUTES)
}
```

---

<a name="13-http"></a>
## 13. HTTP & INTERCEPTORS

### 13.0 Como o HttpClient Funciona

O `HttpClient` do Angular é uma abstração sobre o `XMLHttpRequest` (ou `fetch` em versões mais recentes). Ele retorna **Observables** — não Promises — o que permite:
- Cancelar requisições (unsubscribe)
- Aplicar operadores RxJS
- Compor requisições complexas

> **Importante:** Por padrão, o Observable do HttpClient é **cold** — a requisição só é feita quando alguém faz subscribe. Se ninguém subscrever, a requisição não acontece.

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
  
  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>('/api/users', user);
  }
  
  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`/api/users/${id}`, user);
  }
  
  patchUser(id: number, changes: Partial<User>): Observable<User> {
    return this.http.patch<User>(`/api/users/${id}`, changes);
  }
  
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`/api/users/${id}`);
  }
  
  // Com parâmetros de query
  searchUsers(term: string, page = 1): Observable<User[]> {
    const params = new HttpParams()
      .set('search', term)
      .set('page', page.toString());
    return this.http.get<User[]>('/api/users', { params });
  }
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    const message = error.error?.message || 'Erro desconhecido';
    console.error('Erro:', error.status, message);
    return throwError(() => new Error(message));
  }
}
```

### 13.2 HTTP Interceptor

**Interceptors** são middlewares para requisições HTTP — funcionam como um pipeline que toda requisição e resposta atravessa. Casos de uso comuns:
- Adicionar token de autenticação em todas as requisições
- Logging centralizado
- Tratamento global de erros
- Mostrar/esconder loading global
- Retry automático

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Clonar a requisição (é imutável) e adicionar header
    const token = this.authService.getToken();
    
    const authReq = token
      ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
      : req;
    
    return next.handle(authReq).pipe(
      retry(2),
      catchError(error => {
        if (error instanceof HttpErrorResponse) {
          if (error.status === 401) {
            // Token expirado
            this.authService.logout();
            this.router.navigate(['/login']);
          }
          if (error.status === 403) {
            this.router.navigate(['/forbidden']);
          }
        }
        return throwError(() => error);
      })
    );
  }
}
```

```typescript
// Registro (AppModule)
providers: [
  {
    provide: HTTP_INTERCEPTORS,
    useClass: AuthInterceptor,
    multi: true  // 'multi: true' permite múltiplos interceptors
  }
]

// Registro (Standalone — interceptor funcional)
// main.ts
provideHttpClient(
  withInterceptors([authInterceptorFn, loggingInterceptorFn])
)
```

> **Ordem dos Interceptors:**  
> Interceptors são aplicados na ordem em que são declarados. Para requisições, o primeiro declarado é o primeiro a executar. Para respostas, a ordem é inversa (o último declarado trata a resposta primeiro). Pense como uma cebola — cada interceptor envolve o próximo.

---

<a name="14-testing"></a>
## 14. TESTING

### 14.0 Pirâmide de Testes no Angular

```
        E2E Tests (Cypress, Playwright)
      ————————————————————————————————————
    Integration Tests (TestBed com dependências reais)
  ——————————————————————————————————————————————————————
Unit Tests (Jasmine/Jest — mocks de tudo externo)
```

- **Unit Tests:** testam uma unidade isolada (component, service, pipe). Dependências externas são mockadas.
- **Integration Tests:** testam a integração entre partes (component + template + service real ou parcialmente mockado).
- **E2E Tests:** testam fluxos completos no browser real.

> **Regra geral:** a base da pirâmide deve ser maior. Unit tests são rápidos e baratos. E2E são lentos e frágeis — use com moderação.

### 14.1 Teste de Component

```typescript
describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let mockService: jasmine.SpyObj<UserService>;
  
  beforeEach(() => {
    // Cria um mock com os métodos que precisamos
    mockService = jasmine.createSpyObj('UserService', ['getUsers', 'deleteUser']);
    
    TestBed.configureTestingModule({
      declarations: [UserListComponent],
      providers: [
        { provide: UserService, useValue: mockService }  // Injeta o mock
      ]
    });
    
    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
  });
  
  it('should load users on init', () => {
    const mockUsers: User[] = [{ id: 1, name: 'João', email: 'joao@test.com' }];
    mockService.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges();  // Dispara ngOnInit
    
    expect(component.users).toEqual(mockUsers);
    expect(mockService.getUsers).toHaveBeenCalledOnce();
  });
  
  it('should render users in template', () => {
    const mockUsers: User[] = [{ id: 1, name: 'João', email: 'joao@test.com' }];
    mockService.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges();
    
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('.user-name')?.textContent).toContain('João');
  });
  
  it('should show error when service fails', () => {
    mockService.getUsers.and.returnValue(throwError(() => new Error('Network error')));
    
    fixture.detectChanges();
    
    expect(component.error).toBeTruthy();
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
      imports: [HttpClientTestingModule],  // Substitui HttpClient por mock
      providers: [UserService]
    });
    
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify();  // Garante que não sobrou requisições pendentes
  });
  
  it('should fetch users', () => {
    const mockUsers: User[] = [{ id: 1, name: 'João', email: 'joao@test.com' }];
    
    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
      expect(users.length).toBe(1);
    });
    
    // Intercepta e responde à requisição
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);  // Simula resposta do servidor
  });
  
  it('should handle HTTP error', () => {
    service.getUsers().subscribe({
      next: () => fail('Deveria ter falhado'),
      error: (err) => expect(err.message).toBeTruthy()
    });
    
    const req = httpMock.expectOne('/api/users');
    req.flush('Server error', { status: 500, statusText: 'Internal Server Error' });
  });
});
```

### 14.3 Teste de Pipe

```typescript
describe('TruncatePipe', () => {
  const pipe = new TruncatePipe();
  
  it('should truncate long text', () => {
    const longText = 'a'.repeat(100);
    expect(pipe.transform(longText, 50)).toBe('a'.repeat(50) + '...');
  });
  
  it('should not truncate short text', () => {
    expect(pipe.transform('curto', 50)).toBe('curto');
  });
  
  it('should use custom ellipsis', () => {
    const longText = 'a'.repeat(100);
    expect(pipe.transform(longText, 10, '---')).toBe('a'.repeat(10) + '---');
  });
});
```

---

<a name="15-performance"></a>
## 15. PERFORMANCE OPTIMIZATION

### 15.0 Como Pensar em Performance Angular

Performance no Angular tem duas dimensões:
1. **Tempo de carregamento inicial** (bundle size, lazy loading)
2. **Performance em runtime** (change detection, DOM operations)

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

> **Por que usar?**  
> Renderizar 10.000 elementos DOM de uma vez é muito custoso. O Virtual Scrolling renderiza apenas os itens **visíveis na viewport** — os outros são representados apenas como espaço vazio. Resultado: performance constante independente do tamanho da lista.

```typescript
import { ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  imports: [ScrollingModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <!-- itemSize: altura de cada item em pixels -->
      <div *cdkVirtualFor="let item of items; trackBy: trackById">{{ item.name }}</div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport { height: 500px; }  /* Altura fixa obrigatória */
  `]
})
export class LargeListComponent {}
```

### 15.3 Pure Pipes

```typescript
// ❌ RUIM - função chamada a cada change detection
<div *ngFor="let user of filterUsers(users, term)">

// ✅ BOM - pipe puro é memoizado
@Pipe({ name: 'filterUsers', pure: true })
<div *ngFor="let user of users | filterUsers:term">
```

### 15.4 OnPush Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

### 15.5 Lazy Loading e Code Splitting

```typescript
// Divide o bundle em chunks carregados sob demanda
{
  path: 'dashboard',
  loadComponent: () => import('./dashboard/dashboard.component')
    .then(m => m.DashboardComponent)
}
```

### 15.6 Deferrable Views (Angular 17+)

> Permite adiar o carregamento de partes pesadas do template com condições declarativas:

```html
<!-- Carrega apenas quando o elemento entra na viewport -->
@defer (on viewport) {
  <app-heavy-chart [data]="data" />
} @placeholder {
  <div class="chart-skeleton">Carregando gráfico...</div>
} @loading (minimum 200ms) {
  <app-spinner />
} @error {
  <p>Erro ao carregar gráfico</p>
}
```

### 15.7 NgOptimizedImage (Angular 15+)

```typescript
import { NgOptimizedImage } from '@angular/common';

// No component
@Component({
  imports: [NgOptimizedImage],
  template: `
    <img ngSrc="profile.jpg" width="200" height="200" priority>
    <!-- priority: preload para imagens above the fold -->
  `
})
```

> **Benefícios:** lazy loading automático, `srcset` responsivo, Core Web Vitals otimizados (LCP).

### 15.8 Bundle Analysis

```bash
# Analisar o tamanho do bundle
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

---

<a name="16-content-projection"></a>
## 16. CONTENT PROJECTION

### 16.0 O que é Content Projection?

Content Projection (projeção de conteúdo) é o mecanismo que permite passar template HTML de um component pai para dentro de um component filho. É o equivalente Angular ao `{children}` do React ou `<slot>` do Vue.

Isso permite criar **components altamente reutilizáveis** que definem estrutura e comportamento, mas deixam o conteúdo flexível.

### 16.1 Projeção Simples

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content></ng-content>  <!-- Slot padrão -->
    </div>
  `
})
export class CardComponent {}
```

```html
<app-card>
  <p>Este parágrafo será projetado para dentro do card</p>
</app-card>
```

### 16.2 Multi-Slot (Named Slots)

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="header">
        <ng-content select="[slot='header']"></ng-content>
      </div>
      <div class="body">
        <ng-content></ng-content>  <!-- Slot padrão (pega o que sobrar) -->
      </div>
      <div class="footer">
        <ng-content select="[slot='footer']"></ng-content>
      </div>
    </div>
  `
})
export class CardComponent {}
```

```html
<app-card>
  <h2 slot="header">Título do Card</h2>
  <p>Conteúdo principal do card</p>
  <button slot="footer">Fechar</button>
</app-card>
```

> **O `select` do `ng-content` aceita qualquer seletor CSS válido** — atributos, classes, elementos, etc. O conteúdo não selecionado vai para o `<ng-content>` sem `select`.

### 16.3 ContentChild e ContentChildren

Permitem acessar os elementos projetados via content projection:

```typescript
@Component({ selector: 'app-tabs' })
export class TabsComponent implements AfterContentInit {
  @ContentChildren(TabComponent) tabs!: QueryList<TabComponent>;
  
  ngAfterContentInit() {
    // Aqui o conteúdo projetado já está disponível
    this.tabs.first.isActive = true;
  }
}
```

---

<a name="17-viewchild"></a>
## 17. VIEWCHILD & CONTENTCHILD

### 17.0 ViewChild vs ContentChild

| | ViewChild / ViewChildren | ContentChild / ContentChildren |
|--|--------------------------|-------------------------------|
| **O que acessa** | Elementos do próprio template | Elementos projetados via `ng-content` |
| **Disponível em** | `ngAfterViewInit` | `ngAfterContentInit` |
| **Uso típico** | Manipular DOM, acessar filho component | Acesso a slots projetados |

### 17.1 ViewChild

```typescript
// Acessar elemento DOM nativo
@ViewChild('input') input!: ElementRef<HTMLInputElement>;

// Acessar instância de component filho
@ViewChild(ChildComponent) child!: ChildComponent;

// Acessar como ViewContainerRef (para criar components dinamicamente)
@ViewChild('container', { read: ViewContainerRef }) container!: ViewContainerRef;

// Acessar template ref
@ViewChild('myTemplate') template!: TemplateRef<any>;

ngAfterViewInit() {
  // Disponível aqui
  this.input.nativeElement.focus();
  this.child.somePublicMethod();
}
```

### 17.2 ViewChildren (Lista)

```typescript
// Acessa todos os filhos de um tipo
@ViewChildren(ItemComponent) items!: QueryList<ItemComponent>;

ngAfterViewInit() {
  console.log(this.items.length); // Quantidade de items
  
  // QueryList é observável — reage quando a lista muda (ex: *ngFor)
  this.items.changes.subscribe(list => {
    console.log('Lista de items mudou:', list.length);
  });
  
  // Iterar
  this.items.forEach(item => item.highlight());
}
```

### 17.3 Static Option

```typescript
// static: true — disponível NO ngOnInit (não precisa esperar AfterViewInit)
// Use apenas quando o elemento SEMPRE existe no template (sem *ngIf)
@ViewChild('input', { static: true }) input!: ElementRef;

ngOnInit() {
  this.input.nativeElement.focus(); // Funciona porque static: true
}
```

---

<a name="18-dynamic-components"></a>
## 18. DYNAMIC COMPONENTS

### 18.0 Por que Criar Components Dinamicamente?

Em alguns cenários, você não sabe em tempo de compilação qual component renderizar — modais, toasts, wizards dinâmicos, dashboards configuráveis. A criação dinâmica resolve isso.

### 18.1 Criação com ViewContainerRef

```typescript
@Component({
  template: `<ng-container #container></ng-container>`
})
export class DynamicHostComponent {
  @ViewChild('container', { read: ViewContainerRef }) container!: ViewContainerRef;
  
  loadComponent(type: 'alert' | 'modal') {
    this.container.clear();  // Remove componente anterior
    
    if (type === 'alert') {
      const ref = this.container.createComponent(AlertComponent);
      // Passando dados para o componente
      ref.instance.message = 'Operação realizada!';
      ref.instance.type = 'success';
      // Escutando outputs
      ref.instance.closed.subscribe(() => ref.destroy());
    }
  }
}
```

### 18.2 Passando @Input e Escutando @Output

```typescript
const ref = this.container.createComponent(ModalComponent);

// @Inputs (Angular 14+)
ref.setInput('title', 'Confirmar ação');
ref.setInput('message', 'Deseja realmente deletar?');

// @Outputs
ref.instance.confirmed.subscribe(() => {
  this.doDelete();
  ref.destroy();
});
ref.instance.cancelled.subscribe(() => ref.destroy());
```

---

<a name="19-animations"></a>
## 19. ANIMATIONS

### 19.0 Como Funciona o Sistema de Animações

O Angular usa a **Web Animations API** internamente, com uma DSL declarativa. As animações são definidas na metadata do component via `animations: []` e aplicadas no template com `[@triggerName]`.

### 19.1 Animação Básica

```typescript
import { trigger, state, style, transition, animate, keyframes } from '@angular/animations';

@Component({
  animations: [
    // trigger: nome da animação usada no template
    trigger('fadeIn', [
      // :enter = quando o elemento é adicionado ao DOM
      transition(':enter', [
        style({ opacity: 0, transform: 'translateY(-10px)' }),
        animate('300ms ease-out', style({ opacity: 1, transform: 'translateY(0)' }))
      ]),
      // :leave = quando o elemento é removido do DOM
      transition(':leave', [
        animate('200ms ease-in', style({ opacity: 0, transform: 'translateY(-10px)' }))
      ])
    ]),
    
    // State-based animation
    trigger('expand', [
      state('collapsed', style({ height: '0px', overflow: 'hidden' })),
      state('expanded', style({ height: '*' })),  // '*' = altura natural
      transition('collapsed <=> expanded', animate('300ms ease'))
    ])
  ]
})
export class MyComponent {
  isExpanded = false;
}
```

```html
<!-- Uso no template -->
<div *ngIf="show" @fadeIn>
  Este elemento faz fade ao aparecer/desaparecer
</div>

<div [@expand]="isExpanded ? 'expanded' : 'collapsed'">
  Conteúdo expansível
</div>
```

### 19.2 Animação de Lista (query + stagger)

```typescript
trigger('listAnimation', [
  transition('* => *', [
    query(':enter', [
      style({ opacity: 0, transform: 'translateX(-20px)' }),
      stagger(100, [  // 100ms de delay entre cada item
        animate('300ms ease', style({ opacity: 1, transform: 'translateX(0)' }))
      ])
    ], { optional: true })
  ])
])
```

---

<a name="20-ssr"></a>
## 20. SSR (SERVER-SIDE RENDERING)

### 20.0 O que é SSR e por que usar?

**Client-Side Rendering (CSR):** o browser baixa um HTML vazio, baixa o JavaScript, executa o JavaScript, que então renderiza a página. O usuário vê uma tela em branco ou loading durante esse processo.

**Server-Side Rendering (SSR):** o servidor renderiza o HTML completo e envia ao browser. O usuário vê conteúdo imediatamente — depois o JavaScript "hidrata" a página para torná-la interativa.

**Benefícios do SSR:**
- ✅ **SEO:** crawlers leem o HTML completo (não precisam executar JS)
- ✅ **Performance percebida:** FCP (First Contentful Paint) muito mais rápido
- ✅ **Redes lentas:** o conteúdo aparece antes do JS carregar

**Quando NÃO usar SSR:**
- Apps atrás de login (SEO não importa)
- Dashboards com muita interatividade
- Apps que dependem muito de APIs do browser (canvas, WebGL)

### 20.1 Configuração

```bash
# Adicionar SSR a um projeto existente
ng add @nguniversal/express-engine

# Ou ao criar um novo projeto
ng new my-app --ssr
```

### 20.2 Cuidados com Plataforma

```typescript
import { isPlatformBrowser, isPlatformServer } from '@angular/common';
import { PLATFORM_ID, inject } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class PlatformService {
  private platformId = inject(PLATFORM_ID);
  
  get isBrowser(): boolean {
    return isPlatformBrowser(this.platformId);
  }
  
  get isServer(): boolean {
    return isPlatformServer(this.platformId);
  }
}

// No component
export class MyComponent implements OnInit {
  private platformService = inject(PlatformService);
  
  ngOnInit() {
    if (this.platformService.isBrowser) {
      // APIs exclusivas do browser
      localStorage.setItem('key', 'value');
      window.scrollTo(0, 0);
      const width = window.innerWidth;
    }
  }
}
```

### 20.3 Hydration (Angular 16+)

A partir do Angular 16, o Angular suporta **Non-Destructive Hydration** — em vez de descartar o HTML renderizado pelo servidor e recriar tudo no cliente, ele reutiliza o DOM existente. Isso elimina o "flash" de conteúdo e melhora o Core Web Vitals.

```typescript
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration()  // Habilita hydration
  ]
});
```

---

<a name="21-pwa"></a>
## 21. PWA (PROGRESSIVE WEB APPS)

### 21.0 O que é uma PWA?

Uma **Progressive Web App** é uma web app que usa APIs modernas do browser para oferecer experiência similar a apps nativos: instalável, funciona offline, envia notificações push, carrega instantaneamente.

**Componentes principais:**
- **Service Worker:** script que roda em background, intercepta requisições de rede, habilita cache offline
- **Web App Manifest:** arquivo JSON que define ícone, nome, cor de tema para instalação
- **HTTPS:** obrigatório para Service Workers

### 21.1 Configuração

```bash
ng add @angular/pwa
```

Isso adiciona automaticamente:
- `ngsw-config.json` — configuração do Service Worker
- `manifest.webmanifest` — metadados para instalação
- `ServiceWorkerModule` no AppModule

### 21.2 Estratégias de Cache

```json
// ngsw-config.json
{
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",    // Baixa durante a instalação do SW
      "resources": {
        "files": ["/favicon.ico", "/index.html", "/*.css", "/*.js"]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "api-performance",
      "urls": ["/api/config"],
      "cacheConfig": {
        "strategy": "performance",  // Cache primeiro (mais rápido)
        "maxSize": 10,
        "maxAge": "1d"
      }
    },
    {
      "name": "api-freshness",
      "urls": ["/api/users/**"],
      "cacheConfig": {
        "strategy": "freshness",    // Network primeiro (dados mais atuais)
        "maxSize": 100,
        "maxAge": "1h",
        "timeout": "10s"           // Timeout antes de usar cache
      }
    }
  ]
}
```

---

<a name="22-boas-praticas"></a>
## 22. BOAS PRÁTICAS & DESIGN PATTERNS

### 22.0 Princípios Gerais

Todo código Angular de qualidade segue três princípios:
1. **Separação de responsabilidades:** cada peça faz uma coisa
2. **Imutabilidade:** não mute objetos — crie novos (funciona bem com OnPush)
3. **Reatividade:** use Observables/Signals para lidar com dados que mudam

### 22.1 SOLID Principles no Angular

- **S**ingle Responsibility: Um component/service faz uma coisa
  - *Ruim:* `UserComponent` que faz HTTP, formata dados e gerencia estado
  - *Bom:* `UserComponent` só renderiza; `UserService` gerencia HTTP; `UserFacade` orquestra

- **O**pen/Closed: Aberto para extensão, fechado para modificação
  - Prefira composição (inject services, use diretivas) a herança
  - Use interfaces e abstrações

- **L**iskov Substitution: Substituível por subtipos
  - Se você tem `HttpClient`, deveria poder trocar por um `MockHttpClient` sem quebrar nada

- **I**nterface Segregation: Interfaces pequenas e específicas
  - Evite mega-interfaces; prefira `Nameable`, `Deletable`, `Editable` separadas

- **D**ependency Inversion: Dependa de abstrações
  - Injete interfaces/tokens, não implementações concretas

### 22.2 Patterns Comuns

#### **Facade Pattern**

```typescript
// UserFacade: esconde a complexidade de múltiplos services
@Injectable({ providedIn: 'root' })
export class UserFacade {
  users$ = this.userService.getUsers();
  isLoading$ = this.uiService.isLoading$;
  
  constructor(
    private userService: UserService,
    private notificationService: NotificationService,
    private uiService: UiService
  ) {}
  
  deleteUser(id: number) {
    this.uiService.setLoading(true);
    this.userService.delete(id).pipe(
      finalize(() => this.uiService.setLoading(false))
    ).subscribe({
      next: () => this.notificationService.success('Usuário deletado'),
      error: () => this.notificationService.error('Erro ao deletar')
    });
  }
}
```

#### **Repository Pattern**

```typescript
// Abstrai o acesso a dados — fácil de trocar para outra fonte (API, localStorage)
@Injectable()
export abstract class UserRepository {
  abstract findAll(): Observable<User[]>;
  abstract findById(id: number): Observable<User>;
  abstract save(user: User): Observable<User>;
  abstract delete(id: number): Observable<void>;
}

@Injectable()
export class HttpUserRepository extends UserRepository {
  constructor(private http: HttpClient) { super(); }
  
  findAll(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
  // ...
}
```

#### **Observer Pattern** (RxJS)

```typescript
// Já embutido no RxJS — Subjects são Observables que também aceitam valores
@Injectable({ providedIn: 'root' })
export class EventBusService {
  private events$ = new Subject<AppEvent>();
  
  emit(event: AppEvent) {
    this.events$.next(event);
  }
  
  on(type: string): Observable<AppEvent> {
    return this.events$.pipe(filter(e => e.type === type));
  }
}
```

### 22.3 Nomenclatura e Estrutura

```
src/
  app/
    core/           # Services singleton, guards, interceptors
    shared/         # Components, pipes, directives reutilizáveis
    features/
      users/        # Feature module/folder
        components/ # Components desta feature
        services/   # Services desta feature
        models/     # Interfaces/types
        store/      # State management
        users.routes.ts
        users.component.ts
```

```typescript
// Convenções de nomenclatura
user.component.ts       // Component
user.service.ts         // Service
user.model.ts           // Interface/Type
user.pipe.ts            // Pipe
user.directive.ts       // Directive
user.guard.ts           // Guard
user.resolver.ts        // Resolver
user.interceptor.ts     // Interceptor
user.spec.ts            // Teste
```

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
// Typed Forms — FormControl agora sabe o tipo
const name = new FormControl<string>('');
name.value;       // string | null
name.getRawValue(); // string
```

### 23.3 Angular 15

**Principais Features:**
- ✅ Standalone APIs stable
- ✅ Directive Composition API
- ✅ Image directive (NgOptimizedImage)
- ✅ Functional router guards

```typescript
// Functional guards — sem necessidade de classe
export const authGuard = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() || router.createUrlTree(['/login']);
};

// Directive Composition — componha comportamento via hostDirectives
@Component({
  hostDirectives: [
    { directive: TooltipDirective, inputs: ['text: tooltip'] },
    FocusTrapDirective
  ]
})
export class DialogComponent {}
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

// Required inputs — erro em compile-time se não passado
@Input({ required: true }) user!: User;

// DestroyRef — cleanup sem ngOnDestroy
constructor() {
  const destroyRef = inject(DestroyRef);
  destroyRef.onDestroy(() => {
    console.log('Cleanup sem implementar OnDestroy');
  });
}

// takeUntilDestroyed — mais simples que o pattern destroy$
this.service.getData().pipe(
  takeUntilDestroyed()  // Auto cleanup quando o component é destruído!
).subscribe();
```

### 23.5 Angular 17 ⭐⭐

**Principais Features:**
- ✅ **Novo Control Flow** (@if, @for, @switch) — built-in, sem NgIf/NgFor
- ✅ **Deferrable Views** (@defer)
- ✅ Novo builder (esbuild) — builds até 87% mais rápidos
- ✅ SSR/SSG melhorados

```html
<!-- Novo control flow — sem diretivas, sem NgIf/NgFor no imports -->
@if (user) {
  <div>{{ user.name }}</div>
} @else if (isLoading) {
  <app-spinner />
} @else {
  <button (click)="login()">Entrar</button>
}

@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
} @empty {
  <p>Nenhum item encontrado</p>
}

@switch (status) {
  @case ('loading') { <app-spinner /> }
  @case ('success') { <app-content /> }
  @default { <p>Estado desconhecido</p> }
}

<!-- Deferrable views — lazy loading de components no template -->
@defer (on viewport) {
  <app-heavy-chart />
} @placeholder {
  <div class="skeleton">Carregando...</div>
} @loading (minimum 200ms) {
  <app-spinner />
} @error {
  <p>Erro ao carregar</p>
}
```

### 23.6 Angular 18 (Atual)

**Principais Features:**
- ✅ **Zoneless** (experimental) — sem Zone.js
- ✅ Material 3 stable
- ✅ Hydration melhorado (i18n support)
- ✅ Server route config (granular SSR/SSG por rota)
- ✅ Signals forms (developer preview)

```typescript
// Zoneless (sem Zone.js) — performance máxima
// Requer que todo update seja feito via Signals ou markForCheck explícito
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});

// Server route config — SSR ou SSG por rota
export const serverRouteConfig: ServerRoute[] = [
  { path: '/home', mode: RenderMode.Prerender },    // SSG
  { path: '/dashboard', mode: RenderMode.Server },  // SSR
  { path: '/profile', mode: RenderMode.Client },    // CSR
];
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
| NgOptimizedImage | ❌ | ✅ | ✅ | ✅ | ✅ |
| Functional Guards | ❌ | ✅ | ✅ | ✅ | ✅ |
| esbuild (build) | ❌ | Preview | Preview | Stable | Stable |

### 23.8 Migrações Importantes

**Angular 14 → 15:**
- Standalone components de preview pra stable
- Migrar guards para functional guards
- Adicionar `NgOptimizedImage` em imagens críticas

**Angular 15 → 16:**
- **Começar a usar Signals** para state management
- Adicionar `required: true` em @Input obrigatórios
- Substituir pattern `destroy$` por `takeUntilDestroyed()`

**Angular 16 → 17:**
- **Migrar para novo control flow** (@if, @for) — schematics disponíveis
- Considerar usar @defer para lazy loading de components pesados
- Atualizar para o novo builder (esbuild) se ainda não feito

**Angular 17 → 18:**
- Testar zoneless (se quiser performance máxima e tiver Signals em todo lugar)
- Migrar para Material 3
- Configurar `serverRouteConfig` para SSR/SSG granular

### 23.9 Compatibilidade e Atualização

**Regra geral:** Angular mantém compatibilidade com **N-1** (versão anterior).

**Exemplo:**
- Angular 18 é compatível com Angular 17
- Angular 17 é compatível com Angular 16
- Mas Angular 18 pode quebrar coisas do Angular 14

**Processo seguro de atualização:**
```bash
# Sempre use ng update — nunca atualize package.json manualmente
ng update @angular/core @angular/cli

# Atualizar incrementalmente (14 → 15 → 16, não 14 → 18 direto)
# Cheque https://update.angular.io para guia interativo

# Rodar schematics de migração automática
ng update @angular/core@17 --migrate-only
```

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
- Fazer HTTP calls no **constructor** em vez do **ngOnInit**
- Não conhecer o papel do **Zone.js** na change detection
- Não saber quando usar **snapshot vs Observable** em parâmetros de rota

### 24.2 ✅ O que Mencionar ao Refatorar

1. "Vou separar concerns: **model, service, component**"
2. "Preciso de **tipagem forte** (interfaces, evitar `any`)"
3. "Garantir **cleanup** com takeUntilDestroyed/async pipe"
4. "**Error handling** estruturado, não console.log"
5. "**Cache** faz sentido aqui (shareReplay)"
6. "**OnPush** para performance"
7. "**trackBy** no ngFor para não recriar DOM"
8. "**Smart/Dumb** component split para reutilização"
9. "**Validação** customizada e cross-field para UX"

### 24.3 🎯 Perguntas Comuns e Respostas

**P:** "Por que não usar NgRx aqui?"  
**R:** "Overhead desnecessário para feature isolada. State local com Signals/BehaviorSubject é suficiente. NgRx faz sentido quando: múltiplos modules precisam do mesmo estado, há lógica complexa de side-effects (NgRx Effects), ou a equipe precisa de DevTools/time-travel debugging."

**P:** "Como você testaria isso?"  
**R:** "Service: mockar HttpClient com HttpClientTestingModule. Component: mockar Service com jasmine.createSpyObj. Para a UI: usar fixture.nativeElement para asserções no DOM. Cada teste deve ter um propósito claro e ser isolado."

**P:** "Performance com 10k usuários?"  
**R:** "Virtual scrolling (CDK), paginação no backend, OnPush change detection, trackBy no ngFor. Se o componente for pesado, @defer para carregar sob demanda."

**P:** "Diferença entre switchMap e mergeMap?"  
**R:** "switchMap cancela requisição anterior quando chega nova — ideal para search/autocomplete onde só importa o resultado mais recente. mergeMap executa todas em paralelo sem cancelar — ideal para batch uploads onde todos os resultados importam."

**P:** "Como evitar memory leaks?"  
**R:** "Três abordagens: (1) `takeUntilDestroyed()` no Angular 16+ — mais limpo; (2) `async` pipe no template — unsubscribe automático; (3) pattern `destroy$ + takeUntil` para versões anteriores. Prefiro async pipe quando possível, já que é declarativo e não precisa de cleanup manual."

**P:** "Signals vão substituir Observables?"  
**R:** "Não completamente. Signals são melhores para estado síncrono e reatividade granular no template. Observables são melhores para sequências de eventos, operações assíncronas complexas e quando precisar de operadores RxJS. A interoperabilidade com `toSignal`/`toObservable` permite usar os dois onde cada um brilha."

**P:** "Quando usar `providedIn: 'root'` vs `providers: [Service]` no component?"  
**R:** "`providedIn: 'root'` cria um singleton global — ideal para services que guardam estado compartilhado. `providers: [Service]` no component cria uma nova instância por component — ideal quando você quer estado isolado por instância do component, como um FormBuilder local ou um estado de paginação específico."

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
- ✅ **Zone.js** — o que faz e por que existe
- ✅ **Lifecycle hooks** — ordem e quando usar cada um
- ✅ **Interceptors HTTP** — casos de uso
- ✅ **Standalone Components** — o que resolvem

### 24.5 🚀 Durante o Live Coding

**Verbalize seu raciocínio:**
- "Vou criar um service para isolar a lógica de API"
- "Preciso garantir que não vaze memória, então vou usar takeUntilDestroyed"
- "OnPush aqui vai melhorar performance porque os dados chegam do service como Observable"
- "trackBy é essencial para não recriar DOM elements quando a lista atualizar"
- "Vou usar switchMap aqui porque se o usuário digitar rápido, só quero o resultado da última busca"

**Mostre que você pensa em produção:**
- Error handling
- Loading states
- Empty states
- Edge cases (lista vazia, dados nulos)
- Accessibility (aria, roles)
- Performance (bundle size, renders desnecessários)

### 24.6 🧠 Checklist de Code Review

Quando fizer code review ou refatoração ao vivo, verifique:

- [ ] Subscriptions têm cleanup?
- [ ] `any` sendo usado desnecessariamente?
- [ ] HTTP calls no lugar certo (ngOnInit, não constructor)?
- [ ] OnPush pode ser aplicado?
- [ ] trackBy no ngFor?
- [ ] Lógica de negócio no service, não no component?
- [ ] Erros sendo tratados?
- [ ] Loading states considerados?
- [ ] Tipagem adequada nas interfaces?
- [ ] Lazy loading configurado para features grandes?

### 24.7 📚 Estude Antes da Entrevista

1. **RxJS Operators** - Saiba na ponta da língua (especialmente os 4 flatMap operators)
2. **Change Detection** - OnPush vs Default, Zone.js, Signals
3. **Forms** - Reactive Forms (validação customizada, cross-field, async)
4. **Routing** - Guards, Lazy Loading, Resolve, parâmetros
5. **Testing** - Jasmine, mocking, HttpClientTestingModule
6. **Performance** - trackBy, Virtual Scroll, OnPush, @defer
7. **Signals** - Se a vaga menciona Angular 16+
8. **Standalone** - Estrutura sem NgModule (Angular 14+)

---

## 🎉 BOA SORTE NA SUA ENTREVISTA!

Este guia cobre **tudo que você precisa** para uma entrevista de nível Senior. Continue praticando com:

1. ✅ Exercícios de lógica RxJS
2. ✅ Live coding de features completas (CRUD, autenticação, busca)
3. ✅ Refatoração de código ruim (encontre os memory leaks, o `any`, o código sem error handling)
4. ✅ Code reviews (olhar código e achar problemas antes de rodar)

**Lembre-se:** Na entrevista, **verbalizar seu raciocínio** é TÃO importante quanto escrever código correto. Mostre que você pensa em produção, não só em fazer funcionar.

**Referências para aprofundar:**
- [angular.dev](https://angular.dev) — Documentação oficial
- [rxjs.dev](https://rxjs.dev) — Documentação do RxJS
- [update.angular.io](https://update.angular.io) — Guia de migração interativo
- [angular.dev/guide/signals](https://angular.dev/guide/signals) — Guia de Signals

**Próximo passo:** Resolver exercícios de lógica para treinar! 🔥
