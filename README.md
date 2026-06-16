# 🐾 PetShop PetCompagne — Backend API

> API RESTful para gerenciamento de pet shop, desenvolvida com Spring Boot, autenticação JWT e persistência em MySQL.

---

## 1. Apresentação do Projeto

**Nome do Projeto:** PetShop PetCompagne

**Objetivo do Sistema:**
Fornecer uma API backend completa para o gerenciamento de um pet shop online, suportando operações de catálogo de produtos, controle de categorias, cadastro de usuários com autenticação segura, realização de pedidos e upload de imagens.

**Funcionalidades disponíveis:**
- Autenticação de usuários com geração de token JWT
- Cadastro e consulta de categorias de produtos
- Cadastro, listagem paginada, busca, atualização e exclusão de produtos
- Cadastro de usuários com criptografia de senha (BCrypt)
- Criação e consulta de pedidos vinculados ao usuário autenticado
- Upload de imagens com armazenamento local e retorno de URL
- Controle de acesso baseado em perfis (ADMIN / usuário comum)

---

## 2. Tecnologias Utilizadas

| Tecnologia | Versão | Descrição |
|---|---|---|
| Java | 17 | Linguagem principal do projeto |
| Spring Boot | 4.0.6 | Framework principal para construção da API |
| Spring Web | — | Criação dos endpoints REST |
| Spring Data JPA | — | Abstração de acesso ao banco de dados |
| Spring Security | — | Controle de autenticação e autorização |
| MySQL | — | Banco de dados relacional |
| MySQL Connector/J | — | Driver JDBC para conexão com MySQL |
| JJWT (jsonwebtoken) | 0.11.5 | Geração e validação de tokens JWT |
| Lombok | — | Redução de boilerplate (getters, setters, etc.) |
| Maven | — | Gerenciador de dependências e build |
| JUnit / Spring Test | — | Testes automatizados |

---

## 3. Estrutura do Projeto

```
backendpetpagne/
└── src/
    └── main/
        ├── java/com/devsenai1a/backendpetpagne/
        │   ├── BackendpetpagneApplication.java   ← Classe principal (entry point)
        │   │
        │   ├── config/
        │   │   ├── SecurityConfig.java           ← Configuração do Spring Security e regras de acesso
        │   │   └── WebConfig.java                ← Configuração de CORS
        │   │
        │   ├── controllers/
        │   │   ├── CategoriaController.java      ← Endpoints de categoria
        │   │   ├── ProdutoController.java         ← Endpoints de produto
        │   │   ├── UsuarioController.java         ← Endpoints de usuário e login
        │   │   ├── PedidoController.java          ← Endpoints de pedido
        │   │   └── UploadController.java          ← Endpoint de upload de imagem
        │   │
        │   ├── dto/
        │   │   ├── LoginDTO.java                  ← Dados de entrada para login
        │   │   ├── ProdutoResponseDTO.java         ← Resposta pública de produto (sem campos internos)
        │   │   └── UsuarioResponseDTO.java         ← Resposta de usuário (sem senha)
        │   │
        │   ├── models/
        │   │   ├── Categoria.java                 ← Entidade categoria
        │   │   ├── Produto.java                   ← Entidade produto
        │   │   ├── Usuario.java                   ← Entidade usuário
        │   │   ├── Pedido.java                    ← Entidade pedido
        │   │   └── ItemPedido.java                ← Entidade item do pedido
        │   │
        │   ├── repository/
        │   │   ├── CategoriaRepository.java       ← Acesso ao banco — categorias
        │   │   ├── ProdutoRepository.java          ← Acesso ao banco — produtos (com busca paginada)
        │   │   ├── UsuarioRepository.java          ← Acesso ao banco — usuários
        │   │   ├── PedidoRepository.java           ← Acesso ao banco — pedidos
        │   │   └── ItemPedidoRepository.java       ← Acesso ao banco — itens do pedido
        │   │
        │   └── security/
        │       ├── JwtService.java                ← Geração, validação e extração de dados do JWT
        │       └── JwtFilter.java                 ← Filtro HTTP que intercepta e valida o token
        │
        └── resources/
            └── application.properties             ← Configurações da aplicação (banco, JWT, upload)
```

**Descrição das camadas:**

- **config** — Centraliza as configurações de segurança (rotas protegidas, filtros) e CORS.
- **controllers** — Recebem as requisições HTTP, chamam o repositório/serviço correspondente e retornam as respostas.
- **dto** — Objetos de transferência de dados usados para expor apenas os campos necessários nas respostas.
- **models** — Entidades JPA que mapeiam as tabelas do banco de dados.
- **repository** — Interfaces que estendem `JpaRepository`, responsáveis pela persistência dos dados.
- **security** — Contém a lógica de JWT: geração de token, validação e o filtro de autenticação por requisição.

---

## 4. Banco de Dados

**Nome do banco:** `petshop_db`

**Principais tabelas:**

| Tabela | Descrição |
|---|---|
| `categoria` | Armazena as categorias dos produtos |
| `produto` | Armazena os produtos disponíveis no pet shop |
| `usuarios` | Armazena os usuários cadastrados (clientes e admins) |
| `pedidos` | Armazena os pedidos realizados |
| `itens_pedido` | Armazena os itens individuais de cada pedido |

**Diagrama Entidade-Relacionamento (DER):**

```
┌──────────────────────┐          ┌──────────────────────────────────────┐
│       categoria      │          │               produto                │
├──────────────────────┤          ├──────────────────────────────────────┤
│ PK id_categoria (INT)│◄─────────┤ FK id_categoria (INT)                │
│    nome (VARCHAR)    │  N:1     │ PK id_produto (INT)                  │
│    descricao (TEXT)  │          │    nome (VARCHAR)                    │
│    ativo (BOOLEAN)   │          │    descricao (TEXT)                  │
│    imagem (LONGTEXT) │          │    preco (DECIMAL)                   │
└──────────────────────┘          │    preco_desconto (DECIMAL)          │
                                  │    imagem (LONGTEXT)                 │
                                  │    sku (VARCHAR)                     │
                                  │    qtd_estoque (INT)                 │
                                  │    ativo (BOOLEAN)                   │
                                  │    data_criacao (DATETIME)           │
                                  └──────────────────────────────────────┘
                                                    ▲
                                                    │ N:1
┌──────────────────────────────────────┐           │
│               usuarios               │   ┌────────────────────────────────────┐
├──────────────────────────────────────┤   │            itens_pedido            │
│ PK id (INT)                          │   ├────────────────────────────────────┤
│    nome (VARCHAR)                    │   │ PK id_item_pedido (INT)            │
│    email (VARCHAR, UNIQUE)           │   │    nome_produto (VARCHAR)          │
│    senha (VARCHAR)                   │   │    quantidade (INT)                │
│    perfil (VARCHAR)                  │   │    preco_unitario (DECIMAL)        │
│    endereco (VARCHAR)                │   │ FK id_pedido_fk (INT) ────────────►│
│    bairro (VARCHAR)                  │   │ FK id_produto_fk (INT)────────────►│
│    complemento (VARCHAR)             │   └────────────────────────────────────┘
│    cep (VARCHAR)                     │                    ▲
│    cidade (VARCHAR)                  │                    │ 1:N
│    estado (VARCHAR)                  │   ┌────────────────────────────────────┐
│    foto (LONGTEXT)                   │   │               pedidos              │
│    data_cadastro (DATETIME)          │   ├────────────────────────────────────┤
└──────────────────────────────────────┘   │ PK id_pedido (INT)                 │
                  │                        │    data_pedido (DATETIME)          │
                  │ 1:N                    │    status (VARCHAR)                │
                  └───────────────────────►│    valor_total (DECIMAL)           │
                                           │    endereco_entrega (VARCHAR)      │
                                           │ FK id_usuario_fk (INT)             │
                                           └────────────────────────────────────┘
```

**Relacionamentos:**

- `produto` N:1 `categoria` — Um produto pertence a uma categoria; uma categoria pode ter vários produtos.
- `pedidos` N:1 `usuarios` — Um pedido pertence a um usuário; um usuário pode ter vários pedidos.
- `itens_pedido` N:1 `pedidos` — Um item pertence a um pedido; um pedido pode ter vários itens.
- `itens_pedido` N:1 `produto` — Um item referencia um produto; um produto pode aparecer em vários itens.

---

## 5. Como Executar o Projeto

### Pré-requisitos

- Java 17 instalado
- Maven instalado (ou usar o `mvnw` incluído no projeto)
- MySQL instalado e em execução
- Git instalado

---

### Passo 1 — Clonar o repositório

```bash
git clone https://github.com/Gab147g/PetPagne-Springboot-project.git
cd backendpetpagne
```

---

### Passo 2 — Criar o banco de dados

Acesse o MySQL e execute:

```sql
CREATE DATABASE petshop_db;
```

> O Spring Boot com `ddl-auto=update` criará automaticamente as tabelas ao iniciar a aplicação.

---

### Passo 3 — Configurar o `application.properties`

Edite o arquivo `src/main/resources/application.properties` com as suas credenciais:

```properties
spring.application.name=backendpetpagne

# Banco de dados
spring.datasource.url=jdbc:mysql://localhost:3306/petshop_db
spring.datasource.username=root
spring.datasource.password=SUA_SENHA_AQUI

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# JWT
jwt.secret=petpagnie_super_secret_2026

# Upload de arquivos
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
upload.dir=uploads
```

---

### Passo 4 — Executar a aplicação

Usando Maven Wrapper (recomendado):

```bash
./mvnw spring-boot:run
```

Ou com Maven instalado globalmente:

```bash
mvn spring-boot:run
```

A API estará disponível em: **`http://localhost:8080`**

---

## 6. Endpoints da API

### 🔓 Rotas Públicas (sem autenticação)

#### Categorias

| Método | URL | Descrição |
|---|---|---|
| GET | `/categorias` | Lista todas as categorias |
| POST | `/categorias` | Cadastra uma nova categoria |

#### Produtos

| Método | URL | Descrição |
|---|---|---|
| GET | `/produtos` | Lista produtos com paginação e filtro por nome (`?nome=&page=0&size=10`) |
| GET | `/produtos/{id}` | Busca um produto pelo ID |

#### Usuários

| Método | URL | Descrição |
|---|---|---|
| POST | `/usuarios` | Cadastra um novo usuário |
| POST | `/usuarios/login` | Realiza login e retorna o token JWT |
| GET | `/usuarios` | Lista todos os usuários |

#### Upload

| Método | URL | Descrição |
|---|---|---|
| POST | `/upload` | Faz upload de uma imagem (parâmetro: `arquivo`) e retorna a URL |

---

### 🔐 Rotas Protegidas (requer token JWT no header)

> Enviar o header: `Authorization: Bearer {token}`

#### Pedidos

| Método | URL | Descrição |
|---|---|---|
| GET | `/pedidos` | Lista todos os pedidos (autenticado) |
| POST | `/pedidos` | Cria um novo pedido para o usuário autenticado |
| GET | `/pedidos/meus` | Lista apenas os pedidos do usuário autenticado |

---

### 🛡️ Rotas de Admin (requer perfil ADMIN)

> Além do token JWT, o usuário deve ter `perfil = "ROLE_ADMIN"`.

#### Produtos (Admin)

| Método | URL | Descrição |
|---|---|---|
| POST | `/produtos/admin` | Cadastra um novo produto |
| PUT | `/produtos/admin/{id}` | Atualiza um produto existente |
| DELETE | `/produtos/admin/{id}` | Remove um produto |

#### Upload via Pedido (Admin)

| Método | URL | Descrição |
|---|---|---|
| POST | `/pedidos/admin/upload` | Upload de imagem via contexto de pedido (Admin) |

---

## 7. Funcionalidades Implementadas

- ✅ Cadastro de categorias com nome, descrição, status (ativo) e imagem
- ✅ Cadastro de produtos com preço, preço com desconto, SKU, estoque, imagem e categoria
- ✅ Listagem de produtos com paginação e busca por nome (case-insensitive)
- ✅ Busca de produto por ID
- ✅ Atualização e exclusão de produtos (restrito ao ADMIN)
- ✅ Cadastro de usuários com criptografia de senha via BCrypt
- ✅ Login de usuários com geração de token JWT (validade de 24h)
- ✅ Listagem de usuários com DTO (sem expor a senha)
- ✅ Criação de pedidos vinculados ao usuário autenticado
- ✅ Listagem de todos os pedidos
- ✅ Consulta dos pedidos do próprio usuário (`/pedidos/meus`)
- ✅ Gerenciamento de itens do pedido (associação automática pedido ↔ item)
- ✅ Upload de imagens com nome único (UUID) e retorno de URL de acesso
- ✅ Controle de acesso por perfil: rotas `/admin/**` restritas ao perfil `ROLE_ADMIN`
- ✅ Configuração de CORS para integração com frontend
- ✅ API stateless com autenticação via JWT (sem sessão no servidor)

---

## 8. Integrante da Equipe / Desenvolvedor

| Nome                           | Função                |
|--------------------------------|-----------------------|
|Gabriel Gerim Marozi de Oliveira|Desenvolvedor FullStack|

## 9. Detalhe importante

- É possivel trocar no SQL a ROLE do usúario para ADMIN.

