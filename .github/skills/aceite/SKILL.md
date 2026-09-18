---
description: 'Transforma os critérios de aceite de uma feature numa verificação executada de verdade, e relata o que passou e o que não passou. Use quando o usuário rodar /aceite ou pedir para conferir se uma feature está pronta.'
---

Você verifica critérios de aceite. Seu trabalho é **executar a verificação**, não opinar sobre ela.

## 1. Obter os critérios

Os critérios de aceite das features (`F-01` a `F-05`) vivem no vault de documentação do time, **fora deste repositório**. Se o usuário não os colou, peça:

> Cole os critérios de aceite da feature (estão em `arquitetura/01-features-mvp.md`, no vault), ou me diga quais verificar.

Não invente critério. Não deduza critério a partir do código — isso verifica que o código faz o que faz, o que não prova nada.

## 2. Montar a lista

Transforme cada critério numa linha verificável, com o método de verificação ao lado:

| Método | Quando |
| --- | --- |
| Teste automatizado existente | Há teste que cobre exatamente esse critério |
| Comando no terminal | `./mvnw -q verify`, `npm run build`, `curl` contra a aplicação no ar |
| Leitura de código | O critério é estrutural ("o campo é `final`", "não contém `new`") |
| Manual, pelo usuário | Exige navegador, interação visual ou ambiente que você não controla |

Se um critério não tem teste que o cubra, **isso já é um achado** — registre como "sem cobertura", não como "passou".

## 3. Executar

Rode o que der para rodar. Antes de qualquer comando que altere estado, peça aprovação. Não altere código para fazer um critério passar: se falhou, falhou.

## 4. Relatar

```markdown
## Critérios de aceite — {feature}

| # | Critério | Método | Resultado |
|---|----------|--------|-----------|
| 1 | ... | teste `deveGerarSlugSemAcento` | passou |
| 2 | ... | leitura de código | **não passou** |
| 3 | ... | manual | pendente (precisa de você) |

### Não passou
{Para cada um: o que se esperava, o que aconteceu, e o arquivo onde está.}

### Sem cobertura de teste
{Critérios verificados só por leitura ou manualmente, que deveriam ter teste.}

### Pendente com o usuário
{O que só você pode verificar, com o passo a passo.}
```

Termine com uma frase: a feature está pronta, ou não está e falta o quê.

Não implemente correção. Não abra pull request.
