# mobile_arquitetura_02

Repositório de atividades da matéria de Desenvolvimento para Dispositivos Móveis II. 



## Questionário de Reflexão - Atividade 2

### 1. Em qual camada foi implementado o mecanismo de cache? Explique por que essa decisão é adequada dentro da arquitetura proposta.

O mecanismo de cache foi implementado na **camada Data**, especificamente como um `ProductCacheDatasource`.

**Esta decisão é adequada porque:**

- **Separação de responsabilidades**: O cache é um detalhe de infraestrutura/armazenamento, não uma regra de negócio. Pertence naturalmente à camada responsável por acesso a dados.

- **Independência do domínio**: A camada de domínio (entidades e contratos) não precisa saber da existência do cache, mantendo-se pura e focada nas regras de negócio.

- **Facilidade de manutenção**: Se no futuro quisermos trocar o cache em memória por um cache persistente (como Hive ou SharedPreferences), só precisamos modificar o `ProductCacheDatasource`, sem afetar as outras camadas.

- **Respeito ao princípio da inversão de dependência**: O repositório (`ProductRepositoryImpl`) depende de abstrações (interfaces), mas o cache é uma implementação concreta que fica isolada na camada de dados.

---

### 2. Por que o ViewModel não deve realizar chamadas HTTP diretamente?

O ViewModel não deve realizar chamadas HTTP diretamente por vários motivos arquiteturais:

- **Separação de responsabilidades**: O ViewModel é responsável por coordenar o estado da interface e o fluxo da aplicação, não por detalhes técnicos de comunicação com servidores.

- **Testabilidade**: Se o ViewModel fizer chamadas HTTP diretamente, os testes unitários se tornam complexos, exigindo mocks de requisições de rede. Com a separação, podemos testar o ViewModel isoladamente, mockando apenas o repositório.

- **Reutilização**: Se a mesma lógica de negócio precisar ser usada em diferentes contextos (mobile, web, desktop), o ViewModel com chamadas HTTP diretas estaria acoplado a um mecanismo específico de rede.

- **Manutenção**: Mudanças na API (endpoints, headers, autenticação) exigiriam modificações em todos os ViewModels que fazem chamadas diretas. Com a arquitetura em camadas, essas mudanças ficam isoladas nos DataSources.

- **Princípio da Responsabilidade Única**: O ViewModel tem a responsabilidade de gerenciar o estado da UI. Chamadas HTTP são responsabilidade da camada de dados.

---

### 3. O que poderia acontecer se a interface acessasse diretamente o DataSource?

Se a interface acessasse diretamente o DataSource, várias consequências negativas ocorreriam:

- **Acoplamento excessivo**: A UI ficaria fortemente acoplada a detalhes de infraestrutura. Qualquer mudança na API ou no formato dos dados exigiria alterações na interface.

- **Dificuldade de manutenção**: O código se tornaria mais difícil de manter, com lógica de apresentação misturada com lógica de acesso a dados.

- **Ausência de tratamento de estados**: A interface não teria uma camada intermediária para gerenciar estados como loading, erro ou cache. Isso resultaria em:
  - Sem indicadores de carregamento
  - Tratamento de erros inconsistente
  - Sem fallback para cache quando offline

- **Violação do princípio de separação de responsabilidades**: A UI passaria a ter múltiplas responsabilidades: renderização, gerenciamento de estado e acesso a dados.

- **Código duplicado**: Se diferentes partes da UI precisassem dos mesmos dados, cada uma faria sua própria chamada ao DataSource, sem compartilhamento de cache ou estado.

- **Dificuldade de testar**: Testes de widget precisariam lidar com chamadas de rede reais, tornando-os lentos e não confiáveis.

---

### 4. Como essa arquitetura facilitaria a substituição da API por um banco de dados local?

A arquitetura em camadas facilita enormemente a substituição da API por um banco de dados local:

- **Contratos no domínio**: O `ProductRepository` (abstração) define apenas o que o repositório deve fazer, não como. A camada de domínio não sabe se os dados vêm de uma API, banco local ou cache.

- **Implementações substituíveis**: Para trocar de API para banco local, basta trocar o DataSource utilizado pelo repositório:
  ```
  // Antes (API)
  final datasource = ProductRemoteDatasource(httpClient);
  
  // Depois (Banco local)
  final datasource = ProductLocalDatasource(database);
  ```

- **Injeção de dependência**: A mudança ocorre apenas no ponto de injeção (main.dart). As camadas superiores (Presentation e Domain) permanecem completamente inalteradas.

- **DataSources especializados**: Podemos ter múltiplos DataSources e o repositório decidir a estratégia:
  ```
  Future<List<Product>> getProducts() async {
  try {
    return await remote.getProducts();  // Tenta API primeiro
  } catch (e) {
    return await local.getProducts();    // Fallback para banco local
    }
  }
  ```
- **Escalabilidade**: Se amanhã precisarmos adicionar outra fonte de dados (Firebase, outra API, etc.), a arquitetura já está preparada para isso sem refatoração massiva.
