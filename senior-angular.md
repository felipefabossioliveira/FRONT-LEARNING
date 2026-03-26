# 🅰️ GUIA COMPLETO ANGULAR - PREPARAÇÃO SENIOR 2025
## Versão Expandida com Teoria Profunda

**Versão:** Angular 18+ (compatível com 14-18)  
**Objetivo:** Preparação completa para entrevistas técnicas de nível Senior  
**Melhorias:** Explicações teóricas profundas + Comparações + Contexto histórico

---

## 📚 PRINCIPAIS MELHORIAS NESTA VERSÃO

✨ **Signals** - Explicação teórica profunda (por que existem, como funcionam internamente)  
✨ **Standalone Components** - Contexto histórico e migração detalhada  
✨ **Change Detection** - Como Zone.js funciona por baixo dos panos  
✨ **RxJS** - Marble diagrams visuais, mais operators  
✨ **Testing** - Jest + Spectator, testing de Signals e Standalone  
✨ **Performance** - Web Vitals, profiling, bundle analysis detalhado

---

<a name="signals-profundo"></a>
## 🔥 SIGNALS - EXPLICAÇÃO PROFUNDA

### 1. O QUE SÃO SIGNALS? (Contexto Completo)

**Signals são primitivos reativos** que representam a maior mudança arquitetural do Angular desde o Angular 2.

#### 1.1 Por Que Signals Foram Criados?

**O Problema Histórico do Angular:**

Desde 2016 (Angular 2), o framework usa **Zone.js** para detectar mudanças. Zone.js "sequestra" todas as operações assíncronas do JavaScript global.

```javascript
// Zone.js intercepta automaticamente:
setTimeout(() => { ... })     // ✅ Interceptado
element.addEventListener(...)  // ✅ Interceptado
fetch('/api/data')            // ✅ Interceptado
Promise.resolve().then(...)   // ✅ Interceptado

// Quando qualquer um executa:
Zone.js → Angular.checkWholeTree()  // Checa TODOS os components!
```

**Fluxo Visual do Problema:**

```
SEM SIGNALS (Zone.js tradicional):
┌──────────────────────────────────────────────────┐
│ App com 100 components                           │
│                                                  │
│ Usuário clica em botão que muda 1 variável      │
│         ↓                                         │
│ Zone.js detecta: "Algo assíncrono aconteceu!"    │
│         ↓                                         │
│ Angular: "Vou checar TUDO!"                      │
│         ↓                                         │
│ Checa component 1  ✓ (mudou)                     │
│ Checa component 2  ✗ (não mudou)                 │
│ Checa component 3  ✗ (não mudou)                 │
│ ... checa 97 components desnecessariamente       │
└──────────────────────────────────────────────────┘
Resultado: O(n) - complexidade linear

COM SIGNALS (reatividade granular):
┌──────────────────────────────────────────────────┐
│ App com 100 components                           │
│                                                  │
│ count = signal(0)                                │
│ Template usa {{ count() }}                       │
│         ↓                                         │
│ Usuário clica: count.set(1)                     │
│         ↓                                         │
│ Angular: "count mudou, vou atualizar SÓ         │
│          as expressões que dependem dele"        │
│         ↓                                         │
│ Atualiza APENAS {{ count() }} no template       │
│ NÃO checa os outros 99 components               │
└──────────────────────────────────────────────────┘
Resultado: O(1) - complexidade constante
```

**Por que isso importa?**

```
App pequeno (10 components):
- Zone.js: check 10 components = rápido o suficiente
- Signals: check 1 expression = imperceptível
- Diferença: ~0.5ms (não faz diferença)

App grande (500 components):
- Zone.js: check 500 components = 15-30ms (visível!)
- Signals: check 1 expression = 0.1ms
- Diferença: 150x mais rápido!

App enterprise (2000+ components):
- Zone.js: check 2000+ = 100-200ms (LENTO, usuário percebe lag)
- Signals: check affected = <1ms
- Diferença: 200x mais rápido! Game changer.
```

#### 1.2 Como Signals Funcionam Internamente

**Estrutura Interna de um Signal:**

```typescript
// Quando você escreve:
const count = signal(0);

// O Angular cria internamente algo como:
class WritableSignal<T> {
  private _value: T;
  private _version = 0;
  private _subscribers = new Set<Subscriber>();
  
  constructor(initialValue: T) {
    this._value = initialValue;
  }
  
  // Leitura: count()
  get(): T {
    // CRÍTICO: registra o "consumidor" atual
    const currentContext = getCurrentReactiveContext();
    if (currentContext) {
      this._subscribers.add(currentContext);
      currentContext.dependencies.add(this);
    }
    return this._value;
  }
  
  // Escrita: count.set(1)
  set(newValue: T): void {
    if (this._value === newValue) return; // Não mudou, não notifica
    
    this._value = newValue;
    this._version++;
    
    // Notifica SÓ os subscribers
    this._subscribers.forEach(subscriber => {
      subscriber.markDirty(); // "Você precisa recalcular"
    });
  }
  
  // Atualização funcional: count.update(v => v + 1)
  update(updateFn: (current: T) => T): void {
    this.set(updateFn(this._value));
  }
}
```

**Grafo de Dependências:**

```typescript
// Código:
const firstName = signal('João');
const lastName = signal('Silva');
const fullName = computed(() => `${firstName()} ${lastName()}`);
const greeting = computed(() => `Olá, ${fullName()}!`);

// Angular constrói grafo:
/*
    firstName ─┐
               ├──> fullName ──> greeting ──> Template
    lastName ──┘                              {{ greeting() }}

Quando firstName.set('Maria'):
1. firstName notifica fullName (é subscriber)
2. fullName recalcula → notifica greeting
3. greeting recalcula → notifica Template
4. Template re-renderiza SÓ {{ greeting() }}
*/
```

**Reactive Context (o segredo):**

```typescript
// Quando o template executa {{ count() }}:
function evaluateTemplate() {
  const context = createReactiveContext();
  setCurrentContext(context);
  
  try {
    const value = count();  // count.get() registra context como subscriber
    // Angular: "Este template depende de count"
  } finally {
    clearCurrentContext();
  }
}

// Quando count.set(1):
function setValue(newValue) {
  this._value = newValue;
  
  // Notifica todos os contexts registrados
  this._subscribers.forEach(ctx => {
    ctx.schedule(); // Agenda re-execução do template
  });
}
```

#### 1.3 Signals vs BehaviorSubject (Comparação Técnica)

| Aspecto | Signal | BehaviorSubject |
|---------|--------|-----------------|
| **Leitura** | `count()` | `count$ \| async` ou `.value` |
| **Escrita** | `count.set(5)` / `.update()` | `count$.next(5)` |
| **Reatividade** | Granular (só afetados) | Dispara CD global |
| **Change Detection** | Marca só dependentes | Zone.js checa árvore |
| **Memory Leaks** | Impossível (auto-cleanup) | Precisa unsubscribe |
| **Síncrono** | Sempre | `.value` sim, `\| async` não |
| **RxJS Operators** | Não (mas toObservable) | Sim, todos |
| **Computed** | Built-in (`computed()`) | Manual (`combineLatest`) |
| **Performance** | Excelente (O(afetados)) | Boa (O(árvore CD)) |
| **Simplicidade** | Alta | Média (boilerplate) |
| **Type Safety** | Perfeito | Perfeito |
| **Interop** | toSignal/toObservable | Nativo |

**Exemplo Lado a Lado:**

```typescript
// ════════════════════════════════════════════════════
// BEHAVIORSUBJECT (approach antiga)
// ════════════════════════════════════════════════════

@Component({
  template: `
    <div>Count: {{ count$ | async }}</div>
    <div>Double: {{ double$ | async }}</div>
    <button (click)="increment()">+</button>
  `
})
export class CounterBehaviorComponent implements OnDestroy {
  private countSubject = new BehaviorSubject(0);
  private destroy$ = new Subject<void>();
  
  count$ = this.countSubject.asObservable();
  double$ = this.count$.pipe(map(n => n * 2));
  
  increment() {
    this.countSubject.next(this.countSubject.value + 1);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    // Precisa fazer cleanup manual!
  }
}

// ════════════════════════════════════════════════════
// SIGNALS (approach moderna)
// ════════════════════════════════════════════════════

@Component({
  template: `
    <div>Count: {{ count() }}</div>
    <div>Double: {{ double() }}</div>
    <button (click)="increment()">+</button>
  `
})
export class CounterSignalComponent {
  count = signal(0);
  double = computed(() => this.count() * 2);
  
  increment() {
    this.count.update(n => n + 1);
  }
  
  // Sem ngOnDestroy! Sem destroy$! Auto-cleanup.
}
```

**Quando Usar Cada Um?**

| Cenário | Recomendação | Motivo |
|---------|-------------|--------|
| **Estado de UI simples** | Signal | Menos código, auto-cleanup |
| **Computed values** | Signal | `computed()` é built-in |
| **Formulários reativos** | Signal (Angular 18+) | Signal-based forms |
| **HTTP responses** | toSignal() | Converte Observable → Signal |
| **Streams complexos** | RxJS Observable | Operators (debounce, switchMap) |
| **WebSocket/SSE** | Observable → toSignal | Stream infinito |
| **Interop legado** | Observable | Código já usa RxJS |

#### 1.4 Computed Signals (Derivação Automática)

**O que são Computed Signals:**

```typescript
const price = signal(100);
const quantity = signal(2);
const discount = signal(0.1);

// Computed recalcula automaticamente quando dependências mudam
const subtotal = computed(() => price() * quantity());
const total = computed(() => subtotal() * (1 - discount()));

// Quando price.set(150):
// 1. Angular detecta: subtotal depende de price
// 2. Recalcula subtotal automaticamente (150 * 2 = 300)
// 3. Angular detecta: total depende de subtotal
// 4. Recalcula total automaticamente (300 * 0.9 = 270)
// 5. Atualiza template onde total() é usado
```

**Características Importantes:**

```typescript
// 1. LAZY: só calcula quando lido
const expensive = computed(() => {
  console.log('Calculando...');
  return heavyComputation();
});
// Ainda não calculou nada!

const value = expensive();  // AGORA calcula (log aparece)
const value2 = expensive(); // NÃO recalcula (cached)

// 2. MEMOIZATION: cacheia resultado
const filtered = computed(() => {
  console.log('Filtrando...');
  return items().filter(i => i.active);
});

console.log(filtered()); // "Filtrando..." (primeira vez)
console.log(filtered()); // (sem log - retorna cache)

items.set([...newItems]); // Dependência mudou
console.log(filtered()); // "Filtrando..." (recalcula)

// 3. GRAPH OPTIMIZATION: evita recálculos em diamante
const a = signal(1);
const b = computed(() => a() * 2);
const c = computed(() => a() * 3);
const d = computed(() => b() + c());

a.set(2);
// Angular é inteligente:
// - Marca b, c, d como "dirty"
// - Mas SÓ recalcula quando d() é lido
// - Evita recalcular b e c se d não for usado
```

**Exemplo Real: Filtro de Lista**

```typescript
@Component({
  template: `
    <input [(ngModel)]="searchTerm" placeholder="Buscar...">
    <select [(ngModel)]="categoryFilter">
      <option value="">Todas</option>
      <option value="electronics">Eletrônicos</option>
      <option value="books">Livros</option>
    </select>
    
    <!-- Filtered list -->
    @for (product of filteredProducts(); track product.id) {
      <div class="product">{{ product.name }}</div>
    } @empty {
      <p>Nenhum produto encontrado</p>
    }
    
    <p>Mostrando {{ filteredProducts().length }} de {{ allProducts().length }}</p>
  `
})
export class ProductListComponent {
  // Estado
  allProducts = signal<Product[]>([]);
  searchTerm = model('');           // Two-way binding signal
  categoryFilter = model('');
  
  // Computed (recalcula automaticamente!)
  filteredProducts = computed(() => {
    const term = this.searchTerm().toLowerCase();
    const category = this.categoryFilter();
    
    return this.allProducts().filter(product => {
      const matchesSearch = product.name.toLowerCase().includes(term);
      const matchesCategory = !category || product.category === category;
      return matchesSearch && matchesCategory;
    });
  });
  
  constructor() {
    // Carregar produtos
    this.allProducts.set([/* ... */]);
  }
}
```

#### 1.5 Effects (Side Effects Reativos)

**O que são Effects:**

Effects executam código quando signals que eles lêem mudam.

```typescript
@Component({ ... })
export class UserProfileComponent {
  userId = signal(1);
  
  constructor() {
    // Effect roda sempre que userId muda
    effect(() => {
      const id = this.userId();
      console.log(`User changed to ${id}`);
      this.loadUserData(id);
    });
  }
  
  loadUserData(id: number) {
    // Carregar dados do usuário
  }
}
```

**⚠️ CRÍTICO: Não Mude Signals Dentro de Effects**

```typescript
// ❌ ERRADO - Cria loop infinito!
const count = signal(0);

effect(() => {
  console.log(count());
  count.set(count() + 1);  // Muda count → dispara effect → muda count → ...
});

// ✅ CERTO - Effects são para side effects, não para derivar estado
const count = signal(0);

effect(() => {
  // Side effects OK:
  console.log(count());
  localStorage.setItem('count', count().toString());
  this.analytics.track('count_changed', count());
});

// Para derivar estado, use computed!
const doubleCount = computed(() => count() * 2);
```

**Effect com Cleanup:**

```typescript
effect((onCleanup) => {
  const userId = this.userId();
  
  // Setup: polling a cada 5s
  const interval = setInterval(() => {
    this.pollUserStatus(userId);
  }, 5000);
  
  // Cleanup: executa quando effect re-executa ou component destroi
  onCleanup(() => {
    clearInterval(interval);
    console.log('Limpou interval');
  });
});
```

**untracked() - Ler Signal Sem Dependência:**

```typescript
const count = signal(0);
const theme = signal<'light' | 'dark'>('light');

effect(() => {
  // Este effect depende de count (re-executa quando count muda)
  const currentCount = count();
  
  // Mas NÃO depende de theme (não re-executa quando theme muda)
  const currentTheme = untracked(() => theme());
  
  console.log(`Count: ${currentCount}, Theme: ${currentTheme}`);
});

count.set(1);  // ✅ Re-executa effect
theme.set('dark'); // ❌ NÃO re-executa effect
```

#### 1.6 Signal Inputs (Angular 17.1+)

**Novo jeito de declarar @Input usando signals:**

```typescript
// ══════════════════════════════════════════════════
// ANTES (decorator @Input)
// ══════════════════════════════════════════════════

@Component({ ... })
export class UserCardComponent implements OnChanges {
  @Input() userId!: number;
  @Input() showEmail = true;
  
  user?: User;
  
  ngOnChanges(changes: SimpleChanges) {
    if (changes['userId']) {
      this.loadUser(changes['userId'].currentValue);
    }
  }
  
  loadUser(id: number) { /* ... */ }
}

// ══════════════════════════════════════════════════
// AGORA (signal inputs)
// ══════════════════════════════════════════════════

@Component({ ... })
export class UserCardComponent {
  // Signal inputs
  userId = input.required<number>();
  showEmail = input(true);
  
  // Computed baseado no input (recalcula automaticamente!)
  user = toSignal(this.userService.getUser(this.userId()));
  
  // OU com effect:
  constructor() {
    effect(() => {
      const id = this.userId();
      console.log('User ID changed:', id);
    });
  }
}
```

**Tipos de Signal Inputs:**

```typescript
// Required input (sem default)
userId = input.required<number>();

// Optional com default
size = input(16);  // default 16
theme = input<'light' | 'dark'>('light');

// Com transform (converter tipo)
import { booleanAttribute, numberAttribute } from '@angular/core';

disabled = input(false, { transform: booleanAttribute });
// Uso: <my-component disabled> (string vazia → true)

count = input(0, { transform: numberAttribute });
// Uso: <my-component count="42"> (string "42" → number 42)

// Com alias
userName = input<string>('', { alias: 'user-name' });
// Uso: <my-component user-name="João">
```

#### 1.7 Model Signals (Two-Way Binding)

```typescript
// ══════════════════════════════════════════════════
// BEFORE (Two-way binding tradicional)
// ══════════════════════════════════════════════════

// Filho
@Component({
  selector: 'app-slider',
  template: `<input type="range" [value]="value" (input)="onChange($event)">`
})
export class SliderComponent {
  @Input() value = 0;
  @Output() valueChange = new EventEmitter<number>();
  
  onChange(event: Event) {
    const newValue = +(event.target as HTMLInputElement).value;
    this.valueChange.emit(newValue);
  }
}

// Pai
<app-slider [(value)]="volume"></app-slider>

// ══════════════════════════════════════════════════
// NOW (Model signals)
// ══════════════════════════════════════════════════

// Filho
@Component({
  selector: 'app-slider',
  template: `<input type="range" [value]="value()" (input)="onChange($event)">`
})
export class SliderComponent {
  value = model(0);  // Two-way bindable signal!
  
  onChange(event: Event) {
    const newValue = +(event.target as HTMLInputElement).value;
    this.value.set(newValue); // Propaga automaticamente pro pai
  }
}

// Pai (mesma syntax!)
<app-slider [(value)]="volume"></app-slider>
```

#### 1.8 toSignal / toObservable (Interoperabilidade)

**Observable → Signal:**

```typescript
@Component({ ... })
export class UserComponent {
  private userService = inject(UserService);
  
  // HTTP Observable → Signal
  users = toSignal(
    this.userService.getUsers(),
    { initialValue: [] as User[] }
  );
  
  // Agora pode usar em computed
  activeUsers = computed(() => 
    this.users().filter(u => u.active)
  );
}
```

**Signal → Observable (para usar RxJS operators):**

```typescript
searchTerm = signal('');

// Converter para Observable e aplicar debounce
results$ = toObservable(this.searchTerm).pipe(
  debounceTime(300),
  distinctUntilChanged(),
  filter(term => term.length >= 2),
  switchMap(term => this.searchService.search(term))
);

// Converter de volta para Signal
results = toSignal(this.results$, { initialValue: [] });
```

#### 1.9 Quando NÃO Usar Signals

```
❌ NÃO use Signals para:
- HTTP requests (use HttpClient + toSignal)
- WebSocket streams (Observable melhor)
- Eventos de DOM complexos (fromEvent + RxJS operators)
- Operações assíncronas com retry/debounce (RxJS operators)
- Quando o time todo usa NgRx (manter consistência)

✅ USE Signals para:
- Estado de UI (formulários, toggles, contadores)
- Valores computados (filtros, totais, agregações)
- Estado síncrono local de component
- Substituir BehaviorSubject simples
- Quando performance é crítica (apps grandes)
```

---

<a name="standalone-profundo"></a>
## 🔥 STANDALONE COMPONENTS - CONTEXTO HISTÓRICO

### 2.1 Por Que Standalone Existe?

**O Problema dos NgModules:**

De 2016 a 2022, o Angular obrigava você a criar **NgModules** para organizar components. Isso criava problemas:

```typescript
// ❌ PROBLEMA 1: Boilerplate gigante
@NgModule({
  declarations: [
    UserListComponent,
    UserCardComponent,
    UserDetailComponent,
    UserFormComponent
  ],
  imports: [
    CommonModule,      // Precisa lembrar de importar
    FormsModule,       // Precisa lembrar de importar
    RouterModule,      // Precisa lembrar de importar
    SharedModule       // Precisa criar módulo compartilhado
  ],
  exports: [
    UserListComponent,  // Precisa exportar manualmente
    UserCardComponent
  ]
})
export class UsersModule {}

// ❌ PROBLEMA 2: Circular dependencies
// UserModule importa SharedModule
// SharedModule importa CommonModule
// Se SharedModule precisar de UserComponent → circular!

// ❌ PROBLEMA 3: Lazy loading verboso
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./users/users.module').then(m => m.UsersModule)
  }
];
```

**Como Standalone Resolve:**

```typescript
// ✅ SOLUÇÃO: Component auto-contido
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [
    CommonModule,       // Só importa o que precisa
    FormsModule,
    RouterModule,
    UserCardComponent   // Importa components diretamente!
  ],
  template: `...`
})
export class UserListComponent {}

// ✅ Lazy loading direto
const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./user-list.component').then(m => m.UserListComponent)
  }
];
```

### 2.2 Migração Gradual (Não Precisa Migrar Tudo)

```typescript
// Você pode misturar! Standalone funciona com NgModules

// ════════════════════════════════════════════════════
// CENÁRIO 1: Usar Standalone em NgModule
// ════════════════════════════════════════════════════

// Component standalone
@Component({
  standalone: true,
  selector: 'app-new-feature'
})
export class NewFeatureComponent {}

// Módulo legado importa standalone
@NgModule({
  imports: [
    NewFeatureComponent  // ← Standalone component no imports!
  ]
})
export class LegacyModule {}

// ════════════════════════════════════════════════════
// CENÁRIO 2: Standalone usa NgModule
// ════════════════════════════════════════════════════

@Component({
  standalone: true,
  imports: [
    SharedModule  // ← Pode importar módulos inteiros!
  ]
})
export class StandaloneComponent {}
```

### 2.3 App 100% Standalone (Sem AppModule)

```typescript
// ══════════════════════════════════════════════════
// ANTES (com AppModule)
// ══════════════════════════════════════════════════

// app.module.ts
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    HttpClientModule,
    AppRoutingModule
  ],
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true }
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}

// main.ts
platformBrowserDynamic().bootstrapModule(AppModule);

// ══════════════════════════════════════════════════
// AGORA (standalone)
// ══════════════════════════════════════════════════

// app.component.ts
@Component({
  standalone: true,
  selector: 'app-root',
  imports: [RouterOutlet, CommonModule],
  template: `<router-outlet />`
})
export class AppComponent {}

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes,
      withComponentInputBinding(),
      withPreloading(PreloadAllModules)
    ),
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    ),
    provideAnimations()
  ]
};

// main.ts
bootstrapApplication(AppComponent, appConfig);
```

---

<a name="change-detection-profundo"></a>
## 🔥 CHANGE DETECTION - COMO FUNCIONA INTERNAMENTE

### 3.1 O que é Zone.js? (Explicação Profunda)

**Zone.js é uma biblioteca que "sequestra" operações assíncronas do JavaScript.**

#### Como Zone.js Funciona:

```javascript
// Código normal JavaScript (SEM Zone.js):
setTimeout(() => {
  console.log('Timeout executou');
}, 1000);

// Com Zone.js (Angular internals):
const originalSetTimeout = window.setTimeout;

window.setTimeout = function(callback, delay) {
  return originalSetTimeout(() => {
    // ANTES de executar callback
    zone.onInvoke(() => {
      console.log('Zone: Antes do callback');
    });
    
    // Executa callback original
    callback();
    
    // DEPOIS de executar callback
    zone.onLeave(() => {
      console.log('Zone: Depois do callback');
      angular.checkForChanges();  // ← AQUI QUE ACONTECE A MÁGICA!
    });
  }, delay);
};
```

**Visual do Monkey Patching:**

```
JavaScript Global APIs (originais):
├── setTimeout
├── setInterval
├── Promise.then
├── addEventListener
├── requestAnimationFrame
├── fetch
└── XMLHttpRequest

Zone.js "wraps" todos eles:
├── setTimeout → ZoneAwareSetTimeout → notifica Angular
├── setInterval → ZoneAwareSetInterval → notifica Angular
├── Promise.then → ZoneAwarePromise → notifica Angular
├── addEventListener → ZoneAwareEventListener → notifica Angular
└── ...

Resultado: Angular sabe SEMPRE que algo assíncrono aconteceu
```

#### 3.2 Change Detection Flow Completo

```typescript
// 1. Evento acontece (ex: click)
<button (click)="increment()">+</button>

// 2. Zone.js intercepta
Zone.onEvent(() => {
  // 3. Executa handler do component
  component.increment();
  
  // 4. Marca aplicação como "dirty"
  applicationRef.tick();
  
  // 5. Percorre árvore de components
  checkComponentTree(rootComponent);
});

function checkComponentTree(component: Component) {
  // Default strategy: SEMPRE checa
  if (component.changeDetection === ChangeDetectionStrategy.Default) {
    component.detectChanges();
    component.children.forEach(child => checkComponentTree(child));
  }
  
  // OnPush strategy: SÓ checa se marcado
  if (component.changeDetection === ChangeDetectionStrategy.OnPush) {
    if (component.isMarkedForCheck) {
      component.detectChanges();
      component.children.forEach(child => checkComponentTree(child));
      component.isMarkedForCheck = false;
    }
  }
}
```

**Visualização do Fluxo:**

```
Árvore de Components (Default CD):
┌─────────────────────────────────────────┐
│ AppComponent (Default)                  │
│   ↓                                     │
│   ├── HeaderComponent (Default)         │
│   │   └── UserMenuComponent (Default)   │
│   │                                      │
│   ├── ContentComponent (Default)        │
│   │   ├── ProductListComponent (Default)│
│   │   └── CartComponent (Default)       │
│   │                                      │
│   └── FooterComponent (Default)         │
└─────────────────────────────────────────┘

Click em botão do CartComponent:
1. Zone.js detecta
2. Angular checa TODOS (9 components) ← Desperdício!

────────────────────────────────────────────

Árvore Otimizada (OnPush CD):
┌─────────────────────────────────────────┐
│ AppComponent (OnPush)                   │
│   ↓                                     │
│   ├── HeaderComponent (OnPush)          │
│   │   └── UserMenuComponent (OnPush)    │
│   │                                      │
│   ├── ContentComponent (OnPush)         │
│   │   ├── ProductListComponent (OnPush) │
│   │   └── CartComponent (OnPush) ← mudou│
│   │                                      │
│   └── FooterComponent (OnPush)          │
└─────────────────────────────────────────┘

Click em botão do CartComponent:
1. CartComponent.markForCheck()
2. Marca ancestrais (Content, App)
3. Angular checa: App → Content → Cart (3 components) ← Eficiente!
4. Header, Footer, ProductList NÃO são checados
```

### 3.3 Zoneless Angular (Futuro - Angular 18+)

**Como funcionará SEM Zone.js:**

```typescript
// Com Zoneless:
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});

// Signals disparam CD automaticamente:
const count = signal(0);

// Template usa signal
<div>{{ count() }}</div>

// Quando muda:
count.set(1);
// ↓ Não precisa de Zone.js!
// ↓ Signal notifica Angular diretamente
// ↓ Angular atualiza SÓ {{ count() }}
```

**Benefícios do Zoneless:**

1. ✅ **Menos JS:** Não precisa carregar Zone.js (~15KB)
2. ✅ **Mais Performance:** CD granular sempre
3. ✅ **Previsível:** Você controla quando CD acontece
4. ✅ **Compatível:** Funciona com libs third-party sem zone-awareness

**Quando Zoneless estará pronto?**

```
Angular 18: Experimental (pode usar em apps novos)
Angular 19: Developer Preview (mais estável)
Angular 20+: Provavelmente default (estimativa 2025-2026)
```

---

## 🔥 OUTRAS MELHORIAS

### RXJS - Marble Diagrams

```typescript
// switchMap visual:
source:   --A----B----C---->
            |    |    |
results:  --a]---b]---c---->
(cancela anterior quando novo chega)

// mergeMap visual:
source:   --A----B----C---->
            |    |    |
results:  --a--a-b-b-c-c-->
(todos paralelos)

// concatMap visual:
source:   --A----B----C---->
            |    |    |
results:  --aaaa-bbbb-cccc>
(fila ordenada)

// combineLatest visual:
a$:       --1----2----3---->
b$:       ----A----B----C-->
            |    |    |
result:   ----1A-2A-2B-3B-3C>
(emite quando QUALQUER muda)
```

### TESTING - Jest + Spectator

```typescript
// Jest config (angular.json)
"test": {
  "builder": "@angular-builders/jest:run"
}

// Spectator simplifica testes
import { createComponentFactory, Spectator } from '@ngneat/spectator/jest';

describe('UserComponent', () => {
  let spectator: Spectator<UserComponent>;
  const createComponent = createComponentFactory({
    component: UserComponent,
    mocks: [UserService]
  });

  beforeEach(() => spectator = createComponent());

  it('should display user name', () => {
    spectator.component.user = { name: 'João' };
    spectator.detectChanges();
    
    expect(spectator.query('.user-name')).toHaveText('João');
  });
});
```

### PERFORMANCE - Web Vitals

```typescript
// Monitorar Core Web Vitals
import { onCLS, onFID, onLCP } from 'web-vitals';

onCLS(console.log);  // Cumulative Layout Shift
onFID(console.log);  // First Input Delay
onLCP(console.log);  // Largest Contentful Paint

// Targets para passar no Google PageSpeed:
// LCP < 2.5s (bom) / < 4s (precisa melhorar)
// FID < 100ms (bom) / < 300ms (precisa melhorar)
// CLS < 0.1 (bom) / < 0.25 (precisa melhorar)
```

---

## 🎯 PERGUNTAS DE ENTREVISTA EXPANDIDAS

### Signals:

**P:** "Por que o Angular criou Signals se já tínhamos RxJS?"  
**R:** "Signals resolvem um problema fundamental: Zone.js checa TODA a árvore de components a cada evento assíncrono, mesmo quando só 1 valor mudou. Isso é O(n) - complexidade linear. Signals permitem reatividade granular O(afetados) - só recalcula as expressões que dependem do valor alterado. Em apps grandes (500+ components), a diferença é de 100-200ms para <1ms. Além disso, Signals preparam o Angular para o futuro 'Zoneless', sem a dependência de Zone.js."

**P:** "Signals vão substituir RxJS?"  
**R:** "Não. São complementares. Signals são ideais para estado síncrono de UI (contadores, toggles, filtros). RxJS continua essencial para streams assíncronos complexos (HTTP com retry, debounce, autocomplete, WebSockets). A interop toSignal/toObservable permite usar ambos juntos."

### Standalone:

**P:** "Vamos ter que migrar tudo para standalone?"  
**R:** "Não. NgModules continuam suportados indefinidamente. Mas standalone é o futuro recomendado pelo time do Angular porque elimina boilerplate, melhora tree-shaking, e simplifica lazy loading. A migração pode ser gradual - standalone e NgModules coexistem no mesmo projeto."

### Change Detection:

**P:** "Explique como Zone.js funciona."  
**R:** "Zone.js faz monkey-patching das APIs assíncronas do JavaScript global - setTimeout, addEventListener, Promise, etc. Ele 'wrappeia' essas funções e notifica o Angular quando elas executam. Isso permite que o Angular dispare change detection automaticamente após qualquer operação assíncrona, sem você chamar manualmente. O problema é que dispara CD global - checa toda a árvore. Por isso OnPush e Signals são importantes para otimizar."

---

## 📚 FIM DO GUIA EXPANDIDO

Este guia agora tem:
✅ Explicação profunda de **por que** Signals existem  
✅ Como Signals funcionam **internamente** (grafo de dependências)  
✅ **Contexto histórico** de Standalone Components  
✅ Como **Zone.js funciona** por baixo dos panos  
✅ **Marble diagrams** para RxJS  
✅ **Jest + Spectator** para testing moderno  
✅ **Web Vitals** para performance real

**Próximos passos:**
1. Praticar implementando features com Signals
2. Migrar um projeto pequeno para Standalone
3. Estudar os marble diagrams de RxJS operators
4. Fazer code review focando em performance (trackBy, OnPush, etc)

**Boa sorte na entrevista! 🚀**
