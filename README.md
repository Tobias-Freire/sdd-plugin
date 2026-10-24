# sdd-plugin

Plugin pessoal para o Claude Code com skills e contextos que guiam o agente em tarefas recorrentes.

## Estrutura

```
sdd-plugin/
├── .claude-plugin/
│   ├── plugin.json         # manifesto do plugin
│   └── marketplace.json    # permite instalar o repositório como marketplace
├── skills/
│   └── commit/
│       └── SKILL.md        # skill /commit
└── README.md
```

Quando fizer sentido, adicione também `agents/` (subagentes) e `references/` (material de apoio compartilhado entre skills).

## Instalação

Direto do GitHub:

```
/plugin marketplace add Tobias-Freire/sdd-plugin
/plugin install sdd@sdd-marketplace
```

Ou a partir de um clone local:

```
/plugin marketplace add /caminho/para/sdd-plugin
/plugin install sdd@sdd-marketplace
```

Para atualizar depois de novas mudanças:

```
/plugin marketplace update sdd-marketplace
```

## Skills

### `/commit`

Cria commits atômicos no padrão Conventional Commits a partir das mudanças feitas na sessão atual.

- Arquivos de partes diferentes da tarefa vão para commits separados.
- Arquivos modificados que não pertencem à sessão atual não são commitados, a menos que o chamado peça explicitamente para commitar tudo (`--all` ou texto equivalente).
- Por padrão: mensagem em inglês, somente a linha de assunto, sem corpo e sem coautoria de IA.

| Argumento | Efeito |
|-----------|--------|
| `--with-body` | Adiciona mensagem de corpo ao commit. |
| `--with-ai` | Adiciona corpo e coautoria de IA (`Co-Authored-By`). |
| `--pt-br` | Escreve a mensagem em português brasileiro. |
| `--all` | Inclui também as mudanças pendentes que não são da sessão. |

Os argumentos podem ser combinados:

```
/commit
/commit --with-body
/commit --with-ai --pt-br
/commit --with-body --pt-br
```

> Skills de plugin ficam sob o namespace do plugin, então a forma garantida é `/sdd:commit`. Digite `/` no Claude Code para ver como ela aparece na sua instalação.
