## Fundamentos do Blazor Server

Projeto seguindo o curso do [Balta](https://github.com/balta-io/2821).

O Blazor habilita a execução do CSharp no lado do cliente, sem a necessidade de usar
Javascript.

Para criar um App use:

```csharp
dotnet new blazorserver -o BlazingShop -f net7.0
```

Ou utilizando o Visual Studio, escolha a opção aplicativo Blazor Server

## Estrutura do App

wwwroot   | Arquivos estáticos
Pages     | Paginas Razor
Shared    | Layouts
_Imports  | Importação global de componentes
App.razor | Rotas

## Diretivas

A diferença entre um componente e uma pagina é a diretiva @page

## _Host

O arquivo _Host.cshtml é um arquivo Razor que serve como ponto de entrada para uma
aplicação Blazor Server. Ele é responsável por carregar os componentes e scripts necessários
para executar a aplicação.

O arquivo _Host.cshtml é semelhante ao arquivo index.html em uma aplicação web tradicional.
Ele contém o HTML, CSS e JavaScript necessários para renderizar a aplicação.

No arquivo _Host.cshtml, você pode especificar o layout da aplicação, especificar estilos
e scripts a serem incluídos, e definir o componente raiz da aplicação.

## Router

Qual pagina ou componente será renderizado. O arquivo principal é o App.razor

## Fluxo de execução

Quando executamos a aplicação Blazor, seguimos a seguinte ordem:

1 - Program chama o **app.MapFallbackToPage("/_Host")**.
2 - O **_Host** carrega a rota raiz "**/**" e renderiza o html.
3 - O **<component type="typeof(HeadOutlet)".../>** renderiza o titulo da pagina Index.razor
4 - Agora o **<component type="typeof(App)".../>** renderiza o App.razor

## SPA em ação

Crie uma pagina com a extensão .razor e defina a rota.

```csharp
@page "/products"
```

Basta atualizar agora no NavMenu, chamando a pagina criada com o nome da sua rota.

```csharp
<NavLink class="nav-link" href="products">
    <span class="oi oi-plus" aria-hidden="true"></span> Products
</NavLink>
```

## Gerando os Models

A principal diferença do Razor pro Blazor é que o codigo já fica embutido dentro da
pagina .razor. No Razor, são criados 2 arquivos .cshtml.

Crie uma pasta Models e crie os modelos para Categoria e Produto.

## Listando dados

Com as Models criadas, adicione ao _Imports.razor o namespace **@using BlazingShop.Models**.

Agora na pagina Products.razor, adicione o codigo para instanciar uma lista de produtos
e depois faça a exibição na tela.

```csharp
@page "/products"

<PageTitle>Products</PageTitle>
<h1>Products</h1>
<ul>
  @foreach (var item in _products)
  {
    <li>@item.Title</li>
  }
</ul>

@code {
  private List<Product> _products = new();

  protected override async Task OnInitializedAsync()
  {
    await Task.Delay(500);
    for (int i = 0; i < 25; i++)
    {
      _products.Add(new Product(i + 1, $"Produto {i}", i * 13.75M, 1));
    }
  }
}
```

## Async e Await

Utilizar os métodos asincronos evitam que a navegação fique travada, esperando
carregar alguma soliticação.

## Components

Criar componentes torna tudo mais organizado em nossa aplicação. O componente não tem
a diretiva @page no inicio do codigo do arquivo. Só usa o código direto.

Crie um novo componente Shared -> ProductListComponent.razor. Agora mova o codigo.

```csharp
<ul>
  @foreach (var item in _products)
  {
    <li>@item.Title</li>
  }
</ul>

@code {
  private List<Product> _products = new();

  protected override async Task OnInitializedAsync()
  {
    await Task.Delay(500);
    for (int i = 0; i < 25; i++)
    {
      _products.Add(new Product(i + 1, $"Produto {i}", i * 13.75M, 1));
    }
  }
}

// Agora, a pagina Products.razor chama apenas o componente ProductListComponent.
@page "/products"

<PageTitle>Products</PageTitle>
<h1>Products</h1>
<ProductListComponent></ProductListComponent>
```

## Layouts

A unica diferença pra um componente é utilizar o tagHelper para herdar o layout.
Para aplicar um layout diferença a uma pagina, utilize o @layout e chame o layout personalizado.

```csharp
// Exemplo em uma pagina de Login (Login.razor)

@page "/login"
@layout HeadlessLayout

<h1>Login</h1>
```

Agora o que for definido no arquivo HeadlessLayout.razor será aplicado nessa pagina.

## Passando parametros nos componentes

Para utilizar parametros como paginação em listas, faça dessa forma:

```csharp
// Exemplo no componente ProductListComponent
@code {
  [Parameter]
  public int Skip { get; set; } = 0;

  [Parameter]
  public int Take { get; set; } = 10;

  private List<Product> _products = new();
  ...

// Exemplo na pagina Product.razor
@page "/products/{skip:int?}/{take:int?}"

<PageTitle>Products</PageTitle>
<h1>Products</h1>
<ProductListComponent Skip="@Skip" Take="@Take"></ProductListComponent>

@code {
  [Parameter]
  public int Skip { get; set; } = 0;

  [Parameter]
  public int Take { get; set; } = 10;
}
```

## CRUD com EF

Adicione o pacote do EntityFramework na aplicação. Em seguida, adicione as
Annotations nas models.

## Configurando o EF

Agora crie uma classe para representar o contexto do banco ou AppDbContext.

```csharp
public class AppDbContext : DbContext
{
  public AppDbContext(DbContextOptions<AppDbContext> options)
      : base(options)
  {
  }

  public DbSet<Product> Products { get; set; } = null!;
  public DbSet<Category> Categories { get; set; } = null!;
}
```

Adicione a string de conexão a inicialização da aplicação em Program.cs. Essa configuração
irá recuperar os dados do appssettings.json

```csharp
// Program.cs
var cnnStr = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<AppDbContext>(x => x.UseSqlServer(cnnStr));

// appsettings.json
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=blzshop;User ID=sa;Password=1q2w3e4r@#$;Trusted_Connection=False; TrustServerCertificate=True;"
  },
```

## Migrations

Instale os pacotes SqlServer e Design do EF. 

**dotnet add package Microsoft.EntityFrameworkCore.SqlServer**
**dotnet add package Microsoft.EntityFrameworkCore.Design**

Compile a aplicação e crie uma migration.

**dotnet ef migrations add v1**

Agora execute o **dotnet ef database update**

## Edit Form

Como o Blazor aceita html e css, utilize uma tag propria do blazor EditForm para facilitar
o binding (vinculo) com as entidades.

```csharp
// Pagina Create.razor
@page "/products/create"

<h1>New Product</h1>

// Aqui fizemos o vinculo com a Entidade Product
<EditForm Model ="_model"></EditForm>

// Codigo Csharp fica sempre dentro do @code
@code{
    Product _model = new();
}
```

Agora, adicionamos o método (ação) executar as validações no envio do formulário.

```csharp
// Passamos apenas a referencia no OnValidSubmit. Utilize parenteses apenas quando for necessário
// invocar o método.
<EditForm Model ="_model" OnValidSubmit="@HandleSubmitAsync"></EditForm>

@code{
    Product _model = new();

    async Task HandleSubmitAsync(){}
}
```

Dentro do EditForm, vamos criar os campos do formulário, ligando cada campo com os campos da Model.
O InputText é mais uma tag do Blazor que facilita o vinculo junto ao @bind-value.
Já o id é o vinculo para referencia no html.

```csharp
<EditForm Model ="_model" OnValidSubmit="@HandleSubmitAsync">

    <div class="mb-3">
        <label for="Title" class="form-label">Title</label>
        <InputText class="form-control" type="text" id="Title" @bind-Value="_model.Title"/>
    </div>

</EditForm>
```

## Select

Para selecionar varios itens em uma lista, utilize a tag do Blazor InputSelect.

Crie uma lista para representar as categorias.

```csharp
@code{
    Product _model = new();
    List<Category> _categories = new();

    async Task HandleSubmitAsync(){}
    }
```

Agora adicione a interação dentro do InputSelect para retornar os itens.

```csharp
<div class="mb-3">
    <label for="CategoryId" class="form-label">CategoryId</label>

    <InputSelect
        id="CategoryId"
        @bind-Value="_model.CategoryId"
        class="form-control">

        @foreach (var category in _categories)
        {
            <option value="@category.Id">
                @category.Title
            </option>
        }
    </InputSelect>
</div>
```

## Preenchendo as Categorias

Obs: Para testes, faça alguns inserts no Banco:

```sql
INSERT INTO Categories VALUES ('Carros');
INSERT INTO Categories VALUES ('Motos');
INSERT INTO Categories VALUES ('Caminhões');
```

Para retornar os itens da categoria do banco de dados, faça a injeção
de depêndencia usando o @inject AppDbContext _context.

Obs: Adicione ao _Imports a importação do @using BlazingShop.Data.

```csharp
@page "/products/create"

@inject AppDbContext _context

<h1>New Product</h1>
...

// Agora faça a query ao inicializar a pagina
// Obs: Adicione ao _Imports a importação do @using Microsoft.EntityFrameworkCore
@code{
  Product _model = new();
  List<Category> _categories = new();

  protected override async Task OnInitializedAsync()
  {
    _categories = await _context
        .Categories
        .AsNoTracking() // Melhores praticas para consultas somente leitura
        .ToListAsync();
  }
```

## Validando Formulários

Adicione as tags no inicio do form. Elas iram recuperar as definições da Model.

```csharp
    <DataAnnotationsValidator/>
    <ValidationSummary/>
```
Para criar uma mensagem de erro personalizada, siga os passos abaixo.

```csharp
// Crie uma variavel para referencia
@code 
{
...
  string? _errorMessage = null;
...
}

// Teste a condição antes do botão Save
@if (!string.IsNullOrEmpty(_errorMessage))
{
<div class="alert alert-danger" role="alert">
    @_errorMessage
</div>
}

// Exibe na tela ao enviar o formulario, usando o metodo HandleSubmit
async Task HandleSubmitAsync()
{
_errorMessage = "Falha ao persistir os dados";
}
```

## Salvando o produto

Para indicar qual é a 1ª categoria selecionada, adicione o CategoryId na posição 0.

```csharp
protected override async Task OnInitializedAsync()
{
_categories = await _context
    .Categories
    .AsNoTracking()
    .ToListAsync();

_model.CategoryId = _categories[0].Id;
}
```

Agora, vamos adicionar ao método de envio do form o codigo para persistir o produto.

```csharp
// Obs: Faça a injeção das dependencias para o banco de dados e navegação de paginas com as tags
// @inject AppDbContext _context e @inject NavigationManager _navigationManager

async Task HandleSubmitAsync()
{
    try
    {
        await _context.Products.AddAsync(_model);
        await _context.SaveChangesAsync();

        // Após salvar, retorna para pagina selecionada abaixo
        _navigationManager.NavigateTo("/products");
    }
    catch (Exception ex)
    {
        _errorMessage = ex.Message;
    }
}
```

## Listando os produtos

Crie uma pagina Index.razor dentro da pasta Products.

```csharp
@page "/products" // rota da pagina
@inject AppDbContext _context // Injeta o contexto para comunicação com o banco

<h1>Products</h1>

// Botão para criar um novo produto, redirecionando para o form create
<a href="products/create" class="btn btn-primary">
  <i class="oi oi-plus"></i> CREATE
</a>

<table class="table">
  <thead>
    <tr>
      <td>#</td>
      <td>Title</td>
      <td>Price</td>
      <td></td>
    </tr>
  </thead>
  <tbody>
    // Exibe os itens do banco na tabela
    @foreach (var product in _products)
    {
      <tr>
        <td>@product.Id</td>
        <td>@product.Title</td>
        <td>@product.Price</td>
        <td></td>
      </tr>
    }
  </tbody>
</table>

@code {
  // Cria lista de produtos ao inicializar a pagina.
  List<Product> _products = new();

  protected override async Task OnInitializedAsync()
  {
    _products = await _context
        .Products
        .AsNoTracking()
        .ToListAsync();
  }
}
```







