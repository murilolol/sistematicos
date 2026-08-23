<h1 align="center">SISTEMÁTICOS</h1>

<p align="center"><em>Portal acadêmico não oficial para a comunidade da <strong>UniFEF</strong> — uma alternativa moderna e em tempo real ao sistema institucional legado.</em></p>

<p align="center">
  <img src="./screenshots/02-dashboard-dark.png" alt="Sistemáticos — Dashboard" width="100%" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React-19-20232A?style=flat-square&logo=react&logoColor=61DAFB" alt="React 19" />
  <img src="https://img.shields.io/badge/TypeScript-5.7-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-v4-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white" alt="Tailwind CSS v4" />
  <img src="https://img.shields.io/badge/Node.js-Express_5-339933?style=flat-square&logo=nodedotjs&logoColor=white" alt="Node.js / Express 5" />
  <img src="https://img.shields.io/badge/Socket.IO-realtime-010101?style=flat-square&logo=socketdotio&logoColor=white" alt="Socket.IO" />
  <img src="https://img.shields.io/badge/PWA-ready-5A0FC8?style=flat-square&logo=pwa&logoColor=white" alt="PWA" />
</p>

<p align="center"><strong><a href="https://sistematicos.site/">sistematicos.site</a></strong></p>

<br>

> **TL;DR** — entre com seu RA e senha da FEF, veja notas/horários/boletos __em tempo real__ numa interface moderna, sem duplicar seus dados acadêmicos em nenhum lugar. Detalhes técnicos completos em [ARCHITECTURE.md](./ARCHITECTURE.md).

<br>

## Índice

- [Sobre](#sobre)
- [Privacidade, dados pessoais e LGPD](#privacidade-dados-pessoais-e-lgpd)
- [Módulos](#módulos)
- [Como usar](#como-usar)
- [Perguntas rápidas](#perguntas-rápidas)
- [Capturas de tela](#capturas-de-tela)
- [Stack](#stack)
- [Arquitetura](#arquitetura)
- [Autoria](#autoria)
- [Aviso legal](#aviso-legal)

<br>

## Sobre

O sistema acadêmico oficial da UniFEF roda sobre uma stack legada (**JSF/PrimeFaces**): recarregamento síncrono de página, navegação lenta e pouquíssimo suporte a celular. Todo semestre, a mesma rotina se repete — abrir o portal para ver uma nota, consultar frequência em outra aba, entrar de novo para ver o boleto.

O **Sistemáticos** nasceu dessa frustração como aluno. Em vez de mais um sistema paralelo com seu próprio cadastro, ele funciona como uma camada moderna sobre o portal oficial: o aluno entra com o mesmo RA e senha que já usa, e passa a ver tudo — notas, frequência, horários, boletos, avisos — em uma interface única, rápida e em tempo real.

É um **projeto pessoal e acadêmico**, sem fins lucrativos e sem qualquer pretensão de concorrer comercialmente com sistemas de mercado — o objetivo é estritamente prático: resolver, de forma simples e intuitiva, um problema real da rotina de um aluno.

- **Zero atrito** — notas, faltas, horários, boletos e materiais em uma SPA fluida, sem recarregar página.
- **Sem duplicação de dados** — cada informação é buscada ao vivo direto do portal institucional; nada de notas desatualizadas em um banco paralelo.
- **Ecossistema completo** — financeiro com PIX, chat em tempo real, integração com Google Classroom, Kanban de tarefas e uma pequena rede social entre os próprios alunos do curso.

> **Nota:** este repositório é uma *vitrine técnica*. O código-fonte do motor de sincronização e automação é proprietário e não é distribuído publicamente.

> A própria plataforma também tem uma versão *ao vivo* deste documento: a página **[Sobre o Projeto](https://sistematicos.site/sobre)**, dentro do portal, reúne visão geral, arquitetura, mapa de API e postura de segurança em abas navegáveis — pensada tanto para aluno curioso quanto para quem só quer confirmar o que é feito com os próprios dados.

### Sistemáticos vs. portal oficial

| | Portal oficial (JSF) | Sistemáticos |
| :-- | :--: | :--: |
| Recarrega a página a cada clique | ✅ | ❌ |
| Layout responsivo para celular | ❌ | ✅ |
| Chat em tempo real entre alunos | ❌ | ✅ |
| Simulador de médias | ❌ | ✅ |
| PIX para mensalidade | ❌ | ✅ |
| Instalável como app (PWA) | ❌ | ✅ |
| Fonte oficial dos seus dados acadêmicos | ✅ | _consulta o mesmo portal_ |

<br>

## Privacidade, dados pessoais e LGPD

Pedir para um aluno digitar o RA e a senha institucional em um site que não é o oficial é, com razão, motivo de desconfiança. Por isso, o funcionamento é deliberadamente simples de explicar — e as práticas abaixo seguem, na medida de um projeto pessoal, os princípios da **Lei Geral de Proteção de Dados (LGPD)**:

- **Finalidade e minimização** — a senha é usada **apenas para autenticar contra o portal oficial da FEF**, na hora do login, e nunca é enviada para nenhum outro lugar. O Sistemáticos **não tem banco de dados de notas, frequência ou matrícula** — essas informações são sempre lidas ao vivo do portal institucional, nunca armazenadas por aqui.
- **Segurança e não retenção** — a senha é mantida **cifrada em memória** (nunca em disco, nunca em texto puro) só para permitir que a sessão continue válida enquanto o aluno navega, e é descartada quando a sessão expira ou o servidor reinicia.
- **Consentimento livre e informado** — o tratamento de dados só ocorre porque o próprio aluno opta, voluntariamente, por inserir suas credenciais e usar a plataforma; o **modo convidado** existe justamente para quem prefere explorar a interface sem fornecer nenhum dado real. Um banner de cookies, exibido no primeiro acesso, deixa explícita a distinção entre cookies essenciais (login) e opcionais (preferências).
- **Auditoria com minimização** — cada login grava IP aproximado, cidade/dispositivo/navegador e horário, **nunca a senha** — e esse registro é **sobrescrito a cada novo login do mesmo RA**, em vez de acumular um histórico crescente: existe só o suficiente para investigar um acesso indevido, não para rastrear o aluno ao longo do tempo.
- **Transparência** — os detalhes técnicos completos de como isso é implementado (criptografia, cookies, expiração) estão documentados e abertos em **[ARCHITECTURE.md](./ARCHITECTURE.md)**, para qualquer pessoa auditar o raciocínio.
- **Direitos do titular** — dados gerados pelo próprio uso da plataforma (mensagens de chat, tarefas, sugestões) podem ter acesso, correção ou eliminação solicitados diretamente ao autor, a qualquer momento.

> Este README descreve as práticas técnicas em alto nível. A plataforma mantém, dentro dela mesma, uma **[Política de Privacidade](https://sistematicos.site/privacidade)** e **[Termos de Uso](https://sistematicos.site/termos)** completos, nos moldes da LGPD — vinculados também no rodapé do banner de cookies.

<br>

## Módulos

| Área | Descrição |
| :-- | :-- |
| **Boletim** | Notas, frequência por disciplina e simulador de médias — descubra quanto falta tirar na próxima prova para passar. |
| **Vida acadêmica** | Horário de aulas, cronograma, matriz curricular, calendário de provas e atividades complementares — tudo lido ao vivo do portal. |
| **Financeiro** | Mensalidades em aberto e quitadas, boleto e QR Code PIX para pagamento instantâneo. |
| **Comunicação** | Chat em tempo real da turma/curso, mensagens diretas privadas entre colegas e central de suporte. |
| **Classroom** | Feed de materiais e atividades sincronizado com o Google Classroom das disciplinas. |
| **Hub Acadêmico** | Calculadora de médias, banco de provas antigas, trilha de carreira e networking entre alunos — extensões que rodam em cima dos dados já carregados, sem novas chamadas ao portal. |
| **Ferramentas** | Gerador de capa no padrão ABNT com autopreenchimento (curso, autor, instituição) e simulador de aprovação. |
| **Tarefas** | Quadro Kanban pessoal para organizar entregas, trabalhos e provas. |
| **Avisos e gestão** | Mural de comunicados por curso/semestre, com painel administrativo para representantes de turma. |
| **Comunidade** | Diretório de estudantes e professores, central de sugestões com votação, e loja da atlética. |
| **Apoio ao projeto** | Doações voluntárias via PIX, com ranking opcional de apoiadores — mantém a infraestrutura sem cobrar do aluno. |
| **Interface** | Tema claro/escuro, busca universal (`Ctrl+K`) e instalação como app (PWA) no celular ou computador. |

<br>

## Como usar

1. Acesse **[sistematicos.site](https://sistematicos.site/)**.
2. Entre com o **RA e a senha** que você já usa no portal oficial da FEF — ou clique em **"Entrar como Convidado"** para explorar a interface sem se autenticar.
3. Pronto: o dashboard já carrega suas notas, horários e avisos em tempo real.

<br>

## Perguntas rápidas

> ### Isso é oficial da UniFEF?
> __Não.__ É um projeto independente, feito por um aluno — veja a seção [Aviso legal](#aviso-legal).

> ### Minha senha fica salva em algum lugar?
> Não em texto puro, e não permanentemente. Veja [Privacidade, dados pessoais e LGPD](#privacidade-dados-pessoais-e-lgpd) e, para o detalhe técnico exato, [ARCHITECTURE.md](./ARCHITECTURE.md).

> ### Minhas notas aparecem desatualizadas às vezes?
> Não deveriam — cada tela busca os dados *ao vivo* no portal oficial no momento em que você abre. Se o portal da FEF estiver instável, o Sistemáticos também sente.

> ### Posso usar sem ser aluno da FEF?
> Sim, pelo modo convidado (botão `Entrar como Convidado` na tela de login) — você navega pela interface com dados de exemplo, sem autenticar.

> ### O app funciona offline?
> Parcialmente: é um PWA instalável, mas só páginas públicas (avisos, notícias institucionais) ficam em cache. Boletim, financeiro e horário sempre exigem conexão, *de propósito* — para nunca guardar seus dados pessoais no dispositivo.

> ### Achei um bug ou tenho uma ideia — pra onde eu mando?
> Dentro do próprio portal, use a **Central de Sugestões** — é lida direto pelo autor. Para algo técnico sobre este repositório, abra uma [issue](https://github.com/murilolol/sistematicos/issues).

> ### O Sistemáticos cobra alguma coisa do aluno?
> Não, o uso da plataforma é **inteiramente gratuito**. Existem duas frentes financeiras opcionais e desacopladas do login: a **Loja da Atlética** (pedidos via WhatsApp, o pagamento não passa pelo Sistemáticos) e **doações voluntárias** via PIX para ajudar a custear a infraestrutura — nenhuma das duas é necessária para usar boletim, horários, chat ou qualquer outro módulo acadêmico.

<br>

## Capturas de tela

<p align="center">
  <img src="./screenshots/01-login.png" alt="Login" width="49%" />
  <img src="./screenshots/03-dashboard-light.png" alt="Dashboard" width="49%" />
</p>

<p align="center">
  <img src="./screenshots/04-boletim.png" alt="Boletim" width="49%" />
  <img src="./screenshots/05-mensalidade.png" alt="Financeiro" width="49%" />
</p>

<details>
<summary>Ver o dashboard inteiro (scroll completo, claro e escuro)</summary>
<p align="center">
  <img src="./screenshots/dashboard-dark-full.png" alt="Dashboard completo — tema escuro" width="90%" />
  <img src="./screenshots/dashboard-light-full.png" alt="Dashboard completo — tema claro" width="90%" />
</p>
</details>

### Mobile

<p align="center">
  <img src="./screenshots/06-mobile-login.png" alt="Login em celular" width="24%" />
  <img src="./screenshots/07-mobile-dashboard.png" alt="Dashboard em celular" width="24%" />
  <img src="./screenshots/08-mobile-boletim.png" alt="Boletim em celular" width="24%" />
</p>

<details>
<summary>Ver o dashboard mobile inteiro (scroll completo)</summary>
<p align="center">
  <img src="./screenshots/mobile-dashboard-full.png" alt="Dashboard completo em celular" width="32%" />
</p>
</details>

> A sidebar desktop vira navegação inferior (bottom nav) em telas de celular — não é a mesma interface encolhida, é outro layout, pensado para uso de uma mão.

<br>

## Stack

- **Frontend**
  - React 19 + TypeScript
  - Vite 6
  - Tailwind CSS v4 + shadcn/ui
  - Framer Motion
- **Backend**
  - Node.js + Express 5
  - Socket.IO _(tempo real, multiplexado em salas)_
  - Cheerio _(parsing/sincronização)_
- **Sessão e segurança**
  - Cookies `httpOnly`
  - Sessão criptografada em memória (**AES-256-GCM**)
  - Zero persistência de credenciais

<br>

## Arquitetura

O servidor não guarda o registro acadêmico do aluno — ele mantém apenas uma sessão-ponte em memória e resolve cada requisição ao vivo contra o portal institucional (JSF/PrimeFaces), emulando o formulário real de login e navegação.

```mermaid
flowchart LR
    Client["Cliente\nReact 19 + PWA"]
    Server["Servidor\nExpress 5 + Socket.IO"]
    Session[("Sessão em memória")]
    FEF["Portal institucional\nJSF/PrimeFaces"]

    Client <-- "cookie httpOnly" --> Server
    Client <-- "WebSocket" --> Server
    Server --> Session
    Server <-- "scraping ao vivo" --> FEF
```

O documento **[ARCHITECTURE.md](./ARCHITECTURE.md)** é a referência completa para desenvolvedores e curiosos: diagramas de sequência do fluxo de login e da renovação automática de sessão, o adaptador de scraping reaproveitado por toda a camada acadêmica, o mapa completo das mais de 60 rotas da API (com exemplos de request/response em JSON), a topologia de salas do Socket.IO e as decisões de arquitetura (e seus trade-offs) por trás de cada escolha.

Um gostinho do que tem lá — o formato de resposta do login, já sem nenhum dado da sessão institucional:

```json
{
  "user": {
    "fullName": "Murilo Rocha Silva",
    "course": "Sistemas de Informação",
    "currentSemester": "4º Semestre"
  }
}
```

<br>

## Autoria

Idealizado, desenhado e desenvolvido **individualmente** por **Murilo Rocha Silva** — 19 anos, programando há 3, cursando o 4º semestre de Sistemas de Informação na UniFEF — sem financiamento externo ou equipe.

[![GitHub](https://img.shields.io/badge/GitHub-murilolol-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/murilolol)
[![Instagram](https://img.shields.io/badge/Instagram-@muriloodev-E4405F?style=flat-square&logo=instagram&logoColor=white)](https://instagram.com/muriloodev)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-murilodev-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/murilodev)

> © Murilo Rocha Silva. Todos os direitos sobre o design, a arquitetura e o código-fonte são reservados ao autor.

<br>

## Aviso legal

_O Sistemáticos é um projeto pessoal, acadêmico e sem fins lucrativos, independente e sem vínculo formal ou endosso institucional por parte da UniFEF (Fundação Educacional de Fernandópolis). Não tem finalidade comercial nem pretensão de concorrer com sistemas de mercado, e não substitui os canais oficiais para atos jurídicos e matrículas. O tratamento de dados pessoais segue os princípios descritos em [Privacidade, dados pessoais e LGPD](#privacidade-dados-pessoais-e-lgpd)._

### Canais oficiais da instituição

Para qualquer assunto relativo à **instituição de ensino** em si — matrícula, registro acadêmico oficial, ou uma solicitação formal de LGPD sobre os dados que a própria FEF mantém — use os canais oficiais, e não este repositório:

| Canal | Contato |
| :-- | :-- |
| Site institucional | [fef.br](https://fef.br) |
| Endereço | Av. Theotônio Vilela, 1685 — Jardim Vitória, Fernandópolis/SP — CEP 15608-380 |
| Ouvidoria | [fef.br/fefsisweb/#/ouvidoria](https://fef.br/fefsisweb/#/ouvidoria) |
| Encarregado de Dados (LGPD) | lgpd@fef.edu.br |

> O Encarregado de Dados acima é o **da instituição oficial**, não do Sistemáticos — ele trata de solicitações sobre o registro acadêmico mantido pela FEF. Para qualquer questão sobre este projeto especificamente, use os canais descritos em [Autoria](#autoria) ou abra uma [issue](https://github.com/murilolol/sistematicos/issues).
