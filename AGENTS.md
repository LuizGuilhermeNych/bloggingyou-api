# Regras para agentes de IA — bloggingyou-api

Backend do bloggingyou. Este arquivo é lido pelo GitHub Copilot e pelo Kilo Code. Ele é a única fonte de contexto do projeto para as ferramentas de IA: o que não estiver aqui, elas não sabem.

## Contexto

Projeto de estudo. Dois desenvolvedores com base em Java e **sem experiência anterior em Spring** estão aprendendo o framework construindo este blog. O objetivo é o aprendizado; código pronto que ninguém entende é um resultado ruim, mesmo que funcione.

- Spring Boot 4.1 / Java 25 / PostgreSQL 17 / Maven wrapper (`./mvnw`)
- Frontend separado, em `bloggingyou-web` (Angular 22)
- Documentação de arquitetura fora deste repositório, em vault privado do time

## Processo

- Antes de editar, liste os arquivos que vai tocar e espere aprovação.
- Faça só o que foi pedido. Não refatore, não renomeie, não adicione dependência sem pedir.
- Ao terminar código Java, rode `./mvnw -q verify` e corrija até passar.
- Ao final de cada tarefa, explique em português, em 3 a 6 linhas, **o que fez e por quê**. Se usou um recurso do Spring que ainda não apareceu no projeto, explique o recurso.
- Se não tiver certeza de uma API do Spring, diga que não tem certeza em vez de inventar. A versão 4 renomeou starters e mudou assinaturas; material escrito para o Spring Boot 3 frequentemente não se aplica.
- Nunca faça `git push`. Nunca crie pull request sem pedido explícito.

## Arquitetura

### Pacotes por feature

A raiz é `com.bloggingyou.api`. Cada feature é uma pasta fechada com as classes de todas as camadas:

```
com.bloggingyou.api
├── post            PostController, PostService, PostRepository, Post, dto/
├── autenticacao    SessaoController, UsuarioRepository, Usuario, ConfiguracaoSeguranca
└── comum           TratadorDeErros, ErroResponse
```

- Uma feature não importa classe interna de outra.
- Só entra em `comum` o que é usado por duas ou mais features **e** não pertence ao domínio de nenhuma.
- **Não** criar pacotes `controller/`, `service/`, `repository/`, `model/`. Essa é a organização que a maioria dos tutoriais usa e não é a deste projeto.

### Responsabilidade das camadas

| Camada | Faz | Nunca faz |
| --- | --- | --- |
| Controller | Recebe HTTP, valida com Bean Validation, converte DTO, devolve status | Regra de negócio, acesso a repositório |
| Service | Regra de negócio, transação | Conhecer HTTP (`HttpServletRequest`, `ResponseEntity`) |
| Repository | Consulta | Regra de negócio |
| Entidade | Estado e invariante do próprio objeto | Ser serializada na resposta |

**Entidade JPA nunca sai pela API.** Todo endpoint recebe e devolve `record` de DTO.

### Banco de dados

- Esquema versionado por Flyway, em `src/main/resources/db/migration`, no formato `V<n>__<descricao>.sql`.
- `spring.jpa.hibernate.ddl-auto` é `validate` e **não** deve ser alterado.
- **Nunca edite um arquivo de migração já existente.** Correção se faz com migração nova.
- Colunas de data e hora são `timestamptz`, mapeadas para `OffsetDateTime`.
- Nomes de tabela e coluna em português, no singular.

### API

- Todo endpoint sob `/api`. JSON em `camelCase`.
- Post é lido publicamente por `slug` e administrado por `id`.
- Erro sempre no formato de `ErroResponse`: `timestamp`, `status`, `erro`, `caminho` e, em erro de validação, `campos`.
- Resposta `500` tem mensagem genérica. **Nunca** devolva stack trace, nome de classe de exceção ou SQL ao cliente.
- `401` de login é idêntico para e-mail inexistente e senha errada.

### Segurança

- Autenticação por sessão, com cookie `HttpOnly`, `Secure` em produção e `SameSite=Lax`. **Não** implemente JWT.
- Senha com BCrypt, pelo `PasswordEncoder` do Spring Security.
- Proteção CSRF fica **habilitada**. Não a desabilite "porque é uma API REST" — a credencial é cookie, então a API é vulnerável a CSRF.
- Do Actuator, só `health` é público.
- Segredo vem de variável de ambiente. Nunca escreva credencial em arquivo versionado.

## Código

- Java 25. `record` para DTO; **nunca** `record` para `@Entity`.
- Injeção por construtor, campos `final`. **Nunca** `@Autowired` em campo.
- Sem `System.out`; use o logger.
- Sem `catch (Exception e)` vazio ou que apenas registre e siga.
- Sem lógica de negócio em controller.
- Sem campo mutável em bean (`@Service`, `@Component`, `@RestController`).
- Nome de classe, método e variável em português, seguindo o domínio (`GeradorDeSlug`, `PostService`, `slugPara`). Anotações e APIs do framework permanecem em inglês.
- Comentário só quando explica **por quê**, não o que a linha faz.

## Testes

- Toda classe de service tem teste unitário correspondente.
- Teste de unidade instancia a classe com `new` e usa Mockito para as dependências — **sem** `@SpringBootTest`.
- Teste de integração usa Testcontainers com PostgreSQL. **Não** use H2.
- Nome de teste em português, descrevendo o comportamento: `deveGerarSlugSemAcento`.
- Asserções com AssertJ.
- Cobertura não é meta. Todo critério de aceite de uma feature tem um teste que o verifica.

## Git

- Branches: `<tipo>/<descricao-kebab>` — `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `spike`.
- Branch de feature inclui o identificador: `feat/f02-gerador-de-slug`.
- Commits em português, no imperativo, curtos: `feat: adiciona gerador de slug`.
- Nunca faça push sem pedido explícito.

## Quando parar e perguntar

- A tarefa exige adicionar dependência ao `pom.xml`.
- A tarefa exige criar ou alterar migração de banco.
- A solução mais direta contradiz alguma regra acima.
- Algo neste arquivo está em conflito com o que foi pedido no chat.
