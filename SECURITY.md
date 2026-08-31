# Política de Segurança e Privacidade

A segurança das credenciais dos estudantes e a conformidade com a **Lei Geral de Proteção de Dados (LGPD - Lei nº 13.709/2018)** são pilares fundamentais da engenharia do **Sistemáticos**.

Este documento formaliza as práticas de segurança adotadas na arquitetura e o procedimento para reporte responsável de vulnerabilidades.

---

## 1. Postura e Controles de Segurança

O Sistemáticos foi projetado sob o princípio de **Zero Duplicação de Dados Acadêmicos** e **Privilégio Mínimo**:

- **Criptografia em Repouso e Trânsito:**
  - As senhas nunca são armazenadas em texto claro. Ficam cifradas com **AES-256-GCM** (autenticado) em memória enquanto a sessão estiver ativa.
  - Toda a comunicação entre cliente, servidor e portal institucional ocorre exclusivamente sob **TLS/HTTPS**.
- **Cookies de Sessão Seguros:**
  - O identificador de sessão (`sistematicos_sid`) é emitido com as flags `httpOnly`, `SameSite=Lax` e `Secure` (em produção), mitigando ataques de XSS e interceptação client-side.
- **Autorização Derivada do Servidor:**
  - Em nenhuma rota da API o servidor confia em identificadores (como RA, turma ou id de usuário) passados no corpo da requisição ou parâmetros de rota para autorização de leitura acadêmica. O contexto do aluno é sempre extraído diretamente da sessão criptografada validada no servidor.
- **Ciclo de Vida Efêmero da Sessão:**
  - Sessões possuem TTL de **12 horas** e são varridas a cada **5 minutos**. Ao expirar ou ao executar logout, a chave e a credencial são purgadas da memória.
- **Minimização de Logs:**
  - Logs de acesso gravam apenas dados estritamente necessários para auditoria de anomalias (IP truncado/aproximado, user-agent e timestamp). Senhas e payloads sensíveis são filtrados antes da camada de logging.

---

## 2. Reporte Responsável de Vulnerabilidades

Se você identificar qualquer falha de segurança, brecha em potencial ou comportamento anômalo no portal ou na arquitetura descrita:

1. **Não abra uma Issue pública** para falhas que possam comprometer dados de usuários.
2. Envie um e-mail confidencial para:
   - **Autor e Desenvolvedor:** Murilo Rocha Silva
   - **E-mail Institucional:** `murilosilva@aluno.unifef.edu.br`
   - **Assunto:** `[SECURITY] Vulnerabilidade no Sistemáticos - <Breve resumo>`
3. Forneça detalhes suficientes para reprodução (passo a passo, request de exemplo ou prova de conceito).

### Prazos de Resposta:
- **Confirmação de recebimento:** até 24 horas.
- **Avaliação e plano de correção:** até 72 horas.
- **Agradecimento/Créditos:** Pesquisadores que reportarem falhas de forma ética e responsável serão devidamente reconhecidos.

---

## 3. Atendimento a Direitos do Titular (LGPD)

- **Exclusão de Conta Self-Service:** Alunos podem solicitar e executar a exclusão imediata e definitiva dos seus dados (tarefas, perfil, histórico) diretamente na plataforma.
- **Dados Institucionais Oficiais:** O Sistemáticos não é o custodiante do registro acadêmico oficial (vestibular, histórico escolar, notas finais). Para solicitações relativas ao banco oficial da UniFEF, o canal apropriado é o Encarregado de Dados oficial da Fundação Educacional de Fernandópolis (`lgpd@fef.edu.br`).
