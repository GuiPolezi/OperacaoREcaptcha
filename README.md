# Operação reCAPTCHA 🛡️

Este repositório foi criado com o intuito de centralizar, documentar e padronizar as correções aplicadas nos formulários públicos (ciclos de envio, validação de dados, entrega de e-mails e, por fim, integração com o Google reCAPTCHA) de diversos sites e portais governamentais institucionais.

Muitas vezes, falhas no recebimento de mensagens de cidadãos são tratadas erroneamente como problemas no reCAPTCHA, quando na verdade decorrem de travas de arquitetura de software, falta de parametrização de servidores SMTP ou erros de validação no servidor. Este projeto serve como um guia prático de engenharia reversa e depuração para mitigar esses gargalos.

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
