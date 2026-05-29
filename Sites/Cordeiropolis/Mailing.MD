## Sino.Sistema.Cordeirópolis.Web — Formulário "Mailing"
 
**Data:** 2026
**Status:** ✅ RESOLVIDO
 
### Resumo
 
O formulário de Mailing não exibia validação de reCAPTCHA para o usuário, permitindo submissão sem que o campo fosse preenchido. A solução seguiu o padrão estabelecido em Novo Horizonte.
 
---
 
### Observação Importante — Atributo `[ValidateRecaptcha]`
 
> ⚠️ O atributo `[ValidateRecaptcha]` **deve estar presente no código do back-end** (`*.cshtml.cs`). Sem ele, a validação do reCAPTCHA não é executada pelo servidor, independente do que estiver no front-end.
 
---
 
### Problema Identificado e Resolução
 
#### 🔹 Ausência de Validação de reCAPTCHA no Mailing
 
- **Sintoma:** O formulário era submetido sem exigir que o usuário marcasse a caixa "Não sou um robô", sem nenhum aviso de erro.
- **Causa Raiz:** A verificação do token `g-recaptcha-response` não estava implementada no método `OnPostAsync`.
- **Correção:** Adicionada verificação explícita do token antes do processamento do `ModelState`, forçando um erro de validação caso o campo esteja vazio:
**Arquivo:** `Mailing/Index.cshtml.cs`
 
```csharp
// Verifica se o usuário clicou no reCAPTCHA (o token foi gerado)
var recaptchaResponse = Request.Form["g-recaptcha-response"].ToString();
if (string.IsNullOrEmpty(recaptchaResponse))
{
    // Força um erro no ModelState
    ModelState.AddModelError(string.Empty, "Por favor, marque a caixa 'Não sou um robô' para continuar.");
}
```
 
- **Resultado:** ✅ Funcionou. A mensagem de erro passa a ser exibida ao usuário caso o reCAPTCHA não seja resolvido antes do envio.
---
 
### Padrão de Implementação Recomendado
 
Este padrão de validação manual via `ModelState.AddModelError` é o **padrão adotado na rede de sistemas Sino** para garantir a exibição de mensagem amigável ao usuário nos formulários que utilizam reCAPTCHA v2 ("Não sou um robô"). Replicar em todos os formulários que não possuam essa verificação.
 
---