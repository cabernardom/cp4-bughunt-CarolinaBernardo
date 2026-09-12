# Checkpoint 4 - Bug Hunt StreamFIAP

## Identificacao

**Grupo:** CarolinaBernardo

| Integrante | RM | Turma |
|---|---|---|
| Carolina Bernardo | 564651 | 2CCPW |

| Campo | Resultado |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |

## Parte 1 - Bugs encontrados

| # | Sintoma observado | Causa raiz | Correcao aplicada | Conceito |
|---|---|---|---|---|
| bug01 | Serie cadastrada sem titulo, categoria, duracao ou classificacao; o preco padrao era retornado. | `Serie` nao chamava o construtor de `Conteudo` e declarava overload em vez de override. | Inicializacao da superclasse e `@Override` em `calcularPrecoAluguel()`, cobrando R$ 4,90 por temporada. | Heranca e polimorfismo de sobrescrita |
| bug02 | O preco promocional de filme aumentava 20%. | `Filme.aplicarPromocao` multiplicava por `1.2`. | Multiplicacao por `0.8`, aplicando 20% de desconto. | Interface e regra de negocio |
| bug03 | Documentario era cobrado como conteudo comum. | `Documentario` herdava o preco padrao de `Conteudo`. | Override retornando `0.0`. | Heranca e sobrescrita |
| bug04 | Nome do usuario ficava nulo e usuarios com pouco credito eram aceitos. | Atribuicao `nome = nome` nao alterava o atributo; comparacao de saldo estava invertida. | `this.nome = nome` e comparacao `creditos >= preco`. | Encapsulamento e estado do objeto |
| bug05 | Era possivel alugar novamente um conteudo ja alugado. | `Usuario.alugar` nao verificava `isDisponivel()`. | Lancamento de `ConteudoIndisponivelException` antes das demais regras. | Validacao e excecoes |
| bug06 | Busca por ID inexistente retornava HTTP 200 com corpo nulo. | O controller capturava qualquer excecao e retornava `null`. | Remocao do catch vazio e propagacao de `ConteudoNaoEncontradoException`. | Tratamento de excecoes |
| bug07 | Busca por categoria retornava resultado vazio mesmo com categoria igual. | Comparacao de `String` com `==` e filtragem manual. | Uso de `conteudoRepository.findByCategoria(categoria)`. | Igualdade de objetos e Spring Data |
| bug08 | Restricao etaria chegava ao cliente como erro generico. | Excecao checked nao tinha handler global. | Excecao runtime e resposta HTTP 403 com mensagem da regra. | Excecoes checked/unchecked |
| bug09 | Cadastro aceitava duracao zero ou negativa. | `duracaoMinutos` era publico e o setter nao validava. | Atributo privado, setter com validacao e construtor usando o setter. | Encapsulamento e invariantes |
| bug10 | Cadastro e atualizacao aceitavam creditos negativos. | `setCreditos` aceitava qualquer valor. | Validacao centralizada no setter e no construtor parametrizado. | Encapsulamento |
| bug11 | Usuario inexistente era tratado como `IllegalArgumentException` generica. | Nao havia excecao de dominio nem handler especifico. | Criacao de `UsuarioNaoEncontradoException` e resposta HTTP 404. | Excecoes customizadas |
| bug12 | Entrada invalida gerava erro sem contrato HTTP claro. | `IllegalArgumentException` nao era mapeada pelo advice. | Handler global com resposta HTTP 400 e campo `erro`. | API REST e tratamento global |

## Parte 2 - Ajustes de Clean Code

| # | Onde estava | Principio violado | O que foi mudado |
|---|---|---|---|
| clean01 | Controllers com `@Autowired` em campos mutaveis. | Dependencias ocultas e baixo testabilidade. | Injecao explicita por construtor e campos `final`. |
| clean02 | Metodo `calcularDescontoAntigo` e bloco comentado de cupons. | Codigo morto e comentarios obsoletos. | Remocao do codigo sem uso. |
| clean03 | Valores `9.90`, `5.00`, `4.90` e `0.8` espalhados nos modelos. | Numeros magicos. | Constantes nomeadas por regra de negocio. |
| clean04 | Variaveis `c` e `p` em `Usuario.alugar`. | Nomes pouco expressivos. | `conteudo` e `preco`, que tornam o fluxo autoexplicativo. |
| clean05 | Uso de `HttpStatus.UNPROCESSABLE_ENTITY` deprecated. | Dependencia de API obsoleta. | Uso de `HttpStatus.valueOf(422)`, preservando o contrato. |
| clean06 | `throws ClassificacaoIndicativaException` apos a excecao virar unchecked. | Assinatura redundante e ruido na API. | Remocao da declaracao e do import sem uso. |

## Parte 3 - Perguntas de reflexao

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

### 1. Injecao de dependencia (Aula 13)

`ConteudoController` precisa de um `ConteudoRepository` gerenciado pelo Spring porque o repository e um bean criado pelo container. O Spring instancia a implementacao do `JpaRepository`, configura a conexao com o banco e injeta essa referencia no construtor do controller. Assim, o controller nao conhece a implementacao concreta nem precisa controlar seu ciclo de vida. Um `new ConteudoRepository()` nao funcionaria porque repository e uma interface e porque a instancia criada manualmente nao receberia os proxies, transacoes e configuracoes do Spring Data. A injecao por construtor tambem deixa a dependencia obrigatoria e facilita testes com um mock.

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)

No JDBC da Aula 12, o DAO abre `Connection`, monta `PreparedStatement`, executa SQL e transforma cada `ResultSet` em objeto. Isso da controle fino sobre SQL, transacoes e recursos do banco, mas exige muito codigo repetitivo. Neste projeto, `ConteudoRepository extends JpaRepository<Conteudo, Long>` ja fornece save, findAll, findById e delete. O Spring Data cria a implementacao em runtime e interpreta `findByCategoria` pelo nome do metodo, gerando a consulta equivalente. JPA resolve o mapeamento objeto-relacional e o ciclo de persistencia; JDBC ainda pode ser melhor para consultas muito especificas ou otimizacoes de baixo nivel.

### 3. Excecoes checked vs unchecked (Aula 11)

Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

A `ClassificacaoIndicativaException` originalmente estendia `Exception`, obrigando cada camada a declarar ou capturar a excecao. O problema era que o controller declarava a excecao, mas nao havia um tratamento HTTP global para transformar a regra em resposta util. Ela agora estende `RuntimeException`, como as demais excecoes de negocio do projeto, e o `GlobalExceptionHandler` captura o tipo explicitamente. O cliente recebe HTTP 403 e a mensagem com idade do usuario, titulo e classificacao exigida. Isso separa a regra do modelo da apresentacao HTTP sem esconder a informacao do erro.

### 4. Sobrescrita vs sobrecarga (Aula 7)

Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

`Conteudo` define `calcularPrecoAluguel()` sem parametros. A `Serie` tinha `calcularPrecoAluguel(double desconto)`, que era outro metodo por ter assinatura diferente, portanto nao participava do polimorfismo usado por `Usuario.alugar`. O resultado era o preco padrao da superclasse, apesar de o codigo compilar. A anotacao `@Override` teria exigido exatamente a assinatura herdada e apontado o erro durante a compilacao. A correcao adicionou `@Override` e implementou o metodo sem parametros, retornando R$ 4,90 por temporada.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)

O construtor parametrizado deve garantir que um objeto criado diretamente ja nasca valido, por isso ele delega para setters validados. Os setters tambem precisam validar porque o Spring desserializa o JSON usando o construtor vazio e depois altera propriedades. A duracao e validada em `Conteudo.setDuracaoMinutos`, e os creditos em `Usuario.setCreditos`, evitando estados negativos por qualquer caminho. Regras de transicao ficam em metodos do modelo: `Usuario.alugar` verifica idade, disponibilidade e saldo antes de debitar. Validar somente no controller deixaria o dominio vulneravel a outros usos, como testes ou chamadas internas.

### 6. Abstracao e interface (Aulas 8 e 9)

`Conteudo` e abstrata porque concentra identidade, dados comuns e o comportamento base de qualquer item alugavel. `Promocionavel` e uma interface porque representa uma capacidade opcional: filme e serie aplicam desconto, mas documentario nao participa da promocao. Se documentario passasse a ter promocao, ele implementaria `Promocionavel` e criaria `aplicarPromocao`; o metodo `calcularPrecoPromocional` de `Conteudo` ja detecta essa capacidade. As classes `Filme`, `Serie`, `Usuario` e os controllers ficariam intactos. Isso mostra que a interface reduz acoplamento e permite adicionar comportamento sem alterar o fluxo geral.

## Execucao

O projeto usa Java 17 ou superior e Spring Boot. Configure as credenciais Oracle localmente em `src/main/resources/application.properties`; elas nao devem ser commitadas. Neste ambiente, a validacao Maven nao foi executada porque o repositorio nao possui `mvnw` e o comando `mvn` nao esta instalado. Os arquivos Java alterados foram validados pelo diagnostico do Java Language Server, sem erros de compilacao nos trechos modificados.
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```
