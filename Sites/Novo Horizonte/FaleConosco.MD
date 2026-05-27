# Relatório Técnico de Depuração e Resolução
## Sistema de Formulário "Fale Conosco" e Entrega de E-mails

**Data:** 27 de Maio de 2026  
**Projeto:** Sino.Sistema.NovoHorizonte.Web  
**Status:** ✅ RESOLVIDO  

---

### 1. Resumo Executivo
Este documento descreve detalhadamente o processo de diagnóstico, depuração (*debugging*) e resolução de falhas no formulário de contato (**Fale Conosco**) da Câmara Municipal de Novo Horizonte. O formulário apresentava impedimentos tanto no ciclo de validação de dados no servidor quanto na infraestrutura de mensageria (SMTP), impossibilitando o envio e a entrega final das mensagens encaminhadas pelos cidadãos.

A investigação seguiu uma abordagem estruturada "do front-end ao banco de dados", resolvendo sucessivos gargalos de arquitetura de software e infraestrutura.

---

### 2. Linha do Tempo de Diagnósticos e Soluções

#### 🔹 Fase 1: Falha na Validação do Modelo C# (`ModelState.IsValid`)
* **Sintoma:** Ao clicar no botão "Enviar Mensagem", a página simplesmente recarregava sem disparar erros visíveis e sem enviar o e-mail.
* **Diagnóstico via Debugger:** Inserido um *breakpoint* no método `OnPostAsync` do arquivo `index.cshtml.cs` na linha `if (!ModelState.IsValid)`. A inspeção paralela do `ModelState` revelou o erro crítico:  
  `"O campo 'ErrorMessage' é obrigatório."`
* **Causa Raiz:** A propriedade `public string ErrorMessage { get; set; }` possuía a anotação `[BindProperty]`. No ambiente .NET moderno com *Nullable Reference Types* habilitado por padrão, o framework interpretou que o HTML era obrigado a enviar uma string preenchida para esta propriedade. Como ela é puramente de exibição interna, a validação falhava.
* **Ação Corretiva:** Removida a decoração `[BindProperty]` da variável `ErrorMessage` (ou alternativamente tipada como anulável `string?`), permitindo que o formulário avance legítimo pelo fluxo do servidor.

#### 🔹 Fase 2: Ausência de Configurações de Destinatário no Ambiente Local
* **Sintoma:** O formulário passava da validação inicial, mas caía no bloco de tratamento de erro genérico do e-mail.
* **Causa Raiz:** A linha de código `var email = _enderecoOptions.CurrentValue.Faleconosco;` retornava um valor nulo/vazio. Ao rodar o método `IsValidEmail(email)`, o C# rejeitava o envio. O arquivo de configuração local não continha a seção mapeada pela aplicação.
* **Ação Corretiva:** Adicionado o bloco de opções correto no arquivo `appsettings.json`:
  ```json
  "Endereco": {
      "Faleconosco": "suporte@sinoinformatica.com.br"
  }
  ```

#### 🔹 Fase 3: Bloqueio por Falsificação de Identidade (*Spoofing*) e *Throttling*
* **Sintoma:** A aplicação tentava realizar o disparo, mas os logs do servidor de e-mail registravam o erro:  
  `Delivery for noreply9@siscam.com.br to suporte@sinoinformatica.com.br has bounced. Reason: Sender has been throttled`
* **Causa Raiz:** O componente interno de envio (`ViewEmailService`), oriundo de um pacote Nuget fechado corporativo (`Sino.Sistema.UI`), estava injetando de forma fixa ou parametrizada o remetente `noreply9@siscam.com.br`. No entanto, as credenciais de autenticação SMTP de desenvolvimento utilizavam o domínio `@sinoinformatica.com.br`. O servidor de e-mail barrou a entrega por proteção contra fraudes de e-mail (*Spoofing*).
* **Investigação Avançada:** Uma busca global na solução inteira (`Ctrl + Shift + F`) por `noreply9` retornou zero resultados, indicando que a configuração não estava no código-fonte. A informação foi localizada diretamente em uma tabela de parâmetros no **Banco de Dados** (`sitenovohorizonte`), arquitetada para gerenciar tenants de forma dinâmica.

#### 🔹 Fase 4: Erro de Autenticação SMTP na Porta 587
* **Sintoma:** Após a alteração manual do campo de remetente no banco de dados para o e-mail legítimo da Sino, a aplicação estourou uma exceção em tela:  
  `SmtpException: Mailbox unavailable. The server response was: This is a submission only port. You must authenticate before sending mail`
* **Causa Raiz:** O ecossistema da DLL da empresa foi desenvolvido para priorizar os dados do banco sobre o `appsettings.json`. Ao atualizar apenas a coluna de remetente, a aplicação tentou se conectar de forma anônima à porta de submissão do SMTP (587), que exige obrigatoriamente a passagem de credenciais de login.
* **Ação Corretiva:** Foram atualizadas as demais colunas associadas às configurações de e-mail na mesma tabela do Banco de Dados, incluindo:
  * Usuário SMTP legítimo (`noreply-local@sinoinformatica.com.br`)
  * Senha SMTP criptografada/correta
  * Porta de conexão (`587`)
  * Flag de uso de SSL/TLS desabilitada conforme política do servidor local de testes.

---

### 3. Matriz de Arquivos Modificados e Impactados

| Arquivo / Origem | Tipo | Modificação Realizada | Objetivo |
| :--- | :--- | :--- | :--- |
| `index.cshtml.cs` | Código C# | Remoção do `[BindProperty]` de `ErrorMessage` | Sanar o bloqueio falso no `ModelState.IsValid`. |
| `appsettings.json` | Configuração | Inclusão da seção `"Endereco": { "Faleconosco": "..." }` | Fornecer um destinatário válido para a validação. |
| Banco `sitenovohorizonte` | Dados | Atualização do Remetente, Usuário, Senha e Porta SMTP | Homologar a autenticação ponta a ponta e mitigar o *Spoofing*. |

---

### 4. Status Atual do Fluxo do Formulário
Atualmente, o ciclo de vida da requisição funciona de forma perfeitamente integrada:
1. **Frontend (HTML/JS):** Captura os dados textuais e gera o token dinâmico do Google reCAPTCHA v2/v3 de forma bem-sucedida.
2. **Controller (Razor Pages):** Valida os dados de entrada, reconhece a ausência de propriedades espúrias e consome o token do reCAPTCHA com sucesso.
3. **Serviço Corporativo (`ViewEmailService`):** Renderiza o layout HTML estilizado da Câmara (Brasão + Dados do Usuário) em formato string.
4. **Camada de Dados / Infraestrutura:** Autentica com sucesso no servidor SMTP externo usando os dados parametrizados no banco e entrega a mensagem na caixa de entrada sem sofrer rejeições (*bounces*).

---

### 5. Boas Práticas Recomendadas para o Ambiente de Desenvolvimento
Para evitar bloqueios de segurança por excesso de requisições involuntárias (*throttling*) nas contas oficiais da empresa durante manutenções futuras, sugere-se a adoção de um simulador SMTP local (ex: **Papercut SMTP** ou **smtp4dev**) ou um gateway isolado de testes (ex: **Mailtrap**). Dessa forma, os e-mails disparados em computadores locais (*localhost*) são retidos em uma interface de testes, poupando a reputação de IP dos domínios corporativos oficiais.
