<!-- README.md -->
# pr-comment

Action para publicar comentários padronizados em Pull Requests usando um template Markdown (header/body/footer).

## Uso rápido

### Passo 1: Criar o arquivo de template

Crie um arquivo no seu repositório de testes (ex: .github/templates/custom.md) com o conteúdo abaixo. Fiz um design diferente do padrão para você notar a mudança visualmente:

```markdown
# 🎨 Check Automático (Customizado)

**Iniciado por:** @$ACTOR
**Assunto:** $SUBJECT

---

### 📝 Mensagem Principal
$BODY_MESSAGE

---

<details open>
<summary>🔍 Detalhes do Escopo</summary>

$BODY_SCOPE_BLOCK
</details>

<details>
<summary>📋 Pendências</summary>

$BODY_TODO_BLOCK
</details>

---
$FOOTER_BLOCK

###### *Gerado via Template Personalizado v1.0*
```

### Passo 2: Ajustar o Workflow

No seu arquivo .github/workflows/pr-comment.yml, você precisa adicionar o actions/checkout (se ainda não tiver) e o parâmetro template_path.

```yaml
# .github/workflows/pr-comment.yml
name: "Example PR comment"

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write

jobs:
  comment:
    runs-on: ubuntu-latest
    steps:
      # 1. IMPORTANTE: Checkout é obrigatório para ler o arquivo local
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Sua Action chamando o template padrão
      - name: Post PR comment
        uses: Malnati/pr-comment@v4.0.2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.pull_request.number }}
          header_actor: ${{ github.actor }}
          header_title: "🧪 Testing default PR message"
          header_subject: "Hearder de teste"
          body_message: |
            Este é um teste usando um **arquivo Markdown local** como template padrão.
          body_scope: |
            - base: `${{ github.event.pull_request.base.ref }}`
            - head: `${{ github.event.pull_request.head.ref }}`
          body_todo: |
            - Validar se o layout mudou.
          footer_result: "Template carregado com sucesso."
          footer_advise: "Nenhuma ação requerida."

      # 2. Sua Action chamando o template custom
      - name: Post PR comment
        uses: Malnati/pr-comment@v4.0.2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.pull_request.number }}
          header_actor: ${{ github.actor }}
          header_title: "🧪 Testing custom PR message"
          header_subject: "Hearder de teste"
          body_message: |
            Este é um teste usando um **arquivo Markdown local** como template personalizado.
          body_scope: |
            - base: `${{ github.event.pull_request.base.ref }}`
            - head: `${{ github.event.pull_request.head.ref }}`
          body_todo: |
            - Validar se o layout mudou.
          footer_result: "Template carregado com sucesso."
          footer_advise: "Nenhuma ação requerida."
          
          # 3. Caminho relativo à raiz do repositório
          template_path: ".github/templates/custom.md"
```

## Inputs

Liste todos os inputs com tabela (token, pr_number, header_, body_, footer_*).

## Outputs

comment_body: corpo final em Markdown, caso queira reutilizar em outro lugar.

## Versão

Instale sempre com tag: `uses: Malnati/pr-comment@v4.0.2`
