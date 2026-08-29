# Referência da API

> Isto é a **referência de rotas** do Sistemáticos. Para entender *como* a sessão é criada, renovada e protegida — o que faz essas rotas funcionarem contra o portal institucional real — veja [ARCHITECTURE.md](../ARCHITECTURE.md), em especial [Fluxo de autenticação](../ARCHITECTURE.md#fluxo-de-autenticação) e [Autorização: o RA vem da sessão, nunca do cliente](../ARCHITECTURE.md#autorização-o-ra-vem-da-sessão-nunca-do-cliente). Uma versão navegável desta mesma referência, com syntax highlighting, está em [site/api.html](../site/api.html).

O servidor expõe **80 rotas HTTP** e **22 eventos de Socket.IO**, agrupados em 13 domínios. A coluna **Autorização** diz o que protege cada rota — não é uma escolha individual, é um middleware aplicado uma vez, na entrada:

- **Sessão** — basta um cookie `sistematicos_sid` válido (`requireAuth`); qualquer aluno autenticado pode chamar.
- **Sessão (implícita)\*** — a rota não usa `requireAuth` explicitamente, mas depende de `ssid`/`ssidchk` resolvidos pelo `fefSessionMiddleware`; sem sessão, a chamada ao portal falha adiante, só sem o `401` limpo.
- **Dono ou admin** — a rota também confere que o RA da sessão é o dono do recurso pedido, ou está em `ADMIN_RAS` (`requireOwnRaOrAdmin`).
- **Admin** — restrita a `ADMIN_RAS` (`requireAdmin`).
- **Pública** — não exige sessão nenhuma; é um espelho do que já é público no site institucional, ou a própria rota que cria a sessão.

Todas as rotas sob `/api/*` (exceto login/logout/session) passam primeiro pelo rate limiter geral e pelo `fefSessionMiddleware` que resolve a sessão institucional antes de qualquer handler rodar.

---

## Mapa de rotas — as 80, por domínio

#### Autenticação e sessão

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `POST` | `/api/login` | Pública | Login real contra o portal FEF (rate limited: 10/15min por IP) |
| `POST` | `/api/login/guest` | Pública | Ativa o modo convidado (cliente-side, sem sessão real) |
| `POST` | `/api/logout` | Pública | Encerra a sessão ativa |
| `GET` | `/api/session/me` | Pública | Checa se existe sessão ativa e devolve o perfil |
| `POST` | `/api/recover-password` | Pública | Recuperação de senha institucional |

#### Acadêmico (ao vivo)

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/boletim` | Sessão (implícita)* | Notas e frequência por disciplina |
| `GET` | `/api/horario-aula` | Sessão (implícita)* | Horário de aulas da semana |
| `GET` | `/api/calendario` | Sessão (implícita)* | Calendário de provas e eventos |
| `GET` | `/api/atividades` | Sessão (implícita)* | Atividades complementares registradas |
| `GET` | `/api/atividades/resumo` | Sessão (implícita)* | Resumo/contagem de horas de atividades |
| `GET` | `/api/nupex` | Sessão (implícita)* | Banco de provas antigas (NUPEX) |
| `GET` | `/api/matriz` | Sessão (implícita)* | Matriz curricular do curso |
| `GET` | `/api/dashboard/home` | Sessão (implícita)* | Resumo agregado para o dashboard inicial |
| `GET` | `/api/cronograma` | Sessão (implícita)* | Cronograma de aulas por disciplina |
| `POST` | `/api/cronograma/baixar-pdf` | Sessão | Gera PDF do cronograma |
| `GET` | `/api/fefsis/aulas` | Sessão (implícita)* | Feed do Google Classroom, ao vivo |
| `POST` | `/api/fefsis/aulas/changeselector` | Sessão (implícita)* | Troca o filtro de disciplina do Classroom |
| `POST` | `/api/fefsis/aulas/select` | Sessão (implícita)* | Seleciona um item/atividade do Classroom |
| `POST` | `/api/fefsis/aulas/files` | Sessão (implícita)* | Lista arquivos de uma atividade |
| `POST` | `/api/fefsis/aulas/download-arquivo` | Sessão (implícita)* | Baixa um arquivo de atividade |
| `POST` | `/api/fefsis/aulas/view-arquivo` | Sessão (implícita)* | Abre um arquivo de atividade para visualização |

#### Financeiro

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/mensalidade` | Sessão | Boletos e histórico de pagamentos |
| `POST` | `/api/mensalidade/pix` | Sessão (implícita)* | Gera QR Code PIX dinâmico para o boleto atual |
| `POST` | `/api/mensalidade/confirmar` | Sessão | Confirma pagamento manual (upload de comprovante) |
| `DELETE` | `/api/mensalidade/:id` | Sessão | Exclui um registro de pagamento manual |
| `POST` | `/api/mensalidade/baixar-pdf` | Sessão | Emite o PDF do boleto |

#### Portal institucional (público)

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/fef-landing` | Pública | Conteúdo da página inicial institucional |
| `GET` | `/api/fef/noticias` | Pública | Notícias do site institucional |
| `GET` | `/api/fef/cursos` | Pública | Lista de cursos oferecidos |
| `GET` | `/api/fef/curso-detalhes` | Pública | Detalhe de um curso específico |
| `GET` | `/api/fef-institutional` | Pública | Páginas institucionais diversas (sobre, história etc.) |
| `GET` | `/api/fef-notifications` | Pública | Espelha as notificações nativas do portal oficial |
| `POST` | `/api/avisos/read` | Pública | Marca um comunicado do portal oficial como lido |

#### Comunidade e social

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/messages` | Sessão | Histórico do chat da sala atual |
| `POST` | `/api/messages` | Sessão | Envia mensagem (texto ou anexo) — tempo real via socket |
| `GET` | `/api/public/alunos` | Pública | Diretório público de estudantes |
| `POST` | `/api/public/alunos/upload` | Sessão | Upload de foto de perfil |
| `POST` | `/api/public/alunos/like` | Sessão | Curte o perfil de outro aluno |
| `POST` | `/api/public/alunos/update` | Dono ou admin | Atualiza os próprios dados no diretório |
| `POST` | `/api/profile/update` | Dono ou admin | Atualiza o próprio perfil (nome, curso, foto) |
| `GET` | `/api/direct/conversations` | Sessão | Lista as conversas diretas do usuário |

#### Produtividade pessoal

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/reminders/:ra` | Dono ou admin | Lembretes pessoais do RA |
| `POST` | `/api/reminders` | Sessão | Cria um lembrete |
| `DELETE` | `/api/reminders/:id` | Sessão | Apaga um lembrete |
| `GET` | `/api/tasks/:ra` | Dono ou admin | Tarefas do quadro Kanban do RA |
| `POST` | `/api/tasks` | Sessão | Cria uma tarefa |
| `PATCH` | `/api/tasks/:id` | Sessão | Atualiza uma tarefa (allowlist de campos) |
| `DELETE` | `/api/tasks/:id` | Sessão | Apaga uma tarefa |

#### Conta

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `DELETE` | `/api/account` | Sessão | Exclusão/anonimização de conta (LGPD self-service) |

#### Avisos e gestão

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/avisos` | Pública | Mural de comunicados (dados locais) |
| `POST` | `/api/avisos` | Admin | Cria um comunicado |
| `DELETE` | `/api/avisos/:id` | Admin | Remove um comunicado |
| `GET` | `/api/notifications/active` | Pública | Popups/avisos de sistema ativos no momento |
| `POST` | `/api/system/popup` | Admin | Cria um popup de sistema |
| `GET` | `/api/admin/admins` | Admin | Lista administradores da plataforma |
| `POST` | `/api/admin/admins` | Admin | Adiciona/remove um administrador |

#### Suporte

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/public/suporte/tickets/:ra` | Dono ou admin | Histórico de chamados do RA |
| `GET` | `/api/public/suporte/admins` | Sessão | Lista atendentes disponíveis |
| `GET` | `/api/public/suporte/ticket/:id` | Pública | Detalhe de um chamado específico |
| `DELETE` | `/api/admin/suporte/ticket/:id` | Admin | Apaga um chamado (moderação) |

#### Sugestões

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/public/sugestoes` | Pública | Mural de sugestões com votos |
| `POST` | `/api/public/sugestoes` | Sessão | Cria uma sugestão |
| `PUT` | `/api/public/sugestoes` | Sessão | Edita uma sugestão própria |
| `DELETE` | `/api/public/sugestoes/:id` | Sessão | Apaga uma sugestão |
| `POST` | `/api/public/sugestoes/vote` | Sessão | Vota numa sugestão |
| `POST` | `/api/public/sugestoes/comment` | Sessão | Comenta numa sugestão |
| `POST` | `/api/public/sugestoes/comment/edit` | Sessão | Edita um comentário próprio |
| `POST` | `/api/public/sugestoes/comment/delete` | Sessão | Apaga um comentário próprio |
| `POST` | `/api/public/sugestoes/status` | Admin | Muda o status de uma sugestão (moderação) |

#### Doações

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `POST` | `/api/donations/checkout` | Pública | Cria um link de checkout PIX (InfinitePay) |
| `POST` | `/api/donations/webhook` | Pública | Confirmação de pagamento (valida segredo próprio do webhook) |
| `GET` | `/api/donations/verify/:id` | Pública | Verifica o status de uma doação |
| `GET` | `/api/donations/stats` | Pública | Ranking público de apoiadores |

#### Atlética / Classroom / Loja

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/payments` | Pública | Lista de pagamentos registrados (loja/atlética) |
| `GET` | `/api/atletica` | Pública | Catálogo e agenda da atlética |
| `POST` | `/api/atletica/upload` | Admin | Upload de imagem de produto |
| `POST` | `/api/atletica/update` | Admin | Atualiza catálogo/agenda da atlética |
| `GET` | `/api/classroom` | Pública | Feed de materiais em cache local |

#### Infra

| Método | Rota | Autorização | Propósito |
| :-- | :-- | :-- | :-- |
| `GET` | `/api/proxy/photo` | Sessão | Proxy de foto de perfil (não expõe o cookie da FEF) |
| `GET` | `/api/proxy/ping` | Pública | Healthcheck do servidor |

---

## Eventos de Socket.IO — os 22

Uma única conexão por cliente, multiplexada em salas (`Geral`, o curso do aluno, `dm_<ra1>_<ra2>`, `ticket_<id>`). `socket.authRa` é resolvido uma vez, na conexão, a partir do cookie `sistematicos_sid` — nunca de um campo do payload do evento.

| Evento | Direção | Propósito |
| :-- | :-- | :-- |
| `register_online_admin` | cliente→servidor | Admin se marca online para a lista de presença |
| `register_student_presence` | cliente→servidor | Aluno se marca online (multi-aba consciente) |
| `request_online_students` | cliente→servidor | Pede a lista atual de estudantes online |
| `disconnect` | automático | Remove presença ao fechar a aba/perder conexão |
| `join_room` | cliente→servidor | Entra numa sala (sai de todas as anteriores primeiro) |
| `new_message` | cliente→servidor | Envia mensagem de chat — retransmitida à sala |
| `typing_start` | cliente→servidor | Indicador de "digitando..." no chat |
| `typing_stop` | cliente→servidor | Encerra o indicador de digitação |
| `edit_message` | cliente→servidor | Edita uma mensagem própria |
| `delete_message` | cliente→servidor | Apaga uma mensagem própria |
| `support_ticket_create` | cliente→servidor | Abre um novo chamado de suporte |
| `support_ticket_join` | cliente→servidor | Entra na sala de um chamado existente |
| `support_typing_start` | cliente→servidor | Indicador de digitação dentro de um chamado |
| `support_typing_stop` | cliente→servidor | Encerra indicador de digitação do chamado |
| `support_message` | cliente→servidor | Mensagem dentro de um chamado de suporte |
| `support_ticket_close` | cliente→servidor | Encerra um chamado |
| `new_payment` | cliente→servidor | Notifica novo pagamento registrado (broadcast) |
| `delete_payment` | cliente→servidor | Remove um pagamento (broadcast) |
| `support_ticket_room_join` | cliente→servidor | Reconecta à sala do chamado após refresh |
| `support_ticket_transfer` | cliente→servidor | Transfere o chamado para outro atendente |
| `support_ticket_transfer_team` | cliente→servidor | Transfere o chamado para outra equipe |
| `update_atletica` | cliente→servidor | Notifica atualização do catálogo da atlética (broadcast) |

---

## Exemplos de resposta (JSON)

### Login — o que não aparece é o ponto

Nenhum campo `fefssid`/`fefssidchk` — a sessão institucional nunca chega ao cliente:

```json
{
  "user": {
    "username": "125314106669",
    "fullName": "Murilo Rocha Silva",
    "course": "Sistemas de Informação",
    "currentSemester": "4º Semestre",
    "photoUrl": "/api/proxy/photo?target=...&ssid=..."
  }
}
```

### GET /api/boletim

Dado acadêmico normalizado — nunca HTML bruto do portal:

```json
{
  "boletim": [
    {
      "disciplina": "Estrutura de Dados I",
      "docente": "Prof. Exemplo da Silva",
      "cargaHoraria": "30h",
      "frequencia": 95,
      "notas": { "n1": 8.5, "n2": 7.0, "media": 7.6 },
      "status": "Em Curso"
    }
  ],
  "availableYears": [2023, 2024, 2025, 2026],
  "availableSemesters": [1, 2]
}
```

### GET /api/mensalidade

```json
{
  "historico": [
    {
      "id": "mens_2026_09",
      "mes": "09", "ano": "2026",
      "valorOriginal": "R$ 1.530,92",
      "valorComDesconto": "R$ 765,46",
      "vencimento": "2026-09-08",
      "status": "Pendente"
    }
  ],
  "totalInvestido": "R$ 3.827,30",
  "pendentes": 1
}
```

### GET /api/tasks/:ra

```json
[
  {
    "id": "task_8f2a1",
    "ra": "125314106669",
    "title": "Entregar trabalho de BD",
    "status": "todo",
    "createdAt": "2026-08-20T14:03:00.000Z"
  }
]
```

### GET /api/public/sugestoes

```json
[
  {
    "id": "sug_142",
    "autorRa": "125314106669",
    "autorNome": "Murilo Rocha Silva",
    "titulo": "Modo escuro automático por horário",
    "descricao": "Alternar tema sozinho ao anoitecer.",
    "votos": 14,
    "status": "em_analise",
    "comentarios": []
  }
]
```

### GET /api/donations/stats

```json
{
  "totalArrecadado": "R$ 412,00",
  "totalApoiadores": 9,
  "ranking": [
    { "nome": "Anônimo", "valor": "R$ 100,00", "isAnonymous": true },
    { "nome": "Murilo R.", "valor": "R$ 50,00", "isAnonymous": false }
  ]
}
```

### Socket.IO · message_received

```json
{
  "id": 1755882345123,
  "username": "125314106669",
  "fullName": "Murilo Rocha Silva",
  "content": "Alguém tem o slide da aula de hoje?",
  "room": "Sistemas de Informação",
  "timestamp": "2026-08-29T19:32:25.123Z",
  "admin": false
}
```

### Erros comuns

Rate limit de login excedido:

```json
{ "error": "Muitas tentativas de login. Tente novamente em alguns minutos." }
```
```http
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 10
RateLimit-Remaining: 0
```

Acesso negado (RA da sessão não é dono do recurso, nem admin):

```json
{ "error": "Acesso negado" }
```
```http
HTTP/1.1 403 Forbidden
```

---

## Excluindo uma conta (LGPD self-service)

`DELETE /api/account` distingue o que apaga de fato do que anonimiza — e a razão é o art. 16 da LGPD: excluir mensagens de chat ou comentários corromperia conversas e totais que outras pessoas ainda dependem, então esses são anonimizados em vez de removidos.

```bash
curl -X DELETE https://sistematicos.site/api/account \
  -H "Cookie: sistematicos_sid=..."
```

```json
{
  "message": "Conta excluída com sucesso",
  "removido": ["perfil", "tarefas", "lembretes", "mensalidades", "logins"],
  "anonimizado": ["mensagens", "comentarios", "doacoes"]
}
```

---

## Chamando a API

A única rota seguramente testável sem sessão é uma pública:

```bash
curl https://sistematicos.site/api/fef/cursos
```

```python
import requests

resp = requests.get("https://sistematicos.site/api/fef/cursos", timeout=10)
resp.raise_for_status()
for curso in resp.json():
    print(f"{curso['title']} -> {curso['url']}")
```

```typescript
async function listarCursos(): Promise<Curso[]> {
  const res = await fetch("https://sistematicos.site/api/fef/cursos");
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}
```

Login com sessão persistente em Python:

```python
session = requests.Session()  # guarda o cookie sistematicos_sid entre chamadas
resp = session.post("https://sistematicos.site/api/login",
                     json={"username": "SEU_RA", "password": "SUA_SENHA"})
if resp.status_code == 401:
    raise SystemExit("RA ou senha incorretos")
boletim = session.get("https://sistematicos.site/api/boletim").json()
```

---

## Exemplo: rota "dono ou admin", em código

O trecho real de `server/index.js` que protege `DELETE /api/tasks/:id` — o RA vem do cookie de sessão, nunca do corpo da requisição:

```js
const getAuthRa = (req) => req.fefSession?.ra || null;

const requireOwnRaOrAdmin = (extractTargetRa) => (req, res, next) => {
  const ra = getAuthRa(req);
  const targetRa = extractTargetRa(req);
  if (!ra || !targetRa || (ra !== String(targetRa) && !ADMIN_RAS.includes(ra))) {
    return res.status(403).json({ error: 'Acesso negado' });
  }
  req.authRa = ra;
  next();
};

app.delete(
  '/api/tasks/:id',
  requireOwnRaOrAdmin((req) => tasks.find((t) => t.id === req.params.id)?.ra),
  deleteTaskHandler
);
```

Os middlewares `requireAuth` e `requireAdmin` completos estão em [ARCHITECTURE.md § Autorização](../ARCHITECTURE.md#autorização-o-ra-vem-da-sessão-nunca-do-cliente).

---

<p align="center"><em>Voltar para <a href="../README.md">README</a> · <a href="../ARCHITECTURE.md">ARCHITECTURE.md</a> · <a href="../site/api.html">versão navegável</a></em></p>
