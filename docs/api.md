# Referência da API

> Isto é a **referência de rotas** do Sistemáticos. Para entender *como* a sessão é criada, renovada e protegida — o que faz essas rotas funcionarem contra o portal institucional real — veja [ARCHITECTURE.md](../ARCHITECTURE.md), em especial [Fluxo de autenticação](../ARCHITECTURE.md#fluxo-de-autenticação) e [Autorização: o RA vem da sessão, nunca do cliente](../ARCHITECTURE.md#autorização-o-ra-vem-da-sessão-nunca-do-cliente).

O servidor expõe pouco mais de 60 rotas HTTP, agrupadas por domínio. A coluna **Autorização** diz o que protege cada grupo — não é uma escolha por rota, é um middleware aplicado uma vez, na entrada:

- **Sessão** — basta um cookie `sistematicos_sid` válido (`requireAuth`); qualquer aluno autenticado pode chamar.
- **Dono ou admin** — a rota também confere que o RA da sessão é o dono do recurso pedido, ou está em `ADMIN_RAS` (`requireOwnRaOrAdmin`).
- **Admin** — restrita a `ADMIN_RAS` (`requireAdmin`).
- **Pública** — não exige sessão nenhuma; é um espelho do que já é público no site institucional.

| Domínio | Rotas principais | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| **Autenticação e sessão** | `POST /api/login`, `/api/login/guest`, `/api/logout`, `GET /api/session/me`, `POST /api/recover-password` | Pública (é o que cria a sessão) | Login real, modo convidado, encerramento de sessão, checagem de sessão ativa, recuperação de senha |
| **Acadêmico (proxy ao vivo)** | `GET /api/boletim`, `/api/horario-aula`, `/api/calendario`, `/api/matriz`, `/api/atividades(/resumo)`, `/api/nupex`, `/api/cronograma`, `/api/fefsis/aulas/*`, `/api/dashboard/home` | Sessão | Notas, frequência, horários, matriz curricular, atividades complementares, aulas/materiais e o resumo do dashboard — sempre lidos ao vivo do portal |
| **Financeiro** | `GET /api/mensalidade`, `POST /api/mensalidade/pix`, `/mensalidade/confirmar`, `/mensalidade/baixar-pdf`, `DELETE /api/mensalidade/:id` | Sessão (exclusão: dono ou admin) | Boletos, PIX dinâmico, confirmação de pagamento e emissão de PDF |
| **Portal institucional (público)** | `GET /api/fef-landing`, `/api/fef/noticias`, `/api/fef/cursos`, `/api/fef/curso-detalhes`, `/api/fef-institutional` | Pública | Notícias, cursos e páginas institucionais da FEF — não exigem sessão de aluno |
| **Comunidade e social** | `GET/POST /api/messages`, `GET /api/public/alunos`, `/api/direct/conversations`, `POST /api/profile/update` | Sessão (DMs: dono ou admin) | Chat (histórico via HTTP, tempo real via socket), diretório de estudantes, conversas diretas |
| **Produtividade pessoal** | `GET/POST/DELETE /api/tasks`, `/api/reminders` | Dono ou admin | Kanban de tarefas e lembretes por RA |
| **Conta** | `DELETE /api/account` | Sessão (só a própria conta) | Exclusão/anonimização de dados a pedido do titular — ver [Alinhamento com a LGPD](../ARCHITECTURE.md#alinhamento-com-a-lgpd) |
| **Avisos e gestão** | `GET/POST/DELETE /api/avisos`, `GET/POST /api/admin/admins`, `GET /api/notifications/active`, `POST /api/system/popup` | Admin (leitura de avisos: sessão) | Mural de comunicados, gestão de administradores, popups de sistema |
| **Suporte** | `GET /api/public/suporte/tickets/:ra`, `/suporte/admins`, `/suporte/ticket/:id`, `DELETE /api/admin/suporte/ticket/:id` | Dono ou admin (exclusão: admin) | Central de atendimento (histórico via HTTP; conversa ao vivo via socket) |
| **Sugestões** | `GET/POST/PUT/DELETE /api/public/sugestoes` (+ `/vote`, `/comment`, `/status`) | Sessão (moderação: admin) | Mural de ideias com votação e comentários |
| **Doações** | `POST /api/donations/checkout`, `/webhook`, `GET /verify/:id`, `/stats` | Pública (webhook valida segredo próprio) | Doações voluntárias via PIX (InfinitePay), com ranking de apoiadores |
| **Atlética / Classroom** | `GET/POST /api/atletica*`, `GET /api/classroom` | Sessão (escrita: admin) | Loja e agenda da atlética, feed de materiais do Classroom |
| **Infra** | `GET /api/proxy/photo`, `/api/proxy/ping` | Sessão (proxy de foto) / pública (ping) | Proxy de foto de perfil (evita expor o cookie da FEF ao navegador do aluno) e healthcheck |

Todas as rotas sob `/api/*` (exceto login/logout/session) passam primeiro pelo rate limiter geral e pelo `fefSessionMiddleware` que resolve a sessão institucional antes de qualquer handler rodar.

---

## Exemplo: rota "dono ou admin"

`DELETE /api/tasks/:id` só apaga a tarefa se o RA da sessão for o mesmo RA dono do card do Kanban — nunca o `:id` sozinho é suficiente, mesmo que o chamador saiba (ou adivinhe) um ID válido de outra pessoa:

```http
DELETE /api/tasks/task_8f2a1 HTTP/1.1
Cookie: sistematicos_sid=3f9a1c...b02e
```

Se o RA da sessão não bate com o dono da tarefa (e não é admin):

```json
{ "error": "Acesso negado" }
```

```http
HTTP/1.1 403 Forbidden
```

## Exemplo: rota pública

`GET /api/fef/cursos` não exige sessão — é só um espelho dos cursos listados no site institucional, então dá para testar direto:

```bash
curl https://sistematicos.site/api/fef/cursos
```

```json
[
  { "title": "Administração", "url": "https://fef.br/graduacao/administracao", "icon": "https://fef.br/icons/adm.svg" },
  { "title": "Sistemas de Informação", "url": "https://fef.br/graduacao/sistemas-de-informacao", "icon": "https://fef.br/icons/si.svg" }
]
```

---

<p align="center"><em>Voltar para <a href="../README.md">README</a> · <a href="../ARCHITECTURE.md">ARCHITECTURE.md</a></em></p>
