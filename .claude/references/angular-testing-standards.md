# Padrões de Teste — Angular + Jasmine/Karma

Contexto sob demanda. Carregue antes de escrever ou revisar specs de tela.

Stack: **Jasmine + Karma** (padrão do Angular CLI), `TestBed`, `provideHttpClientTesting`.
Arquivo ao lado do componente: `produto-list.component.spec.ts`.

---

## Nomenclatura

Português, no mesmo espírito dos testes da API:

```
deve carregar os produtos ao iniciar
deve exibir confirmação antes de excluir
não deve salvar quando o formulário está inválido
deve chamar o endpoint de update quando há id na rota
```

`describe` pelo nome da classe, `describe` aninhado por método/comportamento quando ajudar.

---

## Service — o mais valioso e o mais barato

```ts
import { TestBed } from '@angular/core/testing';
import { provideHttpClient } from '@angular/common/http';
import { HttpTestingController, provideHttpClientTesting } from '@angular/common/http/testing';
import { ProdutoService } from './produto.service';
import { environment } from '../../../../environments/environment';

describe('ProdutoService', () => {
  let service: ProdutoService;
  let http: HttpTestingController;
  const url = `${environment.apiUrl}/api/produtos`;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [provideHttpClient(), provideHttpClientTesting()]
    });
    service = TestBed.inject(ProdutoService);
    http = TestBed.inject(HttpTestingController);
  });

  afterEach(() => http.verify());

  it('deve buscar a lista de produtos', () => {
    const esperado = [{ id: 1, sku: 'ABC', nome: 'Produto', preco: 10, ativo: true }];

    service.getAll().subscribe(produtos => expect(produtos).toEqual(esperado));

    const req = http.expectOne(r => r.url === url);
    expect(req.request.method).toBe('GET');
    req.flush(esperado);
  });

  it('deve enviar POST ao criar produto', () => {
    const novo = { sku: 'ABC', nome: 'Produto', preco: 10, ativo: true };

    service.create(novo).subscribe();

    const req = http.expectOne(url);
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(novo);
    req.flush({ id: 1, ...novo });
  });
});
```

`http.verify()` no `afterEach` é obrigatório — é ele que pega requisição sobrando.

---

## Componente

```ts
describe('ProdutoListComponent', () => {
  let fixture: ComponentFixture<ProdutoListComponent>;
  let component: ProdutoListComponent;
  let serviceSpy: jasmine.SpyObj<ProdutoService>;

  beforeEach(async () => {
    serviceSpy = jasmine.createSpyObj('ProdutoService', ['getAll', 'delete']);
    serviceSpy.getAll.and.returnValue(of([]));

    await TestBed.configureTestingModule({
      imports: [ProdutoListComponent],           // standalone: importa o próprio componente
      providers: [
        provideNoopAnimations(),                 // PO-UI exige animações
        { provide: ProdutoService, useValue: serviceSpy }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(ProdutoListComponent);
    component = fixture.componentInstance;
  });

  it('deve carregar os produtos ao iniciar', () => {
    const esperado = [{ id: 1, sku: 'ABC', nome: 'Produto', preco: 10, ativo: true }];
    serviceSpy.getAll.and.returnValue(of(esperado));

    fixture.detectChanges();                     // dispara ngOnInit

    expect(serviceSpy.getAll).toHaveBeenCalled();
    expect(component.produtos()).toEqual(esperado);
    expect(component.loading()).toBeFalse();
  });

  it('deve desligar o loading quando a busca falha', () => {
    serviceSpy.getAll.and.returnValue(throwError(() => new HttpErrorResponse({ status: 500 })));

    fixture.detectChanges();

    expect(component.loading()).toBeFalse();
  });
});
```

Pontos que sempre escapam:
- Componente standalone entra em **`imports`**, não em `declarations`.
- `provideNoopAnimations()` — sem ele, componente PO com animação quebra o teste.
- `fixture.detectChanges()` é o que dispara `ngOnInit`; sem ele o teste passa sem testar nada.
- Signals: leia com `component.produtos()`, não `component.produtos`.

## Mock dos serviços do PO

```ts
const notificationSpy = jasmine.createSpyObj('PoNotificationService', ['success', 'error', 'warning']);
const dialogSpy = jasmine.createSpyObj('PoDialogService', ['confirm']);

// simular o usuário confirmando o diálogo:
dialogSpy.confirm.and.callFake((args: { confirm: () => void }) => args.confirm());
```

## Interceptor

```ts
it('deve notificar o title do ProblemDetails vindo da API', () => {
  service.getAll().subscribe({ error: () => {} });

  http.expectOne(url).flush(
    { title: 'Produto não encontrado', status: 404 },
    { status: 404, statusText: 'Not Found' }
  );

  expect(notificationSpy.error).toHaveBeenCalledWith('Produto não encontrado');
});
```

Este teste é o que garante que o contrato de erro da API chega correto na tela — vale mais que
qualquer teste de layout.

---

## Cobertura mínima por feature

| Alvo | Cenário | Obrigatório |
|---|---|---|
| Service | um teste por método (URL, verbo, body) | sim |
| Componente lista | carga inicial popula o estado | sim |
| Componente lista | erro na carga desliga o loading | sim |
| Componente lista | exclusão pede confirmação antes de chamar o service | sim |
| Componente form | formulário inválido não chama o service | sim |
| Componente form | modo edição carrega e faz `update`; modo novo faz `create` | sim |
| Interceptor | `ProblemDetails` da API vira notificação | uma vez no projeto |

Teste de layout (existe tal `div`) não conta como cobertura — teste comportamento.

## Comandos

```bash
npm test -- --watch=false --browsers=ChromeHeadless
npm test -- --watch=false --browsers=ChromeHeadless --code-coverage
npm test -- --watch=false --include='**/produto*.spec.ts'
```

Em CI, `--watch=false --browsers=ChromeHeadless` é obrigatório; sem isso o processo trava
esperando browser.
