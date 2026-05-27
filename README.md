# Operação reCAPTCHA 🛡️

Este repositório foi criado com o intuito de centralizar, documentar e padronizar as correções aplicadas nos formulários públicos (ciclos de envio, validação de dados, entrega de e-mails e, por fim, integração com o Google reCAPTCHA) de diversos sites e portais governamentais institucionais.

Muitas vezes, falhas no recebimento de mensagens de cidadãos são tratadas erroneamente como problemas no reCAPTCHA, quando na verdade decorrem de travas de arquitetura de software, falta de parametrização de servidores SMTP ou erros de validação no servidor. Este projeto serve como um guia prático de engenharia reversa e depuração para mitigar esses gargalos.

---

## 📌 Histórico de Intervenções e Casos de Sucesso

### 📁 Caso #1: Câmara Municipal de Novo Horizonte
* **Tecnologia:** ASP.NET Core 8.0 (Razor Pages) + Arquitetura de Pacotes Corporativos (`Sino.Sistema.*`).
* **Status:** ✅ 100% Funcional e Homologado.

#### 🛠️ Problemas Detectados & Soluções Aplicadas

1. **Gargalo no `ModelState.IsValid` (Validação Falsa no Servidor)**
   * **Sintoma:** O formulário travava no envio e a página recarregava sem exibir erros.
   * **Causa:** A propriedade interna `ErrorMessage` estava decorada com `[BindProperty]`. O framework interpretava que o HTML frontend era obrigado a enviar dados para esse campo (devido ao comportamento padrão de *Nullable Reference Types* do .NET moderno), causando uma falha silenciosa de validação.
   * **Correção:** Remoção do atributo `[BindProperty]` da string de exibição do servidor.

2. **Ausência de Mapeamento de Destinatários em Ambiente Local**
   * **Sintoma:** O envio passava da validação inicial, mas falhava ao identificar o destino.
   * **Causa:** O dicionário de propriedades do `appsettings.json` local não possuía a chave esperada pelo bloco de opções de endereço (`_enderecoOptions`).
   * **Correção:** Injeção manual do bloco `"Endereco": { "Faleconosco": "..." }` para fins de homologação em ambiente local.

3. **Rejeição por *Spoofing* e *Throttling* no Servidor SMTP**
   * **Sintoma:** Log de erro acusando descarte do e-mail: `Reason: Sender has been throttled`.
   * **Causa:** O componente de mensageria injetava o remetente `noreply9@siscam.com.br`, mas as credenciais locais autenticavam com o domínio `@sinoinformatica.com.br`. O servidor de e-mail barrou o envio por entender que se tratava de uma falsificação de identidade (*Spoofing*).
   * **Correção:** Localização dos parâmetros de configuração armazenados de forma dinâmica na memória de tabelas do **Banco de Dados** (`sitenovohorizonte`).

4. **Falha de Autenticação na Porta 587 (`SmtpException`)**
   * **Sintoma:** Exceção explícita em tela informando que o servidor exigia autenticação anônima bloqueada.
   * **Causa:** Ao alterar o remetente no banco de dados, o ecossistema tentava conexão sem passar usuário e senha.
   * **Correção:** Parametrização completa das colunas de infraestrutura na tabela do banco (Host, Porta, Usuário SMTP e Senha), consolidando o handshake e a entrega final na caixa de entrada corporativa.

---

## 🚀 Como Contribuir para Novos Formulários/Sites

Se você encontrar um novo portal com formulário de contato ou reCAPTCHA quebrado, siga este roteiro de depuração padrão do projeto:

1. **Inspeção Paralela de Validação:** Insira um breakpoint na chegada do método HTTP POST (`OnPostAsync` ou `Action`) e inspecione o estado de validação de modelo (`ModelState.Values.SelectMany(v => v.Errors)`) para identificar campos fantasmas travando o envio.
2. **Validação do Token do reCAPTCHA:** Certifique-se de inspecionar a coleção da requisição (`Request.Form["g-recaptcha-response"]`). Se o token estiver sendo gerado mas a validação falhar, o erro estará na vinculação das chaves privada/pública do Google ou nas permissões de domínio (ex: falta liberar o `localhost`).
3. **Casamento de Identidade SMTP:** Nunca tente submeter e-mails cujo cabeçalho "De:" (*From*) difira da conta de autenticação usada no servidor SMTP. O uso de remetentes incompatíveis gerará bloqueios automáticos de segurança de infraestrutura.

---

## 🛠️ Boas Práticas para Testes Locais (*Localhost*)
Durante o desenvolvimento e correção da esteira de submissões, evite disparar requisições em massa usando as credenciais SMTP de produção para não penalizar a reputação de IP do domínio corporativo. Recomenda-se a utilização de interceptadores locais de e-mail:
* [Mailtrap](https://mailtrap.io/) (Gateway em Nuvem)
* [smtp4dev](https://github.com/rnwood/smtp4dev) (Servidor de testes fictício em Docker/Local)
* [Papercut SMTP](https://github.com/ChangemakerStudios/Papercut-SMTP) (Interface Desktop local)

---

### 📝 Licença
Este repositório é de uso estritamente técnico de engenharia de software e infraestrutura. As chaves, credenciais e dados exibidos nos históricos servem exclusivamente como referências estruturais e devem ser rotacionadas e protegidas em ambientes de produção.
