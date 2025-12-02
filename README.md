<!-- README.md -->
# pr-comment

GitHub Action para publicar comentários em Pull Requests usando um layout padronizado em três blocos: **header**, **body** e **footer**.

A Action é pensada para ser chamada por qualquer workflow, recebendo o conteúdo já montado em variáveis de entrada. Ela apenas formata e envia o comentário para a PR.

## Formato da mensagem

A Action organiza o comentário da seguinte forma:

- **Header**
  - `header_actor` → ator responsável (ex.: `github.actor`)
  - `header_title` → título do comentário
  - `header_subject` → assunto ou contexto (ex.: `Sincronização base → head`)

- **Body**
  - `body_message` → texto principal (pode ser multilinha)
  - `body_scope` → lista ou bullets com o escopo do que foi verificado/afetado
  - `body_todo` → lista ou bullets com ações pendentes / TODO

- **Footer**
  - `footer_result` → resumo do resultado
  - `footer_advise` → orientação ou próximos passos para quem vai revisar/mesclar

## Inputs

| Nome            | Obrigatório | Descrição                                                       |
|-----------------|------------:|-----------------------------------------------------------------|
| `token`         | sim         | Token GitHub (geralmente `${{ secrets.GITHUB_TOKEN }}`)        |
| `pr_number`     | sim         | Número da Pull Request a ser comentada                         |
| `header_actor`  | sim         | Ator (ex.: `${{ github.actor }}`)                              |
| `header_title`  | sim         | Título do comentário                                           |
| `header_subject`| sim         | Assunto/contexto                                               |
| `body_message`  | sim         | Texto principal do comentário                                  |
| `body_scope`    | não         | Escopo (lista/bullets)                                         |
| `body_todo`     | não         | Ações pendentes / TODO                                         |
| `footer_result` | não         | Resumo do resultado                                            |
| `footer_advise` | não         | Orientação ou próximos passos                                  |

## Uso básico

Exemplo de workflow em `.github/workflows/pr-comment.yml`:

```yaml
name: "Exemplo de uso da pr-comment"

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  pr-comment-example:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Montar mensagem padronizada
        id: build-pr-message
        env:
          GITHUB_ACTOR: ${{ github.actor }}
          BASE_BRANCH: ${{ github.event.pull_request.base.ref }}
          HEAD_BRANCH: ${{ github.event.pull_request.head.ref }}
          CHANGED_FILES: ${{ github.event.pull_request.changed_files }}
        run: |
          set -euo pipefail

          HEADER_ACTOR="${GITHUB_ACTOR}"
          HEADER_TITLE="🔁 auto-sync"
          HEADER_SUBJECT="Sincronização de ${BASE_BRANCH} → ${HEAD_BRANCH}"

          BODY_MESSAGE="Relatório automático do workflow.\n\nEsta mensagem resume o estado da sincronização entre os branches."
          BODY_SCOPE="- Branch base: ${BASE_BRANCH}\n- Branch de trabalho: ${HEAD_BRANCH}\n- Arquivos alterados: ${CHANGED_FILES}"
          BODY_TODO=""

          if [ "${CHANGED_FILES:-0}" -gt 0 ]; then
            FOOTER_RESULT="Esta Pull Request contém ${CHANGED_FILES} arquivo(s) alterado(s)."
            FOOTER_ADVISE="O fluxo de validação continuará normalmente."
          else
            FOOTER_RESULT="Pull Request sem arquivos alterados."
            FOOTER_ADVISE="Verifique se isso é esperado antes de prosseguir com o merge."
          fi

          {
            echo "header_actor=${HEADER_ACTOR}"
            echo "header_title=${HEADER_TITLE}"
            echo "header_subject=${HEADER_SUBJECT}"
            printf 'body_message<<EOF\n%s\nEOF\n' "${BODY_MESSAGE}"
            printf 'body_scope<<EOF\n%s\nEOF\n' "${BODY_SCOPE}"
            printf 'body_todo<<EOF\n%s\nEOF\n' "${BODY_TODO}"
            echo "footer_result=${FOOTER_RESULT}"
            echo "footer_advise=${FOOTER_ADVISE}"
          } >> "${GITHUB_OUTPUT}"

      - name: Publicar comentário padronizado
        uses: seu-usuario/pr-comment@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.pull_request.number }}
          header_actor: ${{ steps.build-pr-message.outputs.header_actor }}
          header_title: ${{ steps.build-pr-message.outputs.header_title }}
          header_subject: ${{ steps.build-pr-message.outputs.header_subject }}
          body_message: ${{ steps.build-pr-message.outputs.body_message }}
          body_scope: ${{ steps.build-pr-message.outputs.body_scope }}
          body_todo: ${{ steps.build-pr-message.outputs.body_todo }}
          footer_result: ${{ steps.build-pr-message.outputs.footer_result }}
          footer_advise: ${{ steps.build-pr-message.outputs.footer_advise }}
