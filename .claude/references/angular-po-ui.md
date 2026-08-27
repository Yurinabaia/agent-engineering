# Angular 20 + PO-UI — Referência de Implementação

Contexto sob demanda. **Carregue antes de criar ou alterar qualquer tela.**
Alvo: Angular 20 (standalone, `inject()`, signals, novo control flow) + PO-UI (TOTVS).

> **Verifique a versão antes de usar API menos comum:** `cat package.json | grep @po-ui` e, na
> dúvida, confira `node_modules/@po-ui/ng-components` ou <https://po-ui.io>. Os componentes deste
> documento são estáveis há muitas versões, mas *inputs* novos variam.

---

## 1. Estrutura de pastas

Mesma filosofia da API: **infra estável na borda, uma pasta vertical por feature**.

```
src/app/
  core/                          # infra transversal — criada uma vez, quase nunca muda
    interceptors/
      auth.interceptor.ts        #   injeta o Bearer token
      error.interceptor.ts       #   ProblemDetails da API → PoNotificationService
    guards/auth.guard.ts
    services/                    #   config, storage, sessão
    models/                      #   tipos globais (ProblemDetails, PagedResponse)
  shared/                        # reutilizável por 3+ features
    components/ pipes/ utils/
  features/                      # UMA PASTA POR FEATURE
    produtos/
      models/produto.model.ts            # interface espelhando o DTO da API
      services/produto.service.ts        # HttpClient tipado
      pages/
        produto-list/
          produto-list.component.ts|html|css|spec.ts
        produto-form/
          produto-form.component.ts|html|css|spec.ts
      produtos.routes.ts                 # rotas lazy da feature
  app.config.ts                  # providers da aplicação
  app.routes.ts                  # rotas raiz (lazy por feature)
  app.ts                         # shell: po-toolbar + po-menu + router-outlet
environments/
```

**Regra de dependência (espelha a da API):**
`features/*` → `shared/` → `core/`. Nunca o contrário, e **nunca uma feature importando de outra**.
Precisa compartilhar? Sobe para `shared/`.

### Nomenclatura de arquivo

Este projeto usa o sufixo explícito `.component.ts` + classe `ProdutoListComponent`.
O CLI do Angular 20 **omite** o sufixo por padrão — gere assim:

```bash
ng g c features/produtos/pages/produto-list --type=component --style=css
ng g s features/produtos/services/produto --type=service
```

---

## 2. Bootstrap — `app.config.ts`

```ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideAnimations } from '@angular/platform-browser/animations';
import { routes } from './app.routes';
import { authInterceptor } from './core/interceptors/auth.interceptor';
import { errorInterceptor } from './core/interceptors/error.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor, errorInterceptor])),
    provideAnimations() // OBRIGATÓRIO para o PO-UI (modal, notification, loading)
  ]
};
```

O tema entra no `angular.json`:

```json
"styles": [
  "node_modules/@po-ui/style/css/po-theme-default.min.css",
  "src/styles.css"
]
```

Esquecer `provideAnimations()` faz modal e notification simplesmente não aparecerem — sem erro
no console. É a primeira coisa a conferir quando "o PO não abre".

---

## 3. Model — espelha o DTO da API

`features/produtos/models/produto.model.ts`

```ts
export interface Produto {
  id: number;
  sku: string;
  nome: string;
  preco: number;
  ativo: boolean;
}

export type ProdutoCreate = Omit<Produto, 'id'>;
```

- Um campo por campo do DTO C#, **mesmo nome em camelCase** (o ASP.NET serializa assim por padrão).
- `interface`, não `class`. Sem lógica.
- Campos de auditoria (`recCreatedBy`, `recCreatedOn`…) só entram se a tela os exibir.

---

## 4. Service — o único que fala HTTP

`features/produtos/services/produto.service.ts`

```ts
import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '../../../../environments/environment';
import { Produto, ProdutoCreate } from '../models/produto.model';

@Injectable({ providedIn: 'root' })
export class ProdutoService {
  private readonly http = inject(HttpClient);
  private readonly url = `${environment.apiUrl}/api/produtos`;

  getAll(incluirInativos = false): Observable<Produto[]> {
    const params = new HttpParams().set('incluirInativos', incluirInativos);
    return this.http.get<Produto[]>(this.url, { params });
  }

  getById(id: number): Observable<Produto> {
    return this.http.get<Produto>(`${this.url}/${id}`);
  }

  create(produto: ProdutoCreate): Observable<Produto> {
    return this.http.post<Produto>(this.url, produto);
  }

  update(id: number, produto: Produto): Observable<Produto> {
    return this.http.put<Produto>(`${this.url}/${id}`, produto);
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.url}/${id}`);
  }
}
```

Regras:
- `inject()`, não construtor. `providedIn: 'root'`.
- **Nenhum `subscribe` aqui** — o service devolve `Observable`, quem assina é o componente.
- Nada de `catchError` engolindo erro: o `errorInterceptor` é quem notifica.
- A URL sai de `environment`, nunca hard-coded no componente.

---

## 5. Interceptors — onde o contrato de erro da API vira UI

`core/interceptors/error.interceptor.ts`

```ts
import { HttpErrorResponse, HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { PoNotificationService } from '@po-ui/ng-components';
import { catchError, throwError } from 'rxjs';

interface ProblemDetails {
  title?: string;
  status?: number;
  detail?: string;
  log?: { id: string };
}

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const notification = inject(PoNotificationService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      const problem = error.error as ProblemDetails | null;
      const mensagem =
        problem?.title ??
        (error.status === 0
          ? 'Não foi possível conectar ao servidor.'
          : 'Ocorreu um erro inesperado.');

      notification.error(problem?.log?.id ? `${mensagem} (log: ${problem.log.id})` : mensagem);

      return throwError(() => error);
    })
  );
};
```

Este é o par exato do `ErrorHandlingFilterAttribute` da API: lá a exceção vira `ProblemDetails`
(RFC 7807) com `title` e `status`; aqui o `title` vira a mensagem do toast. **Um contrato só,
ponta a ponta.**

`core/interceptors/auth.interceptor.ts` segue o mesmo formato funcional, adicionando
`Authorization: Bearer <token>`.

---

## 6. Rotas

`features/produtos/produtos.routes.ts`

```ts
import { Routes } from '@angular/router';

export const PRODUTOS_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./pages/produto-list/produto-list.component').then(m => m.ProdutoListComponent)
  },
  {
    path: 'novo',
    loadComponent: () =>
      import('./pages/produto-form/produto-form.component').then(m => m.ProdutoFormComponent)
  },
  {
    path: ':id',
    loadComponent: () =>
      import('./pages/produto-form/produto-form.component').then(m => m.ProdutoFormComponent)
  }
];
```

`app.routes.ts`

```ts
export const routes: Routes = [
  { path: '', redirectTo: 'produtos', pathMatch: 'full' },
  {
    path: 'produtos',
    canActivate: [authGuard],
    loadChildren: () => import('./features/produtos/produtos.routes').then(m => m.PRODUTOS_ROUTES)
  }
];
```

Toda feature é **lazy**. Rota em kebab-case, igual à rota da API.

---

## 7. Tela de listagem — `po-page-list` + `po-table`

`produto-list.component.ts`

```ts
import { ChangeDetectionStrategy, Component, OnInit, inject, signal } from '@angular/core';
import { Router } from '@angular/router';
import { PoPageModule, PoTableModule } from '@po-ui/ng-components';
import {
  PoDialogService, PoNotificationService, PoPageAction, PoPageFilter,
  PoTableAction, PoTableColumn
} from '@po-ui/ng-components';
import { Produto } from '../../models/produto.model';
import { ProdutoService } from '../../services/produto.service';

@Component({
  selector: 'app-produto-list',
  standalone: true,
  imports: [PoPageModule, PoTableModule],
  templateUrl: './produto-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ProdutoListComponent implements OnInit {
  private readonly service = inject(ProdutoService);
  private readonly router = inject(Router);
  private readonly dialog = inject(PoDialogService);
  private readonly notification = inject(PoNotificationService);

  readonly produtos = signal<Produto[]>([]);
  readonly loading = signal(false);

  readonly columns: PoTableColumn[] = [
    { property: 'sku', label: 'SKU', width: '15%' },
    { property: 'nome', label: 'Nome' },
    { property: 'preco', label: 'Preço', type: 'currency', format: 'BRL', width: '15%' },
    { property: 'ativo', label: 'Situação', type: 'boolean',
      boolean: { trueLabel: 'Ativo', falseLabel: 'Inativo' }, width: '12%' }
  ];

  readonly pageActions: PoPageAction[] = [
    { label: 'Novo produto', icon: 'po-icon-plus', action: () => this.router.navigate(['/produtos/novo']) }
  ];

  readonly tableActions: PoTableAction[] = [
    { label: 'Editar', icon: 'po-icon-edit', action: (p: Produto) => this.editar(p) },
    { label: 'Excluir', icon: 'po-icon-delete', type: 'danger', action: (p: Produto) => this.confirmarExclusao(p) }
  ];

  readonly filter: PoPageFilter = {
    action: (termo: string) => this.filtrar(termo),
    placeholder: 'Buscar por SKU ou nome'
  };

  ngOnInit(): void {
    this.carregar();
  }

  private carregar(): void {
    this.loading.set(true);
    this.service.getAll().subscribe({
      next: produtos => { this.produtos.set(produtos); this.loading.set(false); },
      error: () => this.loading.set(false)   // o interceptor já notificou
    });
  }

  private editar(produto: Produto): void {
    this.router.navigate(['/produtos', produto.id]);
  }

  private confirmarExclusao(produto: Produto): void {
    this.dialog.confirm({
      title: 'Excluir produto',
      message: `Deseja excluir o produto ${produto.sku}?`,
      confirm: () => this.excluir(produto)
    });
  }

  private excluir(produto: Produto): void {
    this.service.delete(produto.id).subscribe({
      next: () => {
        this.notification.success('Produto excluído com sucesso.');
        this.carregar();
      }
    });
  }

  private filtrar(termo: string): void { /* ... */ }
}
```

`produto-list.component.html`

```html
<po-page-list p-title="Produtos" [p-actions]="pageActions" [p-filter]="filter">
  <po-table
    [p-columns]="columns"
    [p-items]="produtos()"
    [p-actions]="tableActions"
    [p-loading]="loading()"
    p-striped>
  </po-table>
</po-page-list>
```

Pontos obrigatórios:
- `standalone: true` + `imports` com os **módulos** do PO (`PoPageModule`, `PoTableModule`,
  `PoFieldModule`…). Importar `PoModule` inteiro funciona, mas infla o bundle.
- `ChangeDetectionStrategy.OnPush` sempre; estado em `signal()`.
- `error:` no subscribe **não** exibe toast — o interceptor já fez isso. Aqui só se desliga o loading.
- Ação destrutiva **sempre** passa por `PoDialogService.confirm`.
- Sucesso de escrita **sempre** confirma com `PoNotificationService.success`.

---

## 8. Tela de formulário — `po-page-edit` + reactive forms

`produto-form.component.ts` (essencial)

```ts
@Component({
  selector: 'app-produto-form',
  standalone: true,
  imports: [ReactiveFormsModule, PoPageModule, PoFieldModule],
  templateUrl: './produto-form.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ProdutoFormComponent implements OnInit {
  private readonly fb = inject(FormBuilder);
  private readonly service = inject(ProdutoService);
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);
  private readonly notification = inject(PoNotificationService);

  readonly id = signal<number | null>(null);
  readonly salvando = signal(false);

  readonly form = this.fb.nonNullable.group({
    sku: ['', [Validators.required, Validators.maxLength(30)]],
    nome: ['', [Validators.required, Validators.maxLength(200)]],
    preco: [0, [Validators.required, Validators.min(0.01)]],
    ativo: [true]
  });

  ngOnInit(): void {
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.id.set(Number(id));
      this.service.getById(Number(id)).subscribe(p => this.form.patchValue(p));
    }
  }

  salvar(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }
    this.salvando.set(true);
    const valor = this.form.getRawValue();
    const req = this.id()
      ? this.service.update(this.id()!, { id: this.id()!, ...valor })
      : this.service.create(valor);

    req.subscribe({
      next: () => {
        this.notification.success('Produto salvo com sucesso.');
        this.router.navigate(['/produtos']);
      },
      error: () => this.salvando.set(false)
    });
  }

  cancelar(): void {
    this.router.navigate(['/produtos']);
  }
}
```

```html
<po-page-edit
  [p-title]="id() ? 'Editar produto' : 'Novo produto'"
  [p-disable-submit]="form.invalid || salvando()"
  (p-save)="salvar()"
  (p-cancel)="cancelar()">

  <form [formGroup]="form" class="po-row">
    <po-input class="po-md-4" formControlName="sku" p-label="SKU" p-required p-maxlength="30"></po-input>
    <po-input class="po-md-8" formControlName="nome" p-label="Nome" p-required p-maxlength="200"></po-input>
    <po-decimal class="po-md-4" formControlName="preco" p-label="Preço" p-required
                p-decimals-length="2"></po-decimal>
    <po-switch class="po-md-4" formControlName="ativo" p-label="Situação"
               p-label-on="Ativo" p-label-off="Inativo"></po-switch>
  </form>
</po-page-edit>
```

Regras:
- **Reactive forms**, `fb.nonNullable.group`. Nada de `ngModel`.
- As validações espelham as regras da API (`MaxLength`, obrigatoriedade, `preco > 0`) — a
  validação no cliente é conveniência; a **fonte da verdade continua sendo a API**, e o erro dela
  chega pelo interceptor.
- Grid do PO: `po-row` + `po-md-*`/`po-lg-*` (12 colunas). Não use CSS próprio para layout de
  formulário.
- `p-required` no componente PO desenha o asterisco; o `Validators.required` é que bloqueia.

---

## 9. Catálogo rápido — qual componente PO usar

| Precisa de | Componente |
|---|---|
| Página de listagem com filtro e ações | `po-page-list` |
| Página de cadastro/edição | `po-page-edit` |
| Página simples com ações | `po-page-default` |
| Página de visualização | `po-page-detail` |
| Grade de dados | `po-table` |
| Texto | `po-input` |
| Número inteiro / decimal | `po-number` / `po-decimal` |
| Moeda | `po-decimal` + coluna `type: 'currency'` na tabela |
| Lista fixa de opções | `po-select` |
| Lista grande / busca remota | `po-combo` (`p-filter-service`) |
| Busca em outra entidade | `po-lookup` |
| Sim/não | `po-switch` |
| Data | `po-datepicker` / `po-datepicker-range` |
| Texto longo | `po-textarea` |
| Upload | `po-upload` |
| Diálogo de confirmação | `PoDialogService.confirm()` |
| Toast de feedback | `PoNotificationService` |
| Janela modal | `po-modal` + `PoModalComponent` |
| Bloqueio de tela | `po-loading-overlay` |
| Navegação | `po-toolbar` + `po-menu` (`PoMenuItem[]`) |
| Trilha | `p-breadcrumb` (`PoBreadcrumb`) |
| Abas | `po-tabs` |
| Indicador/status | `po-tag` |

### `po-page-dynamic-table` / `po-page-dynamic-edit` — cuidado

Os templates dinâmicos de `@po-ui/ng-templates` geram a tela inteira a partir de
`p-fields` + `p-service-api`, **mas exigem o contrato de API do PO**: a listagem precisa
responder `{ items: [...], hasNext: boolean }`.

A API deste projeto responde array puro ou `PagedResponseContract` (`data`, `page`, `pageSize`,
`totalRecords`). Então:

- **Padrão:** use `po-page-list` + `po-table` + `po-page-edit` com service explícito (controle
  total, sem surpresa).
- Se quiser o dinâmico, escolha conscientemente: adaptar a resposta no BFF/endpoint, ou expor um
  endpoint compatível. Não "meio adapte" — é fonte garantida de bug silencioso de paginação.

---

## 10. Estado, RxJS e vazamentos

- Estado local de tela: `signal()` + `computed()`.
- Assinatura em `ngOnInit` que vive enquanto a tela vive: `takeUntilDestroyed(inject(DestroyRef))`.
- ❌ Nunca `subscribe` dentro de `subscribe` — use `switchMap`/`forkJoin`.
- ❌ Nunca `subscribe` no template ou em getter. Use `async` pipe ou signal.
- Chamada disparada por digitação: `debounceTime(300)` + `distinctUntilChanged()` + `switchMap`.

## 11. Novo control flow (Angular 17+)

```html
@if (loading()) {
  <po-loading-overlay></po-loading-overlay>
}

@for (item of itens(); track item.id) {
  <po-widget [p-title]="item.nome"></po-widget>
} @empty {
  <p>Nenhum registro encontrado.</p>
}
```

`*ngIf`/`*ngFor` em código novo é desvio de padrão. `track` é obrigatório no `@for`.

## 12. Comandos

```bash
npm install
npm start                       # ng serve
npm run build                   # build de produção
npm test -- --watch=false --browsers=ChromeHeadless
npx ng lint
npx tsc --noEmit                # checagem de tipos isolada
```

## 13. Checklist de revisão da feature Angular

- [ ] Pasta `features/<feature>/` com `models/`, `services/`, `pages/`, `*.routes.ts`
- [ ] Model espelhando o DTO da API, em camelCase
- [ ] Service com `inject(HttpClient)`, sem `subscribe`, URL vinda de `environment`
- [ ] Rota lazy registrada em `app.routes.ts`, em kebab-case
- [ ] Componentes `standalone` com `imports` granulares dos módulos PO
- [ ] `ChangeDetectionStrategy.OnPush` + `signal()` para estado
- [ ] Reactive forms com validações espelhando as regras da API
- [ ] Ação destrutiva com `PoDialogService.confirm`
- [ ] Sucesso com `PoNotificationService.success`; erro **só** pelo interceptor
- [ ] Novo control flow (`@if`/`@for` com `track`)
- [ ] Sem `any`, sem `subscribe` aninhado, sem vazamento de subscription
- [ ] `npm run build`, `npx tsc --noEmit` e `npm test` verdes
